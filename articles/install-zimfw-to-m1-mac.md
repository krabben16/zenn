---
title: "M1 MacにZshフレームワークのZimを導入する" # 記事のタイトル
emoji: "🚴" # アイキャッチとして使われる絵文字（1文字だけ）
type: "tech" # tech: 技術記事 / idea: アイデア記事
topics: ["shell", "zsh", "zim"] # タグ。["markdown", "rust", "aws"]のように指定する
published: false # 公開設定（falseにすると下書き）
---

# Zimとは何ですか？
プラグインマネージャ、便利なモジュール、豊富なテーマを速度を犠牲にすることなくバンドルしたZsh構成フレームワークです。
最初のプロンプトが表示されるまでの時間を測定したところ、測定対象の中で上から3番目という結果が示されています。
以前に使ったことのあるPreztoが約0.14sec、Zimはその半分以下の約0.06secでした。
Preztoも十分速いのでは…？🤔

測定方法、結果については以下のWikiで確認できます。
https://github.com/zimfw/zimfw/wiki/Speed

# 環境構築
- M1 MBP@12.4
- Zsh@5.8.1
- Zim@1.9.1

# インストール
自動インストール、手動インストールがあります。
今回は簡単な自動インストールを試します。
インストール手順はREADMEに記載されています。

https://github.com/zimfw/zimfw

```sh
$ curl -fsSL https://raw.githubusercontent.com/zimfw/install/master/install.zsh | zsh
) Using Zsh version 5.8.1
) ZIM_HOME not set, using the default one.
) Zsh is your default shell.
) Downloaded the Zim script to /Users/xxx/.zim/zimfw.zsh
) Prepended Zim template to /Users/xxx/.zimrc
) Prepended Zim template to /Users/xxx/.zshrc
) Installed modules.
All done. Enjoy your Zsh IMproved! Restart your terminal for changes to take effect.
```

コマンドの実行後にターミナルを再起動すれば完了です。
`~/.zshrc`にzimの設定が追記されています。

# 有効になる機能
デフォルトで以下のモジュールが読み込まれています。

|モジュール名|説明|
|---|---|
|environment|正常なZsh組み込み環境オプションを設定します。|
|git|便利なgitエイリアスや関数を提供します。|
|input|入力イベントに対して正しいバインドキーを適用します。|
|termtitle|カスタム端末のタイトルを設定します。|
|utility|ユーティリティのエイリアスや関数。ls, grep, lessに色をつけます。|
|duration-info|asciishipで使用される、最後のコマンドの実行にかかった時間をプロンプトに公開します。|
|git-info|asciishipで使用される、gitリポジトリの状態情報をプロンプトに公開します。|
|asciiship|スペースシップとスターシップのプロンプトを大幅に縮小した、ASCIIのみのバージョンです。|
|zsh-users/zsh-completions --fpath src|Zshの補完定義を追加します。|
|completion|スマートで広範囲なタブ補完を有効にし設定します。completionは補完定義を追加するすべてのモジュールの後に読み込む必要があります。|
|zsh-users/zsh-syntax-highlighting|Zsh用のFishのようなシンタックスハイライト。zsh-users/zsh-syntax-highlightingはCompletionモジュールの後に読み込む必要があります。|
|zsh-users/zsh-history-substring-search|Zsh用のFishのような履歴検索。zsh-users/zsh-history-substring-searchはzsh-users/zsh-syntax-highlightingの後に読み込む必要があります。|
|zsh-users/zsh-autosuggestions|Zsh用のFishのようなオートサジェスト|

# テーマの変更
デフォルトではasciishipというテーマが適用されています。
テーマを変更したい場合は`~/.zimrc`を編集します。

```
# asciishipをコメントアウト
# zmodule asciiship
# 設定したいテーマ名を指定
zmodule steeef
```

テーマの一覧
https://zimfw.sh/docs/themes/

追記後、以下のコマンドを実行します。

```sh
$ zimfw install
```

テーマモジュールのインストール後、ターミナルを再起動すれば完了です。
