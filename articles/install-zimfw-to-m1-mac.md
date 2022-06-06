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
