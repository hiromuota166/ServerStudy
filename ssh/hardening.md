# ssh の強化（鍵だけでログイン）

## ひとことで言うと
パスワードでのログインを禁止して、**鍵ファイルを持っている PC からしか入れない** ようにすること。

## 仕組み
### なぜパスワードは危ないのか
ネットに出ているサーバーには、`ssh root@サーバー` に適当なパスワードを無限に試してくるプログラムが毎日やってくる（総当たり攻撃・辞書攻撃・よそで漏れたパスワードの使い回し）。パスワードは「当てれば入れる」。

### 公開鍵認証
- **秘密鍵**（`id_ed25519`）は Windows にだけある。外に出さない
- **公開鍵**（`id_ed25519.pub`）を Ubuntu の `~/.ssh/authorized_keys` に登録しておく
- ログインのとき、Ubuntu が「問題」を出し、Windows が秘密鍵で「署名」して返す。公開鍵で確かめられれば OK
- 秘密鍵そのものは通信に流れない
- パスフレーズは、秘密鍵のファイルを開くための鍵。Windows の中だけで使われ、Ubuntu には送られない

```
Windows                          Ubuntu
  │ ① つないで暗号化（known_hosts でサーバーが本物か確認）
  │ ② 「この公開鍵で入りたい」 ──→ authorized_keys にある？
  │ ←── ③ 問題を出す
  │ ④ パスフレーズで秘密鍵を開いて署名 ──→ ⑤ 公開鍵で確認 → OK
```

**強くなったこと**：パスワードを何億回試されても、鍵ファイルがないので絶対に入れない。

## 自分の環境では
1. Windows で鍵を作る（パスフレーズ付き）
   ```bash
   ssh-keygen -t ed25519
   ```
2. 公開鍵を Ubuntu に登録する（PowerShell）
   ```
   type $env:USERPROFILE\.ssh\id_ed25519.pub | ssh hiromu@192.168.100.10 "mkdir -p ~/.ssh && chmod 700 ~/.ssh && cat >> ~/.ssh/authorized_keys && chmod 600 ~/.ssh/authorized_keys"
   ```
3. 鍵で入れることを確認してから、`/etc/ssh/sshd_config.d/10-hardening.conf` を作る
   ```
   PasswordAuthentication no
   KbdInteractiveAuthentication no
   PermitRootLogin no
   PubkeyAuthentication yes
   ```
   - `sshd_config.d` の中のファイルは **先に読まれた設定が勝つ**。`10-` にしたのは、もとからある `50-cloud-init.conf` より先に読ませるため
4. 反映と確認
   ```bash
   sudo sshd -t                     # 書き間違いがないか
   sudo systemctl reload ssh
   sudo sshd -T | grep -E "passwordauthentication|permitrootlogin|pubkeyauthentication"
   ```
5. パスワードで入れないことを確認（Windows）
   ```
   ssh -o PubkeyAuthentication=no hiromu@192.168.100.10
   ```
   → `Permission denied (publickey)` になれば成功

## 関連する用語
- [用語集](../glossary.md#ssh)
- 公開鍵認証、known_hosts、ホストキー、[ファイアウォール（ufw）](../security/firewall.md)
