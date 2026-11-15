# ファイアウォール（ufw）

## ひとことで言うと
サーバーの入口に立つ門番。「この出入り口から、この相手が、この番号に来たら通す。それ以外は全部追い返す」を決める。ufw は Ubuntu でそれを簡単に設定するためのツール。

## 仕組み
- **デフォルト拒否**：まず全部拒否にしてから、必要なものだけ許可する。許可リスト方式なので、うっかり開いたポートがあっても外からは届かない
- 「アプリが動いている（ポートが開いている）」ことと「外から届く」ことは別。ufw はその間に立っている
- ルールは「出入り口（`eth0` / `tailscale0`）」「送り主」「ポート」の組み合わせで書く
- 拒否した通信は `/var/log/ufw.log` に `[UFW BLOCK]` として記録される

```
外からの通信
   ↓
ufw（ルールに合う？）── 合わない → 捨てる（相手はタイムアウト）
   ↓ 合う
ssh（22）/ nginx（80）
```

## 自分の環境では
README の「通信経路」の表と同じルールにした。

```bash
sudo ufw default deny incoming
sudo ufw default allow outgoing
sudo ufw allow in on eth0 from 192.168.100.0/24 to any port 22 proto tcp
sudo ufw allow in on eth0 from 192.168.100.0/24 to any port 80 proto tcp
sudo ufw allow in on tailscale0 to any port 22 proto tcp
sudo ufw allow in on tailscale0 to any port 80 proto tcp
sudo ufw show added   # 有効にする前に確認
sudo ufw enable
sudo ufw status verbose
```

| ルール | 意味 |
|---|---|
| 22/80 on eth0 ← `192.168.100.0/24` | Windows（NATSwitch）からの ssh と HTTP。iPhone の 8080 番もポートフォワーディングで `192.168.100.1` から 80 番として届くので、これで通る |
| 22/80 on tailscale0 ← Anywhere | Tailscale のトンネルから来たもの。トンネルに入れるのは自分の機器だけ |
| `(v6)` が tailscale0 にだけある | eth0 は送り主を IPv4 に限定したので IPv4 用だけ。tailscale0 は限定していないので両方作られる |
| `disabled (routed)` | Ubuntu を中継する通信はない（ルーター役ではない）ので OK |

### 締め出されないための約束
- 有効にする前に、22 番の許可があるか `sudo ufw show added` で確認する
- 今の ssh は閉じずに、別のウインドウで新しく入れるか試す
- 入れなくなったら、Hyper-V マネージャーで VM の画面を開いて `sudo ufw disable`

### 実験：表にないポートは拒否される
```bash
python3 -m http.server 8000                        # Ubuntu で 8000 番のサーバーを立てる
curl -I http://localhost:8000                      # 中からは 200 OK
curl.exe -I --max-time 5 http://192.168.100.10:8000 # Windows からはタイムアウト
sudo grep "DPT=8000" /var/log/ufw.log | tail -n 3  # [UFW BLOCK] の記録
```
→ Windows からはタイムアウトになった。サーバーは動いているのに、ufw が門前払いしている。

## 関連する用語
- [用語集](../glossary.md#公開セキュリティ)
- [ssh の強化](../ssh/hardening.md)、[VPN と Tailscale](vpn.md)、[NAT](../network/nat.md)、ポート
