---
title: "マークダウンを校正してPDF形式のスライドに変換する(Slidev+textlint)" # 記事のタイトル
emoji: "👮" # アイキャッチとして使われる絵文字（1文字だけ）
type: "tech" # tech: 技術記事 / idea: アイデア記事
topics: ["slidev", "vue", "vite", "markdown", "textlint", "githubactions"] # タグ。["markdown", "rust", "aws"]のように指定する
published: true # 公開設定（falseにすると下書き）
---

# はじめに
スライドを作る作業って辛くないですか？
内容の構成を考えるだけで頭が痛いのに、テキスト位置を調整したり、書式を変えたり、画像を貼り付けたり…。

そんな鬱屈した日々を過ごしていたある日、この記事を見かけました。Slidevの特徴と使い方が丁寧に解説されています。

https://zenn.dev/ryo_kawamata/articles/introduce-slidev

重要な部分のみ引用します。

> reveal.jsやremarkなどのマークダウン形式でのスライド作成ツールとほぼ同様の記法で、スライドを作成できます。大抵のツールでは、ある程度決まったレイアウトになるのですが、Slidev の場合は拡張性がかなり高いです。Vue Component や、WindiCSS でスタイリングすることでページごとに独自のスタイルを当てて自由なレイアウトで構築することも出来きます。

…マークダウン形式でスライドを作成できます。苦悶と哀愁に満ちたスライド作成作業に光明が差しました。

# 完成形
Slidevで作成できるスライドはこういう感じです。

@[slideshare](39hJ8UIEf2fYVh)

# なにする？
1. SlidevでマークダウンをPDFに変換します。
    - PDFの変換はCIではなくローカル環境で行う
2. PRの作成時/ブランチ同期時にtextlintでマークダウンの文章を校正します。
    - 校正した結果をPRのコメントとして投稿する

# 検証環境
- M1 MBP@12.4
- yarn@1.22.19
- npm packages
    - @slidev/cli@0.33.0
    - textlint@12.1.1

# Slidevの設定
## インストール
ドキュメントの手順に従います。

https://sli.dev/guide/install.html#starter-template

以下のコマンドを実行するとスターターテンプレートが出力されます。プロンプトに従いプロジェクト名を入力したり使用するパッケージマネージャーを選択したりします。

```sh
$ yarn create slidev
```

## 開発環境の起動
スターターテンプレートの中に`slides.md`というファイルがあります。このマークダウンファイルがスライドの雛形です。以下のコマンドを実行するとSlidevのローカルサーバを起動します。`http://localhost:3030/`でスライドのプレビューを確認できます。

```sh
$ yarn dev
yarn run v1.22.19
$ slidev --open


  ●■▲
  Slidev  v0.33.0 

  theme   @slidev/theme-seriph
  entry   /Users/xxx/projects/slides/slides.md

  public slide show   > http://localhost:3030/
  presenter mode      > http://localhost:3030/presenter/
  remote control      > pass --remote to enable

  shortcuts           > restart | open | edit
```

![](https://storage.googleapis.com/zenn-user-upload/22b5d098ee34-20220611.png)

`slides.md`以外のプレビューを確認したい場合は引数を追加します。

```sh
$ yarn slidev slides-2.md
```

## PDFエクスポート
以下のコマンドを実行するとマークダウンをPDFに変換して出力します。レンダリングにPlaywrightを使用しているとのことでパッケージの追加が必要です。

```sh
$ yarn add -D playwright-chromium
```

次にexportコマンドを実行します。デフォルトだと`./slides-exports.pdf`というパスに出力されます。パスを変更したい場合は引数`--output`を追加します。

```sh
$ yarn export slides.md --output output.pdf
yarn run v1.22.19
$ slidev export slides.md --output output.pdf


  ●■▲
  Slidev  v0.33.0 

  theme   @slidev/theme-seriph
  entry   /Users/xxx/projects/slides/slides.md


  ✓ exported to ./output.pdf

✨  Done in 8.03s.
```

https://ja.sli.dev/guide/exporting.html

# Slidevの実装
## Markdownシンタックス
[Markdownの機能](https://github.com/adam-p/markdown-here/wiki/Markdown-Cheatsheet)に加えて、インラインHTMLとVueコンポーネントがサポートされています。

~~~md
<!-- Markdownの機能 (コードブロッック) -->
```html
<Counter :count="10" />
```

<!-- インラインHTML -->
<div></div>

<!-- Vueコンポーネント -->
<Counter :count="10" />
~~~

https://ja.sli.dev/guide/syntax.html

## テーマの変更
Slidevにはテーマファイルの仕組みがあります。マークダウンのFrontmatterに設定するだけで簡単に導入できます。

```md
# try also 'default' to start simple
theme: bricks
```

テーマファイルの一覧

https://ja.sli.dev/themes/gallery.html

![](https://storage.googleapis.com/zenn-user-upload/bc2480390268-20220611.png)

## 環境変数の参照
いろいろ試している時にたまたま見つけて、需要があるのか微妙ですが書いておきます。

Slidevは内部でViteを使っています。なので`.env`に定義した環境変数をVue Componentの`import.meta.env.xxx`で参照することができます。

https://ja.vitejs.dev/guide/env-and-mode.html

# textlintの設定
## textlintとは？

> textlintとはLintと呼ばれる静的解析ツールで、テキストファイル（.txt）やMarkdownファイル（.md）等を対象に、校正ルールにもとづいて文章校正を行うツールです。校正ルールの自作や、提供されているルールを組み合わせて独自のルールセットを作成できます。

https://ics.media/entry/220404/

## インストール
以下のコマンドを実行します。

```sh
$ yarn add -D textlint
```

https://textlint.github.io/docs/getting-started.html

校正したい内容に合わせてtextlintのルールのパッケージを追加します。自分の環境では以下を追加しました。

|ルール名|説明|
|---|---|
|textlint-rule-no-mix-dearu-desumasu|「ですます」調と「である」調の混在をチェックするルール|
|textlint-rule-no-double-negative-ja|二重否定をチェックするルール|
|textlint-rule-no-hankaku-kana|半角カタカナの使用を禁止するルール|
|textlint-rule-no-dropping-the-ra|ら抜き言葉か使われてないかをチェックするルール|
|textlint-rule-ja-no-mixed-period|文末の句点記号(。)の統一 と 抜けをチェックするルール|
|textlint-rule-prefer-tari-tari|例示・並列・対表現の「〜たり〜たりする」をチェックするルール|
|@textlint-ja/textlint-rule-no-synonyms|リポジトリとレポジトリといった同義語の表記ゆれをチェックするルール|
|sudachi-synonyms-dictionary|Sudachi 同義語辞書を扱うパッケージ。textlint-rule-no-synonymsで必要。|

```sh
$ yarn add -D \
    textlint-rule-no-mix-dearu-desumasu \
    textlint-rule-no-double-negative-ja \
    textlint-rule-no-hankaku-kana \
    textlint-rule-no-dropping-the-ra \
    textlint-rule-ja-no-mixed-period \
    textlint-rule-prefer-tari-tari \
    @textlint-ja/textlint-rule-no-synonyms \
    sudachi-synonyms-dictionary
```

## 設定ファイルの作成
インストールしたルールを有効にします。

```js:.textlintrc.js
module.exports = {
    rules: {
        "no-mix-dearu-desumasu": true,
        "no-double-negative-ja": true,
        "no-hankaku-kana": true,
        "no-dropping-the-ra": true,
        "ja-no-mixed-period": true,
        "prefer-tari-tari": true,
        "@textlint-ja/no-synonyms": true,
    }
}
```

## 動作確認
マークダウンがどのように校正されるか試してみます。`package.json`のscriptsに以下のコマンドを追加します。

```json:package.json
"scripts": {
:
  "lint": "textlint"
},
```

テスト用のマークダウンを対象にtextlintを実行します。ルールに違反している箇所と内容がエラーメッセージとして表示されます。

```md:test.md
食べれる
```

```sh
$ yarn lint test.md
yarn run v1.22.19
$ textlint test.md

/Users/xxx/projects/slides/test.md
  1:3  error  ら抜き言葉を使用しています。    no-dropping-the-ra
  1:4  error  文末が"。"で終わっていません。  ja-no-mixed-period

✖ 2 problems (2 errors, 0 warnings)

error Command failed with exit code 1.
info Visit https://yarnpkg.com/en/docs/cli/run for documentation about this command.
```

# GitHub Actionsの設定
## 設定ファイルの作成
textlintの実行を自動化します。PRの作成時とブランチの同期時にtextlintを実行するように設定します。以下のパスにファイルを作成します。

```yml
name: run-textlint-minimum
on: 
  pull_request_target:
    types: [ opened, synchronize ] # PRの作成 or ブランチの同期がトリガー
    paths: [ '*.md' ] # 全てのmdファイルの追加/更新がトリガー
jobs:
  run-textlint-minimum:
    runs-on: ubuntu-latest
    permissions:
      pull-requests: write # "gh pr comment"を実行するための権限を付与
    steps:
      # レビューを行う作業ブランチに切り替え
      - uses: actions/checkout@v3
        with:
          ref: ${{ github.event.pull_request.head.sha }}
      # Node.js環境をセットアップ
      - uses: actions/setup-node@v3
        with:
          node-version: 14
      - run: npm install
      # textlintを実行。実行ログをログファイルに保存。
      - run: npx textlint *.md >> ./.textlint.log
      - if: ${{ failure() }} # 「失敗したときのみ実行する」条件
        run: gh pr comment --body-file ./.textlint.log "${URL}"
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
          URL: ${{ github.event.pull_request.html_url }}
```

こちらのサンプルファイルを参考にさせていただきました。

https://dev.classmethod.jp/articles/markdown-writing-with-textlint-ci/

## PRにtextlintの実行結果が通知されることの確認
PRの作成時、およびブランチの同期時にtextlintが実行されます。どのファイルがどのルールに違反したのか通知してくれます。`Update test.md`のコミットでら抜き言葉を修正したことでtextlintの実行結果が変化しています。

![](https://storage.googleapis.com/zenn-user-upload/20f0db0e9126-20220611.png)

# おわりに
以下の流れでマークダウンをスライドに変換する仕組みを構築しました。
1. スライドにしたいマークダウンを作成 -> PRでtextlintによるレビューを受ける
2. レビューの指摘がなくなったらローカル環境でマークダウンをPDFに書き出す

手動でスライドを作成することに絶望し、マークダウンをスライドに変換する手法に活路を見出しました。Slidevのフォーマットでマークダウンを書くのは、それはそれで苦労しそうと思いつつ…。textlintと組み合わせて文章の校正を機械任せにできるのは大きなメリットだと感じました。

本当はスライドの作成をローカル環境ではなくGitHub Actionsで行いたかったのですが、キリのいいところでいったん止めました。後ほど試してみて記事にしようと思います。
