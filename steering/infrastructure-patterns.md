# StackGen Infrastructure Patterns

Resource patterns and configuration guidance for common workload types across AWS, Azure, and GCP. Use these patterns when proposing infrastructure to a user based on their codebase.

---

## AWS Patterns

### Node.js / Python / Go / Java API (Containerized)

**Typical resources:**
- `ECS_CLUSTER` — container orchestration
- `ECS_SERVICE` (Fargate) — managed container runtime
- `ECR_REPOSITORY` — container image registry
- `APPLICATION_LOAD_BALANCER` — HTTP/HTTPS traffic routing
- `TARGET_GROUP` — ECS service registration
- `SECURITY_GROUP` — network access control

**Add database if needed:**
- `RDS_INSTANCE` (PostgreSQL/MySQL) for relational data
- `DYNAMODB_TABLE` for key-value / document workloads
- `ELASTICACHE_CLUSTER` for caching

**Resource pack shortcut:** Look for `ecs-fargate-api` or `container-api` packs via `get_supported_resource_types`. These bundle ECS Cluster + Service + ECR + ALB + Target Group in one call.

**Key configuration:**
- ECS Service: set `desired_count`, `cpu`, `memory`, container port
- ALB: set listener port (80/443), SSL certificate ARN if HTTPS
- RDS: set `instance_class` (e.g. `db.t3.micro` for dev, `db.r6g.large` for prod), `engine`, `engine_version`

---

### Node.js / Next.js (Static / SSR Frontend)

**Typical resources:**
- `S3_BUCKET` — static asset storage
- `CLOUDFRONT_DISTRIBUTION` — CDN and edge caching
- `ROUTE53_RECORD` — custom domain DNS (if custom domain required)
- `ACM_CERTIFICATE` — TLS certificate for CloudFront

**Key configuration:**
- S3 Bucket: set `website_enabled: true` for static hosting, configure bucket policy for public read or restrict to CloudFront OAI
- CloudFront: set `origin` to S3 bucket, `default_root_object: index.html`, enable compression

---

### Serverless (Lambda + API Gateway)

**Typical resources:**
- `LAMBDA_FUNCTION` — serverless compute
- `API_GATEWAY_REST_API` or `API_GATEWAY_V2` (HTTP API) — HTTP endpoint
- `DYNAMODB_TABLE` — stateful storage for serverless
- `S3_BUCKET` — asset or artifact storage
- `IAM_ROLE` — Lambda execution role with least-privilege policies

**Key configuration:**
- Lambda: set `runtime` (`nodejs20.x`, `python3.12`, `go1.x`, `java21`), `handler`, `memory_size`, `timeout`
- API Gateway: set `stage_name` matching environment (`dev`, `prod`)

---

### Data Pipeline / Batch Processing

**Typical resources:**
- `SQS_QUEUE` — message queuing and decoupling
- `SNS_TOPIC` — pub/sub fan-out
- `LAMBDA_FUNCTION` — event-driven processing
- `S3_BUCKET` — data lake / staging
- `RDS_INSTANCE` or `DYNAMODB_TABLE` — processed data storage

---

## Azure Patterns

### Containerized API (App Service / Container Apps)

**Typical resources:**
- `APP_SERVICE_PLAN` — compute tier and scaling configuration
- `APP_SERVICE` (Web App for Containers) — managed container hosting
- `CONTAINER_REGISTRY` — Azure Container Registry (ACR)
- `APPLICATION_GATEWAY` or `AZURE_LOAD_BALANCER` — traffic routing

**Add database if needed:**
- `AZURE_SQL_DATABASE` for relational workloads
- `COSMOS_DB_ACCOUNT` for globally distributed NoSQL
- `AZURE_CACHE_FOR_REDIS` for caching

**Key configuration:**
- App Service Plan: set `sku` (`B1` for dev, `P2v3` for prod), `os_type` (`Linux`)
- App Service: set `docker_image`, `docker_image_tag`, `docker_registry_url`

---

### Azure Functions (Serverless)

**Typical resources:**
- `FUNCTION_APP` — serverless compute host
- `APP_SERVICE_PLAN` (Consumption or Premium) — hosting plan
- `STORAGE_ACCOUNT` — required for function state
- `COSMOS_DB_ACCOUNT` or `AZURE_SQL_DATABASE` — data storage

---

### AKS (Kubernetes)

**Typical resources:**
- `AKS_CLUSTER` — managed Kubernetes
- `CONTAINER_REGISTRY` — ACR for images
- `AZURE_LOAD_BALANCER` — external traffic ingress
- `VIRTUAL_NETWORK` + `SUBNET` — network topology

---

## GCP Patterns

### Containerized API (Cloud Run)

**Typical resources:**
- `CLOUD_RUN_SERVICE` — serverless container hosting (preferred for stateless APIs)
- `ARTIFACT_REGISTRY` — container image registry
- `CLOUD_LOAD_BALANCING` — global HTTPS load balancer (if custom domain or multi-region)

**Add database if needed:**
- `CLOUD_SQL_INSTANCE` for relational workloads
- `FIRESTORE` for document/NoSQL workloads
- `MEMORYSTORE` (Redis) for caching

**Key configuration:**
- Cloud Run: set `min_instances` (0 for cost savings, 1+ for low latency), `max_instances`, `container_concurrency`, `cpu`, `memory`

---

### GKE (Kubernetes)

**Typical resources:**
- `GKE_CLUSTER` — managed Kubernetes
- `ARTIFACT_REGISTRY` — container images
- `CLOUD_LOAD_BALANCING` — ingress

---

## Resource Sizing Guidelines by Environment

| Resource | Dev | Staging | Production |
|----------|-----|---------|------------|
| ECS Fargate (CPU/Memory) | 256 CPU / 512 MB | 512 CPU / 1 GB | 1024+ CPU / 2+ GB |
| RDS instance | `db.t3.micro` | `db.t3.small` | `db.r6g.large`+ |
| App Service Plan (Azure) | B1 | B2 | P2v3+ |
| Cloud Run min instances | 0 | 1 | 2+ |
| CloudFront / CDN | Standard | Standard | Standard + WAF |

---

## Naming Conventions

Follow this pattern for consistent StackGen resource identifiers:

```
<service>-<resource-type>-<environment>
```

Examples:
- `checkout-api-ecs-prod`
- `payments-rds-staging`
- `frontend-cdn-dev`

AppStack names follow:
```
<repo-name>-<environment>
```

Examples:
- `checkout-api-prod`
- `auth-service-staging`
- `frontend-dev`

---

## Environment Profile Variables

When creating environment profiles, include these standard variables per cloud provider:

### AWS
```
AWS_REGION = us-east-1      # or chosen region
ENVIRONMENT = dev|staging|prod
APP_PORT = <port>
```

### Azure
```
AZURE_LOCATION = eastus     # or chosen location
ENVIRONMENT = dev|staging|prod
APP_PORT = <port>
```

### GCP
```
GCP_REGION = us-central1    # or chosen region
ENVIRONMENT = dev|staging|prod
APP_PORT = <port>
```
