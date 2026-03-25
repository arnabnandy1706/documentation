# Installation Guide

This guide covers production-grade deployment using Docker, Kubernetes, and CI/CD pipelines.

## Contents

- [Docker Deployment](#docker-deployment)
- [Kubernetes Deployment](#kubernetes-deployment)
- [CI/CD Pipeline](#cicd-pipeline)
- [Health Checks](#health-checks)

---

## Docker Deployment

### Build the Images

```bash
# Build all service images
docker build -t example/auth-service:latest   -f services/auth/Dockerfile   .
docker build -t example/core-api:latest       -f services/core/Dockerfile   .
docker build -t example/notif-service:latest  -f services/notif/Dockerfile  .
docker build -t example/worker:latest         -f services/worker/Dockerfile .
```

### Run with Docker Compose (production profile)

```yaml
# docker-compose.prod.yml (excerpt)
services:
  core-api:
    image: example/core-api:latest
    restart: always
    environment:
      NODE_ENV: production
      DATABASE_URL: ${DATABASE_URL}
      REDIS_URL: ${REDIS_URL}
    ports:
      - "3002:3002"
    depends_on:
      postgres:
        condition: service_healthy
      redis:
        condition: service_healthy
```

```bash
docker compose -f docker-compose.prod.yml up -d
```

---

## Kubernetes Deployment

### Architecture

```mermaid
graph TB
    subgraph "Kubernetes Cluster"
        subgraph "Ingress"
            Ingress["Nginx Ingress\nController"]
        end

        subgraph "Application Namespace"
            AuthDeploy["auth-service\nDeployment (2 replicas)"]
            CoreDeploy["core-api\nDeployment (3 replicas)"]
            NotifDeploy["notif-service\nDeployment (2 replicas)"]
            WorkerDeploy["worker\nDeployment (2 replicas)"]
        end

        subgraph "Data Namespace"
            PGStateful["PostgreSQL\nStatefulSet"]
            RedisStateful["Redis\nStatefulSet"]
        end

        subgraph "Messaging Namespace"
            RMQStateful["RabbitMQ\nStatefulSet"]
        end

        ConfigMap["ConfigMap\n& Secrets"]
        HPA["HorizontalPodAutoscaler"]
    end

    Internet --> Ingress
    Ingress --> AuthDeploy & CoreDeploy
    CoreDeploy --> PGStateful & RedisStateful & RMQStateful
    RMQStateful --> WorkerDeploy
    WorkerDeploy --> NotifDeploy
    HPA -.monitors.-> CoreDeploy & WorkerDeploy
    ConfigMap -.injects.-> AuthDeploy & CoreDeploy & NotifDeploy & WorkerDeploy
```

### Apply Manifests

```bash
# Create namespace
kubectl apply -f k8s/namespace.yaml

# Apply secrets (use Sealed Secrets or External Secrets in production)
kubectl apply -f k8s/secrets.yaml

# Deploy services
kubectl apply -f k8s/deployments/
kubectl apply -f k8s/services/
kubectl apply -f k8s/ingress.yaml

# Verify rollout
kubectl rollout status deployment/core-api -n app
```

### Horizontal Pod Autoscaler

```yaml
# k8s/hpa/core-api-hpa.yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: core-api-hpa
  namespace: app
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: core-api
  minReplicas: 2
  maxReplicas: 10
  metrics:
    - type: Resource
      resource:
        name: cpu
        target:
          type: Utilization
          averageUtilization: 70
```

---

## CI/CD Pipeline

```mermaid
flowchart LR
    PR["Pull Request\nopened"] --> Lint["Lint & Type-check"]
    Lint --> UnitTests["Unit Tests"]
    UnitTests --> IntTests["Integration Tests"]
    IntTests --> Build["Docker Build"]
    Build --> Scan["Vulnerability Scan\n(Trivy)"]
    Scan --> PushStaging["Push image\n→ staging registry"]
    PushStaging --> DeployStaging["Deploy to Staging\n(kubectl)"]
    DeployStaging --> SmokeTest["Smoke Tests\n(Playwright)"]
    SmokeTest --> Approve{"Manual\nApproval"}
    Approve -- Approved --> PushProd["Push image\n→ prod registry"]
    PushProd --> DeployProd["Deploy to Production\n(blue/green)"]
    DeployProd --> Monitor["Monitor\n(Datadog)"]
    Approve -- Rejected --> Notify["Notify team\n(Slack)"]
```

### GitHub Actions Workflow (excerpt)

```yaml
# .github/workflows/ci.yml
name: CI

on:
  pull_request:
    branches: [main]

jobs:
  test:
    runs-on: ubuntu-latest
    services:
      postgres:
        image: postgres:16
        env:
          POSTGRES_PASSWORD: password
        options: >-
          --health-cmd pg_isready
          --health-interval 10s
          --health-timeout 5s
          --health-retries 5

    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: 20
          cache: npm
      - run: npm ci
      - run: npm run lint
      - run: npm run test:unit
      - run: npm run test:integration
        env:
          DATABASE_URL: postgresql://postgres:password@localhost:5432/test
```

---

## Health Checks

Each service exposes a `GET /health` endpoint:

```mermaid
flowchart TD
    LB["Load Balancer"] -->|"GET /health"| Service["Service Instance"]
    Service --> DB{PostgreSQL\nreachable?}
    DB -- Yes --> Redis{Redis\nreachable?}
    DB -- No --> Unhealthy["503 Service Unavailable\n{status: degraded}"]
    Redis -- Yes --> Healthy["200 OK\n{status: ok}"]
    Redis -- No --> Unhealthy
```

**Sample health response:**

```json
{
  "status": "ok",
  "version": "1.4.2",
  "uptime": 86400,
  "checks": {
    "database": { "status": "ok", "latencyMs": 3 },
    "redis":    { "status": "ok", "latencyMs": 1 },
    "queue":    { "status": "ok", "latencyMs": 5 }
  }
}
```
