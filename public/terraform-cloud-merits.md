---
title: HCP Terraform/Terraform Cloudのメリットをざっくりご紹介！
tags:
  - AWS
  - CICD
  - terarform
private: false
updated_at: '2024-08-18T16:19:43+09:00'
id: 7d357086a314eadf4ce5
organization_url_name: null
slide: false
ignorePublish: false
---

## はじめに

HCP Terraform が気になる・使ってみたいという人向けにメリットをざっくりご紹介します！

:::note
2024/4/22 に Terraform Cloud は HCP Terraform と名称変更されました。
:::

https://developer.hashicorp.com/terraform/cloud-docs

## HCP Terraform のメリット

### どこからでも実行可能

`terraform login` コマンドでログインすれば、どんな端末でもすぐに同じ環境を使えます。複数端末あっても楽ちんです。

![login.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3852183/57473fa5-4bf6-1b33-c595-0c5355a18e5a.png)

### 実行環境がクラウド

クラウドで実行されるので、サーバーの面倒を見る必要がありません。メンバー端末の環境差分なども関係なくなります。ステートファイルのロックや保管も気にしなくて良いですね。

![plan.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3852183/30135884-c07a-46be-eee0-2c5e5a2deddd.png)
![run.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3852183/be1fafb8-6041-4cf0-fd9c-dc06355ab6aa.png)

### アクセスキーが不要

各種クラウドサービスと OpenID Connect (OIDC) で連携でき、非常にセキュアです。アクセスキーの発行は不要です。~~アクセスキーを使う記事が多すぎるので滅ぼしたい。~~

![id_provider.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3852183/8aa489ab-8b6e-d594-f9f2-645ae256d61c.png)
![variables.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3852183/67b4136e-4bed-4d3a-e9b9-c206d157955d.png)

### CI 実装が簡単

画面をポチポチすればすぐ CI/CD が実装できます。CI の整備やメンテナンスも馬鹿にならないので助かります。

![ci_1.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3852183/4d28677e-1b03-47ca-949d-c781f41f1e1c.png)
![ci_2.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3852183/b52c6fd3-5f15-338b-f2a9-f9320ade512e.png)

### GitHub 連携がイケてる

プルリクエストの画面から Plan 結果や Apply 結果がいい感じに確認できます。レビュアーも大助かりです。

![pr.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3852183/17747862-fd75-7532-a0fe-c54ae2a5ad68.png)
![checks1.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3852183/7ca14cf6-3fa4-840f-b4a4-644023128415.png)
![checks2.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3852183/f2c037e5-b2aa-58ef-9b5f-7f7cde35f9fc.png)
![checks3.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3852183/89d6abbf-1295-de9b-764b-0ad056ed5491.png)

### ガバナンスを効かせやすい

権限を持った人による承認がないと Apply できない設定ができます。コードがマージされたら自動で Apply もできますが、ガバナンスを効かせたい・タイミングを調整したい場合に。

![apply1.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3852183/4e66f555-b14f-27e3-2a35-a40760dbb9ab.png)
![apply2.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3852183/00ea45ef-6a6b-fde5-ca29-8196be90866c.png)
![apply3.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3852183/51eb1486-d79e-26bc-348d-648ee00138f1.png)

## まとめ

メリットが多いので、ぜひ一度体験してみてください！
