---
title: "EC2インスタンスにCloudWatchエージェントをインストールする" # 記事のタイトル
emoji: "🕵️" # アイキャッチとして使われる絵文字（1文字だけ）
type: "tech" # tech: 技術記事 / idea: アイデア記事
topics: ["aws", "ec2", "cloudwatch"] # タグ。["markdown", "rust", "aws"]のように指定する
published: true # 公開設定（falseにすると下書き）
---

# はじめに

EC2インスタンスに `amazon-cloudwatch-agent` をインストールして実行します。
このエージェントを使用することで以下のことを実行できるようになります。
- インスタンスの内部システムレベルのメトリクスの収集
- ログの収集

https://docs.aws.amazon.com/ja_jp/AmazonCloudWatch/latest/monitoring/Install-CloudWatch-Agent.html

# 検証環境

- Amazon Linux 2
  - amzn2-ami-kernel-5.10-hvm-2.0.20220406.1-x86_64
- amazon-cloudwatch-agent
  - 1.247350.0

# CloudWatchエージェントで使用するIAMロールの作成

CloudWatchエージェントがCloudWatchにメトリクスを書き込むために必要なアクセス許可をEC2に付与

1. AWSコンソールでロールを作成
  - 信頼されたエンティティタイプ
    - AWSのサービス
  - 一般的なユースケース
    - EC2
  - ポリシー
    - CloudWatchAgentServerPolicy
2. 作成したロールをEC2インスタンスに設定

https://docs.aws.amazon.com/ja_jp/AmazonCloudWatch/latest/monitoring/create-iam-roles-for-cloudwatch-agent-commandline.html

# CloudWatchエージェントのインストール

```
sudo yum install -y amazon-cloudwatch-agent
```

https://docs.aws.amazon.com/ja_jp/AmazonCloudWatch/latest/monitoring/download-cloudwatch-agent-commandline.html

# CloudWatchエージェント設定ファイルの作成

- 諸事情により[ウィザード](https://docs.aws.amazon.com/ja_jp/AmazonCloudWatch/latest/monitoring/create-cloudwatch-agent-configuration-file-wizard.html)を使わず手動で作成する。
- AWSが提供する[サンプル](https://docs.aws.amazon.com/ja_jp/AmazonCloudWatch/latest/monitoring/CloudWatch-Agent-Configuration-File-Details.html#CloudWatch-Agent-Configuration-File-Complete-Example)をもとに必要に応じて内容を変更する。

```
{
  "agent": {
    "metrics_collection_interval": 10,
    "logfile": "/opt/aws/amazon-cloudwatch-agent/logs/amazon-cloudwatch-agent.log" # エージェントのログファイル出力先
  },
  "metrics": {
    "namespace": "MyCustomNamespace",
    "metrics_collected": {
      "cpu": {
        "resources": [
          "*"
        ],
        "measurement": [
          {"name": "cpu_usage_idle", "rename": "CPU_USAGE_IDLE", "unit": "Percent"},
          {"name": "cpu_usage_nice", "unit": "Percent"},
          "cpu_usage_guest"
        ],
        "totalcpu": false,
        "metrics_collection_interval": 10,
        "append_dimensions": {
          "customized_dimension_key_1": "customized_dimension_value_1",
          "customized_dimension_key_2": "customized_dimension_value_2"
        }
      },
      "disk": {
        "resources": [
          "/",
          "/tmp"
        ],
        "measurement": [
          {"name": "free", "rename": "DISK_FREE", "unit": "Gigabytes"},
          "total",
          "used"
        ],
         "ignore_file_system_types": [
          "sysfs", "devtmpfs"
        ],
        "metrics_collection_interval": 60,
        "append_dimensions": {
          "customized_dimension_key_3": "customized_dimension_value_3",
          "customized_dimension_key_4": "customized_dimension_value_4"
        }
      },
      "diskio": {
        "resources": [
          "*"
        ],
        "measurement": [
          "reads",
          "writes",
          "read_time",
          "write_time",
          "io_time"
        ],
        "metrics_collection_interval": 60
      },
      "swap": {
        "measurement": [
          "swap_used",
          "swap_free",
          "swap_used_percent"
        ]
      },
      "mem": {
        "measurement": [
          "mem_used",
          "mem_cached",
          "mem_total"
        ],
        "metrics_collection_interval": 1
      },
      "net": {
        "resources": [
          "eth0"
        ],
        "measurement": [
          "bytes_sent",
          "bytes_recv",
          "drop_in",
          "drop_out"
        ]
      },
      "netstat": {
        "measurement": [
          "tcp_established",
          "tcp_syn_sent",
          "tcp_close"
        ],
        "metrics_collection_interval": 60
      },
      "processes": {
        "measurement": [
          "running",
          "sleeping",
          "dead"
        ]
      }
    },
    "append_dimensions": {
      "ImageId": "${aws:ImageId}",
      "InstanceId": "${aws:InstanceId}",
      "InstanceType": "${aws:InstanceType}",
      "AutoScalingGroupName": "${aws:AutoScalingGroupName}"
    },
    "aggregation_dimensions" : [["ImageId"], ["InstanceId", "InstanceType"], ["d1"],[]],
    "force_flush_interval" : 30
  },
  "logs": {
    "logs_collected": {
      "files": {
        "collect_list": [ # CloudWatch Logsに送信したいログファイルのリスト
          {
            "file_path": "/opt/aws/amazon-cloudwatch-agent/logs/amazon-cloudwatch-agent.log",
            "log_group_name": "amazon-cloudwatch-agent.log",
            "log_stream_name": "amazon-cloudwatch-agent.log",
            "timezone": "UTC"
          },
          {
            "file_path": "/opt/aws/amazon-cloudwatch-agent/logs/test.log",
            "log_group_name": "test.log",
            "log_stream_name": "test.log",
            "timezone": "Local"
          }
        ]
      }
    },
    "log_stream_name": "my_log_stream_name",
    "force_flush_interval" : 15
  }
}
```

トラブルシューティングを簡単にするため `/opt/aws/amazon-cloudwatch-agent/etc/amazon-cloudwatch-agent.json` に作成することが[推奨](https://docs.aws.amazon.com/ja_jp/AmazonCloudWatch/latest/monitoring/CloudWatch-Agent-Configuration-File-Details.html)されている。

> エージェント設定ファイルを手動で作成または編集する場合は、任意の名前を付けることができます。トラブルシューティングを簡単にするため、Linux サーバーでは、/opt/aws/amazon-cloudwatch-agent/etc/amazon-cloudwatch-agent.json、Windows Server を実行しているサーバーでは、$Env:ProgramData\Amazon\AmazonCloudWatchAgent\amazon-cloudwatch-agent.json という名前を付けることをお勧めします。

# CloudWatchエージェントの起動

```
sudo amazon-cloudwatch-agent-ctl -a fetch-config -m ec2 -s -c file:/opt/aws/amazon-cloudwatch-agent/etc/amazon-cloudwatch-agent.json
```

|オプション|説明|
|---|---|---|
|-a(action) `fetch-config`|apply config for agent, followed by -c or -o or both. Target config can be based on location (ssm parameter store name, file name), or 'default'.|
|-m(mode) `ec2`|indicate this is on ec2 host.|
|-s|optionally restart after configuring the agent configuration this parameter is used for 'fetch-config', 'append-config', 'remove-config' action only.|
|-c `file:<file-path>`|file path on the host.|

https://docs.aws.amazon.com/ja_jp/AmazonCloudWatch/latest/monitoring/install-CloudWatch-Agent-commandline-fleet.html

その他のオプションは `amazon-cloudwatch-agent-ctl --help` で確認できる。

https://docs.aws.amazon.com/ja_jp/AmazonCloudWatch/latest/monitoring/troubleshooting-CloudWatch-Agent.html#CloudWatch-Agent-options-help

# CloudWatchエージェントの動作確認

```
sudo amazon-cloudwatch-agent-ctl -m ec2 -a status
```

statusが`running`になっていればOK

https://docs.aws.amazon.com/ja_jp/AmazonCloudWatch/latest/monitoring/troubleshooting-CloudWatch-Agent.html#CloudWatch-Agent-troubleshooting-verify-running

動作していない場合は[トラブルシューティングページ](https://docs.aws.amazon.com/ja_jp/AmazonCloudWatch/latest/monitoring/troubleshooting-CloudWatch-Agent.html)を参照
