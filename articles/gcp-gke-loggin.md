---
title: "GKEログ監視が便利になりました！"
emoji: "😸"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["GCP", "GKE", "Logging", "Monitoring"]
published: false
---

# 10 秒で概要

- 2021 年 7 月に、GCP のロギング機能にアップデートが入りました。
  - [ログベースのアラート機能がプレビュー版で利用可能に](https://cloud.google.com/blog/ja/products/operations/create-logs-alerts-preview)
- これにより、GKE などと簡単に連携して、ログベースアラートを実現できるようになりました。

**この記事で目指す姿**

![moverview](/images/gcp-gke-loggin/overview.png)

**前提**

- 任意のクラスタ上にソリューションが存在し、既存の Pod のログを通知する。
- クラスタは Autopilot モード にて実現済み。
- この記事では Email 通知を例として記載する。

# GCP における監視ソリューション詳細の抜粋

[オペレーションスイート（旧称 Stackdriver）](https://cloud.google.com/products/operations?hl=ja)でできること。

- Cloud Logging

> Cloud Logging は、大規模に実行されるフルマネージド サービスです。アプリケーションとシステムのログデータのほか、GKE 環境、VM、Google Cloud サービスからのカスタム ログデータも取り込めます。Cloud Logging では、選択したログを分析でき、アプリケーションのトラブルシューティングが加速します。

- Cloud Monitorig

> Cloud Monitoring では、クラウドで実行されるアプリケーションのパフォーマンスや稼働時間、全体的な動作状況を確認できます。Google Cloud サービス、ホステッド アップタイムのプローブ、アプリケーション インストルメンテーション、よく使われるさまざまなアプリケーション コンポーネントから指標、イベント、メタデータを収集し、チャートやダッシュボードで可視化してアラートを管理できます。

その他、APM（Application Performance Management）として、`Cloud Trace`、`Cloud Debugger`、`Cloud Profiler`が存在する。

## ポイント

- [Cloud Operations for GKE](https://cloud.google.com/stackdriver/docs/solutions/gke?hl=ja)という名称で、GKE と`Cloud Logging`、`Cloud Monitoring`のネイティブな統合が成されている。
  - Cloud Operations for GKE を利用する場合、Prometeus とのシームレスな連携が可能

# Cloud Loggin の設定

## Loggin の ON／OFF の設定

`$ gcloud container clusters create/update NAME`実行時における、--logging フラグにてサポートされる。

- `--logging NONE`
  - ロギングを行わない
- `--logging SYSTEM`
  - コントロールプレーンからのログ全般
- `--logging WORKLOAD`
  - ユーザノードで実行されているシステム以外の Pod ログ

※Autopilot モードの場合は、クラスタ作成時点で強制的に WORKLOAD 状態となる。

## ログエクスプローラのアップデート

ログベースアラート機能等の新機能を使うために、ログエクスプローラのアップデートを行う。

- 以前のログビューアより、アップデートを選択

![update](/images/gcp-gke-loggin/log_update.png)

## Pod ログのクエリの検索

- クラスタの Pod 詳細の画面から、ログを表示を押下することでログエクスプローラに遷移する

![pod_log](/images/gcp-gke-loggin/pod_log.png)

- 上記手順によりログエクスプローラのアップデートを行った後であれば、新規エクスプローラ画面に、**Pod ログを検索するクエリ実行済み**のログエクスプローラに遷移する。

![pod_log_query](/images/gcp-gke-loggin/pod_log_query.png)

クエリ例）

```bash
resource.type="k8s_container"
resource.labels.project_id="PROJECT_ID"
resource.labels.location="asia-northeast1"
resource.labels.cluster_name="CLUSTER_NAME"
resource.labels.namespace_name="default"
resource.labels.pod_name="POD_NAME"
resource.labels.container_name="CONTAINER_NAME"
```

### ログアラートの作成

- ログエクスプローラの`操作`から、`ログアラートの作成`を選択する。

![log_alert_1](/images/gcp-gke-loggin/log_alert1.png)

- 以下を入力し、Alert の設定を行う。

  - Alert 名称
  - Alert の Description
  - ポーリング間隔
  - クエリの設定
  - ※通知チャネルを作ってない場合は最後の項目で`通知チャンネルを管理`を選択する

ここでは、一旦

![log_alert_2](/images/gcp-gke-loggin/log_alert2.png)

- ここでは、ベータ版ではない Email を利用して通知チャンネルを作成する。
  - Email 右側の Add New を選択
  - 通知先のメールアドレスと、通知名称を入力して`SAVE`を押下

![monitor_email](/images/gcp-gke-loggin/monitor_email.png)

- ログエクスプローラに戻り更新ボタンを押下することで、作成した通知チャネルが表示される。

![log_alert_3](/images/gcp-gke-loggin/log_alert3.png)

- この時点で飛んでくるメールは以下の通り。

![notify_email](/images/gcp-gke-loggin/notify_email.png)

# Cloud Monitoring の設定

## Monitoring の ON／OFF

`$ gcloud container clusters create/update NAME`実行時における、--monitorign フラグにてサポートされる。

- `--monitorign NONE`
  - ロギングを行わない
- `--monitorign SYSTEM`
  - コントロールプレーンからのログ全般

※Autopilot モードの場合は、クラスタ作成時点で強制的に SYSTEM 状態となる。

## Monitoring アラート画面の利用方法

Cloud Logging にて通知設定を行い、メール通知などに含まれる`VIEW INCIDENT`を押下すると、Monitor のアラートページに遷移する。

この画面では、以下の操作ができる。

- ログ通知の POLICY の検索
- ログ通知の POLICY の編集
- 通知の際のログ表示
- インシデントの管理（認知するか、クローズするか）

インシデントとして認知した場合は、Monitoring のアラート画面にて確認済みとなる。

![monitor_incident_monitor_incident_acknowlegde](/images/gcp-gke-loggin/monitor_incident_acknowlegde.png)

インシデントをクローズした場合は、Monitoring のアラート画面からインシデント行が削除される。`SHOW CLOSED INCIDENTS`を押下することで、クローズ済みののインシデントの一覧を確認することが可能。

![monitor_incident_monitor_incident_closed](/images/gcp-gke-loggin/monitor_incident_closed.png)

# 対象ログの絞り込み例

Cloud Loggin では、監視対象クラスタのコンソールログが json 形式で記載されている。実際のログについては`textPayLoad`フィールドにて格納されているため、通知対象のログを絞り込みたいときはクエリに以下の情報を記載する。

```
textPayLoad="SEVERE hogehoge"
```

# 終わりに

- 既にクラスタが存在する場合は、作業時間 30 分程度で Email 通知ができました。
- ポーリングタイミングで通知がプッシュされますが、Monitor 上同一のインシデントは 1 件しかたまりませんでした。
