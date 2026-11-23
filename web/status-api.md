# 健康診断 API（status.hiromu-lab.dev）

## ひとことで言うと
サーバーの CPU・メモリ・ディスクを JSON で返す自作 API。Python（FastAPI）で書いて、systemd で常駐させ、nginx の裏に置いて、Cloudflare Tunnel で公開した。合言葉（API キー）がないと 401。

```
世界 → Cloudflare（HTTPS）→ トンネル → nginx（受付）→ uvicorn（127.0.0.1:8000）→ FastAPI → psutil → /proc
                                        │                         │
                                        ├ /api/ 以外は 404          ├ 合言葉がないと 401
                                        └ 宛名（Host）で振り分け      └ 説明ページ（/docs）も出さない
```

## 部品

| 部品 | 役割 | Swift でいうと |
|---|---|---|
| FastAPI | 「この URL に来たらこの関数」を書くフレームワーク | Vapor のルーティング |
| uvicorn | 8000番で待ち受けて FastAPI に渡すサーバー | Vapor の中のサーバー部分 |
| psutil | CPU やメモリを読むライブラリ（中で `/proc` を読む） | |
| venv（仮想環境） | プロジェクト専用のライブラリ置き場 | SPM でプロジェクトごとに依存を持つ |

## 1. 作る

```
sudo apt install -y python3-venv
mkdir -p ~/status-api && cd ~/status-api
python3 -m venv .venv
source .venv/bin/activate          # プロンプトに (.venv) が付く
pip install fastapi uvicorn psutil
```

`~/status-api/main.py`：

```python
import os
import secrets
import time

import psutil
from fastapi import FastAPI, Header, HTTPException

API_KEY = os.environ["STATUS_API_KEY"]

app = FastAPI(docs_url=None, redoc_url=None, openapi_url=None)


@app.get("/api/status")
def status(x_api_key: str | None = Header(default=None)):
    if x_api_key is None or not secrets.compare_digest(x_api_key, API_KEY):
        raise HTTPException(status_code=401, detail="unauthorized")

    mem = psutil.virtual_memory()
    disk = psutil.disk_usage("/")
    return {
        "cpu_percent": psutil.cpu_percent(interval=0.5),
        "cpu_count": psutil.cpu_count(),
        "load_average": [round(x, 2) for x in psutil.getloadavg()],
        "memory": {
            "total_mb": mem.total // 1024**2,
            "available_mb": mem.available // 1024**2,
        },
        "disk": {
            "total_gb": round(disk.total / 1024**3, 1),
            "used_gb": round(disk.used / 1024**3, 1),
        },
        "uptime_hours": round((time.time() - psutil.boot_time()) / 3600, 1),
    }
```

- `@app.get("/api/status")`：この URL に GET が来たら下の関数を呼ぶ
- 辞書を返すと自動で JSON になる（`Codable` を書かなくていい）
- `cpu_percent(interval=0.5)`：0.5秒見て使用率を出す。**だから応答は最低 0.5秒かかる**

## 2. なぜ `--host 127.0.0.1` で待つのか

| `--host` | 待つ出入り口 | 守り |
|---|---|---|
| `127.0.0.1` | lo だけ | 外向きの窓口がそもそもない |
| `0.0.0.0` | lo、eth0、tailscale0 全部 | ufw だけが頼り |

ufw（門番）と 127.0.0.1（窓口を作らない）の二重の守り ＝ **多層防御**。話しかけるのは nginx だけなので、外向きの窓口はいらない。`sudo ss -tlnp` で `127.0.0.1:8000` になっているのを確認できる。

## 3. systemd に預ける

ssh の画面から `uvicorn` を起動して画面を閉じると、uvicorn は止まる。

```
systemd（PID 1）
└ sshd-session   ← ssh 接続
   └ bash
      └ uvicorn  ← bash の子なので、接続が切れると SIGHUP（ハングアップ＝電話を切る合図）で終了
```

`/etc/systemd/system/status-api.service`：

```ini
[Unit]
Description=Server status API
After=network.target

[Service]
User=hiromu
EnvironmentFile=/etc/status-api.env
WorkingDirectory=/home/hiromu/status-api
ExecStart=/home/hiromu/status-api/.venv/bin/uvicorn main:app --host 127.0.0.1 --port 8000
Restart=always
RestartSec=3

[Install]
WantedBy=multi-user.target
```

| 行 | 意味 |
|---|---|
| `User=hiromu` | root ではなく自分の権限で動かす（最小権限） |
| `ExecStart=.../.venv/bin/uvicorn` | venv の中をフルパスで指定するので `activate` はいらない |
| `Restart=always` | 落ちたら3秒後に起こし直す |
| `WantedBy=multi-user.target` | `enable` すると起動時に自動で立ち上がる |

```
sudo systemctl daemon-reload
sudo systemctl enable --now status-api
systemctl status status-api
```

実験：`systemctl show -p MainPID status-api` で PID を見て `sudo kill` すると、3秒後に **別の PID で生まれ直す**。`journalctl -u status-api` に再起動の記録が残る。

## 4. nginx の裏につなぐ（リバースプロキシ）

`/etc/nginx/sites-available/status`：

```nginx
server {
    listen 80;
    listen [::]:80;
    server_name status.hiromu-lab.dev;

    location /api/ {
        proxy_pass http://127.0.0.1:8000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }

    location / {
        return 404;
    }
}
```

```
sudo ln -s /etc/nginx/sites-available/status /etc/nginx/sites-enabled/status
sudo nginx -t && sudo systemctl reload nginx
```

- `sites-available` は置き場、`sites-enabled` にリンクを置くとオン
- 設定を変えたら **書く → `nginx -t` → `reload`**。`reload` は止めずに読み直す
- `nginx -T` は **ディスクの設定ファイル** を全部つなげて見せる。動いている nginx の中身ではない

### Host ヘッダー ＝ 宛名
リクエストには必ず **Host**（どのサイト宛てか）が入っている。nginx はそれを見て `server_name` が合う設定を選ぶ。どれにも合わなければ `default_server` の設定が受け取る。Cloudflare も cloudflared も宛名を書き換えずに届ける。

| コマンド | 宛名 | 担当 | 結果 |
|---|---|---|---|
| `curl -H "Host: status.hiromu-lab.dev" localhost/api/status` | status | status | JSON |
| `curl -H "Host: status.hiromu-lab.dev" localhost/` | status | status | 404 |
| `curl localhost/` | localhost（curl が自動で入れる） | default | Hello のページ |

### 落とし穴：IPv6
最初は宛名を変えても default の Hello が返ってきた。

- `localhost` には IPv4 の `127.0.0.1` と **IPv6 の `::1`** がある。curl は `::1` を先に試す
- default は `listen [::]:80` もあったが、status は `listen 80`（IPv4）だけだった → IPv6 で来ると default しか待っていない
- `127.0.0.1` を直接指定したら JSON が返って確定。`listen [::]:80;` を足して解決
- cloudflared も `localhost` を使うので、直さないと公開しても Hello が出ていた

切り分けの流れ：違和感 → 仮説1（reload 忘れ）は外れ → 仮説2（IPv6）を IPv4 指定で確かめて当たり → 設定を見比べて原因特定 → 直して再確認。

## 5. 公開する

Cloudflare の Tunnels → `hiromuserver` → ルートを追加：`status` / `hiromu-lab.dev` / `http://localhost:80`。ルートを足すと DNS レコードも自動で作られる。

- uvicorn（8000）に直接つながず nginx（80）を通す理由：`/api/` 以外を 404 にできる、`/docs` を隠せる、宛名で振り分けられる、ログや回数制限を1か所で扱える
- **確認は必ず外から**。Ubuntu の中で `curl -H "Host: ..." localhost` しても、DNS・Cloudflare・トンネルは通っていない（実際、ルートを保存し忘れていたのに中からの確認では気づけなかった）

## 6. 守る：合言葉（API キー）

```
KEY=$(python3 -c "import secrets; print(secrets.token_urlsafe(32))")
echo "STATUS_API_KEY=$KEY" | sudo tee /etc/status-api.env > /dev/null
sudo chmod 600 /etc/status-api.env
```

- 合言葉は **コードに書かない**。root だけが読めるファイルに置いて、systemd の `EnvironmentFile` で環境変数としてアプリに渡す（`.xcconfig` を git に入れないのと同じ考え方）
- systemd は root として読んでから hiromu の権限でアプリを動かすので、アプリは受け取れるが hiromu はファイルをのぞけない
- 比べるのは `secrets.compare_digest`（`==` より、時間から推測されにくい）
- 新しい画面で使うとき：`export KEY=$(sudo sed -n 's/^STATUS_API_KEY=//p' /etc/status-api.env)`

### 合言葉はヘッダーで送る（URL に付けない）
`?key=...` を付けると **nginx のアクセスログにそのまま残る**（実験で `himitsu123` が残った）。ほかに Cloudflare のログ、ブラウザの履歴、URL を共有したときのプレビューボットにも渡る。ヘッダーはふつうログに残らない。

```
curl -s -H "X-API-Key: $KEY" https://status.hiromu-lab.dev/api/status
```

### 401 はどこに？
HTTP の返事は **ステータスコード・ヘッダー・本文** の3つ。Safari や `curl -s` は本文（`{"detail":"unauthorized"}`）しか見せない。`curl -si` でステータスコードも見える。

| HTTP | URLSession |
|---|---|
| ステータスコード 401 | `(response as? HTTPURLResponse)?.statusCode` |
| 本文 | `data` |

## 7. 負荷をかけて観察
→ [サーバーの健康状態を見る](../linux/system-monitoring.md#負荷をかけて見るstress-ng)

```
watch -n 2 'curl -s -w "\n応答時間: %{time_total}秒" -H "X-API-Key: $KEY" https://status.hiromu-lab.dev/api/status'
```

## これからやれること
- nginx の `limit_req` で回数制限（合言葉の総当たり対策）
- iPhone のショートカットでヘッダーを付けて見る → ウィジェット
- 1分ごとに記録して `/api/history`

## 関連する用語
- [用語集](../glossary.md#web)
