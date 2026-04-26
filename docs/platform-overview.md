# Permyx Platform Overview

> **Last Updated:** April 26, 2026
> **Version:** 3.0 - Pivot Baseline (Authorization + Governance Evidence)

## What is Permyx?

**Execution Authorization and Governance Control Plane for AI Commerce**

Permyx determines what AI agents are allowed to do in commerce before execution, and proves what happened across channels after execution.

We do not replace payment processors, fraud tools, or checkout platforms. We provide deterministic governance for AI-initiated actions: scoped authority, policy enforcement, replay controls, and audit-grade execution evidence.

---

## Why Permyx Exists

Commerce infrastructure was built for humans. Traditional checkout assumptions still are:

- A human is present and consenting
- Authentication equals intent
- Payment approval equals permission to act

AI agents break all three assumptions. They can act at machine speed, with delegated intent, across mixed channel paths where a human is not actively present.

**Payment authorization != execution permission.**

Permyx exists to close that gap.

---

## What Changed in 2026

AI commerce now runs through mixed execution paths:

- Referrer-to-merchant checkout paths
- Built-in AI channel checkout paths
- Direct storefront or custom headless paths

In some built-in channel flows, storefront-side signals do not consistently fire. That makes attribution and order-source metadata first-class governance evidence.

This is an evidence-model expansion, not a category change.

---

## Non-Negotiable Core

- Permyx is an execution governance layer, not a conversion optimization product
- Permyx governs what agents are allowed to do before execution where interception exists
- Permyx issues scoped, time-bound, burn-on-use authority and enforces merchant policy
- Permyx maintains deterministic, queryable execution audit trails

---

## Product Definition

Permyx is the execution authorization and governance control plane for AI commerce.

It has two coupled functions:

### 1) Authorization Function

- Token issuance and validation
- Burn-on-use and replay controls
- Policy-based allow and deny logic
- Merchant-governed scope, limits, and TTL

### 2) Evidence Function

- Channel and referrer attribution ingestion
- Order-source and checkout metadata enrichment
- Confidence-rated execution outcomes
- Unified, queryable governance timeline

The evidence function supports governance truth. It is not a marketing analytics function.

---

## Holistic Governance Flow

Six steps, one invariant: **no valid authority, no valid execution claim.**

| # | Step | What Happens |
|---|------|-------------|
| 01 | Agent requests authority | Agent calls `POST /agents/v1/exec/authorize` with identity, merchant, cart value, SKU scope, and action scope |
| 02 | Policy evaluation | Permyx checks cart cap, SKU whitelist, identity trust, velocity, and TTL. Result: allowed or denied with reason code |
| 03 | Scoped token issuance | Signed execution token is bound to execution context |
| 04 | Validation where supported | Checkout path calls `POST /agents/v1/exec/validate`; token is consumed (burn-on-use) |
| 05 | Evidence ingestion | Permyx correlates execution events with attribution and order-source metadata |
| 06 | Unified timeline | Governance events are queryable for audit, investigation, and reporting |

---

## API Surface

| Method | Endpoint | Purpose |
|--------|----------|---------|
| POST | `/agents/v1/exec/authorize` | Request scoped execution authority |
| POST | `/agents/v1/exec/validate` | Consume or inspect execution token |
| POST | `/agents/v1/exec/observe` | Log observed unauthenticated execution attempts |
| POST | `/agents/v1/exec/introspect` | Inspect token state without consuming |

See [execution-auth-api.md](./execution-auth-api.md) for request and response details.

---

## Promise Boundaries

### Promise We Can Keep Now

- Execution authorization where interception and validation are available
- Policy-based allow and deny outcomes with deterministic reason codes
- Cross-channel governance evidence using attribution and order metadata
- Merchant-governed controls and auditability across AI commerce surfaces

### Promise We Should Not Make

Permyx should not claim universal real-time pre-checkout blocking across every AI channel path today.

Reason: some platform-mediated paths do not expose full interception points to third-party middleware.

---

## Phase Model

### Phase 1 - Log-Only Governance Visibility
- Authorization and validation visibility
- Replay detection and policy simulation
- Channel-attributed evidence

### Phase 2 - Policy Enforcement
- Token-required validation where interception exists
- Burn-on-use, replay, and stale-context rejection where supported

### Phase 3 - Cross-Channel Governance
- Unified policy surface
- Unified timeline across mixed path types

---

## Integration Patterns

### Pattern 1 - Direct Interception Path
Token validation enforced inline for headless or custom stacks.

### Pattern 2 - Platform Hook Path
Token validation enforced through available platform extension points.

### Pattern 3 - Referrer/Built-In Evidence Path
When interception is limited, Permyx still establishes governance truth through attribution and correlated events.

---

## Related Docs

- [architecture.md](./architecture.md) - Technical architecture
- [execution-auth-api.md](./execution-auth-api.md) - API reference
- [data-contract.md](./data-contract.md) - Data contract and protocol capabilities

---

Permyx - Execution Authorization and Governance for AI Commerce
