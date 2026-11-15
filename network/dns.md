# DNS

## ひとことで言うと
`ubuntu.com` のような名前を、IP アドレス（`185.125.190.20` など）に変換する仕組み。スマホの連絡先のようなもの。

## 仕組み
コンピューター同士は IP アドレスでしか通信できない。人間が数字を覚えなくていいように、DNS が名前と番号を結びつけている。

| 連絡先 | DNS |
|---|---|
| 「友だちの名前」で電話をかける | `ubuntu.com` という名前でアクセスする |
| 電話は裏で番号にかけている | 通信は裏で `185.125.190.20` に向けて行われる |

```
「ubuntu.com の IP は？」
Ubuntu ──────────────────→ DNS サーバー（1.1.1.1 など）
Ubuntu ←────────────────── 「185.125.190.20 だよ」
Ubuntu ──── 185.125.190.20 に通信する
```

- **DNS サーバー**：「名前を調べるときは、ここに聞いてね」という相手。`1.1.1.1` は Cloudflare、`8.8.8.8` は Google
- DNS サーバーの設定を間違えると、`ping 185.125.190.20` は通るのに `ping ubuntu.com` は通らない、という状態になる
- iOS アプリでも、`URLSession` で `https://api.example.com` にリクエストを送るたびに、iPhone は裏で DNS に IP を問い合わせている

## 自分の環境では
これまでに4回登場している。

1. **`ping ubuntu.com`**：結果の `PING ubuntu.com (185.125.190.20)` のカッコの中が、DNS で調べた結果
2. **固定 IP の設定（netplan）**：`/etc/netplan/01-eth0.yaml` の次の部分が DNS サーバーの指定
   ```yaml
   nameservers:
     addresses: [1.1.1.1, 8.8.8.8]
   ```
3. **`hiromuserver.mshome.net`**：Hyper-V の Default Switch が、仮想マシンに名前を付けてくれる簡単な DNS の機能
4. **Tailscale の MagicDNS**：`http://hiromuserver` で開けるのは、Tailscale がトンネルの中だけで使える DNS を用意しているから（→ [VPN と Tailscale](../security/vpn.md)）

## これから大事になる場面
「ドメインを取得して HTTPS で公開する」のは、DNS そのものの話。たとえば `hiromu.dev` のようなドメインを買って、「この名前はこのサーバーだよ」と DNS に登録すると、世界中の人がその名前でアクセスできるようになる。

## 関連する用語
- [用語集](../glossary.md#ネットワーク)
- IP アドレス、ドメイン、MagicDNS
