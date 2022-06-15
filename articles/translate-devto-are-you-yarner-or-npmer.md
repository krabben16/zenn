---
title: "[翻訳]dev.to あなたはYARNerまたはNPMerですか？" # 記事のタイトル
emoji: "😺" # アイキャッチとして使われる絵文字（1文字だけ）
type: "idea" # tech: 技術記事 / idea: アイデア記事
topics: ["devto", "npm", "yarn"] # タグ。["markdown", "rust", "aws"]のように指定する
published: false # 公開設定（falseにすると下書き）
---

# はじめに

dev.toに投稿された記事の翻訳です。パッケージマネージャはYarnまたはNPMのどちらを使うか、というテーマで議論されています。

https://dev.to/charliesay/are-you-a-yarner-or-a-npmer-337j

自分はパッケージのインストールが早いイメージのあるYarnを使っていました。pnpmという選択肢もあるのですね🤔
各パッケージマネージャーの特徴は以下の記事にまとめられています。

https://zenn.dev/hibikine/articles/27621a7f95e761

# Yarn派

> I personally prefer Yarn - I think it's general installation of packages is quicker and I REALLY appreciate its peer dependency resolution it saves LOADS of time.

個人的にはYarnが好きです。パッケージのインストールが速いし、ピア依存の解決で時間をたくさん節約できるのが本当にありがたいです。

# NPM派

> NPM, because i'm lazy and it's the default.

NPMです。私が怠惰で、それがデフォルトだからです。

# pnpm派

> pnpm really fast installation ( in one monorepo I have 2,5 min vs 20 min in npm) and stricter in resolving dependencies. Also saves a lot of disk space if you have multiple projects.

pnpmはインストールがとても速く（1つのモノレポで2.5分、npmは20分）、依存関係の解決もより厳格になりました。また、複数のプロジェクトを持っている場合、ディスクスペースを大幅に節約することができます。

> PNPM all the way, unless for some reason I have to use something else for some reason.
> - Install a dependency another project is already using on the same machine > FAST
> - Clear node_modules and reinstall > VERY fast vs others
> - Raw install in a CI worflow or fresh OS distro > usually faster than Yarn, always faster than NPM in my experience
> - Workspaces just kinda works. I fought with workspaces in NPM and had all manner of issues with they way things were hoisted, dependency versioning issues, and even VS Code not recognizing imports from dependency packages I was writing in the same Monorepo. All gone with PNPM
> - MASSIVE savings in disk space, since each dependency version is stored in a global cache once, and then linked in to each project. This is also why things install faster, if a dependency has been installed on the OS before, nothing gets pulled over the network.

何らかの理由で他のものを使わなければならない場合を除き、ずっとPNPMです。
- 他のプロジェクトが既に使っている依存関係を同じマシンにインストールする > 速い
- node_modulesをクリアして再インストール > 他より非常に速い
- CIワークフローや新しいOSディストロで生インストール > 通常Yarnより速く、経験上NPMより常に速い
- ワークスペースはちょっとだけ使える。NPMのワークスペースでは、ホスティングの仕方や、依存関係のバージョン管理の問題、さらにはVS Codeが同じMonorepoで書いている依存パッケージからのインポートを認識しないなど、様々な問題を抱えていました。PNPMですべて解決
- 各依存性のバージョンは、一度グローバルキャッシュに保存され、その後各プロジェクトにリンクされるため、ディスクスペースを依存関係がOSにインストールされたことがある場合、ネットワーク経由で何も引き出されないからです。

