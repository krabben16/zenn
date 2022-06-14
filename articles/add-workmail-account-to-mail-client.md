---
title: "AWS WorkMailのアカウントをThunderbirdに設定する" # 記事のタイトル
emoji: "🐏" # アイキャッチとして使われる絵文字（1文字だけ）
type: "tech" # tech: 技術記事 / idea: アイデア記事
topics: ["aws", "workmail", "thunderbird"] # タグ。["markdown", "rust", "aws"]のように指定する
published: false # 公開設定（falseにすると下書き）
---

# WorkMailとは？
> Amazon WorkMail はセキュリティに優れた企業向け E メールおよびカレンダーのマネージド型サービスで、デスクトップとモバイルの既存の E メールクライアントアプリケーションに対応しています。Amazon WorkMail によって、ユーザーは選択したクライアントアプリケーションを使って自分の E メール、連絡先、およびカレンダーにシームレスにアクセスできるようになります。

https://aws.amazon.com/jp/workmail/

ドメイン登録やアカウント作成などのセットアップ方法はこちらの記事で解説されているので省略します🙏

https://zenn.dev/bun913/articles/work-mail-practice

# Thunderbirdの設定
まず`既存のメールアドレスのセットアップ`画面を表示します。WorkMailで作成したアカウントのメールアドレス、パスワードを入力します。タコがおもろいですね（ハサミ使える…？🐙）

![](https://gyazo.com/83058206e0152bc2bd5ac4600d99d8e0.png)

次に手動設定をクリックします。受信サーバーにAWSのIMAPサーバー、送信サーバーにAWSのSMTPサーバーを設定します。`ホスト名`はWorkMailでアカウントを作成したリージョンを指定します。（例. us-east-1）
`ユーザー名`は（ややこしいですが）WorkMailで作成したアカウントのメールアドレスを入力します。

**受信サーバー**

| 項目 | 値 |
| --- | --- |
| プロトコル | IMAP  |
| ホスト名 | imap.mail.<WorkMailでアカウントを作成したリージョン>.awsapps.com |
| ポート番号 | 993 |
| 接続の保護 | SSL/TLS |
| 認証方式 | 通常のパスワード認証 |
| ユーザー名 | <WorkMailで作成したアカウントのメールアドレス> |

**送信サーバー**

| 項目 | 値 |
| --- | --- |
| ホスト名 | smtp.mail.<WorkMailでアカウントを作成したリージョン>.awsapps.com |
| ポート番号 | 465 |
| 接続の保護 | 自動検出 |
| 認証方式 | 自動検出 |
| ユーザー名 | <WorkMailで作成したアカウントのメールアドレス> |

![](https://gyazo.com/063777fcbfe91d6b44ae5e102a3e9caf.png)

## 接続確認
画面下部の`再テスト`ボタンをクリックします。

![](https://gyazo.com/d324fd6dba59d4f1f1fc1b0043dffeb2.png)

設定が問題なければ以下のような成功メッセージが表示されるので`完了`ボタンをクリックして終了です。

![](https://gyazo.com/49748e64ad0a21182c085eeecb64f320.png)

# おわりに
受信/送信サーバーの接続情報が分からなくて時間を浪費しました。ドキュメントにしっかり書いてありました🤷

https://docs.aws.amazon.com/ja_jp/workmail/latest/userguide/using_IMAP.html
