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

# エージェントで使用するIAMロールの作成

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

# エージェントのインストール

```
sudo yum install -y amazon-cloudwatch-agent
```

https://docs.aws.amazon.com/ja_jp/AmazonCloudWatch/latest/monitoring/download-cloudwatch-agent-commandline.html

# エージェント設定ファイルの作成

:::message
諸事情によりウィザードを使わず手動で作成する。
https://docs.aws.amazon.com/ja_jp/AmazonCloudWatch/latest/monitoring/create-cloudwatch-agent-configuration-file-wizard.html
:::

AWSが提供するサンプルをもとに必要に応じて内容を変更する。設定ファイルはトラブルシューティングを簡単にするため `/opt/aws/amazon-cloudwatch-agent/etc/amazon-cloudwatch-agent.json` のパスに作成することが推奨されている。

> エージェント設定ファイルを手動で作成または編集する場合は、任意の名前を付けることができます。トラブルシューティングを簡単にするため、Linux サーバーでは、/opt/aws/amazon-cloudwatch-agent/etc/amazon-cloudwatch-agent.json、Windows Server を実行しているサーバーでは、$Env:ProgramData\Amazon\AmazonCloudWatchAgent\amazon-cloudwatch-agent.json という名前を付けることをお勧めします。

https://docs.aws.amazon.com/ja_jp/AmazonCloudWatch/latest/monitoring/CloudWatch-Agent-Configuration-File-Details.html#CloudWatch-Agent-Configuration-File-Complete-Example

```json:amazon-cloudwatch-agent.json
{
  "agent": {
    # この設定ファイルで指定されたすべてのメトリクスが収集される頻度を指定
    "metrics_collection_interval": 10,
    # CloudWatch エージェントがログメッセージを書き込む場所を指定
    "logfile": "/opt/aws/amazon-cloudwatch-agent/logs/amazon-cloudwatch-agent.log"
  },
  "metrics": {
    # エージェントによって収集されるメトリクスに使用する名前空間
    "namespace": "MyCustomNamespace",
    # 収集するメトリクスを指定
    "metrics_collected": {
      "cpu": {
        "resources": [
          "*"
        ],
        "measurement": [
          # CPU がアイドル状態の時間の割合。
          {"name": "cpu_usage_idle", "rename": "CPU_USAGE_IDLE", "unit": "Percent"},
          # プロセスの優先度が低く、優先度の高いプロセスによって簡単に中断される場合がある、ユーザーモードになっている CPU の時間の割合。
          {"name": "cpu_usage_nice", "unit": "Percent"},
          # ゲストオペレーティングシステムで CPU が仮想 CPU を実行している時間の割合。
          "cpu_usage_guest"
        ],
        # すべての CPU コア間で集計された cpu メトリクスを報告するかどうかを指定
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
          # ディスクの空き容量。
          {"name": "free", "rename": "DISK_FREE", "unit": "Gigabytes"},
          "total",  # 使用済み容量と空き容量を含む、ディスクの合計容量。
          "used"    # ディスクの使用済み容量。
        ],
        # disk メトリクスを収集するときに除外するファイルシステムのタイプを指定
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
          "reads",      # ディスク読み取り操作の回数。
          "writes",     # ディスク書き込み操作の回数。
          "read_time",  # 読み取りリクエストがディスクで待機した時間の長さ。
          "write_time", # 書き込みリクエストがディスクで待機した時間の長さ。
          "io_time"     # ディスクが I/O リクエストをキューに入れている時間の長さ。
        ],
        "metrics_collection_interval": 60
      },
      "swap": {
        "measurement": [
          "swap_used",        # 現在使用中のスワップスペースの量。
          "swap_free",        # 使用されていないスワップスペースの量。
          "swap_used_percent" # 現在使用中のスワップスペースの割合。
        ]
      },
      "mem": {
        "measurement": [
          "mem_used",   # 現在使用中のメモリの量。
          "mem_cached", # ファイルキャッシュに使用されているメモリの量。
          "mem_total"   # メモリの合計量。
        ],
        "metrics_collection_interval": 1
      },
      "net": {
        "resources": [
          "eth0"
        ],
        "measurement": [
          "bytes_sent", # ネットワークインターフェイスで送信されたバイトの数。
          "bytes_recv", # ネットワークインターフェイスで受信されたバイトの数
          "drop_in",    # このネットワークインターフェイスで受信されたパケットのうち、削除されたものの数。
          "drop_out"    # このネットワークインターフェイスで送信されたパケットのうち、削除されたものの数。
        ]
      },
      "netstat": {
        "measurement": [
          "tcp_established",  # 確立された TCP 接続の数。
          "tcp_syn_sent",     # 接続リクエストを送信したあとに一致する接続リクエストを待機している TCP 接続の数。
          "tcp_close"         # 状態のない TCP 接続の数。
        ],
        "metrics_collection_interval": 60
      },
      "processes": {
        "measurement": [
          "running",  # 実行されているプロセスの数。
          "sleeping", # スリープ状態になっているプロセスの数。
          "dead"      # 「dead」となっているプロセスの数。
        ]
      }
    },
    # エージェントによって収集されたメトリクスに Amazon EC2 メトリクスのディメンションを追加
    "append_dimensions": {
      "ImageId": "${aws:ImageId}",                          # インスタンスの AMI ID を ImageID ディメンションの値として設定
      "InstanceId": "${aws:InstanceId}",                    # インスタンスのインスタンス ID を InstanceID ディメンションの値として設定
      "InstanceType": "${aws:InstanceType}",                # インスタンスのインスタンスタイプを InstanceType ディメンションの値として設定
      "AutoScalingGroupName": "${aws:AutoScalingGroupName}" # インスタンスの Auto Scaling グループ名を AutoScalingGroupName ディメンションの値として設定
    },
    # 収集されたメトリクスが集計されるディメンションを指定
    "aggregation_dimensions" : [["ImageId"], ["InstanceId", "InstanceType"], ["d1"],[]],
    # メトリクスがサーバーに送信されるまでにメモリバッファ内に残留する最大時間を秒単位で指定
    "force_flush_interval" : 30
  },
  "logs": {
    # サーバーから収集するログファイルを指定
    "logs_collected": {
      "files": {
        "collect_list": [
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
    # 個別のログストリーム名が collect_list パラメータに定義されていない場合、代わりに使用するデフォルトのログストリーム名を指定
    "log_stream_name": "my_log_stream_name",
    "force_flush_interval" : 15
  }
}
```

設定ファイルの構造やその他のフィールドについては以下を参照

https://docs.aws.amazon.com/ja_jp/AmazonCloudWatch/latest/monitoring/CloudWatch-Agent-Configuration-File-Details.html

CloudWatchエージェントで取得できるその他のメトリクスは以下を参照

https://docs.aws.amazon.com/ja_jp/AmazonCloudWatch/latest/monitoring/metrics-collected-by-CloudWatch-agent.html

# エージェントの起動

```
sudo amazon-cloudwatch-agent-ctl -a fetch-config -m ec2 -s -c file:/opt/aws/amazon-cloudwatch-agent/etc/amazon-cloudwatch-agent.json
```

|オプション|説明|
|---|---|
|-a(action) `fetch-config`|apply config for agent, followed by -c or -o or both. Target config can be based on location (ssm parameter store name, file name), or 'default'.|
|-m(mode) `ec2`|indicate this is on ec2 host.|
|-s|optionally restart after configuring the agent configuration this parameter is used for 'fetch-config', 'append-config', 'remove-config' action only.|
|-c `file:<file-path>`|file path on the host.|

https://docs.aws.amazon.com/ja_jp/AmazonCloudWatch/latest/monitoring/install-CloudWatch-Agent-commandline-fleet.html

その他のオプションは `amazon-cloudwatch-agent-ctl --help` で確認できる。

https://docs.aws.amazon.com/ja_jp/AmazonCloudWatch/latest/monitoring/troubleshooting-CloudWatch-Agent.html#CloudWatch-Agent-options-help

# エージェントの起動確認

```
sudo amazon-cloudwatch-agent-ctl -m ec2 -a status
```

statusが`running`になっていればOK。[^1]動作していない場合はトラブルシューティングを参照

[^1]: https://docs.aws.amazon.com/ja_jp/AmazonCloudWatch/latest/monitoring/troubleshooting-CloudWatch-Agent.html#CloudWatch-Agent-troubleshooting-verify-running

https://docs.aws.amazon.com/ja_jp/AmazonCloudWatch/latest/monitoring/troubleshooting-CloudWatch-Agent.html
