# ssh の接続先に名前を付ける（~/.ssh/config）

## ひとことで言うと
`~/.ssh/config` に書いておくと、`ssh hiromu@192.168.100.10` の代わりに `ssh hiromuserver` で入れる。

## 仕組み
自分の PC の中だけの「連絡先」。DNS（世界の電話帳）とは違い、ほかの PC からはこの名前では届かない。

```
Host hiromuserver           ← 自分で付けた名前
    HostName 192.168.100.10 ← 本当の住所
    User hiromu
    IdentityFile ~/.ssh/id_ed25519

Host hiromuserver-ts        ← Tailscale 経由（つないでいるときだけ）
    HostName 100.114.194.19
    User hiromu
    IdentityFile ~/.ssh/id_ed25519
```

### alias（zshrc など）より良いところ
- `scp`（ファイルのコピー）や VS Code の Remote-SSH も同じ名前で使える
- Mac でも同じ書き方の `~/.ssh/config` がそのまま使える

## 自分の環境では
- Windows：`C:\Users\Owner\.ssh\config`
- 設定が読めているかの確認：`ssh -G hiromuserver`（実際に使われる HostName や User が出る）

## 関連する用語
- [用語集](../glossary.md#ssh)
- [ssh の強化](hardening.md)
