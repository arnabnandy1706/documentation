---
layout: default
title: Components
parent: Architecture
nav_order: 1
---

# Component Descriptions

This page provides a detailed breakdown of every service in the platform.

## Contents

- [API Gateway](#api-gateway)
- [Auth Service](#auth-service)
- [Core API](#core-api)
- [Notification Service](#notification-service)
- [Worker Service](#worker-service)

---

## API Gateway

| Property | Value |
|----------|-------|
| Technology | Nginx + Kong |
| Port | 443 (HTTPS) |
| Responsibility | TLS termination, rate limiting, request routing |

The gateway is the **only** publicly exposed entry point. All traffic is routed through it before reaching internal services.

```mermaid
graph LR
    Internet -->|HTTPS :443| Gateway
    Gateway -->|/auth/**| AuthSvc
    Gateway -->|/api/**| CoreSvc
    Gateway -->|/notify/**| NotifSvc
```

### Rate Limiting Rules

| Tier | Requests / minute |
|------|-------------------|
| Anonymous | 30 |
| Free plan | 120 |
| Pro plan | 600 |
| Enterprise | Unlimited |

---

## Auth Service

| Property | Value |
|----------|-------|
| Technology | Node.js 20, Express 4 |
| Port | 3001 (internal) |
| Responsibility | Registration, login, JWT issuance, token refresh |

```mermaid
sequenceDiagram
    participant C as Client
    participant G as API Gateway
    participant A as Auth Service
    participant DB as PostgreSQL
    participant R as Redis

    C->>G: POST /auth/login {email, password}
    G->>A: forward request
    A->>DB: SELECT user WHERE email = ?
    DB-->>A: user record
    A->>A: bcrypt.compare(password, hash)
    A->>R: SET session:{userId} TTL 3600
    A-->>G: 200 OK {accessToken, refreshToken}
    G-->>C: 200 OK {accessToken, refreshToken}
```

### Key Files

```
src/
  auth-service/
    controllers/
      auth.controller.ts   # Route handlers
    services/
      token.service.ts     # JWT sign / verify
      password.service.ts  # bcrypt helpers
    middleware/
      validate.ts          # Joi request validation
    routes/
      auth.routes.ts       # Express router
```

---

## Core API

| Property | Value |
|----------|-------|
| Technology | Node.js 20, Express 4, TypeORM |
| Port | 3002 (internal) |
| Responsibility | Business logic, CRUD operations, event publishing |

```mermaid
classDiagram
    class UserController {
        +getUsers(req, res)
        +getUser(req, res)
        +updateUser(req, res)
        +deleteUser(req, res)
    }

    class UserService {
        -userRepo: UserRepository
        +findAll() Promise~User[]~
        +findById(id) Promise~User~
        +update(id, dto) Promise~User~
        +remove(id) Promise~void~
    }

    class UserRepository {
        +find(options) Promise~User[]~
        +findOne(options) Promise~User~
        +save(entity) Promise~User~
        +delete(id) Promise~void~
    }

    class User {
        +id: string
        +email: string
        +name: string
        +role: UserRole
        +createdAt: Date
    }

    UserController --> UserService
    UserService --> UserRepository
    UserRepository --> User
```

---

## Notification Service

| Property | Value |
|----------|-------|
| Technology | Node.js 20, Nodemailer, SendGrid SDK |
| Port | 3003 (internal) |
| Responsibility | Email and push notifications triggered by worker events |

```mermaid
stateDiagram-v2
    [*] --> Received : event arrives via queue
    Received --> Templating : load template
    Templating --> Rendering : inject variables
    Rendering --> Sending : call provider API
    Sending --> Success : 2xx response
    Sending --> Retry : 4xx / 5xx response
    Retry --> Sending : attempt ≤ 3
    Retry --> DeadLetter : attempt > 3
    Success --> [*]
    DeadLetter --> [*]
```

---

## Worker Service

| Property | Value |
|----------|-------|
| Technology | Node.js 20, BullMQ |
| Port | None (outbound only) |
| Responsibility | Consume queue messages, run background jobs |

```mermaid
graph TD
    Q[("RabbitMQ\nMessage Queue")]
    W["Worker Service"]
    J1["Job: send-email"]
    J2["Job: generate-report"]
    J3["Job: cleanup-sessions"]
    NS["Notification Service"]
    DB[("PostgreSQL")]
    S3[("S3")]

    Q -->|consume| W
    W --> J1 & J2 & J3
    J1 --> NS
    J2 --> S3 & DB
    J3 --> DB
```
