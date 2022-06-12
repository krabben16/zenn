# なにする
gitの設定の構成レベルは以下に分かれています。

- `--local`: リポジトリ単位
- `--global`: ユーザー単位
- `-system`: システム単位

https://www.atlassian.com/ja/git/tutorials/setting-up-a-repository/git-config

「組織Aのリポジトリ」と「組織Bのリポジトリ」でconfigに異なる値を設定したいと思ったことはありませんか？（例えばuser.nameやuser.emailなど）まず思いつくやり方はリポジトリ単位（`--local`）で設定する方法です。ただしこの方法はリポジトリを作成するたびに設定が必要なのでリポジトリの数が多いと大変です。かといってユーザー単位（`--global`）で設定すると両方の組織で共通の値が適用されてしまいます。

これらの課題を解決するためディレクトリ単位でconfigを設定できるようにします。

# 対応方法
globalの.gitconfigを編集します。特定のディレクトリで指定した.gitconfigを読み込みます。この例では`~/a/`以下のリポジトリで`~/.gitconfig-a`を読み込むという意味です。

```~/.gitconfig
[user]
    name = default
[includeIf "gitdir:~/organization-a/"]
  	path = ~/.gitconfig-organization-a
```

`~/.gitconfig-organization-a`を作成します。

```~/.gitconfig-organization-a
[user]
    name = user-a
```

# 動作確認
特定のディレクトリに移動してuser.nameの値を確認してみます。`~/.gitconfig-organization-a`に設定したディレクトリ単位の値が適用されています。

```sh
$ mkdir -p ~/organization-a/repo && cd ~/organization-a/repo && git init
$ git config user.name
user-a
```

次に別のディレクトリに移動してuser.nameの値を確認してみます。`~/.gitconfig`に設定したユーザー単位の値が適用されています。

```sh
$ mkdir -p ~/organization-b/repo && cd ~/organization-b/repo && git init
$ git config user.name
default
```

ディレクトリ単位で異なる値を設定できるようになりました🎉
