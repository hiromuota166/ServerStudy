# Cloudflare Tunnel で公開した手順

## ひとことで言うと
`hiromu-lab.dev` を買って、Ubuntu から Cloudflare へトンネル（切らない電話）をつなぎ、`https://test.hiromu-lab.dev` で nginx の画面を世界に公開した。ルーター・Windows・ufw の設定は一切変えていない。

仕組みの比較（ポート開放との違い）は → [世界に公開する方法](publish-methods.md)

## 全体の流れ

| # | やること | どこで |
|---|---|---|
| 0 | nginx とパッケージを最新にする | Ubuntu |
| 1 | Cloudflare のアカウントを作って、ドメインを買う | ブラウザ |
| 2 | Cloudflare の画面でトンネルを作る | ブラウザ |
| 3 | Ubuntu に cloudflared を入れて、トンネルをつなぐ | Ubuntu |
| 4 | 「ドメイン → `http://localhost:80`」のルートを足す | ブラウザ |
| 5 | モバイル回線で `https://test.hiromu-lab.dev` を開いて確認 | iPhone |

## 0. 公開する前に最新にする

```
nginx -v
sudo apt update && apt list --upgradable 2>/dev/null | grep nginx
grep -rn "http2" /etc/nginx/
zgrep -i "123007" /usr/share/doc/nginx*/changelog.Debian.gz
sudo apt upgrade -y
```

- NGINX2Root（CVE-2026-123007）は HTTP/2 を使っていないと突けない。`grep` で何も出なかったので、入口はそもそも閉じている
- `zgrep` で何も出ない ＝ Ubuntu の修正版はまだ届いていない。届いたら `apt upgrade` で入るので、時々確認する
- Cloudflare Tunnel の構成だと、外のブラウザと HTTPS / HTTP/2 で話すのは **Cloudflare**。nginx は Ubuntu の中で cloudflared と HTTP/1.1 で話すだけ

## 1. ドメインを買う

- Cloudflare の管理画面（`dash.cloudflare.com`）→ ドメインの登録
- **ドメインは汎用的な名前にする**。用途ごとにサブドメインで分ける

```
hiromu-lab.dev          ← 買うのはこれ1つ
├ test.hiromu-lab.dev   ← 今回の nginx の画面
└ darts.hiromu-lab.dev  ← 将来のダーツの API
```

- `.dev` を選んだ理由：`.win` などの安い TLD は詐欺業者がよく使うので評判が悪く、学校や企業のネットワークでブロックされることがある。暗号化の強さは同じ
- `.dev` は HTTPS でしか開けない決まりがあるが、Cloudflare が HTTPS にするので問題ない
- 1年目 $8.20、2年目から $12.20。自動更新はオン（切れると他人に取られることがある）
- 登録者情報と請求先住所は **ローマ字**、番地 → 町名の順。23区は「市区町村」に `〇〇-ku`、州で `Tokyo`。WHOIS では公開されない

### ドメインは右から読む（偽サイト対策）

```
dash . cloudflare.com
 ↑         ↑
サブドメイン   本体（持ち主が決まる部分）
```

| URL | 本体 | 判定 |
|---|---|---|
| `dash.cloudflare.com` | cloudflare.com | ✅ 本物 |
| `cloudflare.com.login-check.xyz` | login-check.xyz | ❌ 左に名前があるだけ |
| `dash.cloudfIare.com` | cloudfIare.com（大文字の I） | ❌ 見た目だけ似せている |

自分も `cloudflare.hiromu-lab.dev` を自由に作れる。だから「左に有名な名前がある」だけでは安心できない。

## 2〜3. トンネルを作って cloudflared をつなぐ

- Cloudflare：Zero Trust → ネットワーク → Tunnels → トンネル作成 → Cloudflared → 名前 `hiromuserver`
- OS は **Debian**、**64-bit** を選ぶ（Ubuntu は Debian 系。`uname -m` が `x86_64` なら 64-bit）
- 表示されるコマンドは2つ

| コマンド | 何をするか |
|---|---|
| Install cloudflared | Cloudflare の署名の鍵（gpg）を置き、ソフト置き場を apt に登録して、cloudflared を入れる。Tailscale と同じ仕組みで、今後は `apt upgrade` で一緒に更新される |
| `sudo cloudflared service install eyJ...` | cloudflared をサービスとして登録。再起動しても自動でつなぎ直す |

- ⚠️ `eyJ...` は **トンネルの鍵（トークン）**。これがあると誰でも「自分が hiromuserver だ」と名乗って、ドメインに偽サイトをつなげる。チャットやスクショに出さない
- 確認：`systemctl status cloudflared` が `active (running)`、Cloudflare の画面でステータスが「正常」

## 4. ルートを足す

| 欄 | 入れるもの |
|---|---|
| サブドメイン | `test` |
| ドメイン | `hiromu-lab.dev` |
| サービス URL | **`http://localhost:80`**（https ではない） |

http にする理由は、区間ごとに話す言葉が違うから。

```
ブラウザ ──HTTPS──▶ Cloudflare ──暗号化トンネル──▶ cloudflared ──HTTP（lo の中）──▶ nginx :80
```

## 5. 502 Bad Gateway の切り分け（実際にあった失敗）

最初はサービス URL を `https://localhost:80` にしてしまい、502 が返ってきた。

**① 壊れている区間を絞る**（Windows から）

```
nslookup test.hiromu-lab.dev 1.1.1.1      → Cloudflare の IP（104.21.x.x など）が返る ✅
curl.exe -sS -I https://test.hiromu-lab.dev → Server: cloudflare、CF-RAY: ...-NRT ✅ / 502 ❌
```

- DNS も Cloudflare（成田 = NRT、Anycast で近くに着いた）までも届いている
- 502 ＝「受付は動いているけど、奥に渡したら返事がおかしかった」

```
ブラウザ → Cloudflare ✅ → トンネル → cloudflared → nginx ❌ ここ
```

**② ログで理由を見る**（Ubuntu）

```
sudo journalctl -u cloudflared -n 20 --no-pager
```

```
originService=https://localhost:80
error="... tls: first record does not look like a TLS handshake"
```

- cloudflared が「TLS で話そう」と言ったのに、80 番の nginx は暗号化なしの HTTP で返した → 言葉が合わずに失敗
- サービス URL を `http://localhost:80` に直したら `200 OK`

この「区間を絞る → ログで理由を見る」が運用保守の切り分けの基本。

## なぜ ufw を変えなくていいのか

ufw が止めるのは **外から新しくかかってきた電話** だけ。

```
① cloudflared → Cloudflare   Ubuntu からかけた電話（出ていく通信なので許可。ufw が通話を覚える）
② Cloudflare → cloudflared   ①の通話の中の話なので許可
③ cloudflared → nginx        lo の中（許可）
④ 返事は ①の通話で戻る
```

- トンネルの通信も ufw を **通っている**。ただし外向きにかけた通話の続きなので許可される（NAT のメモ帳と同じ考え方）
- 世界中のアクセスは、全部 ①の1本の通話の中にまとめて運ばれる。Ubuntu から見た相手は Cloudflare だけ
- 自宅のグローバル IP に直接つなごうとする通信は、ルーターで止まり、届いても ufw が拒否する

## 自分の環境では

- ドメイン：`hiromu-lab.dev`（Cloudflare Registrar）
- トンネル名：`hiromuserver`（cloudflared 2026.11.1、QUIC で接続）
- ルート：`test.hiromu-lab.dev` → `http://localhost:80`
- ssh は今までどおり Tailscale からだけ。公開しているのは `test.` の nginx の画面だけ

## これからやること

- nginx のログの相手の IP が全部 `127.0.0.1`（cloudflared）になる → 本当の IP（`CF-Connecting-IP`）を残す設定
- ダーツの API を作って `darts.hiromu-lab.dev` で公開する。書き込みには合言葉（API キー）を付ける

## 関連する用語
- [用語集](../glossary.md#公開セキュリティ)
