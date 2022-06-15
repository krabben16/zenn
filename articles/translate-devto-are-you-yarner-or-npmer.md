---
title: "[翻訳]dev.to あなたはYARNerまたはNPMerですか？" # 記事のタイトル
emoji: "😺" # アイキャッチとして使われる絵文字（1文字だけ）
type: "idea" # tech: 技術記事 / idea: アイデア記事
topics: ["devto", "npm", "yarn", "pnpm", "ni"] # タグ。["markdown", "rust", "aws"]のように指定する
published: true # 公開設定（falseにすると下書き）
---

# はじめに

dev.toに投稿された記事の翻訳です。（2022/06/15時点）
パッケージマネージャはYarnまたはNPMのどちらを使うか、というテーマで議論されています。自分はパッケージのインストールが早いイメージのあるYarnを使っていました。pnpmもインストールが速そうですね🤔

https://dev.to/charliesay/are-you-a-yarner-or-a-npmer-337j

# Yarn派

> I personally prefer Yarn - I think it's general installation of packages is quicker and I REALLY appreciate its peer dependency resolution it saves LOADS of time.

個人的にはYarnが好きです。パッケージのインストールが速いし、ピア依存の解決で時間をたくさん節約できるのが本当にありがたいです。

> Between NPM and Yarn I prefer Yarn for having more intuitive commands in my opinion. Nowadays I don't think there are any big differences in performance between the two anymore, I think yarn should still win because of the cache. Because if it's a clean install, they're very close.
> I've been using pnpm lately and I've found it super interesting and I've been loving it.

NPMとYarnの間ではより直感的なコマンドを使用できるYarnの方が好きです。最近では2つのパフォーマンスに大きな違いはないと思います。キャッシュがあるためyarnはまだ勝つはずです。クリーンインストールの場合、それらは非常に近いからです。
私は最近pnpmを使用していてとても面白くて気に入っています。

> yarn
> Because the pronunciation is a bit similar to my first name. 😁😁😁

yarnです。
発音が私のファーストネームに少し似ているので😁😁😁

> yarn ui is more beautiful than npm

yarnのUIはnpmより美しいです。

# NPM派

> NPM, because i'm lazy and it's the default.

NPMです。私が怠惰で、それがデフォルトだからです。

> Never bet against the platform, I was full into yarn but since npm has now all yarn features even the commands are same there is no need for yarn anymore all my projects just feel super clean now working directly with npm. If you still on yarn I recommend try going back to npm it is awesom.

私はyarnに傾倒していましたが、npmにはyarnのすべての機能があり、コマンドも同じなので、yarnはもう必要ありません。私のプロジェクトはすべてnpmで直接作業しているので、とてもすっきりしています。もしまだyarnを使っているならば、npmに戻ることをお勧めします。

> NPM, because it has more community support. I'd rather have more compatibility between my tools rather than save 10 minutes on an installation I only do a few times a year.

NPMです。より多くのコミュニティサポートがあるためです。年に数回しか実行しないインストールで10分節約するよりも、ツール間の互換性を高めたいと思います。

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
- CIワークフローや新しいOSディストリビューションで生インストール > 経験上、Yarnよりだいたい速く、NPMより常に速い
- ワークスペースはちょっとだけ使える。NPMのワークスペースでは、ホスティングの仕方や、依存関係のバージョン管理の問題、さらにはVS Codeが同じMonorepoで書いている依存パッケージからのインポートを認識しないなど、様々な問題を抱えていました。それらはPNPMですべて解決しました。
- 各依存性のバージョンは一度グローバルキャッシュに保存され、その後各プロジェクトにリンクされるため、ディスクスペースが大幅に節約されます。依存関係が以前からOSにインストールされている場合、ネットワーク経由で何も引き出されないため、インストールが速くなるのもこのためです。

> I use pnpm.
> Since downloading npm dependencies is very slow in my area, pnpm's local cache helps me save a lot of time

pnpmを使用します。
npmの依存関係をダウンロードするのは私の地域では非常に遅いので、pnpmのローカルキャッシュは多くの時間を節約するのに役立ちます。

# ni派

> ni-er?
> github.com/antfu/ni
> I've so many projects with yarn, npm and pnpm that I always forget which project uses which. It is great to just run nr dev to turn on the dev server and start coding right away

nierはどうですか？
yarn、npm、pnpmを使うプロジェクトがたくさんあって、どのプロジェクトでどれを使っているかいつも忘れてしまいます。`nr dev`を実行するだけで開発サーバーが立ち上がり、すぐにコーディングを始められるのは素晴らしいです。

https://github.com/antfu/ni

# Deno派？

> I prefer the Jurassic era

ジュラ紀の時代が好きです。

![](https://storage.googleapis.com/zenn-user-upload/42b047477472-20220615.jpeg =250x)
*DENOLAND*

:::message
画像の出典はおそらくここです。
https://deno-ja.vercel.app/artwork
DenoでNPMパッケージを使う方法があるそうです。
https://zenn.dev/uki00a/articles/how-to-use-npm-packages-in-deno
:::

# 参考

主なパッケージマネージャーの特徴は以下の記事が詳しいです。

https://zenn.dev/hibikine/articles/27621a7f95e761

# 余談

pnpmって何と読むのが正しいんですかね。
ドキュメントに発音の記載が見つからず…。
ピーエヌピーエムと発音されている方はいました🤔

https://www.youtube.com/watch?v=hiTmX2dW84E&t=10s
