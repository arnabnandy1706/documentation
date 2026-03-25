# documentation

This repository contains sample Markdown documentation with Mermaid architecture and flow diagrams.

## Documentation Structure

```
docs/
├── index.md                      ← Documentation home & quick-start
│
├── architecture/
│   ├── overview.md               ← System context & container diagrams
│   ├── components.md             ← Per-service component descriptions
│   └── data-flow.md              ← End-to-end data-flow diagrams
│
├── api/
│   ├── overview.md               ← API conventions & response shapes
│   ├── authentication.md         ← JWT auth flow & RBAC
│   └── endpoints.md              ← Full endpoint reference
│
├── guides/
│   ├── getting-started.md        ← Run the platform locally in minutes
│   ├── installation.md           ← Docker & Kubernetes production deploy
│   └── configuration.md          ← Environment variable reference
│
└── development/
    ├── setup.md                  ← Dev environment setup & scripts
    ├── contributing.md           ← PR workflow & branching strategy
    └── testing.md                ← Testing pyramid, examples & coverage
```

## Mermaid Diagrams Included

Every page contains at least one [Mermaid](https://mermaid.js.org) diagram — rendered natively on GitHub. Types used across the docs:

| Diagram type | Example location |
|---|---|
| `graph` / `flowchart` | [Architecture Overview](./docs/architecture/overview.md) |
| `sequenceDiagram` | [Authentication](./docs/api/authentication.md) |
| `classDiagram` | [Component Descriptions](./docs/architecture/components.md) |
| `stateDiagram-v2` | [API Endpoints](./docs/api/endpoints.md) |
| `gitGraph` | [Contributing Guide](./docs/development/contributing.md) |
| `C4Context` | [Architecture Overview](./docs/architecture/overview.md) |

## Quick Start

```bash
git clone https://github.com/example/project.git
cd project
npm install
docker compose up -d
npm run dev
```

See [Getting Started](./docs/guides/getting-started.md) for the full walkthrough.
