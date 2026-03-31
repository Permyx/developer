# Permyx Systems Architecture

> **Version:** 3.1  
> **Last Updated:** March 2026  
> **Status:** Deployed — Execution Authorization Beta (EAM V1)

---

## Core Philosophy

Permyx is the authorization layer that governs what AI agents are permitted to do — before they do it.

**Fundamental insight:** AI agents don't browse websites — they reason, decide, and act. The critical gap isn't data quality; it's the absence of a merchant-controlled authorization boundary between agent intent and commerce execution. Permyx is that boundary.

---

## The 5-Layer Control Model

The **Authorize** layer is the core of Permyx — everything else exists to make authorization possible and enforceable.

| Layer | Name | Purpose |
|-------|------|--------|
| **Ingest** | Data Collection | Shopify OAuth, WooCommerce, CSV, custom APIs |
| **Transform** | Normalization | Structured, machine-legible schema (prerequisite for authorization) |
| **Govern** | Policy Enforcement | Action policies, freshness, confidence scoring |
| **Authorize** | **Execution Authorization** | **JWT execution tokens, replay protection, intent binding — THE CORE** |
| **Distribute** | API Delivery | Partner APIs, platform integrations (ChatGPT, Perplexity) |
| **Audit** | Observability | Access logs, action tracking, anomaly detection |

---

## The Agent Journey

```
┌─────────────────────────────────────────────────────────────────┐
│  1. DISCOVERY PHASE                                             │
│     Agent: "Find wireless headphones under $200"                │
│                                                                 │
│     Shopify ──┐                                                 │
│     BigComm. ─┼──► Permyx ──► Authorization-governed catalog    │
│     Feeds ────┘    (policies, freshness, execution permissions) │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│  2. AUTHORIZATION PHASE                                         │
│     Agent requests execution authorization from Permyx:         │
│     • POST /agents/v1/exec/authorize                            │
│     • Receives JWT execution token (if policy allows)           │
│     • Token bound to storeId + variantId + price + qty          │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│  3. EXECUTION PHASE                                             │
│     Agent presents token at merchant checkout                   │
│                                                                 │
│     • POST /agents/v1/exec/validate (burn-on-use)               │
│     • Token consumed — single-use, replay-protected             │
│     • Transaction proceeds on merchant's commerce platform      │
│     • Permyx logs execution event for merchant audit trail       │
└─────────────────────────────────────────────────────────────────┘
```

---

## Protocol Integrations

Permyx acts as a governed gateway for agent transaction protocols. When agents use these protocols to execute commerce actions, Permyx sits in front — enforcing merchant policy *before* any transaction reaches the commerce platform.

| Protocol | Vendor | Version | Discovery Method |
|----------|--------|---------|------------------|
| **ACP** (Agentic Commerce Protocol) | OpenAI | v2025-03 | ACP JSONL feed |
| **UCP** (Universal Commerce Protocol) | Google | v1.0 | Merchant Center feed + UCP endpoints |
| **MCP** (Model Context Protocol) | Microsoft | v2025-11 | MCP tool invocation |
| **Commerce Graph API** | Meta | v2025-Q4 | Commerce Graph with social signals |

```
┌─────────────┐     ┌──────────────────────────────────────┐     ┌─────────────┐
│             │     │             Permyx                    │     │             │
│  AI Agent   │────▶│                                      │────▶│  Commerce   │
│             │ ACP │  ┌────────────────────────────────┐  │     │  Platform   │
│  (OpenAI,   │ UCP │  │     Policy Enforcement         │  │     │             │
│   Google,   │ MCP │  │  • Merchant action policies    │  │     │  (Shopify,  │
│   MSFT,     │Meta │  │  • Agent reputation gates      │  │     │  WooComm,   │
│   Meta)     │     │  │  • Rate limiting               │  │     │   etc.)     │
└─────────────┘     │  └────────────────────────────────┘  │     │             │
                    │  ┌────────────────────────────────┐  │     └─────────────┘
                    │  │     Audit & Observability      │  │
                    │  │  • Agent interaction logs      │  │
                    │  │  • Request/response capture    │  │
                    │  │  • Anomaly detection           │  │
                    │  └────────────────────────────────┘  │
                    │  ┌────────────────────────────────┐  │
                    │  │     Validation Layer           │  │
                    │  │  • Variant availability        │  │
                    │  │  • Price verification          │  │
                    │  │  • Inventory confirmation      │  │
                    │  └────────────────────────────────┘  │
                    └──────────────────────────────────────┘
```

### Explicit Boundaries

> **Permyx does not execute transactions.**  
> **Permyx does not replace transaction protocols.**  
> **Permyx enforces merchant policy before transaction requests reach commerce systems.**

---

## Data Sources

| Source Type | Description |
|-------------|-------------|
| **Shopify Native API** | Direct OAuth integration with Shopify stores |
| **BigCommerce API** | Direct integration with BigCommerce stores |
| **WooCommerce API** | WordPress e-commerce integration |
| **Custom Merchant Feeds** | Direct API ingestion for enterprise merchants |

---

## Permyx Value Layer

When data enters Permyx from any commerce platform, we add the infrastructure that makes execution authorization possible:

| Capability | Description |
|------------|-------------|
| **Schema normalization** | Structured, machine-legible product data (prerequisite for evaluating authorization rules) |
| **Freshness tracking** | SLAs on data validity — stale data cannot be authorized for execution |
| **Execution authorization** | JWT-bound, single-use tokens that prove an agent has permission to act |
| **Policy enforcement** | Merchant-defined rules for which actions agents may perform |
| **Risk classification** | Action-safe vs discovery-only, based on data freshness and policy compliance |
| **Replay protection** | Burn-on-use tokens with `jti` tracking prevent duplicate executions |
| **Audit trail** | Every authorization request, denial, and execution logged for merchant visibility |

---

## API Surface

### Execution Authorization Module (EAM)

| Method | Endpoint | Purpose |
|--------|----------|---------|
| POST | `/agents/v1/exec/authorize` | Request execution token |
| POST | `/agents/v1/exec/validate` | Consume execution token (burn-on-use) |
| POST | `/agents/v1/exec/observe` | Log unauthenticated execution attempt |
| POST | `/agents/v1/exec/introspect` | Inspect token without consuming |

### Feed Adapters

| Method | Endpoint | Purpose |
|--------|----------|---------|
| GET | `/feeds/:storeId/google-merchant.xml` | Google Merchant Center RSS 2.0 feed |
| GET | `/feeds/:storeId/openai-acp.jsonl` | OpenAI ACP feed |
| GET | `/feeds/:storeId/gpt-action/openapi.yaml` | OpenAPI spec for GPT Actions |

### Public Catalog API

| Method | Endpoint | Purpose |
|--------|----------|---------|
| POST | `/api/v1/catalog/search` | Search products across all merchants |
| GET | `/api/v1/catalog/products/:permyxId` | Get product by stable Permyx ID |
| GET | `/api/v1/catalog/merchants` | List participating merchants |

---

## Enforcement Model

### V1 — Log-Only (Current)

No checkout is blocked. Merchants gain visibility into agent activity before enforcement activates.

- `POST /authorize` — live
- `POST /validate` — live (log-only)
- Execution event log and merchant dashboard
- Policy simulation (log would-be denials)
- Replay attempt detection (logged, not blocked)

### V1.1 — Policy Enforcement (Next)

- Checkout blocking on invalid/missing token
- Shopify native Checkout Function
- Token burn-on-use enforced at validation
- Policy enforcement: SKU whitelist, cart value caps, velocity limits

---

## Tech Stack

| Layer | Technology |
|-------|------------|
| API | TypeScript, Hono, Node.js 20 |
| Portal | Next.js 14, React 18, Tailwind CSS |
| Background Jobs | Azure Functions (TypeScript) |
| Database | Azure Cosmos DB (NoSQL, RU-based) |
| Messaging | Azure Service Bus (topics + subscriptions) |
| Auth | RS256-signed JWTs (execution tokens), OAuth 2.0 (merchant auth) |
| Infrastructure | Azure App Service, Static Web Apps, Key Vault, Application Insights |
| IaC | Bicep modules |
| Monorepo | pnpm workspaces, shared TypeScript types |
| CI/CD | GitHub Actions |
