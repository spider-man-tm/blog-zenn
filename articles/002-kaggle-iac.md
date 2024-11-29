---
title: "Terraformを使ってGoogleCloud上にKaggle環境をささっと構築する"
emoji: "🌟"
type: "tech" # tech: 技術記事 / idea: アイデア
topics:
  - "Terraform"
  - "Kaggle"
  - "Docker"
published: true
---

# はじめに

サクッと Kaggle 環境を構築したいときってありますよね。今回は Terraform を使って Google Cloud Platform 上に Kaggle 環境を構築する方法を紹介します。この記事を読むことで、以下のような複数のインスタンスを立ち上げることができるようになります。

- 初めから Kaggle API が使える状態
- 初めから Docker が使える状態
- 初めから GitHub が使える状態
- 自身が指定した ssh public key が登録された状態
- GCS へのアクセス権限がある状態
- 静的 IP アドレスが割り当てられている状態

以下、リポジトリになります。

[GitHub Repos](https://github.com/spider-man-tm/kaggle-infrastructure/tree/main)

# GCP Project のセットアップ

まず始めに、GCP のプロジェクトを作成します。プロジェクト名は任意ですが、この記事では `kaggle-infra` としています。必要なコマンドは [setup-gcp-project/Makefile](https://github.com/spider-man-tm/kaggle-infrastructure/blob/main/setup-gcp-project/Makefile) にまとめているので以下のコマンドを叩けば最低限必要な準備ができた状態でプロジェクトが作成されます。

```bash
cd setup-gcp-project
make setup-gcp-project \
    GCP_PROJECT_ID=<your_gcp_project_id> \
    BILLING_ACCOUNT_ID=<your_billing_account_id> \
    KAGGLE_KEY=<your_kaggle_api_key>
```

一応、やっていることをざっと説明すると

- GCP プロジェクトの作成と billing account の紐付け
- terraform 用のサービスアカウントの作成と必要なロールの付与、およびローカル環境へのダウンロード
- 必要な API の有効化
- シークレットマネージャーの作成

最後のシークレットマネージャーですが、この後の手順で作成するインスタンスに対して、Kaggle API キーを渡すために使います。terraform.tfvars に直接書くのはセキュリティ的によくないので、シークレットマネージャーに格納しておきます。

# Terraform でインフラを構築

Terraform では必要最低限のリソースをカバーするためのモジュールをまとめた`modules/`ディレクトリと、複数コンペを同時並行で走らせたり、環境を各々切り分けるために`environments/`ディレクトリを用意しています。
動かし方は README にも記載していますが、まず始めに自身が用意した環境、例えば`terraform/environments/competition01/`配下に`terraform.tfvars`ファイルを作成し、そちらに必要な変数群を定義します。

```hcl
# General
project_id = "kaggle-infra"
region     = "asia-northeast1"

# GCE
zone             = "asia-northeast1-c"
instance_name    = "titanic-instance"
machine_type     = "e2-micro"
image            = "ubuntu-os-cloud/ubuntu-2004-lts"
pub_key_path     = "~/.ssh/id_ed25519.pub" # Key path used for SSH login (local PC)
network_name     = "default"
github_email     = "xyz@gmail.com"
github_user_name = "Your GitHub Username"
kaggle_username  = "spidermandance" # Your Kaggle username

# GCE & Network
instance_count = 1

# Network
static_ip_name = "titanic-static-ip"

# GCS
bucket_name = "titanic-bucket-xyz" # GCS bucket name
location    = "ASIA"               # GCS region
```

インスタンス・静的 IP・GCS に関する基本情報と、GitHub のユーザー情報と、Kaggle User Name の情報、並びにローカルの公開鍵のパス情報を定義しています。[terraform/modules/gce/main.tf](https://github.com/spider-man-tm/kaggle-infrastructure/blob/main/terraform/modules/gce/main.tf)、並びに
[terraform/modules/gce/startup-script.sh.tpl](https://github.com/spider-man-tm/kaggle-infrastructure/blob/main/terraform/modules/gce/startup-script.sh.tpl)を見るとわかりますが、docker コマンドのインストール、GitHub のセットアップ、Kaggle API のセットアップ、自身のローカル環境にある公開鍵の情報受け渡し等を行なった状態で GCE が起動するようにしています。ただし、Kaggle API に利用する Kaagle Key だけは一応プライベートなものなので、先ほど GCP プロジェクトの立ち上げ時に作成した Secret Manager に対して、[data block](https://github.com/spider-man-tm/kaggle-infrastructure/blob/main/terraform/environments/competition01/main.tf) を使って参照する形にしています。

```hcl
# Get the value from secret manager
data "google_secret_manager_secret_version" "kaggle_key" {
  secret  = "kaggle-key"
  version = "latest"
}
```

また、公開鍵も登録されているので、直ぐに ssh 接続できるようになります。

```shell
# Use the private key that matches the public key specified in terraform.tfvars.
ssh -i ~/.ssh/id_ed25519 ubuntu@<External IP Address>
```

# Docker

一応、インスタンスを起動した後に簡単にコンテナも立ち上げられるようにしたかったので、`docker/`ディレクトリ以下に必要なファイルをまとめています。

```shell
cd docker
docker-compose up -d
```

# 終わりに

まだまだ作りかけなので徐々に追加予定ですが、そもそもここ 3 年くらい Kaggle に取り組めてないのでそろそろ再開したい今日この頃です。
