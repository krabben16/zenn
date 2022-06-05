---
title: "CDK+TSのサンプルLambda-Cronを実行する" # 記事のタイトル
emoji: "🥾" # アイキャッチとして使われる絵文字（1文字だけ）
type: "tech" # tech: 技術記事 / idea: アイデア記事
topics: ["aws", "lambda", "eventbridge", "cron", "typescript", "jest"] # タグ。["markdown", "rust", "aws"]のように指定する
published: false # 公開設定（falseにすると下書き）
---

# はじめに
CDK+TypeScriptの勉強のため、Lambda-Cronと呼ばれるサンプルを動かしてみます。
2022/06/05時点のソースコードを実行します。

https://github.com/aws-samples/aws-cdk-examples/tree/master/typescript/lambda-cron

その他の言語や構成のサンプルは以下のリポジトリで確認できます。

https://github.com/aws-samples/aws-cdk-examples

# 検証環境
- node
  - 14.19.3
- npm
  - 6.14.17
- aws-cdk
  - 2.27.0

# AWS CDK Toolkit (aws-cdk) のインストール

```sh
npm install -g aws-cdk
```

https://docs.aws.amazon.com/cdk/v2/guide/cli.html

# 実行するCDKのコード

```ts:index.ts
import events = require('aws-cdk-lib/aws-events');
import targets = require('aws-cdk-lib/aws-events-targets');
import lambda = require('aws-cdk-lib/aws-lambda');
import cdk = require('aws-cdk-lib');

import fs = require('fs');

export class LambdaCronStack extends cdk.Stack {
  constructor(app: cdk.App, id: string) {
    super(app, id);

    // Lambdaの関数を作成
    const lambdaFn = new lambda.Function(this, 'Singleton', {
      code: new lambda.InlineCode(fs.readFileSync('lambda-handler.py', { encoding: 'utf-8' })),
      handler: 'index.main',
      timeout: cdk.Duration.seconds(300),
      runtime: lambda.Runtime.PYTHON_3_6,
    });

    // EventBridgeのルールを作成
    // Run 6:00 PM UTC every Monday through Friday
    // See https://docs.aws.amazon.com/lambda/latest/dg/tutorial-scheduled-events-schedule-expressions.html
    const rule = new events.Rule(this, 'Rule', {
      schedule: events.Schedule.expression('cron(0 18 ? * MON-FRI *)')
    });

    // EventBridgeのルールのターゲットにLambdaの関数を設定
    rule.addTarget(new targets.LambdaFunction(lambdaFn));
  }
}

const app = new cdk.App();
new LambdaCronStack(app, 'LambdaCronExample');
app.synth();
```

```py:lambda-handler.py
def main(event, context):
    print("I'm running!")
```

毎週月〜金の午後6時（UTC）に特定の文字列を出力します。

aws-cdk-libのドキュメント
https://docs.aws.amazon.com/cdk/api/v2/docs/aws-construct-library.html

# アプリケーションのビルド
READMEに記載された以下のコマンドを実行します。

```sh
$ cd ./aws-cdk-examples/typescript/lambda-cron
$ npm install
npm WARN deprecated sane@4.1.0: some dependency vulnerabilities fixed, support for node < 10 dropped, and newer ECMAScript syntax/features added
npm WARN deprecated resolve-url@0.2.1: https://github.com/lydell/resolve-url#deprecated
npm WARN deprecated urix@0.1.0: Please see https://github.com/lydell/urix#deprecated
npm notice created a lockfile as package-lock.json. You should commit this file.
added 513 packages from 353 contributors and audited 513 packages in 17.431s

27 packages are looking for funding
  run `npm fund` for details

found 0 vulnerabilities
```

```sh
$ npm run build
> lambda-cron@1.0.0 build /Users/xxxx/projects/aws-cdk-examples/typescript/lambda-cron
> tsc
```

`npm run build`を実行するとCDKのコード `index.ts` をコンパイルした `index.js` が出力されます。

# アプリケーションのテスト
Jestによるテストファイルが予め用意されています。
以下のコマンドを実行します。

```sh
$ npm run test

> lambda-cron@1.0.0 test /Users/xxxx/projects/aws-cdk-examples/typescript/lambda-cron
> jest --config=jest.config.js

 PASS  ./lambda-cron.test.js
  lambda tests
    ✓ specified resources created
    ✓ lambda function has correct properties (1 ms)
    ✓ lambda has correct iam permissions
    ✓ lambda not running in vpc
  events tests
    ✓ event has correct rule (1 ms)

Test Suites: 1 passed, 1 total
Tests:       5 passed, 5 total
Snapshots:   0 total
Time:        2.527 s
Ran all test suites.
```

正常に実行されました。

# CDKのデプロイ
以下のコマンドを実行します。

```sh
$ cdk deploy
```

CDKにより構築されるAWSリソースの一覧が表示されました。
問題なければ続行します。

```sh
✨  Synthesis time: 1.1s

current credentials could not be used to assume 'arn:aws:iam::xxxx:role/cdk-hnb659fds-lookup-role-xxxx-ap-northeast-1', but are for the right account. Proceeding anyway.
(To get rid of this warning, please upgrade to bootstrap version >= 8)
current credentials could not be used to assume 'arn:aws:iam::xxxx:role/cdk-hnb659fds-deploy-role-xxxx-ap-northeast-1', but are for the right account. Proceeding anyway.
This deployment will make potentially sensitive changes according to your current security approval level (--require-approval broadening).
Please confirm you intend to make the following modifications:

IAM Statement Changes
┌───┬────────────────────┬────────┬────────────────────┬────────────────────┬─────────────────────┐
│   │ Resource           │ Effect │ Action             │ Principal          │ Condition           │
├───┼────────────────────┼────────┼────────────────────┼────────────────────┼─────────────────────┤
│ + │ ${Singleton.Arn}   │ Allow  │ lambda:InvokeFunct │ Service:events.ama │ "ArnLike": {        │
│   │                    │        │ ion                │ zonaws.com         │   "AWS:SourceArn":  │
│   │                    │        │                    │                    │ "${Rule.Arn}"       │
│   │                    │        │                    │                    │ }                   │
├───┼────────────────────┼────────┼────────────────────┼────────────────────┼─────────────────────┤
│ + │ ${Singleton/Servic │ Allow  │ sts:AssumeRole     │ Service:lambda.ama │                     │
│   │ eRole.Arn}         │        │                    │ zonaws.com         │                     │
└───┴────────────────────┴────────┴────────────────────┴────────────────────┴─────────────────────┘
IAM Policy Changes
┌───┬──────────────────────────┬──────────────────────────────────────────────────────────────────┐
│   │ Resource                 │ Managed Policy ARN                                               │
├───┼──────────────────────────┼──────────────────────────────────────────────────────────────────┤
│ + │ ${Singleton/ServiceRole} │ arn:${AWS::Partition}:iam::aws:policy/service-role/AWSLambdaBasi │
│   │                          │ cExecutionRole                                                   │
└───┴──────────────────────────┴──────────────────────────────────────────────────────────────────┘
(NOTE: There may be security-related changes not in this list. See https://github.com/aws/aws-cdk/issues/1299)

Do you wish to deploy these changes (y/n)? y
```

ところがデプロイ時にエラーが発生しました。

```sh
LambdaCronExample: deploying...
current credentials could not be used to assume 'arn:aws:iam::xxxx:role/cdk-hnb659fds-deploy-role-xxxx-ap-northeast-1', but are for the right account. Proceeding anyway.

 ❌  LambdaCronExample failed: Error: LambdaCronExample: SSM parameter /cdk-bootstrap/hnb659fds/version not found. Has the environment been bootstrapped? Please run 'cdk bootstrap' (see https://docs.aws.amazon.com/cdk/latest/guide/bootstrapping.html)
    at CloudFormationDeployments.validateBootstrapStackVersion (/Users/xxxx/.anyenv/envs/nodenv/versions/14.19.3/lib/node_modules/cdk/node_modules/aws-cdk/lib/api/cloudformation-deployments.ts:482:13)
    at processTicksAndRejections (internal/process/task_queues.js:95:5)
    at CloudFormationDeployments.publishStackAssets (/Users/xxxx/.anyenv/envs/nodenv/versions/14.19.3/lib/node_modules/cdk/node_modules/aws-cdk/lib/api/cloudformation-deployments.ts:457:7)
    at CloudFormationDeployments.deployStack (/Users/xxxx/.anyenv/envs/nodenv/versions/14.19.3/lib/node_modules/cdk/node_modules/aws-cdk/lib/api/cloudformation-deployments.ts:339:7)
    at CdkToolkit.deploy (/Users/xxxx/.anyenv/envs/nodenv/versions/14.19.3/lib/node_modules/cdk/node_modules/aws-cdk/lib/cdk-toolkit.ts:209:24)
    at initCommandLine (/Users/xxxx/.anyenv/envs/nodenv/versions/14.19.3/lib/node_modules/cdk/node_modules/aws-cdk/lib/cli.ts:341:12)

LambdaCronExample: SSM parameter /cdk-bootstrap/hnb659fds/version not found. Has the environment been bootstrapped? Please run 'cdk bootstrap' (see https://docs.aws.amazon.com/cdk/latest/guide/bootstrapping.html)
```

エラーメッセージの最後で `cdk bootstrap` を実行するよう指示されており、ドキュメントを確認すると以下の記述がありました。
CDKを実行するために必要なAWSリソース（S3バケットやIAMロールなど）があり、それらのリソースをプロビジョニングする工程を"bootstrap"と呼んでいるそうです。

> Deploying AWS CDK apps into an AWS environment (a combination of an AWS account and region) may require that you provision resources the AWS CDK needs to perform the deployment. These resources include an Amazon S3 bucket for storing files and IAM roles that grant permissions needed to perform deployments. The process of provisioning these initial resources is called bootstrapping.

https://docs.aws.amazon.com/cdk/v2/guide/bootstrapping.html

# bootstrapの実行
以下のコマンドを実行します。

```sh
$ cdk bootstrap
 ⏳  Bootstrapping environment aws://xxxx/ap-northeast-1...
Trusted accounts for deployment: (none)
Trusted accounts for lookup: (none)
Using default execution policy of 'arn:aws:iam::aws:policy/AdministratorAccess'. Pass '--cloudformation-execution-policies' to customize.
CDKToolkit: creating CloudFormation changeset...
 ✅  Environment aws://xxxx/ap-northeast-1 bootstrapped.
```

CloudFormationのスタック `CDKToolkit` が作成されました。

# CDKのデプロイ（2回目）
以下のコマンドを実行します。

```sh
$ cdk deploy   

✨  Synthesis time: 0.78s

This deployment will make potentially sensitive changes according to your current security approval level (--require-approval broadening).
Please confirm you intend to make the following modifications:

IAM Statement Changes
┌───┬─────────────────────────────┬────────┬─────────────────────────────┬─────────────────────────────┬───────────────────────────────┐
│   │ Resource                    │ Effect │ Action                      │ Principal                   │ Condition                     │
├───┼─────────────────────────────┼────────┼─────────────────────────────┼─────────────────────────────┼───────────────────────────────┤
│ + │ ${Singleton.Arn}            │ Allow  │ lambda:InvokeFunction       │ Service:events.amazonaws.co │ "ArnLike": {                  │
│   │                             │        │                             │ m                           │   "AWS:SourceArn": "${Rule.Ar │
│   │                             │        │                             │                             │ n}"                           │
│   │                             │        │                             │                             │ }                             │
├───┼─────────────────────────────┼────────┼─────────────────────────────┼─────────────────────────────┼───────────────────────────────┤
│ + │ ${Singleton/ServiceRole.Arn │ Allow  │ sts:AssumeRole              │ Service:lambda.amazonaws.co │                               │
│   │ }                           │        │                             │ m                           │                               │
└───┴─────────────────────────────┴────────┴─────────────────────────────┴─────────────────────────────┴───────────────────────────────┘
IAM Policy Changes
┌───┬──────────────────────────┬────────────────────────────────────────────────────────────────────────────────┐
│   │ Resource                 │ Managed Policy ARN                                                             │
├───┼──────────────────────────┼────────────────────────────────────────────────────────────────────────────────┤
│ + │ ${Singleton/ServiceRole} │ arn:${AWS::Partition}:iam::aws:policy/service-role/AWSLambdaBasicExecutionRole │
└───┴──────────────────────────┴────────────────────────────────────────────────────────────────────────────────┘
(NOTE: There may be security-related changes not in this list. See https://github.com/aws/aws-cdk/issues/1299)

Do you wish to deploy these changes (y/n)? y
LambdaCronExample: deploying...
[0%] start: Publishing xxxx:current_account-current_region
[100%] success: Published xxxx:current_account-current_region
LambdaCronExample: creating CloudFormation changeset...

 ✅  LambdaCronExample

✨  Deployment time: 131.35s

Stack ARN:
arn:aws:cloudformation:ap-northeast-1:xxxx:stack/LambdaCronExample/xxxx

✨  Total time: 132.13s
```

デプロイが成功しました🎉

# おわりに
CDK+TSのサンプルを動かしました。
CloudFormationのテンプレートはJSON/YAMLで実装する必要があり辛さを感じていましたが、CDKを導入することでテンプレートをプログラミング言語で実装できるようになりコードの抽象化・モジュール化が可能になります。
また、TSを使うことで入力補完や型チェックの恩恵を受けられるようになり開発速度の向上が見込めるのではないでしょうか。（TSに限った話ではないと思いますが）

最近TerraformからCloudFormationへの乗り換えを検討しており、Terraformの独自記法（HCL）ではなく使い慣れたプログラミング言語でリソースを構築できることにかなりのメリットを感じました。
CDKに興味が出てきたのでもっと深掘りしてみようと思います👀
