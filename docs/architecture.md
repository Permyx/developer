# Permyx Systems Architecture

> **Version:** 3.3
> **Last Updated:** April 26, 2026
> **Status:** Deployed - Execution Governance Control Plane (Authorization + Evidence)

---

## Core Philosophy

Permyx is the authorization and governance layer that governs what AI agents are permitted to do before execution, and proves what happened across channels after execution.

**Fundamental insight:** The critical gap is not data quality. The gap is missing merchant-controlled execution authority between agent intent and commerce execution.

---

## 6-Layer Control Model

The Authorize lane is core. Other layers support deterministic governance.

| Layer | Name | Purpose |
|-------|------|---------|
| **Ingest** | Data Collection | Shopify OAuth, WooCommerce, CSV, custom APIs |
| **Transform** | Normalization | Structured schema used for policy evaluation |
| **Govern** | Policy Enforcement | Scope, freshness, confidence, and limits |
| **Authorize** | Execution Authorization | JWT execution tokens, replay controls, intent binding |
| **Distribute** | API Delivery | Agent and partner-facing surfaces |
| **Audit** | Observability | Event timelines, traceability, anomaly visibility |

---

## April 2026 Position Clarification

AI-initiated commerce now uses mixed paths:

- Referrer paths ending on merchant checkout
- Built-in channel paths with limited interception points
- Direct storefront and headless paths

This changes evidence collection, not product category.

### Two-Lane Governance Model

1. **Authorization lane (primary)**
- Token issuance and policy evaluation where interception exists
- Token validation, burn-on-use, replay and stale-context controls

2. **Evidence lane (required)**
- Channel, referrer, and order-source attribution as governance evidence
- Confidence-rated outcomes when direct interception is unavailable

### Promise Scope

Permyx can promise:
- Execution authorization where interception exists
- Governance-grade evidence across all path types

Permyx should not promise:
- Universal real-time pre-checkout blocking across every AI path today

---

## Agent Journey

```
1) Discovery
   Agent queries governed catalog and policy context

2) Authorization
   Agent calls POST /agents/v1/exec/authorize
   Permyx returns allow/deny with token when allowed

3) Execution
   Supported path validates via POST /agents/v1/exec/validate
   Token is consumed and replay-protected

4) Evidence
   Permyx correlates authorization + validation + attribution + order outcomes
```

---

## Protocol Integrations

Permyx interoperates with protocol ecosystems while keeping merchant policy and governance evidence centralized.

| Protocol | Vendor | Version | Discovery Method |
|----------|--------|---------|------------------|
| ACP (Agentic Commerce Protocol) | OpenAI | v2025-03 | ACP JSONL feed |
| UCP (Universal Commerce Protocol) | Google | v1.0 | Merchant Center + UCP endpoints |
| MCP (Model Context Protocol) | Microsoft | v2025-11 | MCP tool invocation |
| Commerce Graph API | Meta | v2025-Q4 | Commerce graph integration |

---

## Explicit Boundaries

- Permyx does not execute transactions
- Permyx does not replace transaction protocols
- Permyx enforces merchant policy before transaction requests reach commerce systems
- Permyx does not own merchant checkout surfaces

---

## API Surfaces

### Execution Authorization

| Method | Endpoint | Purpose |
|--------|----------|---------|
| POST | `/agents/v1/exec/authorize` | Issue execution token |
| POST | `/agents/v1/exec/validate` | Validate and consume token |
| POST | `/agents/v1/exec/observe` | Log observed unauthenticated attempt |
| POST | `/agents/v1/exec/introspect` | Inspect token without consuming |

### Supporting Surfaces

- Integration adapters for supported execution paths
- Policy and onboarding surfaces for merchant control
- Dashboard timelines for governance evidence

---

## Enforcement Model

### Phase 1: Log-Only Governance Visibility
- No universal blocking claim
- End-to-end event and trace visibility

### Phase 2: Policy Enforcement
- Blocking where supported interception points exist
- Replay and stale-context hardening by integration

### Phase 3: Cross-Channel Governance
- Unified policy and timeline across mixed path types

---

## Tech Stack

| Layer | Technology |
|-------|------------|
| API | TypeScript, Hono, Node.js |
| Portal | Next.js, React |
| Functions | Azure Functions |
| Database | Azure Cosmos DB |
| Messaging | Azure Service Bus |
| Auth | RS256 JWT, OAuth 2.0 |
| Infrastructure | Azure App Service, Static Web Apps, Key Vault |
| IaC | Bicep |

---

Permyx - Execution Authorization and Governance for AI Commerce
