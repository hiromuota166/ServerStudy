# ServerStudy

自宅サーバー（Hyper-V + Ubuntu Server）でサーバーサイドを勉強する記録。

- [用語集](glossary.md)

## 詳しいメモ

| フォルダ | 内容 |
|---|---|
| [virtualization/](virtualization/) | 仮想化・OS |
| [linux/](linux/) | Linux の操作 |
| [network/](network/) | ネットワーク |
| [ssh/](ssh/) | ssh |
| [web/](web/) | Web |
| [security/](security/) | 公開・セキュリティ |

## 今の構成

```
iPhone ─ Wi-Fi ─ Windows（192.168.11.10:8080）
                    │ ポートフォワーディング
                    │ NATSwitch（192.168.100.0/24）
                    └ Ubuntu Server 26.04.1 LTS（192.168.100.10）── nginx（80）

iPhone（どこからでも）═ Tailscale ═ Ubuntu（100.114.194.19）
```

## 通信経路

「どこから・どこへ・何番」の一覧（通信要件表）。**この表に載っていない通り道は、存在してはいけない。**

| どこから | どこへ | ポート | 用途 | 通り道 |
|---|---|---|---|---|
| 自宅 Wi-Fi の機器 | Windows `192.168.11.10` | 8080 | HTTP | Windows が Ubuntu の 80 番に転送 |
| Windows | Ubuntu `192.168.100.10` | 22 | ssh | NATSwitch |
| Windows | Ubuntu `192.168.100.10` | 80 | HTTP | NATSwitch |
| Tailscale の機器（iPhone など） | Ubuntu `100.114.194.19` | 22, 80 | ssh, HTTP | Tailscale |
| インターネット | どこにも | - | - | 入口なし |

- 「HTTP」は、ブラウザでも `curl` でも iOS の `URLSession` でも同じ。扉から見ると区別はない
- ⚠️ 今は ufw を入れていないので、Ubuntu の扉は全部開いている（表と実際がズレている）。ufw でデフォルト拒否にして、表と一致させる
- 確認方法：中からは `sudo ss -tlnp`、外からは Mac で `nc -vz IP ポート`

## ロードマップ

- [x] Hyper-V に Ubuntu Server を入れる
- [x] Windows から ssh で接続する
- [x] nginx で Web ページを表示する
- [x] ポートフォワーディングで iPhone から見る
- [x] 内部仮想スイッチを作って IP を固定する
- [x] Tailscale で、自分だけどこからでも入れるようにする
- [ ] ssh の強化とファイアウォール
- [ ] 自作の Web アプリ（API）を作って、nginx の裏で動かす
- [ ] ドメイン + Cloudflare Tunnel で HTTPS 公開する
- [ ] 運用保守（自動アップデート、ログ、バックアップ、デプロイ手順）
