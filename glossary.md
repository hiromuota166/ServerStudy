# 用語集

自宅サーバー構築で出てきた単語のまとめ。

## 仮想化・OS

詳しいメモ → [virtualization/](virtualization/)

| 用語 | 意味 |
|---|---|
| 仮想化 | 1台のPCの中で、別のPC（仮想マシン）を動かす技術 |
| 仮想マシン（VM） | ソフトウェアで作った仮想のPC。今回は Hyper-V の中の Ubuntu |
| ホスト / ゲスト | 仮想マシンを動かしている側（Windows）がホスト、中で動いている側（Ubuntu）がゲスト |
| ハイパーバイザー | 仮想マシンを動かす土台のソフト。Hyper-V はその一種 |
| Hyper-V | Windows Pro などに付いている、Microsoft のハイパーバイザー |
| VT-x / AMD-V | CPU の仮想化支援機能。これがないと Hyper-V は動かない |
| BIOS / UEFI | PC の電源を入れて最初に動くプログラム。UEFI は新しい方式 |
| セキュアブート | 署名のない OS を起動させない仕組み。Ubuntu を使うときはテンプレートを「Microsoft UEFI 証明機関」にする |
| 第1世代 / 第2世代 | Hyper-V の仮想マシンの種類。第2世代は UEFI 対応の新しい方式 |
| ISO | OS のインストール用ディスクを1つのファイルにしたもの |
| 動的メモリ | Hyper-V が仮想マシンのメモリを自動で増減させる機能。オンだとメモリ不足になりやすい |
| チェックポイント | 仮想マシンのある時点の状態を保存したもの（スナップショット） |
| Ubuntu Server | 画面（GUI）のない、サーバー向けの Ubuntu |
| LTS | Long Term Support（長期サポート版）。偶数年4月の「XX.04」。サーバーでは LTS を選ぶ |
| ポイントリリース（.1 など） | リリース後の不具合修正をまとめた版。26.04.1 など |
| カーネル | OS の中心部分。更新したら再起動が必要（`System restart required`） |
| OOM（Out of memory） | メモリ不足。Linux はプログラムを強制終了させて耐える（OOM killer）。iOS の Jetsam に近い |

## Linux の操作

詳しいメモ → [linux/](linux/)

| 用語 | 意味 |
|---|---|
| プロンプト | 入力待ちの表示。`PS C:\...>` は Windows、`hiromu@hiromuserver:~$` は Ubuntu |
| sudo | 管理者（root）権限でコマンドを実行する |
| root | Linux の管理者ユーザー |
| apt | Ubuntu のソフト（パッケージ）を入れたり更新したりするコマンド |
| パッケージ | インストールできるソフトのまとまり |
| ミラー | パッケージを配っているダウンロードサーバー |
| netplan | Ubuntu のネットワーク設定の仕組み。`/etc/netplan/*.yaml` に書く |
| YAML | 設定ファイルの書き方。インデント（スペースの数）で階層を表す |
| systemctl | サービス（常駐プログラム）の起動・停止・状態確認をするコマンド |
| journalctl | サービスの日記（ログ）を見るコマンド。うまく動かないときに理由が書いてある |
| Debian 系 / Red Hat 系 | Linux の2大家系。Debian 系（Debian、Ubuntu）は `apt` と `.deb`、Red Hat 系（RHEL、Fedora、Rocky）は `dnf` と `.rpm`。Ubuntu には Debian 用のソフトがそのまま入る |
| gpg 鍵（署名の鍵） | apt で入れるソフトが本物の配布元のものかを確かめるための鍵。HTTPS の証明書と似た考え方 |
| x86_64 / arm64 | CPU の種類。`uname -m` で確認。今の Ubuntu は `x86_64`（64-bit） |
| PID（プロセス ID） | 動いているプログラムに OS が付ける整理番号。起動するたびに変わる。PID 1 は `systemd` で、全部の親 → [詳しく](linux/system-monitoring.md) |
| /proc | Linux が自分の状態をファイルとして見せる場所。読むたびに OS が中身を作る。`top` も psutil もここを読む |
| CPU 使用率 | どれだけ忙しいか。`top` の `100 − id`。全コアの平均なので、100% で頭打ち |
| ロードアベレージ | CPU を使いたい・待っているプログラムの数の平均（1分・5分・15分）。**コア数（今は3）と比べる**。超えたら行列 → [詳しく](linux/system-monitoring.md) |
| buff/cache / available | buff/cache は空きメモリをファイルのキャッシュに借りている分（返せる）。メモリが足りているかは **available** を見る |
| スケジューラー | カーネルの中の CPU の配り係。動きたいプログラムに数ミリ秒ずつ順番に使わせる |
| enabled（systemd） | 起動時に自動で立ち上がる約束。`systemctl is-enabled サービス名` で確認 |
| ソケット起動 | `ssh.socket` が22番で電話番だけして、接続が来たら `ssh.service` を起こす仕組み。だから ssh は `disabled` でも入れる |
| SIGHUP | 「電話を切った（hang up）」という合図。ssh を切ると、そこから起動したプログラムに送られて止まる。だから常駐させるものは systemd に預ける |
| 仮想環境（venv） | Python のプロジェクト専用のライブラリ置き場。SPM でプロジェクトごとに依存を持つのと同じ |
| 環境変数 | プログラムの外から渡す設定値。合言葉はコードに書かず、これで渡す（systemd の `EnvironmentFile`） |

## ネットワーク

詳しいメモ → [network/](network/)

| 用語 | 意味 |
|---|---|
| ネットワークインターフェース | ネットにつながる出入り口。`eth0`（有線LAN1つ目）、`lo`（自分自身） |
| eth0 | 1つ目の有線LAN（Ethernet）のインターフェース。今の Ubuntu では NATSwitch につながった出入口（`192.168.100.10`） |
| tailscale0 | Tailscale が作った仮想の出入口（`100.114.194.19`）。ufw ではここから来た 22・80 を許可 |
| ループバック（lo / 127.0.0.1） | 自分自身と通信するための仮想の出入り口。localhost |
| IPアドレス | ネットワーク上の住所。例：`192.168.100.10` |
| プライベートIP | 家や会社の中だけで使える住所。始まり方が決まっている：`10.x`、`172.16〜31.x`、`192.168.x`。Ubuntu も Windows もこれだけ → [詳しく](network/global-ip.md) |
| グローバルIP | インターネット上で使える、世界で1つだけの住所。始まり方の決まりはなく、プライベートと特別な番号以外は全部これ。自宅では **ルーターが1つだけ** 持っている（プロバイダから借りている）。サーバーを公開していなくても、ネットが使えている時点で存在する。`curl.exe ifconfig.me` で確認 → [詳しく](network/global-ip.md) |
| CGNAT | プロバイダ側でもう1回 NAT していること。ルーターすら本当のグローバル IP を持っていない。この場合ポートを開けても外から入れないが、Cloudflare Tunnel や Tailscale は使える。`100.64〜127.x` の範囲を使う |
| DHCP | ネットに繋いだときに IP アドレスを自動で配ってくれる仕組み |
| 固定IP | DHCP を使わず、自分で決めた IP アドレスを使うこと |
| サブネット（/24） | 同じネットワークの範囲。`/24` は「先頭3つの数字が同じなら同じネットワーク」 |
| ゲートウェイ | 別のネットワーク（インターネットなど）へ出るときの出口。今回は Windows（`192.168.100.1`） |
| DNS | `ubuntu.com` のような名前を IP アドレスに変換する仕組み。スマホの連絡先のようなもの → [詳しく](network/dns.md) |
| DNS サーバー | 名前を調べるときに問い合わせる相手。netplan の `nameservers` で指定した `1.1.1.1`（Cloudflare）、`8.8.8.8`（Google） |
| ドメイン | `ubuntu.com` のような、インターネット上の名前。買って DNS に登録すると、自分のサーバーをその名前で公開できる。**右から読む**：持ち主は右端の `.com` とそのすぐ左で決まる。今は `hiromu-lab.dev` を持っている → [詳しく](security/cloudflare-tunnel-setup.md) |
| サブドメイン | ドメインの頭に付ける名前（`test.hiromu-lab.dev` の `test.`）。持ち主が自由に、追加料金なしでいくつでも作れる。用途ごとに分ける |
| TLD | ドメインの右端（`.com`、`.dev`、`.jp` など）。安い TLD（`.win`、`.xyz`）は詐欺に使われやすく評判が悪い |
| MagicDNS | Tailscale のトンネルの中だけで使える DNS。`http://hiromuserver` のように名前で接続できる |
| NAT | 内側の住所を外側の住所に書き換えて、通信を中継する仕組み。書き換えた記録（メモ帳）で返事を戻す。外から中へはそのままでは入れない → [詳しく](network/nat.md) |
| 二重 NAT | NAT が2段になっていること。今回は Windows と自宅のルーターの2段 |
| ルーティング表 | 宛先ごとに「どの出口から出すか」を決めた一覧。Windows では `Find-NetRoute` で確認できる |
| 仮想スイッチ | Hyper-V の中の仮想的なネットワーク機器 |
| Default Switch | Hyper-V に最初からある仮想スイッチ。Windows を再起動すると IP が変わりやすい |
| 内部仮想スイッチ（NATSwitch） | 自分で作った仮想スイッチ。番号を固定できる |
| ポート | 1つの IP の中の「窓口番号」。ssh は22番、HTTP は80番、HTTPS は443番 |
| ポートフォワーディング（ポート開放） | ある窓口に来た通信を、別の機器の窓口に転送すること。NAT のメモ帳に手で書いておく設定。今回は Windows の8080番 → Ubuntu の80番 → [公開の方法](security/publish-methods.md) |
| ファイアウォール | 通信を許可・拒否する門番 |
| ネットワークプロファイル | Windows の Wi-Fi の種類。パブリック（厳しい）/ プライベート（自宅向け） |
| ping | 相手に届くか、往復に何秒かかるかを調べるコマンド |
| TTL | ルーターを通るたびに1減る数字。経由したルーターの数がわかる |
| レイテンシ / RTT | 通信の往復にかかる時間。距離が遠いほど長くなる |
| CDN | 世界中にサーバーを置いて、ユーザーの近くから返事をする仕組み |

## ssh

詳しいメモ → [ssh/](ssh/)

| 用語 | 意味 |
|---|---|
| ssh | 別のコンピューターに安全に接続して操作する仕組み |
| OpenSSH server | ssh で接続を受け付けるためのソフト |
| ホストキー（指紋） | サーバーごとの身分証明。初回接続で `yes` と答えると覚える |
| known_hosts | 一度接続したサーバーの指紋の一覧（`C:\Users\Owner\.ssh\known_hosts`） |
| 公開鍵認証 | パスワードの代わりに鍵ファイルでログインする方法。より安全 → [詳しく](ssh/hardening.md) |
| 秘密鍵 / 公開鍵 | 秘密鍵は自分の PC だけに置く。公開鍵はサーバーの `authorized_keys` に登録する |
| パスフレーズ | 秘密鍵のファイルを開くための合言葉。サーバーには送られない |
| ssh config | `~/.ssh/config` に接続先の名前を書いておくと、`ssh hiromuserver` だけで入れる。scp や VS Code でも使える → [詳しく](ssh/ssh-config.md) |

## Web

詳しいメモ → [web/](web/)

| 用語 | 意味 |
|---|---|
| Webサーバー | ブラウザからのリクエストにファイルなどを返すソフト |
| nginx | Webサーバーの一つ。リバースプロキシとしても使われる |
| HTTP / HTTPS | Web の通信方式。HTTPS は暗号化されている |
| リクエスト / レスポンス | ブラウザ（アプリ）からのお願いと、サーバーからの返事 |
| API | プログラム同士がデータをやり取りする窓口。Web API は JSON を返すことが多い |
| JSON | データの書き方。iOS アプリが `URLSession` で受け取っているもの |
| ドキュメントルート | Web サーバーが公開するファイルの置き場。nginx は `/var/www/html` |
| FastAPI / uvicorn | FastAPI は Python で API を書くフレームワーク（Vapor のルーティングのようなもの）。uvicorn はそれを動かすサーバー → [健康診断 API](web/status-api.md) |
| Host ヘッダー | リクエストの **宛名**（どのサイト宛てか）。nginx はこれで `server_name` が合う設定を選ぶ。差出人ではない |
| default_server | どの `server_name` にも当てはまらないときに受け取る nginx の設定 |
| sites-available / sites-enabled | nginx の設定の置き場と、オンにする設定へのリンクを置く場所 |
| ステータスコード | 返事の結果の番号（200 成功、401 誰？、404 ない、502 奥がおかしい）。本文とは別にある。iOS の `HTTPURLResponse.statusCode` |
| 401 Unauthorized | 「あなたは誰？」。合言葉がない・間違っている |
| 400 Bad Request | 「何を言っているか分からない」。HTTP の窓口に HTTPS で話しかけたときなど |
| IPv6（`::1`） | 新しい形式の住所。`localhost` は `127.0.0.1` と `::1` の両方を持つ。nginx は `listen [::]:80` がないと IPv6 で受け取れない |

## 公開・セキュリティ

詳しいメモ → [security/](security/)

| 用語 | 意味 |
|---|---|
| VPN | インターネットの上に作る、自分たち専用の暗号化トンネル。離れた機器同士が同じ Wi-Fi にいるように通信できる。会社の VPN も同じ仕組み → [詳しく](security/vpn.md) |
| Tailscale | 自分の機器だけをつなぐ VPN サービス。機器同士が直接つながる（メッシュ型）。NAT の奥でもポートフォワーディングなしで届く → [詳しく](security/vpn.md) |
| Tailscale IP（100.x.x.x） | Tailscale のトンネルの中だけで使える住所。トンネルに入っていない他人からは届かない。再起動しても変わらない。今の Ubuntu は `100.114.194.19` |
| Cloudflare Tunnel | ルーターのポートを開けずに、サーバーをインターネットに公開できる仕組み。Ubuntu から先に Cloudflare へつないでおく → [仕組み](security/publish-methods.md)、[実際の手順](security/cloudflare-tunnel-setup.md) |
| cloudflared | Cloudflare Tunnel をつなぐための、Ubuntu 側のソフト。サービスとして常駐し、再起動しても自動でつなぎ直す |
| トンネルのトークン | cloudflared が「自分は hiromuserver だ」と名乗るための鍵（`eyJ...`）。漏れると偽サイトをつながれるので、人に見せない |
| ルート（公開ホスト名） | 「このドメインに来たら、Ubuntu の中のどこに渡すか」の設定。今は `test.hiromu-lab.dev` と `status.hiromu-lab.dev` → `http://localhost:80`。足すと DNS レコードも自動で作られる |
| API キー（合言葉） | 知っている人だけが API を使えるようにする文字列。**ヘッダーで送る**（URL に付けるとログや履歴に残る） → [詳しく](web/status-api.md) |
| 多層防御 | 守りを何重にも重ねる考え方。ufw + 127.0.0.1 で待つ + nginx で絞る + 合言葉。どのリクエストがどこで止まるか → [リクエスト路線図](security/request-routes.md) |
| WAF | Web アプリ用のファイアウォール。ufw やルーターは**番号**（IP・ポート）を見るが、WAF は**中身**（URL・ヘッダー）を読む。`/.env` 探しなどを 403 で止める。今は Cloudflare の WAF → [路線図](security/request-routes.md) |
| 最小権限 | 必要な権限だけで動かす考え方。API は root ではなく hiromu で動かす |
| アクセスログ | nginx が「誰が・いつ・何を見に来たか」を1行ずつ記録したもの。`/var/log/nginx/access.log` → [読み方](security/log-reading.md) |
| ログローテーション | ログを毎日新しいファイルに切り替えて、古いものを `.1` や `.gz` に移すこと |
| User-Agent | アクセスしてきた相手が名乗っている名前。`curl/8.x`、`facebookexternalhit`（リンクのプレビューを作るボット）など |
| Certificate Transparency（CT ログ） | 発行された証明書の記録が載る、世界中に公開されたログ。攻撃ボットも見ていて、新しいドメインがすぐ見つかる理由 |
| 502 Bad Gateway | 受付（Cloudflare や nginx）は動いているが、奥に渡したら返事がおかしかった、というエラー。奥の区間を疑う |
| オリジン | 受付の奥にある、本当の返事を作るサーバー。今回は Ubuntu の nginx |
| DDNS | 自宅のグローバル IP が変わっても、ドメイン名と自動で結びつけ直してくれる仕組み |
| リバースプロキシ | 外からのリクエストを受け付けて、裏のアプリに渡す窓口役。HTTPS の処理もここで行う |
| 踏み台サーバー | 管理用の入口を1か所にまとめるためのサーバー。ここを経由しないと中に入れない |
| ufw | Ubuntu の簡単なファイアウォール設定ツール → [詳しく](security/firewall.md) |
| デフォルト拒否 | まず全部拒否して、必要なものだけ許可する考え方。うっかり開いたポートも外から届かない |
| 総当たり攻撃 / 辞書攻撃 | パスワードを片っ端から、またはよく使われる単語から順に試してログインしようとする攻撃 |
| SSL/TLS 証明書 | HTTPS に必要な、サイトの身分証明書。Let's Encrypt で無料で取れる |

## よく使うコマンド

| コマンド | 何をするか |
|---|---|
| `ssh ユーザー名@IP` | サーバーに接続する（Windows） |
| `exit` | ssh を終わる |
| `ip a` | IP アドレスを確認する |
| `ping -c 3 ubuntu.com` | 外に出られるか確認する |
| `free -h` | メモリを確認する |
| `sudo apt update && sudo apt upgrade -y` | ソフトを最新にする |
| `sudo apt install -y パッケージ名` | ソフトを入れる |
| `sudo netplan apply` | ネットワーク設定を反映する |
| `systemctl status ssh` | ssh サーバーの状態を確認する |
| `sudo reboot` | 再起動 |
| `sudo shutdown now` | シャットダウン |
| `netsh interface portproxy show all` | ポートフォワーディングの設定を確認する（Windows） |
| `sudo tailscale up` | Tailscale に接続する（初回は表示された URL でログイン） |
| `tailscale ip -4` | Tailscale 用の IP（`100.x.x.x`）を確認する |
| `sudo ufw status verbose` | ファイアウォールの設定を確認する |
| `sudo ss -tlnp` | 待ち受けているポートを確認する |
| `sudo sshd -t` | ssh の設定ファイルに書き間違いがないか確認する |
| `systemctl status cloudflared` | トンネルがつながっているか確認する（`active (running)` なら OK） |
| `sudo journalctl -u cloudflared -n 20 --no-pager` | cloudflared のログを最新20行見る |
| `uname -m` | CPU の種類（`x86_64` など）を確認する |
| `nslookup ドメイン 1.1.1.1` | ドメインがどの IP になるか調べる（Windows） |
| `curl.exe -sS -I https://ドメイン` | 返事のヘッダーだけ見る。`200 OK` / `502` などがわかる（Windows） |
| `ssh hiromuserver` | ssh config に書いた名前で接続する（Windows） |
| `nproc` | CPU のコア数を見る |
| `uptime` / `cat /proc/loadavg` | 起動時間とロードアベレージを見る |
| `df -h /` | ディスクの使用量を見る |
| `top` | CPU を使っている順にプログラムを見る（`q` で終了） |
| `cat /var/run/reboot-required` | 再起動が必要か確認する |
| `systemctl is-enabled サービス名` | 起動時に自動で立ち上がるか確認する |
| `sudo systemctl enable --now サービス名` | 自動起動にして、今すぐ起動する |
| `sudo systemctl daemon-reload` | systemd の設定ファイルを読み直す |
| `sudo nginx -t && sudo systemctl reload nginx` | nginx の設定を確かめてから、止めずに反映する |
| `curl -si URL` | ステータスコードとヘッダーも表示する |
| `curl -H "Host: 名前" localhost/` | 宛名を指定して送る |
| `sudo zcat -f /var/log/nginx/access.log* \| awk '{print $7}' \| sort \| uniq -c \| sort -rn \| head -20` | どのページが何回見られたかを数える |
| `stress-ng --cpu 3 --timeout 90s` | わざと CPU に負荷をかける |
| `watch -n 2 'コマンド'` | 2秒ごとにコマンドを実行して表示し続ける |
| `curl.exe ifconfig.me` | 外から見た自分のグローバル IP（ルーターの住所）を確認する（Windows） |
