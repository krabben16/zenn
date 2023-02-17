---
title: "TypeScriptで書いたCDKのコードをRomeでフォーマットする" # 記事のタイトル
emoji: "🚪" # アイキャッチとして使われる絵文字（1文字だけ）
type: "tech" # tech: 技術記事 / idea: アイデア記事
topics: ["rome", "cdk", "typescript"] # タグ。["markdown", "rust", "aws"]のように指定する
published: true # 公開設定（falseにすると下書き）
---

# 前提
TypeScriptで書いたCDKコードのフォーマッターを選定中です。
ESLintとPrettierはセットアップが面倒に感じるのでRomeを使ってみます。

https://rome.tools/

# セットアップ
## Romeの設定ファイルを作成
プロジェクトルートにrome.jsonを作成する。リンターとフォーマッターの設定が1ファイルで対応できます。

```json
{
    "linter": {
        "enabled": true,
        "rules": {
            "recommended": true
        }
    },
    "formatter": {
        "enabled": true,
        "formatWithErrors": false,
        "indentStyle": "space",
        "indentSize": 2,
        "lineWidth": 120,
        "ignore": []
    }
}
```

参考 https://docs.rome.tools/configuration/

## エディタの拡張機能にRomeを追加
2023/02/17現在 拡張機能はVSCodeのみ対応しています。

![](https://storage.googleapis.com/zenn-user-upload/294798af3cd6-20230217.png)

参考 https://docs.rome.tools/guides/getting-started/#editor-setup

## VSCodeの設定を変更
1. `cmd+,`で設定を開く
1. Default FormatterにRomeを設定する
1. Format On Saveを有効にする

![](https://storage.googleapis.com/zenn-user-upload/3ae5a484c1df-20230217.png)

![](https://storage.googleapis.com/zenn-user-upload/a1b5259045ea-20230217.png)

コードを修正して保存するタイミングで整形が実行されます。

## 感想
- Romeのエディタ拡張機能はVSCodeのみ対応しているので、他のエディタを使っている方は設定できません。ESlintやPrettierのエディタ拡張機能を使う、Romeのnpmパッケージをインストールしてpre-commitでフォーマットを実行する、など検討した方がいいかもしれません。
- Romeの主要なコントリビュータが開発から離脱したそうで将来性が少し不安です😓
    - 参考 https://zenn.dev/kyrice2525/articles/article_tech_009
