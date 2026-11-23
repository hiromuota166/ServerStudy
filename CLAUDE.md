# CLAUDE.md

このリポジトリを読む Claude 向けのメモ。ここを読めば、勉強の続きから手伝えるようにしてある。

## このリポジトリは何か

自宅サーバー（Windows 11 の Hyper-V + Ubuntu Server 26.04）で、サーバーサイドを勉強した記録。
**公開リポジトリ**なので、書く内容に注意する（下の「書かないこと」）。

- 全体像と今の構成：[README.md](README.md)（構成図、通信経路の表、ロードマップ）
- 用語：[glossary.md](glossary.md)
- 詳しいメモ：各フォルダ（`network/` `security/` `web/` など）。各フォルダの `README.md` に一覧がある
- どのリクエストがどこで止まるか：[security/request-routes.md](security/request-routes.md)

## 勉強している人

- iOS エンジニア。Swift / `URLSession` / iOS のメモリ管理（Jetsam）などは分かる
- サーバー、ネットワーク、Linux、Windows は学び始め。Windows の操作も初心者
- **返事は必ず日本語**。やさしく、かみ砕いて。専門用語を専門用語で説明しない
- iOS 開発とのたとえが効く（例：OOM killer ≒ Jetsam）

## 教え方の好み

- **クイズ形式**：「予想する → 実際にコマンドで試す → 答え合わせ」の順。いきなり答えを書かない
- 図が好き（縦の流れ図、路線図のような図）
- 手を動かすときは、1ステップずつ。コマンドは何をするかを一言添える
- 理解があやしいところは、自分で言語化させてから補足する

## 今どこまで進んだか

ロードマップは [README.md](README.md#ロードマップ) を正とする。2026-11 時点：

- できた：Hyper-V に Ubuntu、ssh（鍵のみ）、nginx、ポートフォワーディング、固定 IP、Tailscale、ufw、自作 API（健康診断 API、FastAPI + uvicorn + systemd）、ドメイン `hiromu-lab.dev` + Cloudflare Tunnel で HTTPS 公開
- まだ：運用保守（自動アップデート、ログ、バックアップ、監視、デプロイ手順）
- 余裕があれば：nginx の `limit_req`、`CF-Connecting-IP` で本当の送り主をログに残す、Cloudflare の Bot Fight Mode を使うか決める

## 復習したいこと（理解があやしい所）

路線図のクイズで間違えた所から。順番もこのとおりがおすすめ。

1. **「返事は通る」の仕組み**（conntrack / ステートフル）。トンネル（cloudflared、Tailscale）が「中から外へかけた電話」なので、ルーターや ufw を素通しできる理由
2. **ルーターの転送、Windows の portproxy、NATSwitch の違い**。NATSwitch をルーターの一部だと混同しがち
3. **HTTPS の暗号はどこで開くか**。iPhone → Cloudflare（TLS）→ トンネル（暗号化）→ cloudflared → nginx（`localhost:80` は平文）
4. **ssh の秘密鍵と公開鍵、どちらが誰の持ち物か**
5. **nginx の振り分け**（Host ヘッダーと `default_server`、IPv6 `::1` の落とし穴）
6. 線を引くときに「**出発地点**」を先に決める癖（出発地点より手前の駅は通らない）

## 書かないこと（大事）

- **仕事・会社に関係すること**（社内の話、評価、上司、社内ツールなど）。純粋な勉強の内容だけを書く
- **秘密**：Cloudflare Tunnel のトークン（`eyJ...`）、API キー、パスワード
- **自宅のグローバル IP**
- 画面のスクショを載せるときも、上の3つが写っていないか確認する

## 書き方のルール

- 日本語で短く。会話で使ったたとえや、自分の環境の具体的な値は残す
- 用語は `glossary.md` の該当セクションの表に1行。詳しく説明した内容は、フォルダに `トピック名.md`（英小文字とハイフン）を作り、用語集とフォルダの `README.md` からリンクする
- 同じ用語を二重に作らない。既存の行を良くする
- ロードマップが進んだら `README.md` のチェックを更新する
- コミットメッセージは英語で短く（例：`Add note: security/request-routes.md`）。書く前に `git pull --rebase`

## 環境のメモ

| もの | 値 |
|---|---|
| Ubuntu（Hyper-V の中） | `192.168.100.10`（NATSwitch）、Tailscale は `100.114.194.19` |
| Windows（ホスト） | 自宅 LAN 側 `192.168.11.10`、NATSwitch 側 `192.168.100.1`。8080 → Ubuntu:80 に portproxy |
| ssh | `ssh hiromuserver`（家の中）/ `ssh hiromuserver-ts`（Tailscale 経由）→ [ssh/ssh-config.md](ssh/ssh-config.md) |
| 公開 | `https://test.hiromu-lab.dev`（Hello）、`https://status.hiromu-lab.dev/api/status`（合言葉が必要） |
| ufw | eth0 は `192.168.100.0/24` から 22・80、tailscale0 は 22・80 だけ許可 |
