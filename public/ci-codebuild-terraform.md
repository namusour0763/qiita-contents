---
title: AWS CodeBuild と GitHub で実現する Terraform CI/CD 入門
tags:
  - GitHub
  - AWS
  - Terraform
  - CodeBuild
  - CICD
private: false
updated_at: '2024-08-06T22:51:33+09:00'
id: 286b43c3f7d6a56c86a3
organization_url_name: null
slide: false
ignorePublish: false
---

## 想定読者

AWS CodeBuild と GitHub による Terraform の CI/CD 実装例を紹介します。

- Terraform の CI/CD ワークフローを実装したい人
- AWS CodeBuild の具体的な使用例をみたい人

## 構成図

![architecture.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3852183/aae01d06-65f5-39ce-8d91-b24811e1f2c4.png)

CodeBuild のビルドプロジェクトを 2 つ作成します。
それぞれプルリクエスト作成/更新とマージに対応します。

:::note warn
`terraform apply` が同時に実行される可能性がある場合は tfstate の破損を防ぐために　DynamoDB による state lock を追加してください。
:::

<details>
<summary>details GitHub Actionsについて</summary>

GitHub Actions を利用すると CI/CD に関連するリソースが GitHub に閉じるため、よりシンプルに利用できます。GitHub Enterprise（いわゆるオンプレ版）の利用が必須の場合、GitHub Actions を使うにはセルフホストランナーを用意する必要があります。インフラを自前で用意することが大変な場合、この記事のように AWS アカウントで CodeBuild を利用するという選択肢が考えられます。GitHub Actions であっても CI/CD の基本概念は同じため、参考にしてください。
</details>

### tfnotify

Terraform の plan/apply 結果を GitHub や Slack に通知してくれるツールです。

https://github.com/mercari/tfnotify

## 事前準備

### リポジトリ

GitHub にて新規にリポジトリを作成します。
プライベートリポジトリで問題ありません。

![repo_create.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3852183/b834491c-66dc-acdf-c4e1-f74ec2336797.png)

### アクセストークン

GitHub にて 個人のアクセストークンを払い出します。
tfnotify が GitHub に通知をするために利用します。
`Settings > Developer Settings > Personal access tokens > Tokens (classic)` より作成できます。
権限は `repo:status` `public_repo` のみで大丈夫です。

![ghp_token.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3852183/65f2356c-9475-ffd9-e420-df9fac4eee86.png)

:::note warn
ウィンドウを閉じるとトークンは二度と確認できません。きちんとメモしておきましょう。
:::

### IAM

Terraform 実行に必要な権限を付与した IAM ロールを作成します。
本記事では IAM ポリシー `AmazonS3FullAccess` を利用します。
信頼関係では CodeBuild を許可しましょう。

![iam_role1.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3852183/1c916234-01b1-974b-27c3-4bcaef768e57.png)
![iam_role2.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3852183/14452a37-cad3-a07e-56e1-9360872dff0b.png)
![iam_role3.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3852183/8bc90e46-abfc-263b-3a20-409328d9bf3e.png)

### S3

Terraform のバックエンド用 S3 バケットを作成します。
デフォルト値から変える設定はありません。

![s3_tfstate.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3852183/a4869e20-b7a0-0e71-4346-1948f33ed20a.png)

## 構築

### GitHub

サンプルコードを参考に、CI/CD 用のコードをプッシュします。

```text:repository.txt
├── README.md
├── codebuild
│   ├── buildspec_apply.yml
│   ├── buildspec_plan.yml
│   └── tfnotify.yml
└─── environments
    ├── main.tf
    └── provider.tf
```

CodeBuild が plan 時に利用する `buildspec_plan.yml` は以下です。

```yml:buildspec_plan.yml
version: 0.2

env:
  variables:
    TFDIR: "environments"
    TFNCONF: "codebuild/tfnotify.yml"
    TITLE: "Terraform Plan"
    MSG: "Plan detail via tfnotify"

phases:
  install:
    commands:
      # terraform
      - curl -sL https://releases.hashicorp.com/terraform/1.6.2/terraform_1.6.2_linux_amd64.zip > terraform.zip
      - unzip terraform.zip
      - cp terraform /usr/local/bin
      # tfnotify
      - curl -sL https://github.com/mercari/tfnotify/releases/download/v0.8.0/tfnotify_linux_amd64.tar.gz > tfnotify.tar.gz
      - tar -zxvf tfnotify.tar.gz
      - cp tfnotify /usr/local/bin
  pre_build:
    commands:
      - terraform -chdir="${TFDIR}" init -no-color
  build:
    commands:
      - |
        PLAN=$(terraform -chdir="${TFDIR}" plan -no-color 2>&1)
        echo "${PLAN}" | tfnotify --config "${TFNCONF}" plan --title "${TITLE}" --message "${MSG}"
```

CodeBuild が apply 時に利用する `buildspec_apply.yml` は以下です。

```yml:buildspec_apply.yml
version: 0.2

env:
  variables:
    TFDIR: "environments"
    TFNCONF: "codebuild/tfnotify.yml"
    TITLE: "Terraform Apply"
    MSG: "Apply detail via tfnotify"

phases:
  install:
    commands:
      # terraform
      - curl -sL https://releases.hashicorp.com/terraform/1.6.2/terraform_1.6.2_linux_amd64.zip > terraform.zip
      - unzip terraform.zip
      - cp terraform /usr/local/bin
      # tfnotify
      - curl -sL https://github.com/mercari/tfnotify/releases/download/v0.8.0/tfnotify_linux_amd64.tar.gz > tfnotify.tar.gz
      - tar -zxvf tfnotify.tar.gz
      - cp tfnotify /usr/local/bin
  pre_build:
    commands:
      - terraform -chdir="${TFDIR}" init -no-color
  build:
    commands:
      - |
        APPLY=$(terraform -chdir="${TFDIR}" apply -auto-approve -no-color 2>&1)
        echo "${APPLY}" | tfnotify --config "${TFNCONF}" apply --title "${TITLE}" --message "${MSG}"
```

:::note info
CI/CD が頻繁に実行される場合は `terraform` `tfnotify` のインストールを毎回行わずに済むよう、カスタムイメージを利用しましょう。`docker build` を行い ECR にプッシュすることで CodeBuild から利用可能です。
:::

tfnotify 用の設定ファイルは以下です。
こちらは plan と apply を 1 ファイルにまとめられます。

```yml:tfnotify.yml
---
ci: codebuild
notifier:
  github:
    token: $GITHUB_TOKEN
    repository:
      owner: "teradatky"
      name: "ci-codebuild-terraform-20231026"
terraform:
  plan:
    template: |
      {{ .Title }} <sup>[CI link]( {{ .Link }} )</sup>
      {{ .Message }}
      {{if .Result}}
      <pre><code>{{ .Result }}
      </pre></code>
      {{end}}
      <details><summary>Details (Click me)</summary>

      <pre><code>{{ .Body }}
      </pre></code></details>
  apply:
    template: |
      {{ .Title }} <sup>[CI link]( {{ .Link }} )</sup>
      {{ .Message }}
      {{if .Result}}
      <pre><code>{{ .Result }}
      </pre></code>
      {{end}}
      <details><summary>Details (Click me)</summary>

      <pre><code>{{ .Body }}
      </pre></code></details>
```

### CodeBuild

ビルドプロジェクトを 2 つ作成します。
以下に plan 用ビルドプロジェクトを作成するスクリーンショットを載せています。
apply 用のリソースのスクリーンショットは省略しますが、同様に作成します。

:::note warn
デフォルト値から変更していない箇所は、スクリーンショットを撮っていません。
名前やログ出力先などもビルドプロジェクトに合わせて適宜変更してください。
:::

![codebuild1.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3852183/c149322d-defe-793e-86c9-40ea58a6c627.png)
![codebuild2.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3852183/d8a8a9dc-4352-2178-44cd-e9c94b50e3cd.png)

:::note info
OAuth 連携が初回の場合、このような画面に遷移します。

![oauth.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3852183/48a149d6-098c-45b7-af20-acbc162f5bf3.png)
:::

![codebuild3.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3852183/9125b8c3-08df-59dd-8324-653eec407756.png)

:::note warn
「プライマリソースのウェブフックイベント」が変わることに注意します。

- plan 用ビルドプロジェクト
  - `PULL_REQUEST_CREATED` `PULL_REQUEST_UPDATED` `PULL_REQUEST_REOPEND`
- apply 用ビルドプロジェクト
  - `PULL_REQUEST_MERGED`
:::

![codebuild4.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3852183/7242b03e-62d9-97ac-c101-6aa34a054c2f.png)
![codebuild5.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3852183/bfec3de3-00c4-f701-6ee7-b3f5ab57a199.png)

:::note warn
サービスロールの編集を許可する場合は、自動的に IAM ロールに必要なポリシーがアタッチされます。許可しない場合、事前に必要な権限を付与しておく必要があります。

https://dev.classmethod.jp/articles/codebuild-service-role-checkbox/
:::

![codebuild6.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3852183/fe02f912-048f-b4f2-d53d-d725129cbd81.png)

:::note alert
個人用の AWS アカウントでない場合、アクセストークンはプレーンテキストではなく Secrets Manager を利用してください。
:::

![codebuild7.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3852183/c56d106f-0e11-b4ee-9f89-41532f3fff97.png)
![codebuild8.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3852183/a3dab8ac-29c1-7428-148c-5b5b68aa939b.png)

:::note info
ストリーム名を空にすることで、ビルドごとに別のストリームにログを記録します。
:::

2 つのビルドプロジェクトが作成できました。

![codebuild9.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3852183/4f3e93bf-91f6-a969-24a9-2d39017a5d99.png)

## CI/CD ワークフロー

実際に Terraform コードを記載し CI/CD ワークフローを回します。
今回は S3 バケットを作成します。 `main.tf` に以下を記載してプッシュします。

```hcl:main.tf
resource "aws_s3_bucket" "my_bucket" {
  bucket = "ci-codebuild-terraform-20231026-my-bucket"
}
```

プルリクエストを作成しました。

![pr1.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3852183/022cc47e-981d-5b69-b73c-bb8731580e2b.png)
![pr1_file.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3852183/f2b65b6b-a29b-ffa9-124f-8392f16b36fd.png)

すると `tfnotify` により plan 結果が通知されます。
レビュアーはこの結果を確認することで、マージ可否を簡単に判断できるようになります。

![pr2.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3852183/0374c68c-09b5-abbb-ada5-1a97f789a2d7.png)

`terraform init` や `terraform plan` で失敗していないことが checks からも分かります。

![pr3.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3852183/af826123-b71e-8c55-262f-ba715b3c0e5a.png)

レビュアーから Approve されたらマージしましょう。

![pr4.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3852183/6019b6a7-9ef9-1fee-882c-2106adac5c80.png)

マージ後 `terraform apply` の結果が通知されます。

![pr5.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3852183/bdb5ff95-5fe2-79b3-8d4c-2ff031c0dced.png)

:::note warn
plan が通っても apply が失敗するケースはよくあるため、きちんと結果を確認しましょう。
:::

以上、CI/CD ワークフローの一例でした。

## まとめ

CodeBuild を使うことで、Terraform の CI/CD が実現できました。
プルリクエストをベースとすることで、作業ミスや認識齟齬をグッと減らすことができます。
また tfnotify は plan や apply 結果が プルリクエスト内で確認できるため、レビュー負荷が軽減されます。
CI/CD を未経験の方はぜひ試してみて欲しいです。
