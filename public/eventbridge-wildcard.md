---
title: EventBridge のルールにワイルドカードがサポートされました！
tags:
  - AWS
  - Cloudtrail
  - AmazonSNS
  - EventBridge
private: false
updated_at: '2024-08-17T21:51:56+09:00'
id: ca5a1e49a33b9c42e509
organization_url_name: null
slide: false
ignorePublish: false
---

## 概要

EventBridge がより便利になるアップデートが行われました！
ルールの中でワイルドカード `*` が使用できるようになり、より柔軟な条件を簡単に表現することが可能になりました。

## 公式ページ

https://aws.amazon.com/jp/about-aws/whats-new/2023/10/amazon-eventbridge-wildcard-filters-rules/

https://aws.amazon.com/jp/blogs/news/aws-weekly-20231002/

## 検証内容

IAM で何かしらのリソースが作成されたら、監査人へ通知する仕組みを作成します。
CloudTrail より `iam:create*` に該当する API コールを EventBridge で拾い、SNS 経由でメール通知します。

![architecture.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3852183/ceb33d00-4cf5-2dd1-a86f-d1eb8ae0230b.png)

:::note
セキュリティ向上を目的とする場合、このような自前の作り込みは避けて SecurityHub や GuardDuty を利用しましょう。
:::

## 構築

:::note warn
IAM の API コールは `us-east-1` リージョンに記録されます。そのため検証リソースはすべてバージニア北部に作成する必要があります。
:::

### SNS

まずは SNS トピックを作成します。タイプはスタンダードを選びます。その他の設定はデフォルトで問題ありません。
次にサブスクリプションの作成をします。プロトコルにEMAILを選び、メールアドレスを設定します。
確認メールが届くので、リンクをクリックします。ステータスが確認済みになればOKです。

![sns.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3852183/3a9622e3-807e-b52d-725b-f7207166bad9.png)

### EventBridge

次に EventBridge ルールを作成します。イベントバスは `default` でOKです。

![eventbridge1.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3852183/07cbd97f-5b15-66c9-9775-8db2ff9375fb.png)

イベントソースは AWS イベントまたは EventBridge パートナーイベントを選択します。

![eventbridge2.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3852183/e3321df0-affb-4f5b-937b-47adcc5891c5.png)

イベントパターンは以下のように設定します。
ここが今回の肝で `"eventName": [{"wildcard": "Create*"}]` と指定しています。

```json
{
  "source": ["aws.iam"],
  "detail-type": ["AWS API Call via CloudTrail"],
  "detail": {
    "eventSource": ["iam.amazonaws.com"],
    "eventName": [{
      "wildcard": "Create*"
    }]
  }
}
```

![eventbridge3.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3852183/b4f962a6-d61d-963e-8084-a3717508139b.png)

:::note
イベントタイプの仕様１から直接記入すると、記号が自動エスケープされて意図した記載ができません。「パターンを編集」から直接 `json` を記入します。
:::

:::note warn
単に `"eventName": ["Create*"]` と書いた場合、意図したワイルドカード動作になりません。詳しい記法は以下を参考にしてください。2023/10/29時点では英語ページにのみワイルドカードの記載があります。

https://docs.aws.amazon.com/eventbridge/latest/userguide/eb-create-pattern-operators.html
:::

ターゲットは作成済みの SNS トピックを選択します。

![eventbridge4.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3852183/8ba31248-fe04-ef0e-9149-c793ee571339.png)

以上で構築は完了です！

## 動作確認

適当な IAM ロールを作成します。信頼されたエンティティに EC2 を選択し、ポリシーのアタッチはしません。

![test1.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3852183/e547dbda-9045-a356-ab28-b7dcf0cb0437.png)

CloudTrail のイベント履歴を確認すると `CreateRole` と `CreateInstanceProfile` が呼び出されていることが分かります。
EC2 を信頼関係に選んだため、IntanceProfile も自動的に作成されます。

![test2.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3852183/dd441f5a-6578-e9ed-12f3-1c56662bc784.png)

メールを確認すると `Create` から始まる API の呼び出しについて 2 件の通知が届いています。成功です！

![test3.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3852183/6ca2f9ba-5c14-e447-e1c6-e39f9f07084c.png)

## まとめ

EventBridge のルールでワイルドカードが使えるようになり、より簡単にイベント駆動型システムを組めるようになりそうです。
一方 IAM ポリシーの感覚で `*` をつけるだけでは期待した動作とならないので注意が必要です。
