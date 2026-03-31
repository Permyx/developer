# Contributing to Permyx

We're building the execution authorization layer for AI commerce — the infrastructure that governs what AI agents are allowed to do before they act.

## What We're Building

Permyx sits between AI agents and merchant checkouts. We issue, validate, and log execution tokens so every machine-initiated transaction is visible, auditable, and policy-governed.

### Core Technical Challenges

- **Cryptographic execution tokens** — RS256-signed JWTs with intent binding, replay protection, and 120-second TTL. Tokens are burn-on-use with `jti` tracking.
- **Sub-second authorization decisions** — Policy evaluation at the speed of agent execution. Every authorize/validate cycle must complete in <100ms.
- **Multi-protocol gateway** — Supporting OpenAI ACP, Google UCP, Microsoft MCP, and Meta Commerce Graph as a governed intermediary.
- **Real-time merchant dashboards** — Next.js portal with live execution event streams, agent activity visualization, and policy configuration.
- **Distributed event architecture** — Azure Service Bus topics for execution event fanout, with Cosmos DB for immutable audit trails.
- **Commerce platform connectors** — Shopify OAuth, BigCommerce, WooCommerce ingestion pipelines that normalize catalog data for authorization rule evaluation.

## Tech Stack

| Layer | Technology |
|-------|------------|
| API | TypeScript, Hono, Node.js 20 |
| Portal | Next.js 14, React 18, Tailwind CSS |
| Background Jobs | Azure Functions (TypeScript) |
| Database | Azure Cosmos DB (NoSQL) |
| Messaging | Azure Service Bus |
| Auth | RS256 JWTs, OAuth 2.0 |
| Infrastructure | Azure (App Service, Static Web Apps, Key Vault, Application Insights) |
| IaC | Bicep |
| Monorepo | pnpm workspaces |

## Areas of Interest

We're particularly interested in engineers with experience in:

- **Security & cryptography** — Token design, JWT best practices, replay protection at scale
- **Distributed systems** — Event-driven architectures, exactly-once semantics, idempotency
- **Commerce platforms** — Shopify APIs, checkout extensibility, platform integration patterns
- **TypeScript/Node.js** — Production API design with Hono or similar frameworks
- **React/Next.js** — Complex dashboard UIs with real-time data
- **Azure infrastructure** — Cosmos DB, Service Bus, App Service, Bicep IaC
- **AI/ML platforms** — Understanding of how AI agents interact with commerce (OpenAI, Google, Microsoft ecosystems)

## Getting Started

1. Review the [architecture docs](docs/architecture.md) to understand the system
2. Read the [API reference](docs/execution-auth-api.md) to see what we've built
3. Check the [platform overview](docs/platform-overview.md) for business context
4. Try the sandbox API at `https://api.sandbox.permyx.io`

## Contact

- **Email:** developers@permyx.io
- **Website:** [permyx.io](https://permyx.io)
- **Beta Access:** [permyx.io/beta](https://permyx.io/beta)
