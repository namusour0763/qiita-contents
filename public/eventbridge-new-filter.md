---
title: EventBridge のフィルターにプレフィックス・サフィックス・以外・大文字小文字無視の条件が追加されました！
tags:
  - 'aws'
  - 'eventbridge'
private: false
updated_at: ''
id: null
organization_url_name: null
slide: false
ignorePublish: false
---

## 概要

Twitter で EventBridge のルールのフィルタリングがパワーアップしたとの情報をキャッチし、実際に調べてみました！

https://twitter.com/nickste/status/1753513030984536380

## ドキュメント確認

AWS の公式ドキュメントを確認すると `Prefix matching`, `Suffix matching`, `Anything-but matching`, `Equals-ignore-case matching` が追加されています！

https://docs.aws.amazon.com/eventbridge/latest/userguide/eb-create-pattern-operators.html

## 実際に確認

EventBridge で実際にイベントパターンを作って確認します。今回はサンプルイベントに `EC2 Instance Launch Unsccessful` イベントを選び、ASG 名を `anything-but` および `prefix` 条件でフィルタリングしてみます。

シチュエーションとしては基本的にはイベントを発火したいが、障害を模したリソース（プレフィックスが broken）では発火させないというケースを想定しています。

![sample-event.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3852183/edbb5af0-7215-a499-29a4-1bf26d634ac0.png)
![filtering-result1.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3852183/acdf8815-a509-9a6c-af9d-a65e73c93738.png)
![filtering-result2.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3852183/9e844516-99a3-1f2a-2a47-7fd96b5ffd90.png)

```json:event-pattern.json
{
  "source": ["aws.autoscaling"],
  "detail-type": ["EC2 Instance Launch Unsuccessful"],
  "detail": {
    "AutoScalingGroupName": [{
      "anything-but": {
        "prefix": "broken"
      }
    }]
  }
}
```

ASG 名のプレフィックスが broken のイベントのみ一致しない（除外できている）ため、新しいフィルター条件 `anything-but`, `prefix` が正しく機能していることが確認できました！

## まとめ

EventBridge 単体でフィルタリングできていなかった処理をシンプルにできる可能性があり、嬉しいアップデートだと思います！
