---
title: "MacにGitHub CLIをセットアップする" # 記事のタイトル
emoji: "🐙" # アイキャッチとして使われる絵文字（1文字だけ）
type: "tech" # tech: 技術記事 / idea: アイデア記事
topics: ["m1", "macos", "git", "github"] # タグ。["markdown", "rust", "aws"]のように指定する
published: false # 公開設定（falseにすると下書き）
---

# はじめに
**GitHub CLIとは？**
リポジトリ、PR、Issueなどの情報を取得したり更新したりできるツールです。かなり色々できます。

# 検証環境
- M1 MBP@12.4
- gh@2.12.1 (GitHub CLI)

# インストール
Homebrew、MacPorts、Conda、Spackを経由してインストールできます。また[releases page](https://github.com/cli/cli/releases/latest)からバイナリをダウンロードすることが可能です。今回はHomebrewでインストールします。

```sh
$ brew install gh
```

https://github.com/cli/cli#macos

# 構成
CLIと自分のGitHubアカウントを紐づける認証を行います。以下のコマンドを実行します。


```sh
$ gh auth login
```

プロンプトが表示されるので回答していきます。

```
? What account do you want to log into?  [Use arrows to move, type to filter]
> GitHub.com
  GitHub Enterprise Server
? What is your preferred protocol for Git operations?  [Use arrows to move, type to filter]
> HTTPS
  SSH
? Authenticate Git with your GitHub credentials? (Y/n) Y
? How would you like to authenticate GitHub CLI?  [Use arrows to move, type to filter]
> Login with a web browser
  Paste an authentication token
? How would you like to authenticate GitHub CLI? Login with a web browser

! First copy your one-time code: XXXX-XXXX
Press Enter to open github.com in your browser...
```

Enterを押すとブラウザでGitHubのDevice Activationページが表示されます。提示されたone-time code(`XXXX-XXXX`の部分)をDevice Activationページのフォームに入力します。

![](https://storage.googleapis.com/zenn-user-upload/b77364ba9fb4-20220612.png)



認証が完了しました。プロンプトにも完了メッセージが表示されています。

![](https://storage.googleapis.com/zenn-user-upload/c0244526781d-20220612.png)

```
✓ Authentication complete.
- gh config set -h github.com git_protocol https
✓ Configured git protocol
✓ Logged in as krabben16
```

https://cli.github.com/manual/
