---
title: RDS のエンジンバージョンアップグレード時にはパラメータグループを確認しよう！
tags:
  - AWS
  - RDS
  - DB
private: false
updated_at: '2024-08-17T22:11:29+09:00'
id: ff746ac11688b7d94218
organization_url_name: null
slide: false
ignorePublish: false
---

## 概要

RDS を運用していると、エンジンバージョンのアップグレードが必要になります。単にエンジンバージョンを変更するだけでは問題が起きる可能性があるため、パラメータグループを確認しましょう。今回はその確認に便利なパラメータグループの比較機能を紹介します。

![overview.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3852183/0f21f98f-6433-cd6e-09f8-4c0d66d09ada.png)

### パラメータグループとは

パラメータグループとは、DB インスタンスやクラスターに適用されるエンジンの設定値の集合です。RDS の作成時に特に設定していなかった、という場合はデフォルトのパラメータグループが割り当てられているはずです。DB のパラメータは動作やパフォーマンスのチューニングに重要な要素です。

https://docs.aws.amazon.com/ja_jp/AmazonRDS/latest/UserGuide/parameter-groups-overview.html

### なぜパラメータグループを確認するのか

- アプリケーションの動作やパフォーマンスに影響を及ぼさないため
- パラメータグループ自体や静的パラメータの変更には DB の再起動が必要になるため

## パラメータグループ比較

パラメータグループには数百のパラメータがあり、手動でひとつずつ確認していくのは大変です。パラメータグループの比較を活用していきましょう。

### 既存のパラメータグループとデフォルトのパラメータグループを比較

まずは現在のパラメータグループがデフォルト値とどのような差分があるか確認しましょう。
デフォルトのパラメータグループは RDS での利用状況で存在有無が変わるため、まずデフォルト値のパラメータグループを作成します。

![create_parameter_group1.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3852183/11ea50dd-b536-6dea-945c-634da3e175b9.png)
![create_parameter_group2.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3852183/890520f1-c1f6-5ef0-a375-8cba7c497648.png)

作成できたらパラメータグループ一覧画面に戻ります。2 つのパラメータグループを選択し、アクション＞比較をクリックすることで、既存のパラメータグループに対する意図した設定変更を調べることができます。

:::note warn
DB インスタンスパラメータグループと DB クラスターパラメータグループは比較することができません
:::

![compare_parameter_group1.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3852183/1b01f492-2cea-77b4-ab0d-00ff6460ab09.png)
![compare_parameter_group2.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3852183/8cc46b58-ad89-552c-800f-9665a1672e35.png)

既存のパラメータグループに加えている変更の多くはアップグレード後も適用すべきだと考えられます。結果を確認して設定を検討してください。

### 旧バージョンと新バージョンのデフォルトのパラメータグループを比較

エンジンバージョンが異なると、設定値の変更、設定項目の追加・削除があります。これもパラメータグループの比較を使うことで簡単に調べることができます。
新バージョンのパラメータグループファミリーでデフォルト値のパラメータグループを作成してください。その後同じように旧バージョンのパラメータグループと比較を行うだけです。

![compare_parameter_group3.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3852183/35b0194c-9ad9-0dd4-02e6-1a1e086769f6.png)
![compare_parameter_group4.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3852183/9cecfd97-19bb-e7ee-1b2b-698d657e6446.png)

こちらも内容を吟味し、新しいパラメータグループに設定する内容を検討してください。

## DB エンジンバージョンのアップグレード

パラメータグループの検討ができたら DB エンジンバージョンのアップグレードに着手しましょう。一般的にはまず開発環境に適用し、以下の点を確認するとよいでしょう。

- エンジンアップグレードの実施手順
- アップグレード作業時間・サービスダウンタイム
- アップグレード後のアプリケーション動作とパフォーマンス
- 万が一に備えた切り戻し方法・連絡・体制・指揮系統

## まとめ

RDS の運用を任されたが DB の設計や構築はしていないという方に、パラメータグループの存在とパラメータグループの比較によるアップグレード時の調査方法が伝われば幸いです！
