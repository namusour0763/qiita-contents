---
title: Infracost で Terraform コードからコストを試算してみる
tags:
  - AWS
  - Terraform
  - Infracost
private: false
updated_at: '2024-08-15T16:52:45+09:00'
id: 7e2f54f90128f718d00e
organization_url_name: null
slide: false
ignorePublish: false
---

## Infracost

Infracost は IaC コードからクラウドサービスの利用料を試算するツール/サービスです。執筆時点では Terraform のみの対応ですが、ロードマップでは Pulumi, CDK, Bicep に対応予定のようです。

https://www.infracost.io/

https://github.com/infracost/infracost

## 導入

VSCode および Infracost Cloud での利用法を記載します。

:::note info
CLI 利用や CI/CD ワークフローへの導入も可能です。公式サイトを確認ください。
:::

VSCode の拡張機能から `Infracost` を検索してインストールします。その後画面の指示に従いサインアップします。私は GitHub アカウントを利用しました。後述の Infracost Cloud でのコスト試算を行う場合は、連携する VCS を選択するまで実施してください。

<details><summary>セットアップ画面</summary>

![vscode_install.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3852183/c6ce6d5a-ff05-d5b9-1cb6-1f5ecfda1a67.png)
![login.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3852183/805dab31-d40c-c1ee-04f3-3eb3a6270a47.png)
![integrations.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3852183/83a9a781-f801-8dc5-e556-0e4c179a1300.png)
</details>

## VSCode でコスト試算

Terraform コードを含むフォルダを VSCode で開くと自動でコスト試算を表示してくれます。

![vscode1.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3852183/06bcff0e-65b5-ba63-62f6-f9574af965a8.png)
![vscode2.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3852183/2e437f6e-29b6-761f-11ea-f40c2cd51ace.png)

:::note warn
VSCode のワークスペース機能で複数のフォルダを開いていると、拡張機能が正しく動作しません。これはかなり残念ポイントです。

https://github.com/infracost/vscode-infracost/issues/1

![bug1.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3852183/710141f8-dfa8-c95b-b80e-2315af0337ae.png)
![bug2.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3852183/9fbe35b9-de1a-05d8-5b14-24ff15d26630.png)
:::

## Infracost Cloud でコスト試算

Infracost Cloud ではより詳しく確認できます。サインアップしてから 2 週間無料で利用できます。期限を過ぎると CLI を除く機能は有料のサブスクリプションが必要になるようです。

https://www.infracost.io/pricing/

ダッシュボードでは 5 項目について、コスト最適化設定の達成率を表示してくれます。

![dashboard.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3852183/1ad0cf6c-89bd-75b0-9636-96a9c9772023.png)

リポジトリを調査してみます。もちろんコスト試算はしてくれるのですが、なにやら FinOps policies と Tagging policies に違反するリソースがあることが分かりました。

![repo_overview.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3852183/2ce98c84-6362-4b27-00c1-c6efe4f24bcd.png)
![finops_policy.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3852183/ae0193a9-43e2-065e-a102-d59e7dda9290.png)
![tagging_policy.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3852183/1a3c90f1-3326-c0b9-d0eb-d61d34b6c965.png)
![project.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3852183/0dcb0f70-bf36-18e6-e40b-5861afedd8e5.png)

Infracost が用意するポリシーが気になるので、調べてみましょう。画面上部の Governance から FinOps policies をクリックします。指摘済みの CloudWatch Logs のログ保持期間以外にも様々なポリシーがあることが分かります。個別にポリシーの ON/OFF もできるようです。

![finops_policies1.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3852183/b941445d-3bb6-7bbf-3941-2f770c01682e.png)
![finops_policies2.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3852183/499fac10-d92b-0521-c104-babc5b9eae24.png)
![finops_policies3.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3852183/94fe59d5-12b4-1655-bccf-be5cc5679adb.png)
![finops_policies4.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3852183/6d7b51f9-83ee-906e-c42d-c5216e4fb1af.png)
![finops_policies5.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3852183/728ed92f-105a-7fae-ed85-13cb7901ab1e.png)

Tagging policies も確認します。デフォルト設定は以下の通りです。タグ付けの強制はクラウドサービス側で行うのが定石かと思いますが、Infracost 側から行うこともできそうです。

![tagging_policies1.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3852183/f3112153-5aef-ed22-369f-74b8226b235a.png)
![tagging_policies2.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3852183/3e13654f-cdc2-ca93-b3f5-2c6bcd440e83.png)

## まとめ

Infracost を利用すると、IaC 化さえしていればコスト試算が簡単に行えます。一方で VSCode の拡張機能はワークスペース利用時に正常動作しなかったり、一部計算の表示がおかしかったりすることが分かりました。改善に期待です。

Infracost Cloud はコスト試算に加え、コスト最適化のベストプラクティスを指摘してくれるのため、一度目を通す価値はあると思います。ただしあくまでコスト影響のあるパラメータに対する指摘であり、ワークロードに対する最適なアーキテクチャを教えてくれるわけではありません。これは業務の中で身につけていきたいです。
