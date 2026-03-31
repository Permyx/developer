# Permyx Platform Overview

> **Last Updated:** March 2026

## What is Permyx?

**Execution Authorization for AI Commerce**

Permyx is the execution authorization layer that governs what AI agents are allowed to do in commerce — before they act.

We sit between AI agents and merchant checkout. We don't replace payment processors, fraud tools, or checkout platforms. We add the deterministic enforcement layer that governs agent authority before any payment or fulfillment action occurs.

---

## Why Permyx Exists

Commerce infrastructure was built for humans. Every checkout flow assumes:

- A human is present and consenting
- Authentication equals intent
- Payment approval equals permission to act

AI agents break all three assumptions. They act at machine speed, without a human present, with intent that may be ambiguous. Fraud models trained on human behavior cannot evaluate them accurately.

**No system governs what an AI agent is actually permitted to execute.** Permyx fills that gap.

---

## The Execution Gap

| What Exists Today | What's Missing |
|-------------------|----------------|
| Card networks authorize payments | No system authorizes agent execution |
| Fraud tools score transaction risk | No policy enforcement for agent scope |
| Merchants set pricing and availability | No token-gated execution for agents |
| Platforms optimize the buyer experience | No replay protection or burn-on-use tokens |
| Authentication verifies identity | No scoped authority for what agents can DO |

**Payment authorization ≠ execution permission.**

This is the gap Permyx fills. It is a different layer — upstream of payment, downstream of agent intent — and it is currently unoccupied.

---

## How It Works: The Authorization Flow

Five steps, one invariant: **no valid token, no execution.**

| # | Step | What Happens |
|---|------|--------------|
| 01 | Agent requests auth | Agent calls `POST /authorize` with identity, cart value, SKU list, merchant ID, and requested scope |
| 02 | Policy evaluation | Permyx validates cart value cap, SKU whitelist, agent identity, velocity rate, and TTL against merchant-defined policies |
| 03 | Token issued | A signed JWT execution token is returned, cryptographically bound to the execution context |
| 04 | Token carried to checkout | The agent passes the token to the checkout surface; the checkout validates via `POST /validate` |
| 05 | Burn-on-use + audit log | Token is consumed on validation — it cannot be reused. Every step is logged with a trace ID |

---

## V1 API Surface

### Execution Authorization Module (EAM)

| Method | Endpoint | Purpose |
|--------|----------|---------|
| POST | `/agents/v1/exec/authorize` | Request execution token. Validates agent, cart, policy. Returns ISSUED or DENIED |
| POST | `/agents/v1/exec/validate` | Consume or inspect an execution token. Enforces burn-on-use. Detects replay |
| POST | `/agents/v1/exec/observe` | Log unauthenticated execution attempt |
| POST | `/agents/v1/exec/introspect` | Inspect a token without consuming |

### Feed Adapters

Feed adapters export Permyx product data in formats required by AI shopping platforms.

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/feeds/:storeId/google-merchant.xml` | Google Merchant Center RSS 2.0 feed |
| GET | `/feeds/:storeId/openai-acp.jsonl` | OpenAI Agentic Commerce Protocol (ACP) feed |
| GET | `/feeds/:storeId/openai-acp/validate` | Validate ACP feed compliance |
| GET | `/feeds/:storeId/stats` | Feed statistics and readiness |
| GET | `/feeds/:storeId/manifest.json` | Feed manifest with all URLs |
| GET | `/feeds/:storeId/gpt-action/openapi.yaml` | OpenAPI spec for GPT Actions |

### Public Catalog API

The primary discovery surface for AI agents. Agents query this API to discover products across all participating merchants.

| Method | Endpoint | Description |
|--------|----------|-------------|
| POST | `/api/v1/catalog/search` | Search products across all merchants |
| GET | `/api/v1/catalog/products/:permyxId` | Get product by stable Permyx ID |
| GET | `/api/v1/catalog/merchants` | List participating merchants |
| GET | `/api/v1/catalog/merchants/check?domain=` | Check if a domain participates |

---

## Enforcement Model

### V1 — Available Now: Full Observability

V1 is **log-only**. No checkout is blocked. Merchants gain visibility into agent activity before enforcement activates.

- Execution authorization API (all 4 endpoints)
- Execution event log and merchant dashboard
- Agent identity tracking
- Policy simulation (log would-be denials)
- Replay attempt detection (logged, not blocked)
- Interactive API Playground in portal
- Shopping Agent Simulator with 4 protocol personas

### V1.1 — Coming Next: Policy Enforcement

- Checkout blocking on invalid/missing token
- Shopify native Checkout Function
- Token burn-on-use enforced at validation
- Policy enforcement: SKU whitelist, cart value caps, velocity limits

---

## Who Uses Permyx

| Customer | What They Get |
|----------|---------------|
| **Merchants** | Control over what AI agents can do with their catalog — authorization, policy enforcement, full audit trail |
| **AI Agent Builders** | Execution authorization tokens that prove permission to act, plus structured catalog data |
| **Commerce Platforms** | Governance infrastructure that protects their merchants from uncontrolled agent behavior |

---

## Protocol Support

Permyx integrates with the emerging AI commerce protocols:

| Protocol | Vendor | Integration |
|----------|--------|-------------|
| **ACP** (Agentic Commerce Protocol) | OpenAI | ACP JSONL feed export, GPT Action specs |
| **UCP** (Universal Commerce Protocol) | Google | Merchant Center feed compatibility |
| **MCP** (Model Context Protocol) | Microsoft | MCP tool invocation support |
| **Commerce Graph API** | Meta | Social signal integration |

Permyx enforces merchant policy before any protocol-based transaction reaches the commerce platform.
