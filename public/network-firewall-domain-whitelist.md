---
title: "AWS Network Firewall でドメインによるホワイトリスト通信制御"
tags:
  - 'AWS'
  - 'NetworkFirewall'
  - 'Security'
  - 'Proxy'
private: false
updated_at: ''
id: null
organization_url_name: null
slide: false
ignorePublish: false
---

## AWS Network Firewall とは

AWS Network Firewall は、インバウンド・アウトバウンドの通信を制御できるクラウド型ファイアウォールです。AWS のフルマネージドサービスであるため、サーバーや仮想アプライアンスの管理が不要になります。

AWS Network Firewall の展開モデルとして、分散型・集約型・複合型の 3 パターンが考えられます。それぞれのモデルについては以下ページを参考にしてください。

https://aws.amazon.com/jp/blogs/news/networking-and-content-delivery-deployment-models-for-aws-network-firewall/

## ユースケース

ホワイトリスト形式のドメインで通信を制御することを考えます。この要件を達成するために自前のプロキシ (Squid) サーバーを運用している組織は多いのではないでしょうか。このユースケースで必要な要素は以下のとおりです。

![architecture1.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3852183/cd160de8-3da1-f4c0-cb9c-21085f9ae289.png)

- AWS Network Firewall 自体
- AWS Network Firewall 用のサブネット
- エンドポイントへ向けたルーティング
  - 内部のルーティング
  - エッジのルーティング (Ingress Routing)
- ステートフルルール
  - ホワイトリスト形式のドメイン

エッジのルーティング (Ingress Routing) は使用経験がない方が多いと思います。これはインバウンドの通信に対してルーティングが可能になる機能です。インバウンドのパケットに対して通信を検査するために設定が必要です。

https://aws.amazon.com/jp/blogs/news/new-vpc-ingress-routing-simplifying-integration-of-third-party-appliances/

AWS Network Firewall にはステートレスルールも存在します。5 タプル形式（ソース IP・ソースポート・宛先 IP・宛先ポート・プロトコル）による制御ができますが、今回は説明を割愛します。

https://docs.aws.amazon.com/ja_jp/network-firewall/latest/developerguide/firewall-rules-engines.html

### 構成図

以下の構成を作成します。Google への通信のみを許可し、それ以外の通信をドロップします。 セッションマネージャー接続用のドメインも許可しています。

![architecture2.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3852183/42868bfd-6e4b-2df9-fc7a-5a31d72bccc8.png)

### 構築

Terraform を利用します。コードのリソースとパラメーターを参考にして、マネジメントコンソールから作成しても構いません。

:::note
AWS Network Firewall のプロビジョニングには 10 分前後の時間がかかります。
:::

AWS Network Firewall の作成コードの一部抜粋です。

```hcl:modules/network_firewall/firewall.tf
# firewall
resource "aws_networkfirewall_firewall" "main" {
  name                = join("-", [var.env, var.name, "firewall"])
  firewall_policy_arn = aws_networkfirewall_firewall_policy.main.arn
  vpc_id              = aws_vpc.main.id
  subnet_mapping {
    subnet_id = aws_subnet.firewall.id
  }

  tags = {
    Name = join("-", [var.env, var.name, "firewall"])
  }
}

# firewall policy
resource "aws_networkfirewall_firewall_policy" "main" {
  name = join("-", [var.env, var.name, "policy"])

  firewall_policy {
    stateless_default_actions          = ["aws:forward_to_sfe"]
    stateless_fragment_default_actions = ["aws:forward_to_sfe"]
    stateful_rule_group_reference {
      resource_arn = aws_networkfirewall_rule_group.main.arn
    }
  }

  tags = {
    Name = join("-", [var.env, var.name, "policy"])
  }
}

# rule group
resource "aws_networkfirewall_rule_group" "main" {
  capacity = 100
  name     = join("-", [var.env, var.name, "rule"])
  type     = "STATEFUL"
  rule_group {
    rules_source {
      rules_source_list {
        generated_rules_type = "ALLOWLIST"
        target_types         = ["HTTP_HOST", "TLS_SNI"]
        targets              = var.allowed_domains
      }
    }
  }

  tags = {
    Name = join("-", [var.env, var.name, "rule"])
  }
}

# logging to cloudwatch
resource "aws_networkfirewall_logging_configuration" "main" {
  firewall_arn = aws_networkfirewall_firewall.main.arn
  logging_configuration {

    # flowlogs
    log_destination_config {
      log_destination = {
        logGroup = aws_cloudwatch_log_group.flow.name
      }
      log_destination_type = "CloudWatchLogs"
      log_type             = "FLOW"
    }

    # alertlogs
    log_destination_config {
      log_destination = {
        logGroup = aws_cloudwatch_log_group.alert.name
      }
      log_destination_type = "CloudWatchLogs"
      log_type             = "ALERT"
    }

  }
}
```

```hcl:environments/dev/main.tf
module "network_firewall" {
  source               = "../../modules/network_firewall"
  name                 = "dev-nfw"
  env                  = "dev"
  vpc_cidr             = "10.1.1.0/24"
  firewall_subnet_cidr = "10.1.1.16/28"
  public_subnet_cidr   = "10.1.1.32/28"
  allowed_domains      = [".google.com", ".amazonaws.com"]
}

module "ec2" {
  source    = "../../modules/ec2"
  name      = "dev-nfw"
  env       = "dev"
  vpc_id    = module.network_firewall.vpc_id
  subnet_id = module.network_firewall.public_subnet_id
}
```

```text:tree
├── environments
│   └── dev
│       ├── main.tf
│       └── provider.tf
└── modules
    ├── ec2
    │   ├── ec2.tf
    │   ├── iam.tf
    │   ├── outputs.tf
    │   ├── security_group.tf
    │   └── variables.tf
    └── network_firewall
        ├── cloudwatch.tf
        ├── firewall.tf
        ├── outputs.tf
        ├── variables.tf
        └── vpc.tf
```

:::note warn
ルーティングに必要なエンドポイント ID は直感的に取り出せません。以下ページを参考にしてください。

```hcl:Single-AZの場合
vpc_endpoint_id = tolist(aws_networkfirewall_firewall.main.firewall_status[0].sync_states)[0].attachment[0].endpoint_id
```

```hcl:Multi-AZの場合
for_each = toset(["ap-northeast-1a", "ap-northeast-1c", "ap-northeast-1d"])
vpc_endpoint_id = [for ss in tolist(aws_networkfirewall_firewall.main.firewall_status[0].sync_states) : ss.attachment[0].endpoint_id if ss.availability_zone == each.value][0]
```

参考：https://zenn.dev/toshikish/articles/fc08c2021811f9
:::

### 動作確認

作成された EC2 へセッションマネージャー経由でアクセスしてください。

:::note
デプロイ完了からアクセス可能になるまで 5 分前後の時間がかかる場合があります。
:::

許可ドメインに含まれる Google へアクセスします。TLS ハンドシェイクに成功しています。

```bash
curl -s -v -sslv3 -m 5 https://www.google.com 1> /dev/null
```

![curl_ok.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3852183/a7a9a546-69f4-a15f-279c-a9b7b2f7d9a8.png)

許可外のドメインである Yahoo へアクセスします。通信はタイムアウトしました。

```bash
curl -s -v -sslv3 -m 5 https://yahoo.co.jp 1> /dev/null
```

![curl_ng.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3852183/915e9c35-5796-a984-3075-bcec063953d5.png)

以上からドメインによる通信制御ができることが確認できました！

:::note
通過した通信はフローログ、ドロップした通信はアラートログに記録可能です。
:::

## 注意点

### セキュリティ対策効果

悪質な内部犯や高度なマルウェア等には効果がない場合があります。これは AWS Network Firewall は Host ヘッダや SNI の server_name を見ているだけで、実際の通信先を検査していないからです。より詳しい解説は以下ページを参照ください。

https://zenn.dev/osprey/articles/networkfirewall-is-not-for-domainbase-restriction

:::note
よくある要件の「アウトバウンド通信制御」自体は行っているため、Squid 等のプロキシサーバーが必要かはお客様や関係者と相談しましょう。
:::

#### ドメイン制御を回避してみる

以下コマンドで SNI の内容を偽装します。

```bash
curl https://www.google.com --resolv www.google.com:443:183.79.250.251 -H "Host: yahoo.co.jp" --insecure -v
```

`183.79.250.251` は Yahoo! JAPAN の IP アドレス（のひとつ）です。
Google への通信を偽装することで、Yahoo に通信できてしまいます。

![spoofing.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3852183/b97fbcdf-6dd7-4e51-6f2b-d41a3ed4b332.png)

### 価格

意外と高額になりがちです。固定費は工夫して削減することが難しく、小～中規模なプロジェクトでは支出の多くを占める可能性があります。以下は東京リージョンでの利用例です。

| Type                    | Amount    | Pricing      | Total     | Remarks            |
| ----------------------- | --------- | ------------ | --------- | ------------------ |
| Endpoint Charges        | 720 h * 2 | 0.395 USD/h  | 568.8 USD | 30 days, Multi-AZ  |
| Data Processing Charges | 3000 GB   | 0.065 USD/GB | 195.0 USD | 30 days, 100GB/day |

:::note
単に AWS リソースの稼働コストだけであれば、ALB + EC2 (Squid) の方が安いです。検討時には EC2 や Squid の運用コストを考慮しましょう。
:::

## まとめ

注意点のように気をつけるべきポイントはありますが、ドメインによるホワイトリスト通信制御として利用可能です。要件を満たせる場合は、マネージドサービスを積極的に利用していきましょう！
