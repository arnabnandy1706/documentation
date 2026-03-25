---
layout: default
title: Guides
nav_order: 8
has_children: true
---

# Getting Started

This guide walks you through running the platform locally for the first time.

## Prerequisites

| Tool | Minimum Version | Install Guide |
|------|-----------------|---------------|
| Node.js | 20 LTS | https://nodejs.org |
| Docker & Docker Compose | 24 | https://docs.docker.com/get-docker |
| Git | 2.40 | https://git-scm.com |

## 1 · Clone the Repository

```bash
git clone https://github.com/example/project.git
cd project
```

## 2 · Start Infrastructure Services

Docker Compose brings up PostgreSQL, Redis, and RabbitMQ:

```bash
docker compose up -d postgres redis rabbitmq
```

Verify that all containers are healthy:

```bash
docker compose ps
```

Expected output:

```
NAME         STATUS          PORTS
postgres     running (healthy)   0.0.0.0:5432->5432/tcp
redis        running (healthy)   0.0.0.0:6379->6379/tcp
rabbitmq     running (healthy)   0.0.0.0:5672->5672/tcp
```

## 3 · Install Node Dependencies

```bash
npm install
```

## 4 · Configure Environment Variables

```bash
cp .env.example .env
```

Edit `.env` and fill in the required values (see [Configuration](./configuration.md) for full reference):

```dotenv
# Database
DATABASE_URL=postgresql://postgres:password@localhost:5432/app

# Redis
REDIS_URL=redis://localhost:6379

# JWT secrets (generate with: openssl rand -base64 64)
JWT_SECRET=<your-secret>
JWT_REFRESH_SECRET=<your-refresh-secret>

# RabbitMQ
RABBITMQ_URL=amqp://guest:guest@localhost:5672
```

## 5 · Run Database Migrations

```bash
npm run db:migrate
```

Seed the database with sample data (optional):

```bash
npm run db:seed
```

## 6 · Start All Services

```bash
npm run dev
```

This starts all microservices with hot-reload using `concurrently`:

```
[auth-service]  ✓  Auth Service listening on :3001
[core-api]      ✓  Core API listening on :3002
[notif-service] ✓  Notification Service listening on :3003
[worker]        ✓  Worker connected to RabbitMQ
```

## 7 · Verify the Setup

Send a test request to confirm everything is working:

```bash
curl -s http://localhost:3001/health | jq .
```

Expected response:

```json
{
  "status": "ok",
  "services": {
    "database": "ok",
    "redis": "ok",
    "queue": "ok"
  }
}
```

## Start-up Flow

```mermaid
flowchart TD
    A([Developer]) --> B["git clone"]
    B --> C["docker compose up -d"]
    C --> D["npm install"]
    D --> E["cp .env.example .env"]
    E --> F["npm run db:migrate"]
    F --> G["npm run db:seed\n(optional)"]
    G --> H["npm run dev"]
    H --> I{All services\nhealthy?}
    I -- Yes --> J["✅ Platform ready\nlocalhost:3002"]
    I -- No --> K["Check logs:\nnpm run logs"]
    K --> L["Consult Troubleshooting\nsection below"]
```

## Troubleshooting

| Symptom | Likely Cause | Fix |
|---------|-------------|-----|
| `ECONNREFUSED 5432` | PostgreSQL not running | `docker compose up postgres` |
| `Error: JWT_SECRET not set` | Missing `.env` | Copy `.env.example` and fill values |
| `Migration already applied` | DB already seeded | Safe to ignore; run `npm run db:migrate:status` |
| Port 3002 in use | Another process | `lsof -i :3002` then kill the process |

## Next Steps

- Read the [Installation Guide](./installation.md) for production deployment
- See [Configuration](./configuration.md) for all environment variables
- Explore the [API Reference](../api/overview.md) to start making requests
