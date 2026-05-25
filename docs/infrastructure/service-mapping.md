# 抽象 → クラウド/オンプレ サービス対照表

[`interview.md`](interview.md) と [`design.md`](design.md) では **抽象名**（コンピュート / RDB / KMS など）で書く。本ファイルで、選んだクラウド/オンプレの **具体サービス名** に翻訳する。

---

## コンピュート

| 抽象 | AWS | GCP | Azure | オンプレ |
|---|---|---|---|---|
| コンピュート (常時稼働 / VM) | EC2 | GCE | VM | KVM / VMware / bare-metal |
| コンピュート (常時稼働 / コンテナ・マネージド) | ECS (Fargate) | Cloud Run | Container Apps | Docker + Nomad |
| コンピュート (常時稼働 / Kubernetes) | EKS | GKE | AKS | k3s / Rancher / vanilla k8s |
| コンピュート (サーバレス関数) | Lambda | Cloud Functions | Functions | OpenFaaS / Knative |
| バッチ / ジョブ | Batch / Step Functions | Cloud Run jobs / Workflows | Batch | systemd timer / Nomad batch |

## ネットワーク

| 抽象 | AWS | GCP | Azure | オンプレ |
|---|---|---|---|---|
| 仮想ネットワーク | VPC | VPC | VNet | 物理ネットワーク + VLAN |
| ロードバランサ (L7) | ALB | Cloud Load Balancing (HTTPS) | Application Gateway | Nginx / HAProxy |
| ロードバランサ (L4) | NLB | Cloud Load Balancing (TCP/UDP) | Load Balancer | HAProxy / LVS |
| CDN | CloudFront | Cloud CDN | Front Door / CDN | Varnish + 自前配信 |
| WAF | AWS WAF | Cloud Armor | Front Door WAF | ModSecurity |
| プライベート接続 | PrivateLink / VPC Peering | Private Service Connect / VPC Peering | Private Link / VNet Peering | VPN / 専用線 |

## データ

| 抽象 | AWS | GCP | Azure | オンプレ |
|---|---|---|---|---|
| RDB (マネージド) | RDS / Aurora | Cloud SQL / AlloyDB | Azure DB for PostgreSQL / MySQL | 自前 PostgreSQL / MySQL |
| NoSQL (キー値) | DynamoDB | Firestore (Datastore) / Bigtable | Cosmos DB | Redis / etcd |
| NoSQL (ドキュメント) | DocumentDB | Firestore | Cosmos DB (Mongo API) | MongoDB |
| キャッシュ | ElastiCache (Redis) | Memorystore | Cache for Redis | Redis (self-hosted) |
| 検索 | OpenSearch Service | （Marketplace） | AI Search | Meilisearch / OpenSearch |
| オブジェクトストレージ | S3 | GCS | Blob Storage | MinIO / Ceph |
| データ転送 / ETL | Glue / DMS | Dataflow / DMS | Data Factory | Airflow / Embulk |

## メッセージング / イベント

| 抽象 | AWS | GCP | Azure | オンプレ |
|---|---|---|---|---|
| キュー | SQS | Pub/Sub (subscription) | Service Bus | RabbitMQ / NATS |
| Pub/Sub | SNS | Pub/Sub | Event Grid | NATS / Kafka |
| ストリーミング | Kinesis | Pub/Sub / Dataflow | Event Hubs | Kafka |
| ワークフロー | Step Functions | Workflows | Logic Apps / Durable Functions | Argo Workflows |

## セキュリティ / IAM

| 抽象 | AWS | GCP | Azure | オンプレ |
|---|---|---|---|---|
| シークレット管理 | Secrets Manager / SSM Parameter Store | Secret Manager | Key Vault | HashiCorp Vault |
| 鍵管理 (KMS) | KMS | Cloud KMS | Key Vault | Vault Transit |
| 証明書 | ACM | Certificate Manager | Key Vault Certificates | Let's Encrypt / 自前 CA |
| OIDC IdP (自前) | Cognito | Identity Platform | AD B2C | Keycloak |
| OIDC IdP (外部委譲) | Google / GitHub / Apple / Auth0 | 同左 | 同左 | 同左 |
| IAM | IAM | Cloud IAM | Entra ID (旧 Azure AD) | sudo / RBAC / LDAP |

## 監視・ログ

| 抽象 | AWS | GCP | Azure | オンプレ |
|---|---|---|---|---|
| メトリクス | CloudWatch Metrics | Cloud Monitoring | Monitor | Prometheus |
| ログ集約 | CloudWatch Logs | Cloud Logging | Log Analytics | Loki / ELK |
| トレース | X-Ray | Cloud Trace | Application Insights | Jaeger / Tempo |
| ダッシュボード | CloudWatch Dashboards | Cloud Monitoring | Monitor Workbooks | Grafana |
| アラート | CloudWatch Alarms + SNS | Cloud Monitoring + Pub/Sub | Monitor Alerts | Alertmanager |
| エラートラッキング | (外部 Sentry 推奨) | 同左 | 同左 | Sentry (self-host) |

## CI/CD・コンテナ

| 抽象 | AWS | GCP | Azure | オンプレ |
|---|---|---|---|---|
| コンテナレジストリ | ECR | Artifact Registry | Container Registry | Harbor |
| CI/CD (マネージド) | CodePipeline + CodeBuild | Cloud Build | Pipelines | Jenkins / Drone |
| (推奨) | GitHub Actions | 同左 | 同左 | 同左 (self-host runner も可) |

---

## 使い方

1. `interview.md` の Q-HOST-02 で採用クラウドを決める
2. `design.md` を書くときは **抽象名で記述**（例: 「コンピュート: コンテナ (マネージド)」）
3. `iac/` で具体実装に落とすとき、本表の該当列を引いて具体サービス名 / Terraform リソース名 / Ansible ロール名を決める
4. 採用列だけ抜粋した縮約版を `design.md` の末尾や ADR にコピーしてもよい
