---
title: "ライブラリアップデート時のスタイルの変化をVisual Regression Testで検知しようとしたけどできなかった話" # 記事のタイトル
emoji: "🖼" # アイキャッチとして使われる絵文字（1文字だけ）
type: "tech" # tech: 技術記事 / idea: アイデア記事
topics: ["nextjs", "githubactions", "storybook", "regsuit", "puppeteer"] # タグ。["markdown", "rust", "aws"]のように指定する
published: true # 公開設定（falseにすると下書き）
---

# はじめに
趣味で開発しているプロダクトのライブラリアップデートの自動化にRenovateを使っています。マイナー/パッチアップデートの場合はアプリケーションの動作確認をユニットテストに任せていましたが、メジャーアップデートの場合は動作確認を手動で行なっていました。

あるときgithub-markdown-cssを4.0.0から5.1.0に変更して動作確認していると「エラーとして報告されないけど見た目が変わってしまっている」ようなアップデートの影響を見つけました。

![](https://storage.googleapis.com/zenn-user-upload/aa6e2e46c078-20220630.png)
*before*

![](https://storage.googleapis.com/zenn-user-upload/ca865528692d-20220630.png)
*after(なぜか背景が黒くなる)*

ユニットテストやNext.jsのビルドでエラーにならないので目視で確認しないと気付きませんでした。こういった影響をライブラリをアップデートするたびに手動で確認するのは手間だなと思ったので、ある程度自動化できないかと思い方法を調べました。今回は以下の記事を参考にVisual Regression Testを使って視覚的な変化を検知する方法を試しました。

https://blog.nnn.dev/entry/2021/04/30/110000

# Visual Regression Test (VRT) とは
こちらの記事が詳しいです。

> Visual Regression Test(VRT)とはその名の通り、視覚的にリグレッションテストを行うことです。
> フロントエンドの開発者、そうでなくてもHTMLやCSSの修正作業をすることのあるエンジニアの方は下記のような問題に遭遇することがあるのではないでしょうか。
> 
> - あるコンポーネントのCSSを変更しただけのつもりだったのに、なぜか他のページのレイアウトが崩れてしまった
> - デザインのレイアウトルールが変わったので、共通で使用しているCSSを修正する必要があるが、影響のあるコンポーネントが正常に表示されるか全てを確認することが難しい、結果的に「えいや」でリリースせざるを得ない
> 
> 通常のロジックのリグレッションテストであれば、自動テストなどで担保できますが、HTMLやCSSなどのレイアウトなどの視覚面の実装に関しては、人の目で差分が想定通りか確認する必要があります。
> この面倒な作業をできる限り自動化してくれる解決策が、Visual Regression Testです。

https://zenn.dev/toshiokun/articles/3d7087b84ba1d9

# やること
- Next.jsアプリケーションへのStorybook, Storycap, reg-suitの追加
- Storycapのスナップショットを保存するS3バケットの作成
- Visual Regression TestをCI（GitHub Actions）で実行する環境構築

# 実行環境
- yarn@1.22.19
- npm packages
    - next@12.1.6
    - @storybook/react@6.5.9
    - storycap@3.1.9
      - puppeteer@13.4.0
    - reg-suit@0.12.1

# サンプルコード
実際に実装した内容は以下のリポジトリで確認できます。

https://github.com/krabben16/yurikago-next

# ライブラリのセットアップ
## Storybookの追加
参考記事の手順でほとんど対応できたので省略します。

```json:package.json
    "storybook": "start-storybook -p 9001",
```

背景色が変化しているあたりのコンポーネントをストーリー（`Content With Code Block`）として作成しました。

![](https://storage.googleapis.com/zenn-user-upload/80dc3839d403-20220630.png)

https://zenn.dev/minguu42/articles/20211226-nextjs-storybook

## Storycapの追加

```
yarn add storycap
```

```json:package.json
    "storycap": "storycap --serverCmd 'yarn storybook' http://localhost:9001 --serverTimeout 600000 --delay 1000",
```

リソースの取得やページのレンダリングの完了を待機するため`--delay`オプションを指定しました。デフォルトだと`__screenshots__`ディレクトリにスナップショットが出力されます。

https://github.com/reg-viz/storycap#install

## reg-suitの追加

```
yarn add reg-suit
yarn reg-suit init
```

いくつか質問に答えて設定ファイルを作成します。

```json:regconfig.json
{
  "core": {
    "workingDir": ".reg",
    "actualDir": "__screenshots__",
    "thresholdRate": 0.001,
    "addIgnore": true,
    "ximgdiff": {
      "invocationType": "client"
    }
  },
  "plugins": {
    "reg-keygen-git-hash-plugin": true,
    "reg-notify-github-plugin": {
      "prComment": true,
      "prCommentBehavior": "default",
      "clientId": "<クライアントID>",
      "setCommitStatus": false
    },
    "reg-publish-s3-plugin": {
      "bucketName": "<S3バケット名>"
    }
  }
}
```

reg-suitの実行結果をPRのステータスに反映させないよう`setCommitStatus`を無効にしました。これがtrueだと、意図した変更なのにステータスが赤いPRをマージすることになり正常な感覚を失う恐れがあるためです。（N予備校のエンジニアの方の受け売り）

```json:package.json
    "regression": "reg-suit run"
```

https://blog.wadackel.me/2018/storybook-chrome-screenshot-with-reg-viz/#reg-suit

# S3バケットの作成
reg-suitを実行したときにスナップショットをアップロードするS3バケットを作成します。

- オブジェクト所有者の設定
  - ACL有効、オブジェクト所有者は`希望するバケット所有者`とする。
- パブリックアクセスの設定
  - 以下の二つにチェック
    - `新しいパブリックバケットポリシーまたはアクセスポイントポリシーを介して付与されたバケットとオブジェクトへのパブリックアクセスをブロックする`
    - `任意のパブリックバケットポリシーまたはアクセスポイントポリシーを介したバケットとオブジェクトへのパブリックアクセスとクロスアカウントアクセスをブロックする`

https://zenn.dev/toshiokun/articles/3d7087b84ba1d9#%E3%82%A2%E3%83%83%E3%83%97%E3%83%AD%E3%83%BC%E3%83%89%E5%85%88%E3%81%AEs3%E3%81%AE%E6%BA%96%E5%82%99%E3%82%92%E3%81%99%E3%82%8B

## S3バケットに書き込みできるIAMユーザーの作成
上記で作成したバケットしか操作できないインラインポリシーをアタッチしました。このユーザーのクレデンシャルを使ってreg-suitを実行します。

# CIの設定
アプリケーションを修正するたびにVRTを手動で実行するのは手間なのでCIで実行するようにします。想定しているフローは以下です。

1. RenovateがライブラリアップデートのブランチとPRを作成
2. CIが動いてライブラリがアップデートされた状態のアプリケーションのスナップショットを作成
3. 基点ブランチのスナップショットとRenovateが作成したブランチのスナップショットを比較してPRに通知

CIでは作成したIAMユーザーのクレデンシャルを環境変数に設定する必要があります。（`AWS_ACCESS_KEY_ID`, `AWS_SECRET_ACCESS_KEY`の部分）

## secretの読み込み
クライアントIDとS3バケット名は環境変数を経由して読み込むようにしました。

```yml:cicd.yml
env:
  REG_SUIT_CLIENT_ID: ${{ secrets.REG_SUIT_CLIENT_ID }}
  REG_SUIT_S3_BUCKET_NAME: ${{ secrets.REG_SUIT_S3_BUCKET_NAME }}
  AWS_ACCESS_KEY_ID: ${{ secrets.AWS_ACCESS_KEY_ID }}
  AWS_SECRET_ACCESS_KEY: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
```

```json:regconfig.json
  "plugins": {
    "reg-keygen-git-hash-plugin": true,
    "reg-notify-github-plugin": {
      "prComment": true,
      "prCommentBehavior": "default",
      "clientId": "$REG_SUIT_CLIENT_ID", # 環境変数 REG_SUIT_CLIENT_ID の値を参照する
      "setCommitStatus": false
    },
    "reg-publish-s3-plugin": {
      "bucketName": "$REG_SUIT_S3_BUCKET_NAME"
    }
  }
```

https://qiita.com/eretica/items/080cf75f8b9d2d928799

## 切り離されたHEADの回避策
READMEのCIセクションに記述されている設定を追加します。

```yml:cicd.yml
      - uses: actions/checkout@v2
        with:
          fetch-depth: 0
```

```yml:cicd.yml
      - name: workaround for detached HEAD
        run: |
          git checkout ${GITHUB_REF#refs/heads/} || git checkout -b ${GITHUB_REF#refs/heads/} && git pull
```

https://github.com/reg-viz/reg-suit#run-with-ci-service

この設定がないとreg-suitの実行時に前回のスナップショットを検出できず比較がスキップされてしまいます。

https://github.com/reg-viz/reg-suit/issues/186

## 日本語フォントの追加
Storycapで取得するスナップショットの日本語が文字化けするので日本語フォントを追加します。

```yml:cicd.yml
- name: Install Japanese fonts
  run: |
    sudo apt install fonts-ipafont fonts-ipaexfont
```

https://www.pnkts.net/2021/10/14/chromium-chromedriver-japanese

# VRTの実行
CIの発火条件は以下を設定しました。何かしらコミットをプッシュすればCIが発火します。

```yml:cicd.yml
on:
  push:
```

VRTのジョブは以下を設定しました。

```yml:cicd.yml
- name: Visual Regression Test
  run: |
    yarn storycap
    yarn regression
```

1. Storycapを実行してストーリーのスナップショットを作成します。
2. reg-suitを実行して前回と今回のスナップショットの比較を行います。
    - 前回のスナップショットをS3バケットから取得します。
    - 今回のスナップショットをS3バケットにアップロードします。

実際にVRTのジョブを実行しました。

![](https://storage.googleapis.com/zenn-user-upload/291da8fb2677-20220630.png)

ログに以下の文言が出力されています。`Content With Code Block`の変化が検知されているのではないでしょうか！？

```log
[reg-suit] debug fail: Blog/Content/Content With Code Block.png
:
[reg-suit] info    Changed items: 1
```

![](https://storage.googleapis.com/zenn-user-upload/7c342a2f3214-20220630.png)

## PRへの通知
ジョブが完了すると比較結果がPRにコメントされます。Changed itemsは赤い丸として表示されます。

![](https://storage.googleapis.com/zenn-user-upload/327823eb8a22-20220630.png)

コメントの`this report`をクリックすると比較結果のレポートを閲覧できます。

## 比較結果のレポート
CHANGED ITEMSに`Content With Code Block`が表示されています。

![](https://storage.googleapis.com/zenn-user-upload/1e3a084369a2-20220630.png)

ライブラリをアップデートして背景が黒くなったことが検知されているに違いありません。実際にどういうスナップショットが比較されたのか見てみましょう。CHANGED ITEMSのサムネイルをクリックします。

![](https://storage.googleapis.com/zenn-user-upload/df898574f484-20220630.png)
*左：基点ブランチのスナップショット 右：Renovateブランチのスナップショット*

………右の背景が黒くなっていませんね。

ちなみにローカル環境でRenovateブランチをcheckoutして`next`コマンドを実行してみると背景は黒くなりました。ローカル環境とCI環境の間に何らかの違いがあり、その違いが原因で背景が黒くなるかどうかが変化するようです🤔

## CI環境で背景が黒くならない原因
ローカル環境で背景を黒くしているCSSを確認するとbackground-colorに変数`--color-canvas-default`が設定されていました。その値は`#0d1117`でした。

![](https://storage.googleapis.com/zenn-user-upload/8bcadb9a4ca2-20220630.png)

github-markdown-cssのソースを確認するとメディアクエリのprefers-color-schemeが`dark`のときに上記の背景色が設定されるようです。

```css
@media (prefers-color-scheme: dark) {
  .markdown-body {
:
    --color-canvas-default: #0d1117;
```

https://github.com/sindresorhus/github-markdown-css/blob/v5.1.0/github-markdown.css#L37

つまりブラウザのテーマがダークモードなら背景が黒くなり、ライトモードなら背景は黒くならないということです。実際、ブラウザのテーマをライトモードに変更してローカル環境で`next`コマンドを実行してみると背景は白くなりました。

## CI環境で背景を黒くする方法
Storycapは内部でPuppeteerを使用しているようです。

https://github.com/reg-viz/storycap#features

以下のように`emulateMediaFeatures`を実行すればダークモードでスナップショットを作成できそうな気がします。

```js
await page.emulateMediaFeatures([
  { name: "prefers-color-scheme", value: "dark" },
]);
```

https://design.dena.com/engineering/puppeteer-v2

ところがStorycapのCLIのオプションの中に`emulateMediaFeatures`を実行できそうなものはありませんでした。

https://github.com/reg-viz/storycap#command-line-options

`--puppeteerLaunchConfig`は指定できますがその中でprefers-color-schemeを指定できそうなものはありませんでした。

https://pptr.dev/#?product=Puppeteer&version=v13.4.0&show=api-puppeteerlaunchoptions

# 結論

現時点ではCI環境でStorycapの実行時にダークモードでスナップショットを作成できず、ローカル環境で発生する背景が黒くなる現象を再現できないという結論に至りました…。もしダークモードでスナップショットを作成する方法をご存じの方はぜひ教えていただきたいです。

# 感想
Visual Regression TestをCIで実行する環境を構築して、ライブラリをアップデートした影響によるスタイルの変化を自動で検知しようとしましたが今回のようなケースでは対応できませんでした。

ただ、冒頭で引用した通りVRTを導入することでメリットが発生するケースがいくつかあります。
- あるコンポーネントのCSSを変更したら意図せず別のページのレイアウトが崩れたことを検知したい
- 共通で使っているCSSを修正したとき影響のあるコンポーネントの変化を検知したい

またVRTをCIで実行することで修正による影響の確認フローを自動化できます。PRを作ると基点ブランチとfeatureブランチのスナップショットを比較して結果をPRにコメントするような仕組みを構築できます。

修正確認を手動で行うことに厳しさを感じている方はVRTを試してみてはいかがでしょうか。
