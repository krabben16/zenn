---
title: "web.dev/measureとPageSpeed Insightsの違いとは何か？" # 記事のタイトル
emoji: "📏" # アイキャッチとして使われる絵文字（1文字だけ）
type: "idea" # tech: 技術記事 / idea: アイデア記事
topics: ["webperf", "corewebvitals", "lighthouse", "webdev", "pagespeedinsights"] # タグ。["markdown", "rust", "aws"]のように指定する
published: true # 公開設定（falseにすると下書き）
---

# はじめに
web.devのこれ（web.dev/measureと呼ばれているらしい）とPageSpeed Insights（PSI）は何が違うの、というテーマです。どちらもLighthouseを使用してCore Web Vitalsを測定するためのツールという認識ですが果たして🤔

![](https://storage.googleapis.com/zenn-user-upload/72ea59efad2f-20220615.png)

# Core Web Vitalsを測定するツールのカテゴリ
予備知識として、Googleが提供するCore Web Vitalsを測定するツールには2つのカテゴリがあります。

1. ラボデータを報告するツール
   - 例. LighthouseやWebPageTestなど
2. フィールドデータを報告するツール
   - 例. CrUX

https://web.dev/lab-and-field-data-differences/

https://web.dev/speed-tools/

## ラボデータとは
- あらかじめ定義されたネットワークおよびデバイスの条件を持つ**制御された環境でウェブページを読み込む**ことによって決定されます。これらの条件はラボ環境として知られており合成環境とも呼ばれることがあります。
- ラボデータを報告するChromeツールは一般にLighthouseを実行しています。
- ラボテストの目的はできる限り多くの要因を制御して実行ごとに（可能な限り）**一貫性/再現性のある結果を得ること**です。

## フィールドデータとは
- あるページにアクセスしたすべてのユーザーを監視し、それらのユーザーの個々の体験について所定のパフォーマンス指標群を測定することによって決定されます。フィールドデータは実際のユーザーの訪問に基づいているため、**ユーザーの実際のデバイス、ネットワーク条件、地理的な位置が反映されます。**
- 一般にリアルユーザーモニタリング（RUM）データとも呼ばれます。
- フィールドデータを報告するChromeツールは**Chromeユーザーエクスペリエンスレポート（CrUX）からデータを取得します。**
  - CrUXレポートは28日間にわたる実際のChromeユーザーからのパフォーマンスメトリクスの分布を示します。
- フィールドデータについて理解すべき最も重要なことは、それは単なる1つの数値ではなく数値の分布であるということです。 **サイトを訪れる人には読み込みが非常に速い人もいれば読み込みが非常に遅い人もいます。** フィールドデータはユーザーから収集したすべてのパフォーマンスデータの完全なセットです。

## ラボとフィールドの使い分け
- あるページについてフィールドデータとラボデータの両方がある場合、取り組みの優先順位付けに使用すべきはフィールドデータです。フィールドデータは実際のユーザーが体験していることを表しているため、**ユーザーが何に困っているのか、何を改善すべきなのかを理解する最も正確な方法です。**
- フィールドデータでは実際のユーザー体験を把握することができますが、サイトを正常に読み込むことができたユーザーに対してのみ可能です。ラボデータは**低速のネットワークや下位デバイスのユーザーからのアクセスを向上させる機会を特定するために役立つ場合があります。**
- ラボデータとフィールドデータの両方が効果的なパフォーマンス測定の重要な要素です。どちらにも長所と限界があり、どちらか一方だけを使用している場合はユーザー体験を向上させる機会を逃している可能性があります。

# web.dev/measureが報告するデータの種類
ではweb.dev/measureが報告するデータはラボ or フィールドのどちらでしょうか。

答えは**ラボデータ**でした。web.dev/measureのページ下部FAQの`How does web.dev/measure differ from PageSpeed Insights?`というセクションで回答されていました。ちなみにPSIが報告するデータの種類はラボ+フィールドとのことです。

![](https://storage.googleapis.com/zenn-user-upload/e79caf87d400-20220615.png)

上記の回答のまとめです。

- web.dev/measureは **ラボ環境でウェブページの全体的な品質を測定する。** パフォーマンス、アクセシビリティ、SEOなどLighthouseの複数の品質カテゴリに関するインサイトを提供する。
- PSIはパフォーマンスに焦点を当て**ラボ（Lighthouse）とフィールド（CrUX）の両方のデータを使用して**パフォーマンスの概要を提供する。
- PSIはフィールドでオリジンレベルの洞察を提供する。web.dev/measureはラボで1つのURLを深く掘り下げることを目的とする。
  - 「オリジンレベルの洞察」とは？と思って調べましたが明確な答えを見つけられませんでした。以下の記事で言及されているOrigin Summary（サイト全体の評価）のことかなーと予想しています。

https://zenn.dev/skillshare/articles/61450a88fc7b00981117#origin-summary%E3%81%A8%E3%81%AF%E4%BD%95%E3%81%8B%EF%BC%9F

# それぞれのツールの測定結果
実際にweb.dev/measureとPSIの測定結果を見てみます。

## web.dev/measure
以下の項目が計算されています。

- Performance
- Accessibility
- Best Practicies
- SEO

![](https://gyazo.com/5e85bb1a1e6cc5cbff2bceb29bcf21bb.png)

https://web.dev/measure/

## PSI
以下の項目が計算されています。

- Performance

![](https://gyazo.com/787427bb8b59297c7f3204bf91df81b6.png)

https://pagespeed.web.dev/

# おわりに
web.dev/measureとPSIの違いは以下でした。

|ツール名|報告するデータの種類|対象の品質カテゴリ|
|---|---|---|
|web.dev/measure|ラボデータ|パフォーマンス、アクセシビリティ、SEOなど|
|PSI|ラボデータ+フィールドデータ|パフォーマンスのみ|

どちらもCore Web Vitalsを計測するツールというのは言葉足らずで、それに加えてウェブページの品質向上のための改善項目を提供してくれるツールでした。あと個人的にはラボデータとフィールドデータの使い分けを知れて良かったです。「このデータから得た課題を解決したらこういう属性のユーザーが良い影響を受ける」みたいなデータ基準の考え方が自分の中に芽生えました😺

# 余談
Core Web Vitalsの測定でラボデータとフィールドデータの間に差異が生じる一般的な理由については以下のページで語られています。

https://web.dev/lab-and-field-data-differences/#why-lab-and-field-data-are-different
