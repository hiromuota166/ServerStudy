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

iPhone（どこからでも）═ Tailscale ═ Ubuntu（100.x.x.x）
```

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
