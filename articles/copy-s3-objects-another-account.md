---
title: "AWS S3のオブジェクトを異なるAWSアカウントにコピーする" # 記事のタイトル
emoji: "🪣" # アイキャッチとして使われる絵文字（1文字だけ）
type: "tech" # tech: 技術記事 / idea: アイデア記事
topics: ["aws", "s3"] # タグ。["markdown", "rust", "aws"]のように指定する
published: true # 公開設定（falseにすると下書き）
---

# 何したん
AWSのアカウントAに存在するS3オブジェクトをアカウントBにコピーする作業が発生しました。オブジェクトのコピーはAWS CLIの`aws s3 sync`を実行しましたが、その前にいろいろ対応が必要な作業がありましたので共有します。

# 実行環境
- aws-cli@2.4.18

# 対応方法
以下、A=AWSアカウントA、B=AWSアカウントB

## A コピー作業用のIAMユーザーを作成
ユーザーの作成はAWSコンソール/AWS CLIなどよしなに対応してください。（例. `s3-objects-copy-user`）
次にポリシーを作成します。コピー元バケットのオブジェクトを一覧表示したり、コピー先のバケットにオブジェクトをコピーしたりするのに必要なアクセス許可を設定します。最後に作成したポリシーをIAMユーザーにアタッチします。（例. `s3-objects-copy-policy`）

```json:s3-objects-copy-policy
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "s3:ListBucket",
                "s3:GetObject"
            ],
            "Resource": [
                "arn:aws:s3:::<コピー元のバケット名>",
                "arn:aws:s3:::<コピー元のバケット名>/*"
            ]
        },
        {
            "Effect": "Allow",
            "Action": [
                "s3:ListBucket",
                "s3:PutObject",
                "s3:PutObjectAcl"
            ],
            "Resource": [
                "arn:aws:s3:::<コピー先のバケット名>",
                "arn:aws:s3:::<コピー先のバケット名>/*"
            ]
        }
    ]
}
```

![](https://storage.googleapis.com/zenn-user-upload/0fdd8971c08a-20220613.png)

![](https://storage.googleapis.com/zenn-user-upload/628664786a5b-20220613.png)

## B バックアップ用のS3バケットを作成
バケットの作成はAWSコンソール/AWS CLIなどよしなに対応してください。（例. `s3-bucket-to`）
次にバックアップ用バケットのバケットポリシーを編集します。AのIAMユーザーがコピー先のバケットにオブジェクトをコピーしたり、オブジェクトを一覧表示したりするのに最低限必要なアクセス許可を設定します。

```json:s3-bucket-toのバケットポリシー
{
    "Version": "2012-10-17",
    "Id": "Policy1611277539797",
    "Statement": [
        {
            "Sid": "Stmt1611277535086",
            "Effect": "Allow",
            "Principal": {
                "AWS": "<Aで作成したIAMユーザーのARN>"
            },
            "Action": "s3:PutObject",
            "Resource": "arn:aws:s3:::<コピー先のバケット名>/*",
            "Condition": {
                "StringEquals": {
                    "s3:x-amz-acl": "bucket-owner-full-control"
                }
            }
        },
        {
            "Sid": "Stmt1611277877767",
            "Effect": "Allow",
            "Principal": {
                "AWS": "<Aで作成したIAMユーザーのARN>"
            },
            "Action": "s3:ListBucket",
            "Resource": "arn:aws:s3:::<コピー先のバケット名>"
        }
    ]
}
```

## AWS CLIでコマンドを実行
以下のコマンドを実行すると「コピー元バケットのディレクトリ以下のオブジェクト」が「コピー先バケットのディレクトリ」にコピーされます。コピー完了まで待機しましょう🛌

```sh
aws s3 sync s3://<コピー元のバケット名>/<ディレクトリ> s3://<コピー先のバケット名>/<ディレクトリ>
```

# 参考

https://aws.amazon.com/jp/premiumsupport/knowledge-center/copy-s3-objects-account/
