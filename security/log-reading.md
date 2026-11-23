# ログを読んで「攻撃？」を確かめる

## ひとことで言うと
nginx のアクセスログで「誰が・何を見に来たか」が分かる。怪しい行を見つけたら、**時刻を、ほかのログや自分の作業と突き合わせる**。意外と自分が原因のことが多い。

## ログの形

```
127.0.0.1 - - [21/Nov/2026:15:12:59 +0000] "HEAD / HTTP/1.1" 200 0 "-" "curl/8.x"
   $1    $2 $3          $4          $5      $6  $7   $8     $9 ...           ↑ User-Agent（名乗っている名前）
```

- 場所：`/var/log/nginx/access.log`。毎日切り替わって、古いものは `access.log.1`、`.2.gz`…に移る（ログローテーション）
- Cloudflare 経由だと IP は全部 `127.0.0.1`（相手は cloudflared）。本当の相手は Cloudflare の画面か、`CF-Connecting-IP` を残す設定で見る

## よく使う形：どのページが何回見られたか

```
sudo zcat -f /var/log/nginx/access.log* | awk '{print $7}' | sort | uniq -c | sort -rn | head -20
```

| 処理 | 何をする | Swift でいうと |
|---|---|---|
| `zcat -f access.log*` | 圧縮されたものも含めて全部読む | |
| `awk '{print $7}'` | 各行の7番目（パス）だけ | `.map { $0.split(" ")[6] }` |
| `sort` | 同じものを隣に並べる | `.sorted()` |
| `uniq -c` | 隣の同じ行をまとめて数える | `Dictionary(grouping:)` |
| `sort -rn` | 多い順 | `.sorted { $0.count > $1.count }` |
| `head -20` | 上から20件 | `.prefix(20)` |

`|`（パイプ）で前の結果を次に流している。

## 攻撃っぽいアクセスの例

| パス | 狙い |
|---|---|
| `/.env`、`/.git/config` | 設定ファイルやパスワード |
| `/wp-admin`、`/wp-login.php` | WordPress の管理画面 |
| `/phpmyadmin` | データベースの管理画面 |

404 で追い返せていれば問題ない。

## 実際にあったこと

### 文字化けの 400（`\x16\x03...`）→ 自分の設定ミスだった
`\x16\x03` は **TLS の最初の挨拶**。HTTP の窓口（80番）に HTTPS で話しかけると、nginx は理解できず 400 を返して文字化けとして記録する。

時刻を突き合わせたら正体が分かった。

| 時刻（UTC） | nginx | cloudflared のログ | 自分の作業 |
|---|---|---|---|
| 15:09:49 | 400 ×2 | `tls: first record does not look like a TLS handshake` | iPhone で開いた |
| 15:10:40 | 400 ×3 | 同じ | Windows から curl（502） |
| その後 | 出なくなる | | ルートを `http://` に直した |

→ ルートを `https://localhost:80` にしていたときの cloudflared の挨拶だった（[公開の手順](cloudflare-tunnel-setup.md#5-502-bad-gateway-の切り分け実際にあった失敗)）。1回のアクセスで `/` と `/favicon.ico` の2件。最初「iPhone の Safari が https を試したのでは」と推測したが外れ。**証拠なしの推測は外れる**。

### `facebookexternalhit` が `/robots.txt` を見に来た
Facebook のボット。Messenger・Instagram・WhatsApp などに URL を貼ると、**リンクのプレビューを作るために取りに来る**。`robots.txt` は「ボットはどこを巡回していいか」を書くファイル（ないと「全部いい」の意味）。

## なぜ作ったばかりのドメインが見つかるのか
HTTPS の証明書を発行すると、その記録が **Certificate Transparency（CT）ログ** という世界中に公開されたログに載る。不正発行を見張る仕組みだが、攻撃ボットもここを監視していて、新しいドメインをすぐスキャンしに来る。`https://crt.sh/?q=ドメイン` で見られる（crt.sh 自体がよく 502 になる。アドレスバーを右から読めば、自分の nginx ではないと分かる）。

## 今のところ攻撃が少ない理由
- 「公開すると攻撃が来る」の多くは、**IP を片っ端から叩いて開いているポートを探すスキャン**。自宅のグローバル IP はポートが1つも開いていないので、ルーターで止まりログにも残らない
- ドメインは Cloudflare の IP を指しているので、まず Cloudflare が受け止める（Cloudflare の セキュリティ → 分析 で「不審なアクティビティ 0」を確認）

## Cloudflare のおすすめ設定は全部オンにしない
- Bot Fight モード：怪しいボットを止める。ただし自作アプリやスクリプトからのアクセスもボット扱いされることがあるので保留
- 「提案があるから全部オン」ではなく、自分のサービスに影響がないかを考えてから

## 関連する用語
- [用語集](../glossary.md#公開セキュリティ)
- [健康診断 API](../web/status-api.md)
