---
title: ElasticSearch/Kibanaのトラブルシュート＆Tips
tags:
  - AWS
  - Elasticsearch
  - Kibana
  - OpenSearch
private: false
updated_at: '2024-09-01T16:27:15+09:00'
id: c7aa21686ff2a661a758
organization_url_name: null
slide: false
ignorePublish: false
---

## はじめに

ElasticSearch + Kibana（実際は AWS の OpenSearch Service + OpenSearch Dashboards）の利用時に遭遇したトラブルシュートや Tips を記録しておきます。

## トラブルシュート＆Tips

### Kibana の画面に接続できない

VPC アクセスの ElasticSearch/Kibana ではインターネットからアクセスできません。以下の対応が必要です。

- パブリックアクセスでドメインを作り直す
- VPC に Windows 踏み台を作る・VPN を張るなどして、VPC 内部からアクセスする

https://docs.aws.amazon.com/ja_jp/opensearch-service/latest/developerguide/vpc.html

### ElasticSearch に接続できない＆ログ転送できない

fluentbit を使う想定です。設定ファイルが正しいか、コメントを確認してください。

```fluent-bit.conf
[OUTPUT]
    Name               es
    Match              apache.access                                              # 対象があっているか
    Host               search-${domain}-${random}.ap-northeast-1.es.amazonaws.com # 接続先があっているか
    Port               443            # ポートが443になっているか 9200ではありません！
    tls                On             # 上記と合わせて必須！
    HTTP_User          ${user_name}   # ユーザー名があっているか 
    HTTP_Passwd        ${password}    # パスワードが合っているか
    Suppress_Type_Name On             # OpenSearch 2.0 以上では必須！
    Index              apache_logs
    Type               _doc
    Time_Key           @timestamp
```

fluent-bit の状態確認方法

```bash
# ステータス確認
systemctl status fluent-bit -l

# ログ確認
journalctl -u fluent-bit -e
```

VPC アクセス・IAM 認証・OpenSearch Serverless など、様々なパターンがあるので、詳細は以下を確認ください。

https://docs.fluentbit.io/manual/pipeline/outputs/elasticsearch

### Kibana でフィールドが認識されない・機能しない

下記エラーが出ていませんか？

- `Unable to filter for presence of meta fields`
- `No cached mapping for this fields. Refresh field list from the Management > Index Patterns page`

この場合は `DashBoards Management` > `Index patterns` > `${your_index}` から「🔃 (Refresh)」をクリックすると解決します。

![unable_to_filter.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3852183/19c5b156-f1cc-e08d-70dc-b4dd8cc14998.png)
![no_cached_mapping.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3852183/a20a2338-78cd-a9c8-73ff-896b9812b072.png)
![refresh.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3852183/94bd1094-1ade-23ac-2877-9c5c44c092a3.png)

### Kibana の Visualize で Aggregation にフィールドが出てこない

フィールドの型が Aggregatable でないことが原因です。

同じテキストであっても `keyword` と `text` は以下の通り違いがあります。

- keyword: 文字列全体をひとかたまりで扱う。Aggregation 可。
- text: 文字列をパースして扱う。Aggregation 不可。

![aggregation.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3852183/9d50e2cd-c003-305b-f36b-53075ca6d6ed.png)
![fields.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3852183/02ae3366-6fd1-6507-6afe-fa8209a45c29.png)
![index_mappings.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3852183/86876efc-879a-ee45-0885-93c7ef2fb8b2.png)

よって mappings を変更するしかありません。一度作成したインデックスのマッピングは変更できないため、新しいインデックスにマッピングを再作成し reindex するしかありません。またはデータの再投入でも構いません。

https://tech.legalforce.co.jp/entry/2021/12/21/190129

https://shin0higuchi.hatenablog.com/entry/2017/07/03/233833

### Kibana で部分一致検索や数値の比較がしたい

もしできないのであれば、前述と同じデータ型の問題です。ある文字列が `text` でなく `keyword` として扱われていたら部分一致検索はできませんし、数字が `integer` や `long` でなく `keyword` として扱われていたら大小比較できません。

![search_text.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3852183/b7c45360-86e5-b804-e068-b53c47488429.png)

これも mappings を修正して reindex かデータの再投入が必要です。

![mappings.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3852183/4df3eefb-85ad-d407-f23b-3cbc012bcc40.png)

### Visualize で割合を可視化したい

あるデータの割合やパーセンテージを可視化したい場合、`TSVB (Time Series Visual Builder)` の `Filter Ratio` を利用します。TSVB 以外の Visualize では Filter Ratio が使えません。

![tsvb.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3852183/6eb058f8-e1ee-3a29-b2b8-7776c346dd53.png)

公式ドキュメントがわかりやすいので、要チェックです。

https://www.elastic.co/jp/blog/how-to-display-data-as-a-percentage-in-kibana-visualizations

## 最後に

トラブルがひとつでも解決したら「いいね」をお願いします！
