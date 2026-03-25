# Data Flow Diagrams

This page traces the end-to-end data flow for the platform's most important operations.

## Contents

- [User Registration Flow](#user-registration-flow)
- [Authenticated API Request Flow](#authenticated-api-request-flow)
- [File Upload Flow](#file-upload-flow)
- [Background Job Flow](#background-job-flow)

---

## User Registration Flow

```mermaid
flowchart TD
    A([Start]) --> B[/Client sends POST /auth/register/]
    B --> C{Request valid?}
    C -- No --> D[Return 400 Bad Request]
    C -- Yes --> E{Email already exists?}
    E -- Yes --> F[Return 409 Conflict]
    E -- No --> G[Hash password with bcrypt]
    G --> H[Save user to PostgreSQL]
    H --> I[Publish user.created event to queue]
    I --> J[Worker picks up event]
    J --> K[Notification Service sends welcome email]
    K --> L[Return 201 Created with JWT tokens]
    L --> Z([End])
    D --> Z
    F --> Z
```

---

## Authenticated API Request Flow

```mermaid
flowchart LR
    Client -->|"1. HTTPS request\n+ Bearer token"| Gateway
    Gateway -->|"2. Forward token"| AuthSvc["Auth Service"]
    AuthSvc -->|"3. Verify JWT signature"| AuthSvc
    AuthSvc -->|"4. Check token in Redis\n(not revoked?)"| Redis[("Redis")]
    Redis -->|"5. Token valid"| AuthSvc
    AuthSvc -->|"6. Attach user context"| Gateway
    Gateway -->|"7. Forward request\n+ user context"| Core["Core API"]
    Core -->|"8. Execute business logic"| Postgres[("PostgreSQL")]
    Postgres -->|"9. Return data"| Core
    Core -->|"10. JSON response"| Gateway
    Gateway -->|"11. HTTPS response"| Client
```

---

## File Upload Flow

```mermaid
sequenceDiagram
    participant C as Client
    participant G as API Gateway
    participant A as Core API
    participant S as S3
    participant DB as PostgreSQL

    C->>G: POST /api/files (multipart/form-data)
    G->>A: authenticated request + file
    A->>A: validate file type & size
    A->>S: PutObject (file bytes)
    S-->>A: ETag + S3 key
    A->>DB: INSERT file record {key, url, ownerId}
    DB-->>A: saved record
    A-->>G: 201 Created {fileId, url}
    G-->>C: 201 Created {fileId, url}
```

---

## Background Job Flow

```mermaid
flowchart TB
    subgraph "Trigger"
        API["Core API"]
    end

    subgraph "Async Layer"
        Q[("RabbitMQ\nExchange")]
        DLQ[("Dead-Letter\nQueue")]
    end

    subgraph "Worker"
        W["Worker Service"]
        R["Retry Logic\n(max 3 attempts)"]
    end

    subgraph "Side Effects"
        N["Notification Service"]
        DB[("PostgreSQL")]
        S3[("S3")]
    end

    API -->|publish event| Q
    Q -->|consume| W
    W -->|success| N & DB & S3
    W -->|failure| R
    R -->|retry| W
    R -->|exhausted| DLQ
    DLQ -->|alert on-call| N
```

---

## Data Storage Summary

| Store | Type | Primary Use |
|-------|------|-------------|
| PostgreSQL | Relational RDBMS | Users, orders, products, audit logs |
| Redis | Key-value / cache | Session tokens, rate-limit counters, short-lived cache |
| S3 | Object storage | User-uploaded files, generated reports, static assets |
| RabbitMQ | Message broker | Async event bus between services |
