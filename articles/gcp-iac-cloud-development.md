---
title: "GCPにおけるIaC（Cloud Development Manager）を触ってみる"
emoji: "👌"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["GCP", "IaC", "CDM"]
published: false
---

# 10 秒で概要

GCP にて IaC を行うには、代表的なツールに以下の二つの候補があります。

- Cloud Deployment Manager
- Terraform

このうち Cloud Development Manager（以下 CDM） には、以下の特徴があります。

- GCP リソースを yaml や json で宣言的に記述、デプロイ可能
  - Python 等のスクリプト言語を利用すると、共通部分を可変的に記載することが可能
- AWS でいう CloudFormation
- 実行環境含めて完全無料

この記事では、CDM の手軽さに触れ、さらに参考となるドキュメント等を紹介します。

**前提**

- GCP で IaC を始める手っ取り早さに着目したいため、Python には触れません。

# Cloud Shell

CDM の実行は、Cloud Shell から実行することで、設定が非常に少なくすぐに利用可能です。
Cloud Shell を別ブラウザで開くと、以下のようにエディタと共にコンソールが利用できます。

![cloud-shell](/images/gcp-iac-cloud-development/cloud-shell.png)

- `gcloud config list`にて、project が対象のプロジェクトであることを確認します。

```bash
$ gcloud config list
[accessibility]
screen_reader = True
[component_manager]
disable_update_check = True
[compute]
gce_metadata_read_timeout_sec = 30
[core]
account = myaccount@hoge.com
disable_usage_reporting = True
project = MY_PROJECT
[metrics]
environment = devshell
```

- project が正しくセットされていない場合は、`$ gcloud config set project [MY_PROJECT]`にて自身のプロジェクトをセットします。

# とにかく動かす

一番簡単な GCE インスタンスを立ち上げます。

```yaml
resources:
  - type: compute.v1.instance
    name: test-vm
    properties:
      zone: asia-northeast1-a
      machineType: https://www.googleapis.com/compute/v1/projects/[MY_PROJECT]/zones/asia-northeast1-a/machineTypes/f1-micro
      disks:
        - deviceName: boot
          type: PERSISTENT
          boot: true
          autoDelete: true
          initializeParams:
            sourceImage: https://www.googleapis.com/compute/v1/projects/debian-cloud/global/images/family/debian-9
      networkInterfaces:
        - network: https://www.googleapis.com/compute/v1/projects/[MY_PROJECT]/global/networks/default
          accessConfigs:
            - name: External NAT
              type: ONE_TO_ONE_NAT
```

yaml の説明です。

- `Compute Engine` を１インスタンスデプロイするサンプル。
- `resources.type` に対象となるリソースを指定する。[Supported resource types](https://cloud.google.com/deployment-manager/docs/configuration/supported-resource-types?hl=en)を確認することで、IaC 可能なリソースが分かる。
- Supported resource type から`compute.v1.instance`の[Documentation](https://cloud.google.com/compute/docs/reference/rest/v1/instances)を見ると、yaml フィールドの詳細設定値が分かる。
- `type`には`compute.v1.instance`, `machineType` には一番小さい`f1-micro`を指定
- `properties.disks[].type`は`PERSISTENT`と`SCRATCH`を選択可能。インスタンス削除時に揮発するかどうかの違い。
  - `SCRATCH` の場合は、インスタンス削除時に揮発する。デプロイ中は永続化されたネットワークを超えないため、オーバヘッドが非常に小さくなる。
  - `PERSISTENT`の場合は、外部ボリュームが設けられるため削除してもデータが永続化される。ただしネットワークを超えるため、`SCRATCH`に比べてオーバヘッドが発生する。
- `properties.disks[].initializeParams.sourceImage`にて、初期ブート時のディスクにインストールしたいパブリックオペレーションシステムを選択する。
- `properties.networkInterfaces[].network`では、ネットワークリソースの URL を指定する。デフォルトでは`/global/networks/default`となる。
- `properties.networkInterfaces[].accessConfigs.type`は、設定しないと外部アクセスはできない。`ONE_TO_ONE_NAT`固定。

yaml を直接実行する場合は、一律以下コマンドで実行します。

`$ gcloud deployment-manager deployments METHOD DEPLOY_NAME --config DEPLOY.yaml --preview`

- `METHOD`は、`create`, `delete`, `update`等を指定します。
- `DEPLOY_NAME`は、後述する Deployment Manager コンソールでの表示名です。
- `DEPLOY.yaml`は、実行対象の yaml です。
- `--preview`は、dry-run を意味します。

上記 yaml を`gce-sample.yaml`とし、`DEPLOY_NAME`も同じ名称とした場合、以下の実行結果になります。

```bash
$ gcloud deployment-manager deployments create gce-sample --config gce-sample.yaml --preview
The fingerprint of the deployment is b'i4vkrs9EqX03g7QuYxs-Sw=='
Waiting for create [operation-1637302267341-5d11e2408ebd4-8a8ba119-eabf3332]...done.
Create operation operation-1637302267341-5d11e2408ebd4-8a8ba119-eabf3332 completed successfully.
NAME: test-vm
TYPE: compute.v1.instance
STATE: IN_PREVIEW
ERRORS: []
INTENT: CREATE_OR_ACQUIRE
```

実行すると、Deployment Manager コンソールに、gce-sample が表示されるので、選択するとワンクリックでデプロイが可能です。
![!deployment-manager](/images/gcp-iac-cloud-development/deployment-manager.png)

# TIPS

GCP は、かゆいところに手が届かないドキュメントが多いため、当記事を読むにあたって参考になる情報をいくつか記載します。

## 利用可能な Compute Engine イメージの調べ方

`Compute Engine`を IaC する際の yaml フィールドである`properties.disks[].initializeParams.sourceImage`に記載可能な OS イメージは、以下コマンドで検索可能です。

- `$ gcloud compute images list`

```bash
$ gcloud compute images list | grep debian
NAME: debian-10-buster-v20211105
PROJECT: debian-cloud
FAMILY: debian-10
NAME: debian-11-bullseye-v20211105
PROJECT: debian-cloud
FAMILY: debian-11
NAME: debian-9-stretch-v20211105
PROJECT: debian-cloud
FAMILY: debian-9
```

同様に `machineType` や`network`については、以下コマンドで検索可能です。（machineType の実行結果は膨大なので割愛）

- `$ gcloud compute machine-types list`

- `$ gcloud compute networks list`

```bash
$ gcloud compute networks list
NAME: default
SUBNET_MODE: AUTO
BGP_ROUTING_MODE: REGIONAL
IPV4_RANGE:
GATEWAY_IPV4:
```

## デプロイ可能なリソースの調べ方

基本は、以下に IaC 可能な Resource の一覧が有るため、まずはここを確認します。

- [Supported resource types](https://cloud.google.com/deployment-manager/docs/configuration/supported-resource-types?hl=en)

上記リソースについては、以下の template が用意されています。

- [GoogleCloudPlatform/cloud-foundation-toolkit](https://github.com/GoogleCloudPlatform/cloud-foundation-toolkit/tree/master/dm/templates)

IaC を行いたいリソースが見つからない場合は、以下のサンプルリポジトリに対象のリソースのサンプルが有るかもしれません。これは、最終的には REST API にて Resource を操作するためリソースの操作自体は可能であるが、ドキュメント化がされていないだけに見受けられます。例えば Monitoring の notificationChannels 等は、以下サンプルの公開及び、[リソースの詳細ページ](https://cloud.google.com/monitoring/api/ref_v3/rest/v3/projects.notificationChannels)にて REST API のパラメータが公開されています。

- [GoogleCloudPlatform/deploymentmanager-samples](https://github.com/GoogleCloudPlatform/deploymentmanager-samples)

さらに、Terraform の公式もリソースのパラメータを調べるのに役立ちます。各リソースの詳細からは、GCP 公式の上記リソースの詳細ページへのリンクも記載されています。

- [Terraform - google](https://registry.terraform.io/providers/hashicorp/google/latest/docs)

# 終わりに

- GCE を立ち上げるまでのみであれば、ものの 5 分で IaC ができてしまいました。
- どのクラウドサービスでもそうですが、ドキュメントに癖が有る場合が多いです。今回の場合は、自分の利用したいリソースが IaC 可能かどうかを調べることが一番苦労しました。
- 本来は Python と連携して、自身のプロジェクト ID 等や共通で利用する Docker イメージなど、共通的な部分を外出しできるような運用ができると尚便利ですね。
