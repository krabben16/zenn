---
title: "MacでGitHub CLIをセットアップする" # 記事のタイトル
emoji: "🐙" # アイキャッチとして使われる絵文字（1文字だけ）
type: "tech" # tech: 技術記事 / idea: アイデア記事
topics: ["m1", "macos", "git", "github"] # タグ。["markdown", "rust", "aws"]のように指定する
published: false # 公開設定（falseにすると下書き）
---

# GitHub CLIとは
GitHubをターミナルやスクリプトから操作するためのCLIです。色々できます。

**Core commands**

|コマンド|説明|
|---|---|
|gh auth|GitHubでghとgitを認証する|
|gh browse|WebブラウザでGitHubリポジトリを開く|
|gh codespace|Codespaceに接続して管理する|
|gh gist|gistsを操作する|
|gh issue|イシューを操作する|
|gh pr|PRを操作する|
|gh release|リリースを管理する|
|gh repo|リポジトリを操作する|

**Actions commands**

|コマンド|説明|
|---|---|
|gh run|GitHub Actionsから実行された最近のワークフローを一覧表示、表示、および監視する|
|gh workflow|GitHub Actionsでワークフローを一覧表示、表示、実行する|

**Additional commands**

|コマンド|説明|
|---|---|
|gh alias|エイリアスを使用して、ghコマンドのショートカットを作成したり、複数のコマンドを作成したり|
|gh api|GitHub APIに対して認証済みのHTTPリクエストを作成し、レスポンスを出力する|
|gh completion|GitHubCLIコマンドのshell completion scriptsを生成する|
|gh config|ghの構成設定を表示または変更する|
|gh extension|拡張機能を追加する|
|gh gpg-key|GitHubアカウントに登録されているGPGキーを管理する|
|gh label|ラベルを操作する|
|gh search|GitHub全体を検索する|
|gh secret|secretをリポジトリまたは組織レベルで設定する|
|gh ssh-key|GitHubアカウントに登録されているSSHキーを管理する|
|gh status|サブスクライブしているすべてのリポジトリにわたって、GitHubでの作業に関する情報を出力する|

https://cli.github.com/manual/gh

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
CLIとGitHubアカウントの認証を行います。以下のコマンドを実行します。


```sh
$ gh auth login
```

いくつか質問されるので回答していきます。

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

Enterを押すとブラウザでGitHubのDevice Activationページが表示されます。提示されたone-time code(`XXXX-XXXX`の部分)をDevice Activationページのフォームに入力してContinueをクリックします。

![](https://storage.googleapis.com/zenn-user-upload/b77364ba9fb4-20220612.png)

CLIが要求するアクセス権限のリストが表示されます。問題なければAuthorize gitHubをクリックします。

![](https://storage.googleapis.com/zenn-user-upload/87a6ba2e8e90-20220612.png)

認証が完了しました。ターミナルに完了メッセージが表示されます。

![](https://storage.googleapis.com/zenn-user-upload/c0244526781d-20220612.png)

```
Press Enter to open github.com in your browser...
✓ Authentication complete.
- gh config set -h github.com git_protocol https
✓ Configured git protocol
✓ Logged in as krabben16
```

https://cli.github.com/manual/

# 動作確認
以下の条件でリポジトリの一覧を取得してJSONに整形してみます。

- リポジトリ内で最も使われている言語がTypeScript

JSONに出力するフィールドを指定します。

- リポジトリ名
- リポジトリ内で最も使われている言語
- リポジトリ内で使われている全ての言語とそのサイズ
- 作成日時
- 更新日時

```sh
$ gh repo list krabben16 --language typescript --json name,primaryLanguage,languages,createdAt,updatedAt
```

```json
[
  {
    "createdAt": "2022-06-03T14:21:00Z",
    "languages": [
      {
        "size": 1110,
        "node": {
          "name": "JavaScript"
        }
      },
      {
        "size": 1271,
        "node": {
          "name": "TypeScript"
        }
      }
    ],
    "name": "sandbox-cdk-ts",
    "primaryLanguage": {
      "name": "TypeScript"
    },
    "updatedAt": "2022-06-11T16:40:44Z"
  },
```

取得できました👏
