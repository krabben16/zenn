---
title: "AWS S3のオブジェクトを異なるAWSアカウントにコピーする" # 記事のタイトル
emoji: "🪣" # アイキャッチとして使われる絵文字（1文字だけ）
type: "tech" # tech: 技術記事 / idea: アイデア記事
topics: ["aws", "s3"] # タグ。["markdown", "rust", "aws"]のように指定する
published: false # 公開設定（falseにすると下書き）
---

# 何したん
AWSのアカウントAに存在するS3オブジェクトをアカウントBに退避する作業が発生しました。オブジェクトのコピーはAWS CLIの`aws s3 sync`を実行しましたが、その前にいろいろ対応が必要な作業がありましたので共有します。

# 実行環境
- aws-cli@2.4.18

# 対応方法
以下、A=AWSアカウントA、B=AWSアカウントB

## A IAMユーザーを作成
作成したIAMユーザーにカスタマー管理ポリシーを付与します。コピー元バケットのオブジェクトを一覧表示したり、コピー先のバケットにオブジェクトをコピーしたりするのに必要なアクセス許可を設定します。

```json
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

## B バックアップ用のS3バケットを作成
バケットの作成はAWSコンソール/AWS CLIなどよしなに対応してください。次にバックアップ用バケットのバケットポリシーを編集します。AのIAMユーザーがコピー先のバケットにオブジェクトをコピーしたり、オブジェクトを一覧表示したりするのに最低限必要なアクセス許可を設定します。

```json
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
以下のコマンドを実行します。コピー完了まで待機します。

```sh
aws s3 sync s3://<コピー元のバケット名>/<フォルダ> s3://<コピー先のバケット名>/<フォルダ>
```

# 参考

https://aws.amazon.com/jp/premiumsupport/knowledge-center/copy-s3-objects-account/
