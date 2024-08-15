---
title: Aurora Serverless v1 から v2 への移行手順
tags:
  - AWS
  - RDS
  - Aurora
private: false
updated_at: '2024-08-15T15:45:52+09:00'
id: 69cf637bb57d35b5c2b8
organization_url_name: null
slide: false
ignorePublish: false
---

## はじめに

2024 年 12 月 31 日に Aurora Serverless v1 はサポートを終了します。ここでは Aurora Serverless v2 へアップグレードする手順を紹介します。

### Aurora Serverless v1 と v2 の違い

Aurora Serverless v1 と v2 は DB クラスターおよびインスタンスに大きな違いがあるので、事前に概念を確認してください。

- DB クラスター
  - v1 は Serverless
  - v2 は Provisioned
- DB インスタンス
  - v1 は インスタンスが存在しない
  - v2 は Serverless（Provisioned インスタンスと共存可）

以下記事の「Provisioned と Serverless の違い」が参考になります。

https://dev.classmethod.jp/articles/moving-from-aurora-serverless-v1-tov2/

### アップグレード前提

- Aurora Serverless v2 が対応するエンジンバージョン (Aurora PostgreSQL 13.12) を利用
- アップグレードにブルー/グリーンデプロイは利用しない

対応するエンジンバージョンおよびエンジンバージョンのアップグレード方法は以下を参照してください。

https://docs.aws.amazon.com/ja_jp/AmazonRDS/latest/AuroraUserGuide/Concepts.Aurora_Fea_Regions_DB-eng.Feature.ServerlessV2.html#Concepts.Aurora_Fea_Regions_DB-eng.Feature.ServerlessV2.apg

https://docs.aws.amazon.com/ja_jp/AmazonRDS/latest/AuroraUserGuide/aurora-serverless.modifying.html#aurora-serverless.modifying.upgrade

## アップグレードパス

アップグレード手順の概要です。公式ドキュメントも確認してください。

1. Aurora Serverless v1 DB クラスターをプロビジョニングされた DB クラスターに変換
1. Aurora Serverless v2 リーダー DB インスタンスを DB クラスターに追加
1. Aurora Serverless v2 DB インスタンスにフェイルオーバー
1. リーダー DB インスタンスの追加（オプション）
1. プロビジョニングされた DB インスタンスの削除（オプション）

https://docs.aws.amazon.com/ja_jp/AmazonRDS/latest/AuroraUserGuide/aurora-serverless-v2.upgrade.html#aurora-serverless-v2.upgrade-from-serverless-v1-procedure

## アップグレード手順

### Aurora Serverless v1 クラスターの用意

説明用に Severless v1 を用意します。AWS CLI を実行します。

:::message
2024 年 4 月現在、マネジメントコンソールから Aurora Serverless v1 を起動することはできません。
:::

:::message
アクセスキー発行はせず、CloudShell, Cloud9, IAM ロールアタッチ済みの EC2 での実行をおすすめします。
:::

```bash
aws rds create-db-cluster --db-cluster-identifier sample-cluster \
    --engine aurora-postgresql --engine-version 13.12 \
    --engine-mode serverless \
    --scaling-configuration MinCapacity=2,MaxCapacity=4,SecondsUntilAutoPause=1000,AutoPause=true \
    --master-username username master-user-password password \
    --db-subnet-group-name sample-db-subnet-group
```

![1.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3852183/f0d38580-8147-cfa1-bd51-3503e4d67f7b.png)

### プロビジョニングされた DB クラスターへ変換

AWS CLI を実行してクラスターを変換します。

:::message
この操作はマネジメントコンソールからでは行えません。
:::

:::message
Unknown options: エラーが出た場合、新しい AWS CLI に更新してください。
:::

```bash
aws rds modify-db-cluster \
    --db-cluster-identifier sample-cluster \
    --engine-mode provisioned \
    --allow-engine-mode-change \
    --db-cluster-instance-class db.t3.medium
```

![2.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3852183/94880375-b20d-ad8e-e99d-78d1b2786cad.png)

### Aurora Serverless v2 リーダー DB インスタンスの追加

マネジメントコンソールから Serverless v2 のインスタンスを追加します。

![3.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3852183/7504c5ea-5e58-b4e8-27f9-a397d83eacbe.png)
![4.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3852183/71a01c7e-7d2c-9dde-ebc5-7618582699c6.png)
![5.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3852183/d33fa222-0c78-6101-f716-f459637b9e42.png)

### Serverless v2 リーダー DB インスタンスへフェイルオーバー

マネジメントコンソールからフェイルオーバーを実行します。

![6.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3852183/9d78f1c4-fae5-9b2e-e0f1-25d25c2a2d17.png)
![7.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3852183/b4fde005-082b-4e6f-6711-554497187e79.png)

### リーダー DB インスタンスの追加

可用性確保のため、別の AZ にリーダーを追加します。

![8.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3852183/323c9f50-4266-e4be-e4f9-ed1f8626e25d.png)
![9.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3852183/f91b4ed8-245a-fc95-4ae6-9aab1e6e6943.png)
![10.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3852183/ad879e57-8f7e-541f-e0ce-378e503ddf07.png)

### プロビジョニングされた DB インスタンスの削除

不要になったプロビジョニングされた DB インスタンスを削除します。

![11.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3852183/ed553662-7494-2027-e2e0-e54065fafd21.png)
![12.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3852183/64660d4b-8bf2-e6cb-c527-abb4ed92c3c1.png)

## 移行手順まとめ

キーポイントは以下の通りです。

- Aurora Serverless v2 への移行は、複数のステップを踏む必要がある
- アップグレード作業はマネジメントコンソールでは完結せず、AWS CLI の利用が必要

### その他耳より情報

- Serverless v1 の DB クラスタースナップショットからは、マネジメントコンソールのみで Serverless v1 のリストアができることを確認
- 許容できるのであれば、スナップショットからプロビジョニングされたクラスターと Serverless v2 インスタンスとしてリストアする方法も取れる
