# Permyx Developer Documentation

**Execution Authorization and Governance for AI Commerce**

Permyx is the execution authorization and governance control plane for AI-initiated commerce. We determine what AI agents are allowed to do before execution, and we prove what happened across channels when execution occurs.

> Payment authorization != execution permission.
>
> Enforcement is applied where interception exists; evidence is captured across all channel paths.

---

## Current Baseline

This repository is aligned to the April 2026 pivot baseline.

- Authorization remains the core product function
- Evidence is a first-class governance function
- Promise boundaries are explicit and enforceable
- Messaging avoids conversion optimization framing

---

## Quick Links

| Document | Purpose |
|----------|---------|
| [docs/platform-overview.md](docs/platform-overview.md) | Product and positioning baseline |
| [docs/architecture.md](docs/architecture.md) | System architecture and control model |
| [docs/execution-auth-api.md](docs/execution-auth-api.md) | Execution authorization endpoint reference |
| [docs/data-contract.md](docs/data-contract.md) | Data contract and protocol capabilities |

---

## Product Model

Permyx operates as two coupled lanes:

1. Authorization lane
- Issue execution tokens
- Evaluate merchant policy
- Validate and burn tokens
- Reject replay and stale context where supported

2. Evidence lane
- Correlate authorization and validation events
- Ingest channel and referrer attribution
- Enrich order-source metadata
- Produce governance-grade audit timelines

---

## Authorization Flow

| Step | What Happens |
|------|--------------|
| 1. Agent requests authority | Agent calls `POST /agents/v1/exec/authorize` with store, item, quantity, price, and scope |
| 2. Policy evaluation | Permyx evaluates merchant-defined limits and returns allowed or denied with reason code |
| 3. Token issued | Short-lived signed JWT is bound to execution intent |
| 4. Token carried | Agent carries token to checkout via supported integration path |
| 5. Validate and consume | Checkout path calls `POST /agents/v1/exec/validate`; token is consumed (burn-on-use) |
| 6. Evidence correlation | Events are linked with attribution and order metadata for auditability |

---

## Quickstart

### 1) Authorize

```bash
curl -X POST https://api.sandbox.permyx.io/agents/v1/exec/authorize \
  -H "Content-Type: application/json" \
  -H "X-API-Key: permyx_agent_YOUR_KEY" \
  -d '{
    "storeId": "store-123",
    "permyxProductId": "PRODUCT_UUID",
    "sourceVariantId": "shopify:variant:123456",
    "quantity": 1,
    "price": { "amount": 29.99, "currency": "USD" },
    "scope": "agent_exec"
  }'
```

### 2) Validate

```bash
curl -X POST https://api.sandbox.permyx.io/agents/v1/exec/validate \
  -H "Content-Type: application/json" \
  -H "X-API-Key: permyx_agent_YOUR_KEY" \
  -d '{
    "storeId": "store-123",
    "executionToken": "eyJhbGciOiJSUzI1NiIs..."
  }'
```

### 3) Observe (optional)

```bash
curl -X POST https://api.sandbox.permyx.io/agents/v1/exec/observe \
  -H "Content-Type: application/json" \
  -H "X-API-Key: permyx_agent_YOUR_KEY" \
  -d '{
    "storeId": "store-123",
    "sourceVariantId": "shopify:variant:123456",
    "quantity": 1,
    "hasToken": false
  }'
```

---

## Phase Framing

### Phase 1: Log-Only Governance Visibility
- Authorization and validation outcomes are visible
- No universal checkout blocking claim
- Evidence correlation is active across mixed channels

### Phase 2: Policy Enforcement
- Enforcement is applied on supported interception paths
- Replay and stale-context controls harden by integration type

### Phase 3: Cross-Channel Governance
- Unified policy and timeline across direct, referrer, and built-in paths

---

## Environments

| Environment | API Base URL | Portal |
|-------------|--------------|--------|
| Sandbox | `https://api.sandbox.permyx.io` | `https://portal.sandbox.permyx.io` |
| Production | `https://api.permyx.io` | `https://portal.permyx.io` |

---

## Support

- Email: developers@permyx.io

---

Permyx - Execution Authorization and Governance for AI Commerce
