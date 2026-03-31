# Permyx AI Commerce Data Contract — v1.5

> **Status:** Stable Extension  
> **Compatibility:** Fully backward-compatible with v1.0  
> **Purpose:** Platform compatibility, verification readiness, and certification alignment

---

## Why v1.5 Exists

All three major AI vendors (OpenAI, Microsoft, Google) are converging on:

> "We will only act on commerce data that is versioned, policy-scoped, verified, auditable, and subscribable."

v1.5 formalizes those expectations explicitly.

---

## What v1.5 Adds

| # | Capability | What It Does |
|---|------------|--------------|
| 1 | Protocol Capability Discovery | Lets platforms ask "What kind of commerce system are you?" |
| 2 | Verification Semantics | Explicit verification state on every field |
| 3 | Scope-Aware Enforcement | Chat vs Cart vs Compare vs Ads vs Agent_Exec distinctions |
| 4 | Mandatory Eventing | Full event streaming with typed diffs |
| 5 | Compliance Surface | Machine-verifiable certification checklist |

---

## Protocol Capability Discovery

### Endpoint

```
GET /v1/protocols
```

Allows platforms to ask: "What kind of commerce system are you?"

### Response

```json
{
  "permyx_protocol_version": "1.5",
  "supported_contract_versions": ["1.0", "1.5"],
  "capabilities": {
    "merchant_authoritative": true,
    "policy_enforced": true,
    "verification_exposed": true,
    "event_streaming": true,
    "checkout_execution": false,
    "agent_decision_optimization": false
  },
  "protocols": {
    "openai.shopping": {
      "supported": true,
      "min_contract": "1.0",
      "features": ["discovery", "compare", "safe_linking", "events"]
    },
    "microsoft.copilot.commerce": {
      "supported": true,
      "min_contract": "1.0",
      "features": ["discovery", "compare", "verification", "events"]
    },
    "google.agent.shopping": {
      "supported": true,
      "min_contract": "1.0",
      "features": ["grounded_answers", "price_check", "events"]
    }
  }
}
```

---

## Verification Semantics

v1.5 makes verification state explicit so platforms don't have to infer.

### Verification States

```typescript
type VerificationState =
  | "verified"     // merchant or authoritative platform confirmed
  | "inferred"     // derived with confidence score
  | "stale"        // beyond stale_after timestamp
  | "withheld";    // policy blocked this field
```

### Example: Verified Product Field

```json
{
  "title": {
    "value": "Sony WH-1000XM5 Wireless Headphones",
    "verification": {
      "state": "verified",
      "verified_by": "shopify_api",
      "verified_at": "2026-03-29T10:00:00Z",
      "confidence": 1.0
    }
  },
  "price": {
    "value": { "amount": 349.99, "currency": "USD" },
    "verification": {
      "state": "verified",
      "verified_by": "shopify_api",
      "verified_at": "2026-03-29T10:00:00Z",
      "stale_after": "2026-03-29T16:00:00Z",
      "confidence": 1.0
    }
  }
}
```

---

## Scope-Aware Enforcement

Permyx policies distinguish between what agents can do in different contexts:

| Scope | Agent May | Agent Must Not |
|-------|-----------|----------------|
| `chat` | Quote prices, describe products, compare | Add to cart, initiate checkout |
| `compare` | Show side-by-side pricing, availability | Modify cart, redirect to checkout |
| `cart` | Add items, show totals | Complete purchase without execution token |
| `agent_exec` | Full execution with valid token | Act without burn-on-use token validation |
| `ads` | Display promotional content | Misrepresent pricing or availability |

The `scope` field in execution token requests determines which policy rules are evaluated.

---

## Mandatory Eventing

v1.5 requires all authorization events to be streamed with typed diffs:

```json
{
  "eventType": "execution.authorized",
  "traceId": "trc_abc123",
  "timestamp": "2026-03-29T10:05:00Z",
  "storeId": "store-123",
  "agentId": "agent-456",
  "scope": "agent_exec",
  "decision": "allowed",
  "tokenId": "jti_xyz",
  "expiresAt": "2026-03-29T10:07:00Z"
}
```

Event types: `execution.authorized`, `execution.denied`, `execution.validated`, `execution.expired`, `execution.replay_detected`, `execution.observed`

---

## Compliance Surface

Machine-verifiable checklist for platform certification:

```json
{
  "compliance": {
    "data_versioned": true,
    "policy_scoped": true,
    "verification_explicit": true,
    "events_streaming": true,
    "tokens_burn_on_use": true,
    "replay_protected": true,
    "audit_immutable": true,
    "merchant_authoritative": true,
    "checkout_execution": false
  }
}
```

---

## Backward Compatibility

v1.5 is fully backward-compatible with v1.0:

- All v1.0 endpoints continue to work unchanged
- v1.5 fields are additive — never breaking
- Clients can negotiate version via `Accept-Version` header
- Responses include `permyx_protocol_version` field
