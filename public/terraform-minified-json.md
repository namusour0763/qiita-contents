---
title: Terraform で Service Control Policy の文字数上限に引っかかった場合の対策
tags:
  - AWS
  - Terraform
private: false
updated_at: '2024-08-06T20:42:10+09:00'
id: bff75f83502f109b4c80
organization_url_name: null
slide: false
ignorePublish: false
---

## 課題

Service Control Policy (SCP) ドキュメントは、最大 5120 文字の制限があります。これを超えた場合には `terraform apply` に失敗します。

https://docs.aws.amazon.com/ja_jp/organizations/latest/userguide/orgs_reference_limits.html

## 先に結論

`data.aws_iam_policy_document` の `minified_json` を使うとよい！

https://registry.terraform.io/providers/hashicorp/aws/latest/docs/data-sources/iam_policy_document

## 具体例

以下の通り SCP を作成してみます。

```hcl:main.tf
provider "aws" {
  region = "ap-northeast-1"
}

data "aws_caller_identity" "current" {}

data "aws_iam_policy_document" "main" {
  # S3バケットとオブジェクトの削除を禁止
  statement {
    sid    = "DenyS3BucketAndObjectDeletion"
    effect = "Deny"

    actions = [
      "s3:DeleteBucket",
      "s3:DeleteObject",
      "s3:DeleteObjectVersion"
    ]

    resources = [
      "arn:aws:s3:::do-not-delete-this-bucket",
      "arn:aws:s3:::do-not-delete-this-bucket/*",
      "arn:aws:s3:::very-important-data-store",
      "arn:aws:s3:::very-important-data-store/*",
      "arn:aws:s3:::audit-for-confidential-access",
      "arn:aws:s3:::audit-for-confidential-access/*",
    ]
  }

  # IAMロールとポリシーの削除と編集を禁止
  statement {
    sid    = "DenyIAMRoleAndPolicyModification"
    effect = "Deny"

    actions = [
      "iam:DeleteRole",
      "iam:DeleteRolePolicy",
      "iam:DeletePolicy",
      "iam:DeletePolicyVersion",
      "iam:CreatePolicyVersion",
      "iam:SetDefaultPolicyVersion",
      "iam:UpdateRole",
      "iam:UpdateRoleDescription",
      "iam:PutRolePolicy"
    ]

    resources = [
      "arn:aws:iam::${data.aws_caller_identity.current.account_id}:role/do-not-delete-this-role",
      "arn:aws:iam::${data.aws_caller_identity.current.account_id}:policy/do-not-delete-this-policy",
      "arn:aws:iam::${data.aws_caller_identity.current.account_id}:role/very-important-role",
      "arn:aws:iam::${data.aws_caller_identity.current.account_id}:policy/very-important-policy",
      "arn:aws:iam::${data.aws_caller_identity.current.account_id}:role/audit-for-confidential-access-role",
      "arn:aws:iam::${data.aws_caller_identity.current.account_id}:policy/audit-for-confidential-access-policy",
    ]
  }

  # AWS Organizationsの変更を禁止
  statement {
    sid    = "DenyOrganizationsChanges"
    effect = "Deny"

    actions = [
      "organizations:LeaveOrganization",
      "organizations:DeleteOrganization",
      "organizations:RemoveAccountFromOrganization",
      "organizations:DeleteOrganizationalUnit",
      "organizations:DeletePolicy",
      "organizations:UpdatePolicy"
    ]

    resources = ["*"]
  }
}

resource "aws_organizations_policy" "main" {
  name    = "main"
  content = data.aws_iam_policy_document.main.minified_json # minified_json を指定しよう！
}

output "json_policy_length" {
  value = length(data.aws_iam_policy_document.main.json)
}

output "minified_json_policy_length" {
  value = length(data.aws_iam_policy_document.main.minified_json)
}
```

`terraform plan` すると以下の結果が得られます。

```text
namusour@DESKTOP:~/terraform_test$ terraform plan
data.aws_caller_identity.current: Reading...
data.aws_caller_identity.current: Read complete after 0s [id=xxxxxxxxxxx]
data.aws_iam_policy_document.main: Reading...
data.aws_iam_policy_document.main: Read complete after 0s [id=xxxxxxxxxx]

Terraform used the selected providers to generate the following execution plan. Resource actions are indicated with the following symbols:
  + create

Terraform will perform the following actions:
...（省略）

Plan: 1 to add, 0 to change, 0 to destroy.

Changes to Outputs:
  + json_policy_length          = 1830
  + minified_json_policy_length = 1387
```

この例ではなんと **1830 文字から 1387 文字と 443 文字の削減、75% に圧縮** できました！

## 解説

SCP の文字数は空白や改行をカウントします。Terraform で作成したポリシーは、通常空白等を含むため、文字数は必要以上に多くカウントされます。

> SCP ドキュメントの最大サイズは 5,120 文字です。この最大サイズには、空白を含むすべての文字が含まれます。SCP のサイズを減らすには、引用符の外側にあるすべての空白文字 (スペースや改行など) を削除できます。

https://docs.aws.amazon.com/ja_jp/organizations/latest/userguide/org_troubleshoot_policies.html

この課題に対応するため、`terraform-provider-aws` の `v5.49.0` において `minified_json` の機能が 2024/05/10 に追加されています。利用できない場合は `aws provider` のバージョンを確認してください。

https://github.com/hashicorp/terraform-provider-aws/pull/35677

## まとめ

SCP の文字数削減は、OU の継承を利用する、タグで制御する、制限範囲を狭めるなど根本対策が重要です。しかし Terraform を利用しており、これらの対策が困難な場合にはこの機能を使ってみてください！
