# Configuration Reference

All services are configured via **environment variables**. This page lists every variable, its purpose, and its default value.

## Loading Order

```mermaid
flowchart LR
    OS["OS environment\nvariables"] -->|highest priority| Merged["Merged\nconfig object"]
    DotEnv[".env file\n(local dev only)"] --> Merged
    Defaults["Code defaults\n(lowest priority)"] --> Merged
    Merged --> App["Application"]
```

## Core API (`services/core`)

| Variable | Required | Default | Description |
|----------|----------|---------|-------------|
| `NODE_ENV` | No | `development` | `development`, `test`, or `production` |
| `PORT` | No | `3002` | Port the service listens on |
| `DATABASE_URL` | **Yes** | – | PostgreSQL connection string |
| `REDIS_URL` | **Yes** | – | Redis connection string |
| `RABBITMQ_URL` | **Yes** | – | RabbitMQ AMQP URL |
| `JWT_SECRET` | **Yes** | – | Secret for verifying access tokens |
| `LOG_LEVEL` | No | `info` | `error`, `warn`, `info`, `debug` |
| `CORS_ORIGINS` | No | `*` | Comma-separated list of allowed origins |
| `RATE_LIMIT_WINDOW_MS` | No | `60000` | Rate-limit window in milliseconds |
| `RATE_LIMIT_MAX` | No | `100` | Max requests per window per IP |

## Auth Service (`services/auth`)

| Variable | Required | Default | Description |
|----------|----------|---------|-------------|
| `PORT` | No | `3001` | Port the service listens on |
| `DATABASE_URL` | **Yes** | – | PostgreSQL connection string |
| `REDIS_URL` | **Yes** | – | Redis connection string |
| `JWT_SECRET` | **Yes** | – | RS256 private key (PEM) |
| `JWT_PUBLIC_KEY` | **Yes** | – | RS256 public key (PEM) |
| `JWT_EXPIRES_IN` | No | `15m` | Access token TTL (ms or zeit/ms string) |
| `REFRESH_TOKEN_SECRET` | **Yes** | – | Secret for refresh tokens |
| `REFRESH_TOKEN_EXPIRES_IN` | No | `7d` | Refresh token TTL |
| `BCRYPT_COST` | No | `12` | bcrypt cost factor |
| `MAX_LOGIN_ATTEMPTS` | No | `5` | Consecutive failures before lock-out |
| `LOGIN_LOCKOUT_MINUTES` | No | `15` | Lock-out duration |

## Notification Service (`services/notif`)

| Variable | Required | Default | Description |
|----------|----------|---------|-------------|
| `PORT` | No | `3003` | Port the service listens on |
| `RABBITMQ_URL` | **Yes** | – | RabbitMQ AMQP URL |
| `SENDGRID_API_KEY` | **Yes** | – | SendGrid API key |
| `EMAIL_FROM` | **Yes** | – | Sender email address |
| `EMAIL_FROM_NAME` | No | `Example Platform` | Sender display name |

## Worker Service (`services/worker`)

| Variable | Required | Default | Description |
|----------|----------|---------|-------------|
| `RABBITMQ_URL` | **Yes** | – | RabbitMQ AMQP URL |
| `DATABASE_URL` | **Yes** | – | PostgreSQL connection string |
| `REDIS_URL` | **Yes** | – | Redis connection string |
| `WORKER_CONCURRENCY` | No | `5` | Number of jobs processed in parallel |

## Example `.env` File

```dotenv
# ── Core API ──────────────────────────────────────────────────────────────────
NODE_ENV=development
PORT=3002
DATABASE_URL=postgresql://postgres:password@localhost:5432/app
REDIS_URL=redis://localhost:6379
RABBITMQ_URL=amqp://guest:guest@localhost:5672

# ── Auth Service ──────────────────────────────────────────────────────────────
JWT_SECRET=replace-me-with-a-long-random-string
JWT_EXPIRES_IN=15m
REFRESH_TOKEN_SECRET=replace-me-with-another-long-random-string
REFRESH_TOKEN_EXPIRES_IN=7d
BCRYPT_COST=12

# ── Notification Service ──────────────────────────────────────────────────────
SENDGRID_API_KEY=SG.xxxx
EMAIL_FROM=noreply@example.com
EMAIL_FROM_NAME=Example Platform

# ── Feature Flags ─────────────────────────────────────────────────────────────
FEATURE_NEW_CHECKOUT=false
FEATURE_DARK_MODE=true
```

## Feature Flags

Feature flags are environment variables prefixed with `FEATURE_`. The application reads them at start-up:

```typescript
// src/config/features.ts
export const features = {
  newCheckout: process.env.FEATURE_NEW_CHECKOUT === 'true',
  darkMode:    process.env.FEATURE_DARK_MODE    === 'true',
} as const;
```

```mermaid
flowchart TD
    Start["Request received"] --> Check{"Feature flag\nenabled?"}
    Check -- "FEATURE_NEW_CHECKOUT=true" --> NewFlow["New Checkout\nexperience"]
    Check -- "FEATURE_NEW_CHECKOUT=false" --> OldFlow["Legacy Checkout\nexperience"]
    NewFlow & OldFlow --> Response["Return response"]
```

## Generating Secrets

```bash
# Generate a secure random 64-byte base64 string
openssl rand -base64 64

# Generate RS256 key pair for JWT
openssl genrsa -out private.pem 4096
openssl rsa -in private.pem -pubout -out public.pem
```

> ⚠️ **Never commit secrets to version control.** Use a secrets manager (AWS Secrets Manager, HashiCorp Vault, or GitHub Secrets) in production.
