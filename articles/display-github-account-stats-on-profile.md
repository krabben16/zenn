---
title: "GitHubアカウントの統計情報をプロフィールに表示する" # 記事のタイトル
emoji: "🔍" # アイキャッチとして使われる絵文字（1文字だけ）
type: "tech" # tech: 技術記事 / idea: アイデア記事
topics: ["github", "githubactions"] # タグ。["markdown", "rust", "aws"]のように指定する
published: true # 公開設定（falseにすると下書き）
---

# はじめに
以下のインフォグラフィックスジェネレーターを使用します。GitHubアカウントの統計情報をSVG、Markdown、PDF、JSONなどにレンダリングできます。

https://github.com/lowlighter/metrics

セットアップの方法はいくつかありますが、今回は[GitHub Actionsを使う方法](
https://github.com/lowlighter/metrics/blob/master/.github/readme/partials/documentation/setup/action.md)を試してみます。

# リポジトリの作成
アカウント名と同じpublicリポジトリを作成します。
自分の環境だと [krabben16/krabben16](https://github.com/krabben16/krabben16) というリポジトリになります。

https://docs.github.com/ja/account-and-profile/setting-up-and-managing-your-github-profile/customizing-your-profile/managing-your-profile-readme

# Action codeの生成
以下のサイトで統計情報をどのように表示するかシミュレートできます。

https://metrics.lecoq.io/

また、取得した統計情報を画像に変換するGitHub Actionsのコードが出力されます。デフォルトの設定だと以下のコードが出力されます。

```yml
# Visit https://github.com/lowlighter/metrics/blob/master/action.yml for full reference
name: Metrics
on:
  # Schedule updates (each hour)
  schedule: [{cron: "0 * * * *"}]
  # Lines below let you run workflow manually and on each commit
  workflow_dispatch:
  push: {branches: ["master", "main"]}
jobs:
  github-metrics:
    runs-on: ubuntu-latest
    permissions:
      contents: write
    steps:
      - uses: lowlighter/metrics@latest
        with:
          # Your GitHub token
          # The following scopes are required:
          #  - public_access (default scope)
          # The following additional scopes may be required:
          #  - read:org      (for organization related metrics)
          #  - read:user     (for user related data)
          #  - read:packages (for some packages related data)
          #  - repo          (optional, if you want to include private repositories)
          token: ${{ secrets.METRICS_TOKEN }}

          # Options
          user: krabben16
          template: classic
          base: header, activity, community, repositories, metadata
          config_timezone: Asia/Tokyo
```

# GitHub Actionsの設定
Action codeをリポジトリの以下の場所に保存します。

https://github.com/krabben16/krabben16/blob/master/.github/workflows/metrics.yml

コード内で`secrets.METRICS_TOKEN`を参照しています。これはジェネレーターがGitHubアカウントの統計情報にアクセスするために必要なものです。Personal Access Tokenを生成して、その値をリポジトリのsecretsに設定します。

## Personal Access Tokenの生成
Action code内に記載されている通り、Select scopesで以下のアクセスを許可します。

- read:org
- read:user
- read:packages
- repo

https://github.com/settings/tokens

## secretsの設定
secretsは暗号化された環境変数のことです。生成したPersonal Access Tokenを`METRICS_TOKEN`という名前で登録します。設定場所はリポジトリのSettings > Security > Secrets > Actions です。

![](https://storage.googleapis.com/zenn-user-upload/f8c7dc7a4868-20220609.png)

## ワークフローの動作確認
設定が問題なければGitHub Actionsが正常に動作してリポジトリに`github-metrics.svg`というファイルがコミットされています。画像を生成するワークフローはyamlの`schedule`で設定した通り1時間に1回実行されます。

![](https://storage.googleapis.com/zenn-user-upload/e851a111f463-20220609.png)

# README.mdの作成
生成した画像を読み込むマークダウンファイルを作成します。

https://github.com/krabben16/krabben16/blob/master/README.md

# 画像の表示確認
GitHubアカウントのOverviewを表示した時に画像が表示されていれば完成です！

![](https://storage.googleapis.com/zenn-user-upload/557446a90ad1-20220609.png)

# おわりに
お気に入りはMost used languagesです。その他にも統計情報を視覚的に表示するプラグインがいろいろ用意されているので、プロフィールの見せ方のバリエーションが広がりますね🧐

プラグインの一覧

https://github.com/lowlighter/metrics/blob/master/README.md#-plugins
