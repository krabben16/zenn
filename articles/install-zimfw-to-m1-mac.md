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

デフォルトで以下のモジュールが有効になっています。

|モジュール名|説明|
|---|---|
|environment|Sets sane Zsh built-in environment options.|
|git|Provides handy git aliases and functions.|
|input|Applies correct bindkeys for input events.|
|termtitle|Sets a custom terminal title.|
|utility|Utility aliases and functions. Adds colour to ls, grep and less.|
|duration-info|Exposes to prompts how long the last command took to execute, used by asciiship.|
|git-info|Exposes git repository status information to prompts, used by asciiship.|
|asciiship|A heavily reduced, ASCII-only version of the Spaceship and Starship prompts.|
|zsh-users/zsh-completions --fpath src|Additional completion definitions for Zsh.|
|completion|Enables and configures smart and extensive tab completion.completion must be sourced after all modules that add completion definitions.|
|zsh-users/zsh-syntax-highlighting|Fish-like syntax highlighting for Zsh.zsh-users/zsh-syntax-highlighting must be sourced after completion|
|zsh-users/zsh-history-substring-search|Fish-like history search (up arrow) for Zsh.zsh-users/zsh-history-substring-search must be sourced after zsh-users/zsh-syntax-highlighting|
|zsh-users/zsh-autosuggestions|Fish-like autosuggestions for Zsh.|

# テーマの変更
デフォルトではasciishipというテーマが適用されています。
テーマを変更したい場合は`~/.zimrc`を編集します。
自分の環境ではファイルの末尾に以下を追記しました。

```
# 変更したいテーマ名を指定
zmodule steeef
```

テーマの一覧
https://zimfw.sh/docs/themes/

追記後、以下のコマンドを実行します。

```sh
$ zimfw install
```

テーマモジュールのインストール後、ターミナルを再起動すれば完了です。
