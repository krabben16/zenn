---
title: "AWS CLIを使ってSSMパラメータストアのパラメータを取得、作成する" # 記事のタイトル
emoji: "💻" # アイキャッチとして使われる絵文字（1文字だけ）
type: "tech" # tech: 技術記事 / idea: アイデア記事
topics: ["aws", "awscli", "ssm"] # タグ。["markdown", "rust", "aws"]のように指定する
published: true # 公開設定（falseにすると下書き）
---

# はじめに
AWSアカウントAのSystems Managerパラメータストアに作成したパラメータを別のアカウントBに複製したかったのですが、複数のAWSアカウント間でパラメータを複製または同期する方法が見つかりませんでした。仕方なくAWS CLIを使って「アカウントAからパラメータを取得」「アカウントBにパラメータを作成」を繰り返すことにしました😓

# 取得

```
aws ssm get-parameters-by-path --path "/"
```

パラメータ階層を指定してパラメータのリストを取得します。コマンドを実行するとJSONが返されるのでjqとかでよしなに加工します（省略）

:::message
パラメータタイプがSecureStringの場合には暗号化された値が返されます。復号化された値はAWSコンソールでしか確認できないと思います（多分）
:::

## 参考

https://dev.classmethod.jp/articles/aws-cli-all-ssm-parameter-get/

# 作成

```
aws ssm put-parameter \
    --name "parameter-name" \
    --description "parameter-description" \
    --value "parameter-value" \
    --type String
```

## 参考

https://docs.aws.amazon.com/ja_jp/systems-manager/latest/userguide/param-create-cli.html

https://docs.aws.amazon.com/cli/latest/reference/ssm/put-parameter.html

# 感想
めんどくさかったです。もっといい方法あるで！という方はぜひ教えてください🙏
