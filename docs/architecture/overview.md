# Architecture Overview

This document describes the high-level architecture of the system, the major components, and how they interact.

## Contents

- [System Context](#system-context)
- [Container Architecture](#container-architecture)
- [Component Breakdown](#component-breakdown)
- [See Also](#see-also)

---

## System Context

The system follows a **microservices** pattern with an API Gateway acting as the single entry point for all client traffic.

```mermaid
C4Context
    title System Context Diagram

    Person(user, "End User", "A person using the web or mobile app")
    Person(admin, "Administrator", "Manages system configuration and users")

    System(system, "Example Platform", "Provides core business capabilities via REST APIs and a web UI")

    System_Ext(email, "Email Service", "SendGrid – transactional emails")
    System_Ext(payment, "Payment Gateway", "Stripe – handles billing")
    System_Ext(storage, "Object Storage", "AWS S3 – stores files and assets")

    Rel(user, system, "Uses", "HTTPS")
    Rel(admin, system, "Manages", "HTTPS")
    Rel(system, email, "Sends emails via", "SMTP / API")
    Rel(system, payment, "Processes payments via", "REST / Webhook")
    Rel(system, storage, "Stores/retrieves files via", "S3 API")
```

---

## Container Architecture

```mermaid
graph LR
    subgraph "Client Tier"
        WebApp["Web App\n(React SPA)"]
        MobileApp["Mobile App\n(React Native)"]
    end

    subgraph "API Tier"
        Gateway["API Gateway\n(Nginx / Kong)"]
        AuthSvc["Auth Service\n(Node.js)"]
        CoreSvc["Core API\n(Node.js / Express)"]
        NotifSvc["Notification Service\n(Node.js)"]
    end

    subgraph "Data Tier"
        Postgres[("PostgreSQL\n(primary data)")]
        Redis[("Redis\n(sessions / cache)")]
        S3[("S3\n(file storage)")]
    end

    subgraph "Async Tier"
        Queue["Message Broker\n(RabbitMQ)"]
        Worker["Worker Service\n(Node.js)"]
    end

    WebApp & MobileApp -->|HTTPS| Gateway
    Gateway --> AuthSvc & CoreSvc
    CoreSvc --> Postgres & Redis & S3
    CoreSvc -->|publish events| Queue
    Queue --> Worker
    Worker --> NotifSvc
    NotifSvc --> Postgres
```

---

## Component Breakdown

See [components.md](./components.md) for detailed descriptions of each service, and [data-flow.md](./data-flow.md) for end-to-end data-flow diagrams.

---

## See Also

- [API Reference](../api/overview.md)
- [Getting Started Guide](../guides/getting-started.md)
- [Development Setup](../development/setup.md)
