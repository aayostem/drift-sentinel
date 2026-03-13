<div align="center">

<img src="https://img.shields.io/badge/version-2.0.0-6366f1?style=for-the-badge&labelColor=0f172a" />
<img src="https://img.shields.io/badge/python-3.9+-10b981?style=for-the-badge&labelColor=0f172a" />
<img src="https://img.shields.io/badge/FastAPI-0.104+-06b6d4?style=for-the-badge&labelColor=0f172a" />
<img src="https://img.shields.io/badge/kubernetes-1.28+-3b82f6?style=for-the-badge&labelColor=0f172a" />
<img src="https://img.shields.io/badge/license-MIT-f59e0b?style=for-the-badge&labelColor=0f172a" />

<br /><br />

```
 ██████╗ ██████╗ ██╗███████╗████████╗    ███████╗███████╗███╗   ██╗████████╗██╗███╗   ██╗███████╗██╗
 ██╔══██╗██╔══██╗██║██╔════╝╚══██╔══╝    ██╔════╝██╔════╝████╗  ██║╚══██╔══╝██║████╗  ██║██╔════╝██║
 ██║  ██║██████╔╝██║█████╗     ██║       ███████╗█████╗  ██╔██╗ ██║   ██║   ██║██╔██╗ ██║█████╗  ██║
 ██║  ██║██╔══██╗██║██╔══╝     ██║       ╚════██║██╔══╝  ██║╚██╗██║   ██║   ██║██║╚██╗██║██╔══╝  ██║
 ██████╔╝██║  ██║██║██║        ██║       ███████║███████╗██║ ╚████║   ██║   ██║██║ ╚████║███████╗███████╗
 ╚═════╝ ╚═╝  ╚═╝╚═╝╚═╝        ╚═╝       ╚══════╝╚══════╝╚═╝  ╚═══╝   ╚═╝   ╚═╝╚═╝  ╚═══╝╚══════╝╚══════╝
```

### Close the loop between drift detection and remediation — automatically, at any scale.

<br />

[**Quick Start**](#quick-start) · [**Architecture**](#architecture) · [**API Reference**](#api-reference) · [**Deployment**](#deployment) · [**Roadmap**](#roadmap)

<br />

</div>

---

## The Problem

Production ML models silently degrade. Input distributions shift, relationships between features and targets change, and model accuracy erodes — often for days before anyone notices. Existing monitoring tools surface the alert. Then a human has to investigate, decide what to do, execute a fix, and verify it worked.

**Drift Sentinel closes that loop.** It detects drift in real-time, performs automated root cause analysis across correlated signals, executes safe remediation actions, and verifies recovery — with a full audit trail for SOC2 compliance.

---

## Table of Contents

- [How It Works](#how-it-works)
- [Features](#features)
- [Architecture](#architecture)
- [Tech Stack](#tech-stack)
- [Quick Start](#quick-start)
- [Configuration](#configuration)
- [Project Structure](#project-structure)
- [Core Components](#core-components)
- [API Reference](#api-reference)
- [Deployment](#deployment)
- [Observability](#observability)
- [Security](#security)
- [Troubleshooting](#troubleshooting)
- [Roadmap](#roadmap)
- [Contributing](#contributing)
- [License](#license)

---

## How It Works

```
Model serves predictions
        │
        ▼
Monitoring Service samples prediction distribution continuously
        │
        ├── No drift detected → continue
        │
        └── Drift detected (KS-test / PSI / ADWIN / Page-Hinkley)
                │
                ▼
        RCA Engine correlates signals
        (feature importance + temporal patterns + external factors)
                │
                ▼
        Safety Controller evaluates remediation plan
        (resource limits, rollback triggers, cooldown checks)
                │
                ▼
        Remediation Engine executes action
        (retrain │ rollback │ scale │ threshold adjustment)
                │
                ├── Success → alert team, log to audit trail
                │
                └── Failure → auto-rollback → escalate to PagerDuty
```

**Detection-to-remediation median time: 2m 30s** on default configuration.

---

## Features

### Drift Detection

| Algorithm | Type | Best For | Sensitivity |
|-----------|------|----------|-------------|
| **KS-Test** | Statistical | Distribution shifts in numerical features | Medium |
| **PSI** | Stability Index | Population stability over time | Low |
| **Earth Mover's Distance** | Statistical | Multi-dimensional feature drift | High |
| **ADWIN** | Adaptive windowing | Real-time concept drift | High |
| **DDM** | Online learning | Classification problem drift | Medium |
| **Page-Hinkley** | Change point | Abrupt distribution changes | High |

All algorithms run concurrently. Drift is confirmed only when a configurable quorum of methods agree — dramatically reducing false positives without sacrificing detection speed.

### Automated Remediation

| Action | Safety Check | Rollback | Notes |
|--------|-------------|----------|-------|
| `retrain_model` | Resource availability | No | Triggers full pipeline run |
| `rollback_model` | Version validation | N/A | Instant; previous artifact must exist |
| `scale_resources` | Capacity headroom | Yes | HPA adjustment |
| `update_thresholds` | Performance impact sim | Yes | Dry-run before apply |
| `notify_team` | None | No | Multi-channel: Slack, PagerDuty, email |

### Root Cause Analysis

- **Signal correlation** across features, latency, error rates, and external events
- **Granger causality tests** to distinguish correlation from causation
- **Temporal pattern matching** to identify recurring issue signatures
- **Confidence-scored reports** with prioritized remediation recommendations

### Enterprise Capabilities

- **SOC2-ready audit logging** — every action, actor, and outcome recorded immutably
- **Multi-cloud** — EKS, GKE, AKS with Terraform modules for each
- **Multi-tenant** — namespace-isolated deployments with RBAC
- **HA by default** — minimum 3 replicas per service, HPA pre-configured

---

## Architecture

```
┌───────────────────────────────────────────────────────────────────┐
│                         CLIENT LAYER                              │
│          REST API · gRPC · Python SDK · Java SDK · Web UI         │
└──────────────────────────────┬────────────────────────────────────┘
                               │ HTTPS / gRPC
┌──────────────────────────────▼────────────────────────────────────┐
│                        API GATEWAY                                │
│         FastAPI · Auth (JWT/mTLS) · Rate Limiting · Caching       │
└──────┬─────────────────────────────────────────────────┬──────────┘
       │                                                 │
┌──────▼──────────────┐                   ┌─────────────▼──────────┐
│  Monitoring Service │                   │  Remediation Service   │
│                     │                   │                        │
│  • Anomaly detect   │                   │  • Execution engine    │
│  • Drift detection  │──── incident ────▶│  • Action registry     │
│  • Perf tracking    │                   │  • Safety controller   │
└──────┬──────────────┘                   └─────────────┬──────────┘
       │                                                 │
┌──────▼──────────────┐                   ┌─────────────▼──────────┐
│  Analysis Service   │                   │   Alerting Service     │
│                     │◀─── rca_request ──│                        │
│  • RCA engine       │                   │  • Alert manager       │
│  • Correlation      │──── rca_result ──▶│  • Multi-channel notify│
│  • Pattern detect   │                   │  • Priority escalation │
└──────┬──────────────┘                   └────────────────────────┘
       │
┌──────▼──────────────────────────────────────────────────────────┐
│                        DATA & ML LAYER                           │
│    Feature Store (Feast) · Model Registry (MLflow) · Pipelines  │
└──────┬───────────────────────────────┬───────────────────────────┘
       │                               │
┌──────▼──────┐  ┌──────────┐  ┌──────▼──────┐  ┌──────────────┐
│  PostgreSQL │  │  Redis   │  │  S3 / MinIO │  │  TimescaleDB │
│  (metadata) │  │  (cache) │  │  (artifacts)│  │   (metrics)  │
└─────────────┘  └──────────┘  └─────────────┘  └──────────────┘
                               │
┌──────────────────────────────▼────────────────────────────────┐
│              Kubernetes (EKS / GKE / AKS)                     │
│     Istio service mesh · Cilium CNI · Karpenter autoscaler    │
└───────────────────────────────────────────────────────────────┘
```

**Key design decisions:**

- **Service isolation** — Monitoring and Remediation are separate services. A remediation action that consumes significant resources cannot starve the detection pipeline.
- **Safety controller is a hard gate** — no remediation action executes without passing resource checks, version validation, and cooldown enforcement. This is not configurable away per-action; it's enforced at the engine level.
- **Metrics in TimescaleDB, not Prometheus** — Prometheus is used for scraping and alerting. Historical drift metric storage goes to TimescaleDB for long-range querying without Thanos complexity.
- **Istio for zero-trust networking** — all service-to-service traffic is mTLS. Network policies block anything not explicitly allowed.

---

## Tech Stack

**Core**

| Layer | Technology |
|-------|-----------|
| API framework | FastAPI (async) |
| Runtime | Python 3.9+ |
| Validation | Pydantic v2 |
| ORM | SQLAlchemy 2.0 |

**ML & Data Science**

| Layer | Technology |
|-------|-----------|
| Drift detection | scikit-learn, alibi-detect |
| Anomaly detection | PyOD, Prophet |
| Feature store | Feast |
| Model registry | MLflow |

**Infrastructure**

| Layer | Technology |
|-------|-----------|
| Orchestration | Kubernetes 1.28+ |
| Service mesh | Istio |
| IaC | Terraform |
| Packaging | Helm 3 |

**Observability**

| Layer | Technology |
|-------|-----------|
| Metrics | Prometheus + TimescaleDB |
| Dashboards | Grafana |
| Logging | ELK Stack |
| Tracing | Jaeger |

**Security**

| Layer | Technology |
|-------|-----------|
| Auth | OAuth2 / JWT (RSA256) |
| Secrets | HashiCorp Vault |
| Network | Kubernetes Network Policies |
| Audit | Fluentd → immutable S3 log store |

---

## Quick Start

### Prerequisites

```
Python 3.9+
Docker 20.10+
kubectl (configured against a cluster, or use Docker Desktop)
Helm 3.0+
```

### 1. Clone and configure

```bash
git clone https://github.com/your-org/mlops-platform.git
cd mlops-platform

cp .env.example .env
```

Minimum required variables to run locally:

```env
# Storage
DATABASE_URL="postgresql://user:password@localhost:5432/mlops"
REDIS_URL="redis://localhost:6379"
S3_BUCKET="mlops-artifacts"
S3_ENDPOINT="http://localhost:9000"   # MinIO for local dev

# ML
MLFLOW_TRACKING_URI="http://localhost:5000"

# Security
JWT_SECRET="change-me-in-production"
VAULT_ADDR=""          # Leave blank to use env-based secrets locally

# Alerting (optional for local dev)
SLACK_WEBHOOK_URL=""
PAGERDUTY_API_KEY=""
```

### 2. Start the local stack

```bash
# Launch Postgres, Redis, MinIO, MLflow
docker-compose up -d

# Verify everything is healthy
docker-compose ps

# Run database migrations
make db-migrate

# Start all services with hot reload
make dev
```

Services will be available at:

| Service | URL |
|---------|-----|
| API | `http://localhost:8000` |
| API docs (Swagger) | `http://localhost:8000/docs` |
| Grafana | `http://localhost:3000` (admin / admin) |
| MLflow | `http://localhost:5000` |
| MinIO console | `http://localhost:9001` |

### 3. Run your first drift check

```bash
# Generate sample reference + current data with injected drift
python scripts/generate_sample_data.py --inject-drift

# Trigger drift detection
curl -X POST http://localhost:8000/api/v1/monitoring/drift-check \
  -H "Authorization: Bearer $(make get-dev-token)" \
  -H "Content-Type: application/json" \
  -d '{
    "model_id": "fraud-detector-v1",
    "reference_data": "s3://mlops-artifacts/reference/baseline.parquet",
    "current_data": "s3://mlops-artifacts/current/today.parquet",
    "features": ["amount", "location", "device_id"],
    "threshold": 0.05
  }'
```

Expected response when drift is present:

```json
{
  "drift_detected": true,
  "drift_score": 0.23,
  "affected_features": [
    {
      "name": "amount",
      "drift_type": "data_drift",
      "p_value": 0.001,
      "magnitude": 0.23
    }
  ],
  "incident_id": "inc-2024-03-16-001",
  "timestamp": "2024-03-16T10:30:00Z"
}
```

Open Grafana at `http://localhost:3000` to see the drift event appear on the ML dashboard in real time.

### 4. Verify system health

```bash
make health-check
```

```json
{
  "status": "healthy",
  "components": {
    "database": "connected",
    "redis": "connected",
    "s3": "accessible",
    "mlflow": "reachable",
    "prometheus": "scraping"
  },
  "version": "2.0.0",
  "uptime": "0d 0h 4m"
}
```

---

## Configuration

All configuration is environment-variable driven. See [`.env.example`](./.env.example) for the full annotated reference.

| Group | Variables | Purpose |
|-------|-----------|---------|
| `DATABASE_*` | Connection string, pool size | PostgreSQL |
| `REDIS_*` | URL, password, TTL defaults | Cache and pub/sub |
| `S3_*` | Bucket, endpoint, credentials | Artifact storage |
| `MLFLOW_*` | Tracking URI | Model registry |
| `DRIFT_*` | Thresholds, window sizes, quorum | Detection sensitivity |
| `REMEDIATION_*` | Max retries, cooldown, resource caps | Safety controller |
| `VAULT_*` | Address, role, mount path | Secret injection |
| `ALERT_*` | Slack, PagerDuty, email endpoints | Notification routing |

For production performance tuning, see [`docs/optimization.md`](./docs/optimization.md).

---

## Project Structure

```
mlops-platform/
├── src/
│   ├── api/
│   │   ├── app.py                      # FastAPI application factory
│   │   ├── routes/
│   │   │   ├── monitoring.py           # /api/v1/monitoring/*
│   │   │   ├── remediation.py          # /api/v1/remediation/*
│   │   │   └── analysis.py             # /api/v1/analysis/*
│   │   └── middleware/
│   │       ├── auth.py                 # JWT validation
│   │       └── logging.py              # Structured request logging
│   │
│   ├── core/
│   │   ├── config.py                   # Pydantic settings model
│   │   ├── database.py                 # SQLAlchemy async engine
│   │   └── security.py                 # Token issuance and validation
│   │
│   ├── services/
│   │   ├── monitoring/
│   │   │   ├── anomaly_detector.py     # Isolation Forest + Prophet
│   │   │   ├── drift_detector.py       # KS / PSI / ADWIN / DDM
│   │   │   └── performance_tracker.py  # Accuracy, latency, error rate
│   │   ├── remediation/
│   │   │   ├── engine.py               # Action executor with retry logic
│   │   │   ├── actions.py              # Action registry and implementations
│   │   │   └── safety_controller.py    # Pre-flight checks and rollback
│   │   ├── analysis/
│   │   │   ├── rca_engine.py           # Orchestrates RCA pipeline
│   │   │   ├── correlation_analyzer.py # Granger causality + Pearson
│   │   │   └── pattern_detector.py     # Temporal recurring issue detection
│   │   └── alerting/
│   │       ├── manager.py              # Routes alerts by severity
│   │       ├── notifier.py             # Slack / PagerDuty / email
│   │       └── prioritizer.py          # Severity scoring
│   │
│   ├── ml/
│   │   ├── pipelines/
│   │   │   ├── training_pipeline.py    # Full model training flow
│   │   │   ├── inference_pipeline.py   # Batch and real-time inference
│   │   │   └── preprocessing.py        # Feature transforms
│   │   ├── feature_store/
│   │   │   ├── manager.py              # Feast integration
│   │   │   └── online_store.py         # Low-latency feature retrieval
│   │   └── model_registry/
│   │       ├── manager.py              # MLflow versioning wrapper
│   │       └── deployment.py           # Canary and blue/green deploy
│   │
│   ├── models/
│   │   ├── schemas.py                  # Pydantic request/response models
│   │   └── database_models.py          # SQLAlchemy ORM models
│   │
│   └── utils/
│       ├── logging_utils.py            # Structured logging setup
│       ├── metrics_utils.py            # Prometheus counter/histogram helpers
│       └── data_processor.py           # Data validation and cleaning
│
├── tests/
│   ├── unit/                           # Isolated service tests
│   ├── integration/                    # End-to-end API workflow tests
│   ├── performance/                    # Locust load tests
│   └── fixtures/                       # Shared test data factories
│
├── infrastructure/
│   ├── terraform/                      # Cloud resource definitions (AWS/GCP/Azure)
│   ├── kubernetes/                     # Raw K8s manifests
│   └── helm-charts/                    # Packaged Helm releases
│       ├── mlops-monitoring/
│       ├── mlops-pipelines/
│       └── mlops-remediation/
│
├── monitoring/
│   ├── prometheus.yml
│   ├── alert_rules.yml                 # Severity-tiered alerting rules
│   └── grafana_dashboards/             # Executive, Ops, ML, Incident views
│
├── database/
│   ├── migrations/                     # Alembic migration history
│   └── seeds/                          # Development seed data
│
├── scripts/
│   ├── generate_sample_data.py         # Test data with optional drift injection
│   ├── deploy_cluster.sh
│   └── backup_restore.sh
│
├── docs/
│   ├── architecture.md
│   ├── api_reference.md
│   ├── deployment_guide.md
│   ├── optimization.md
│   └── troubleshooting.md
│
├── deployments/
│   ├── Dockerfile                      # Multi-stage build
│   ├── docker-compose.yml              # Full local stack
│   └── helm-values.yaml
│
├── Makefile                            # All operational commands
├── .env.example
├── requirements.txt
├── requirements-dev.txt
└── pyproject.toml
```

---

## Core Components

### Monitoring Service

The drift detector runs five algorithms in parallel per check. Results are aggregated with a configurable quorum threshold before an incident is raised.

```python
# Configure per-model detection behavior
detection_config = {
    "model_id": "fraud-detector-v1",
    "algorithms": ["ks_test", "psi", "adwin"],   # subset or "all"
    "quorum": 2,                                  # algorithms that must agree
    "window_size": "1h",
    "sensitivity": "medium",                       # low | medium | high
    "features": ["amount", "location", "device_id"]
}
```

Anomaly detection runs separately on system signals (latency, error rates, prediction distribution variance) using Isolation Forest for multivariate outliers and Prophet for time-series seasonality decomposition.

### Remediation Engine

No action executes without passing the safety controller. The controller checks: resource headroom, cooldown period since last action, version availability for rollbacks, and performance impact simulation for threshold changes.

```python
# Safety policy — applied to all remediations for a model
safety_policy = {
    "max_retries": 3,
    "cooldown_period": "5m",          # minimum gap between remediation attempts
    "resource_limits": {
        "cpu": "4 cores",
        "memory": "8Gi"
    },
    "rollback_triggers": {
        "error_rate_increase": 0.20,   # rollback if error rate rises 20%+
        "latency_increase": 0.50       # rollback if p95 latency rises 50%+
    }
}
```

### Root Cause Analysis Engine

RCA correlates drift signals, system metrics, and external event logs. The output is a confidence-scored report with contributing factors ranked by weight.

```json
{
  "incident_id": "inc-2024-03-16-001",
  "root_cause": {
    "type": "data_drift",
    "feature": "transaction_amount",
    "confidence": 0.92,
    "contributing_factors": [
      { "factor": "end_of_month_seasonality", "weight": 0.6 },
      { "factor": "new_user_cohort_onboarding", "weight": 0.3 }
    ]
  },
  "timeline": {
    "drift_start": "2024-03-15T10:00:00Z",
    "detection_time": "2024-03-15T10:02:30Z",
    "remediation_time": "2024-03-15T10:05:00Z"
  },
  "recommendations": [
    "Expand training window to include prior month-end data",
    "Add temporal features (day_of_month, week_of_month) to capture seasonality"
  ]
}
```

---

## API Reference

All endpoints require `Authorization: Bearer <token>`. Full documentation with request/response schemas is at `/docs` (Swagger) or `/redoc`.

### Monitoring

```http
POST   /api/v1/monitoring/drift-check          Run drift detection on a model
GET    /api/v1/monitoring/models/:id/health    Current model health summary
GET    /api/v1/monitoring/incidents            List active incidents
```

### Remediation

```http
POST   /api/v1/remediation/execute             Trigger a remediation action
GET    /api/v1/remediation/:id                 Check remediation status
DELETE /api/v1/remediation/:id                 Abort in-progress remediation
```

### Analysis

```http
GET    /api/v1/analysis/rca/:incident_id       Retrieve RCA report for incident
GET    /api/v1/analysis/patterns               List recurring issue patterns
POST   /api/v1/analysis/correlate              Ad-hoc signal correlation query
```

### System

```http
GET    /health                                 System health check
GET    /metrics                                Prometheus metrics endpoint
GET    /api/v1/models                          Registered model index
```

### Python SDK

```python
from mlops_sdk import MLOpsClient

client = MLOpsClient(api_key="your-api-key", environment="production")

# Detect drift and remediate automatically
result = client.monitor.detect_drift(
    model_id="fraud-detector-v1",
    reference_data="s3://bucket/reference.parquet"
)

if result.drift_detected:
    # Fetch the RCA before acting
    rca = client.analysis.get_rca(result.incident_id)
    print(f"Root cause: {rca.root_cause.type} (confidence: {rca.root_cause.confidence})")

    # Execute remediation with safety checks enabled
    remediation = client.remediation.execute(
        incident_id=result.incident_id,
        action="retrain_model",
        safety_check=True
    )
    print(f"Remediation {remediation.id}: {remediation.status}")
```

---

## Deployment

### Option A: Local (Docker Compose)

```bash
make up          # Full stack
make dev         # With hot reload
make reset       # Tear down and rebuild volumes
```

Suitable for development and single-node testing up to moderate event volumes.

### Option B: Kubernetes

```bash
# Apply all manifests
kubectl apply -f infrastructure/kubernetes/

# Or with Helm (recommended for production)
helm install mlops-platform ./infrastructure/helm-charts/mlops-platform \
  --namespace mlops-platform \
  --create-namespace \
  --values ./deployments/helm-values.yaml

# Scale the monitoring service
kubectl scale deployment monitoring-service \
  --replicas=5 \
  -n mlops-platform
```

The Helm chart includes HPA configuration out of the box:

```yaml
# HPA pre-configured in helm-values.yaml
autoscaling:
  monitoring:
    minReplicas: 3
    maxReplicas: 10
    targetCPUUtilizationPercentage: 70
    customMetric:
      name: drift_detection_queue
      target: 100      # max queue depth per pod before scaling
```

### Option C: Cloud (Terraform)

```bash
cd infrastructure/terraform

terraform init
terraform workspace select prod              # or dev / staging
terraform plan -var-file="environments/prod.tfvars"
terraform apply
```

Terraform modules are provided for AWS (EKS + RDS + ElastiCache + S3), GCP (GKE + Cloud SQL + Memorystore + GCS), and Azure (AKS + Flexible Postgres + Redis Cache + Blob Storage).

### Environment matrix

| Environment | Resources | SLA | Notes |
|-------------|-----------|-----|-------|
| Local | Docker Desktop | — | Full stack via Compose |
| Dev | 4 cores / 8 GB | 95% | Single-replica services |
| Staging | 8 cores / 16 GB | 99% | Production config, reduced data |
| Production | Auto-scaling | 99.9% | Min 3 replicas, HPA enabled |

---

## Observability

### Key Metrics

| Category | Metric | Alert Threshold |
|----------|--------|----------------|
| Drift | `drift_score` | > 0.3 |
| Anomalies | `anomaly_count_per_min` | > 5 |
| Performance | `model_accuracy` | < 0.85 |
| Latency | `api_latency_p95_ms` | > 200 |
| Reliability | `remediation_success_rate` | < 90% |

### Grafana Dashboards

Four pre-built dashboards ship in `monitoring/grafana_dashboards/`:

- **Executive** — system-wide health, SLA status, cost metrics
- **Operations** — service health, resource utilization, active alerts
- **ML Performance** — drift trends, feature importance changes, model accuracy over time
- **Incident** — active incidents, RCA summaries, remediation history

### Alerting Tiers

```yaml
# monitoring/alert_rules.yml

- name: SevereDriftDetected
  condition: drift_score > 0.8
  severity: critical
  channel: pagerduty        # immediate page

- name: PerformanceDegradation
  condition: model_accuracy < 0.9 for 15m
  severity: warning
  channel: slack            # review within 4 hours

- name: RemediationCompleted
  condition: remediation_success == true
  severity: info
  channel: email            # FYI, no action required
```

---

## Security

All service-to-service communication is mTLS via Istio. External API access uses short-lived JWTs (RSA256). API keys use HMAC signatures for service-to-service calls.

Secrets are injected from HashiCorp Vault at pod startup via the Vault Agent Injector. No secrets are present in environment variables, config maps, or container images in production.

```bash
# Store secrets (run once per environment)
vault kv put secret/mlops-platform \
  DATABASE_URL="postgresql://..." \
  REDIS_PASSWORD="..." \
  MLFLOW_TRACKING_URI="..."
```

Network policies restrict pod-to-pod traffic to explicitly declared paths. The monitoring service can reach storage. The API can reach monitoring and analysis. Nothing can reach the API from outside the frontend namespace.

Audit logs are written by Fluentd to an S3 bucket with Object Lock enabled — logs cannot be modified or deleted for the retention period. This satisfies SOC2 CC7.2 and CC7.3 requirements.

To report a vulnerability, email **security@your-org.com** — do not open a public issue.

---

## Troubleshooting

| Issue | Symptom | Resolution |
|-------|---------|------------|
| False positive drift alerts | High alert volume on stable model | Lower quorum requirement or widen `window_size` in detection config |
| Remediation stuck `in_progress` | Status doesn't advance after 10m | Check worker logs; verify IAM/resource permissions for the action type |
| API p95 latency > 500ms | Slow responses under load | Scale monitoring service; check database connection pool exhaustion |
| Missing features in drift check | `KeyError` in drift response | Run data validation step; check feature store for schema changes |

```bash
# Enable debug logging without restart
kubectl set env deployment/monitoring-service LOG_LEVEL=DEBUG -n mlops-platform

# Tail logs across all pods
kubectl logs -f -l app=monitoring-service -n mlops-platform --tail=100

# Health check (all components)
curl https://your-api-host/health | jq .
```

---

## Roadmap

**Q4 2024**
- [ ] AutoML integration — trigger automated HPO on retrain
- [ ] Explainable AI features — SHAP-based drift attribution in RCA reports
- [ ] Compliance reporting — SOC2 evidence export, automated audit prep

**Q1 2025**
- [ ] Real-time feature store — sub-10ms feature serving with Redis-backed Feast
- [ ] A/B testing framework — statistical significance tracking for remediation strategies
- [ ] Cost optimization engine — per-model compute spend tracking and right-sizing recommendations

**Completed**
- [x] Core drift detection (KS, PSI, ADWIN, DDM, Page-Hinkley)
- [x] Automated remediation engine with safety controller
- [x] Root cause analysis with causal inference
- [x] Grafana dashboards and Prometheus integration
- [x] Multi-cloud Terraform modules (AWS, GCP, Azure)

---

## Contributing

See [CONTRIBUTING.md](./CONTRIBUTING.md) for the full guide. The short version:

```bash
# Fork, then clone your fork
git clone https://github.com/your-username/mlops-platform.git

# Create a feature branch
git checkout -b feat/your-feature

# Install dev dependencies
make install-dev

# Run the test suite before pushing
make test
make lint
```

All PRs require passing CI, one approving review from a maintainer, and updated documentation for any user-facing changes. Commit messages follow [Conventional Commits](https://www.conventionalcommits.org/).

---

## License

MIT © 2024 Your Organization — see [LICENSE](./LICENSE) for full terms.

---

<div align="center">

Built for ML teams who are tired of finding out about model failures from their users.

[⬆ Back to top](#the-problem)

</div>
