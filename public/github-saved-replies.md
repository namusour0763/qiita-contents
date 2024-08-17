---
title: GitHub の Saved Replies でコードレビューに心理的安全性を！
tags:
  - 'git'
  - 'github'
  - 'コードレビュー'
private: false
updated_at: ''
id: null
organization_url_name: null
slide: false
ignorePublish: false
---

## はじめに

コードレビューをする際に、「個人の好みかなぁ」というコメントを付けるのをためらったことはありませんか？レビュイーに正しく意図を伝えて心理的安全性を高めたいと思い、Saved Replies を使ってコメント種別をつけることにしました。

## コメント種別

|略語|意味|説明|
|----|----|----|
|MUST|必須：Must|修正されないとApproveできません。修正のアクションが必要です。|
|IMO|提案：In My Opinion|個人的な見解や軽微な提案です。タスク化や修正等のアクションが必要です。|
|Q|質問：Question|質問です。回答のアクションが必要です。|
|NR|お手すきで：No Rush|不急なので今やるかはおまかせします。タスク化や修正等のアクションが必要です。|
|NITS|あらさがし：Nitpick|重箱の隅をつつく提案です。やってもいいがやらなくてもよいです。アクションは不要です。|
|FYI|参考までに：For Your Information|参考までに共有します。アクションは不要です。|

## Saved Replies

返信テンプレートともいいます。Saved Replies を活用することで、手間なくコメントをつけることができます。

### 設定方法

1. <https://github.com/settings/replies> へアクセス
1. タイトルとコメントを記載
1. Add saved reply をクリックし保存

![1.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3852183/e1b4589a-19da-6361-adaa-2fb278608009.png)

### 返信テンプレート例

```md
**[MUST: 必須]**[^1]

xxx

[^1]: 修正されないとApproveできません。修正のアクションが必要です。
```

```md
**[IMO: 提案]**[^1]

xxx

[^1]: 個人的な見解や軽微な提案です。タスク化や修正等のアクションが必要です。
```

```md
**[Q: 質問]**[^1]

xxx

[^1]: 質問です。回答のアクションが必要です。
```

```md
**[NR: お手すきで]**[^1]

xxx

[^1]: 不急なので今やるかはおまかせします。タスク化や修正等のアクションが必要です。
```

```md
**[NITS: あらさがし]**[^1]

xxx

[^1]: 重箱の隅をつつく提案です。やってもいいがやらなくてもよいです。アクションは不要です。
```

```md
**[FYI: 参考までに]**[^1]

xxx

[^1]: 参考までに共有します。アクションは不要です。
```

### 使用例

Saved replies をクリックすることで、返信が選択できます。タイトルで検索も可能です。

![2.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3852183/4a7bbc38-909e-493e-2104-760584c74848.png)
![3.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3852183/941320a3-8aed-813e-82fa-4774fa5beeae.png)

実際にコメントすると、以下のような感じになります。

![4.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3852183/4c314bfc-805f-c984-02a7-5ea2da1ee929.png)
![5.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3852183/55f14aa3-995d-5a0a-f396-b903ea78042d.png)
![6.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3852183/22c50015-91e5-5321-01a7-0fa5a610618f.png)
![7.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3852183/a8741f54-c358-2748-6074-c51459431301.png)
![8.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3852183/d9c5b7b1-c561-9cf5-6333-5f1559838c79.png)
![9.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3852183/3731da68-cd18-7dec-5363-28431fa0f783.png)

## まとめ

レビュアーとしては軽い提案のつもりでも、レビュイーには厳しい指摘コメントばかりに見えることがあります。Saved Replies を駆使して、意図を正確に伝えられると快適なコードレビューになると思います！
