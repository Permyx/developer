# Execution Authorization API Reference

> **Version:** V1 — Log-Only (Execution Authorization Beta)  
> **Base URL:** `https://api.sandbox.permyx.io`  
> **Authentication:** `X-API-Key: permyx_agent_xxx`

The Execution Authorization Module (EAM) provides deterministic authorization for machine-initiated checkout attempts. Agents obtain a short-lived JWT execution token before proceeding with a purchase. Every decision is logged and visible in the merchant dashboard.

> **V1 operates in log-only mode.** No checkout blocking. Authorization decisions are logged for merchant visibility. Enforcement mode unlocks in V1.1.

---

## Endpoints

| Method | Endpoint | Description |
|--------|----------|-------------|
| POST | `/agents/v1/exec/authorize` | Issue a short-lived JWT execution token |
| POST | `/agents/v1/exec/validate` | Validate + consume an execution token |
| POST | `/agents/v1/exec/observe` | Log an unauthenticated execution attempt |
| POST | `/agents/v1/exec/introspect` | Inspect a token without consuming it |

All endpoints require an agent API key in the `X-API-Key` header.

---

## Authorization Flow

```
Agent
──► POST /exec/authorize ──► Permyx issues JWT (120s TTL)
  │
  ├──► POST /exec/introspect ──► Debug: check token claims & status
  │
  └──► POST /exec/validate   ──► Permyx validates + consumes token
          │
          ├── allowed: true  → Agent proceeds to checkout
          └── allowed: false → Agent handles denial
```

---

## POST /agents/v1/exec/authorize

Issue a short-lived JWT execution token for a specific product variant, quantity, and price.

### Request

```bash
curl -X POST https://api.sandbox.permyx.io/agents/v1/exec/authorize \
  -H "Content-Type: application/json" \
  -H "X-API-Key: permyx_agent_xxx" \
  -d '{
    "storeId": "store-123",
    "permyxProductId": "550e8400-e29b-41d4-a716-446655440000",
    "sourceVariantId": "shopify:variant:123456",
    "quantity": 1,
    "price": { "amount": 120.00, "currency": "USD" },
    "scope": "agent_exec"
  }'
```

### Request Body

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `storeId` | string | Yes | Target merchant store ID |
| `permyxProductId` | string | Yes | Permyx product UUID |
| `sourceVariantId` | string | Yes | Source platform variant ID (e.g. `shopify:variant:123456`) |
| `quantity` | number | Yes | Quantity to authorize (1–50) |
| `price.amount` | number | Yes | Price amount (positive) |
| `price.currency` | string | Yes | ISO 4217 currency code (e.g. `USD`) |
| `scope` | string | Yes | Must be `"agent_exec"` |
| `clientContext.cartId` | string | No | Optional cart ID for audit linking |
| `clientContext.sessionId` | string | No | Optional session ID for audit linking |

### Success Response (200)

```json
{
  "executionToken": "eyJhbGciOiJSUzI1NiIs...",
  "expiresAt": "2026-03-30T20:15:45Z",
  "traceId": "trc_abc123",
  "decision": "allowed"
}
```

### Deny Response (403)

```json
{
  "error": {
    "code": "STALE_PRICE",
    "message": "Price data is stale for execution scope (last verified >6 hours ago)."
  },
  "traceId": "trc_def456",
  "decision": "denied"
}
```

---

## POST /agents/v1/exec/validate

Validate and consume an execution token. Tokens are single-use — a second call with the same token returns `REPLAY_DETECTED`.

### Request

```bash
curl -X POST https://api.sandbox.permyx.io/agents/v1/exec/validate \
  -H "Content-Type: application/json" \
  -H "X-API-Key: permyx_agent_xxx" \
  -d '{
    "storeId": "store-123",
    "executionToken": "eyJhbGciOiJSUzI1NiIs..."
  }'
```

### Request Body

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `storeId` | string | Yes | Store ID (must match the token's storeId) |
| `executionToken` | string | Yes | The JWT token from `/authorize` |
| `checkoutContext.cartToken` | string | No | Optional cart token for audit |
| `checkoutContext.checkoutId` | string | No | Optional checkout ID for audit |

### Success Response (200)

```json
{
  "allowed": true,
  "reasonCode": null,
  "traceId": "trc_xyz789",
  "tokenConsumed": true
}
```

### Deny Response (403)

```json
{
  "allowed": false,
  "reasonCode": "REPLAY_DETECTED",
  "traceId": "trc_xyz790",
  "tokenConsumed": false
}
```

---

## POST /agents/v1/exec/observe

Log an unauthenticated execution attempt — use this when you detect an agent-initiated checkout that did not carry a Permyx execution token.

### Request

```bash
curl -X POST https://api.sandbox.permyx.io/agents/v1/exec/observe \
  -H "Content-Type: application/json" \
  -H "X-API-Key: permyx_agent_xxx" \
  -d '{
    "storeId": "store-123",
    "sourceVariantId": "shopify:variant:123456",
    "quantity": 1,
    "hasToken": false
  }'
```

### Request Body

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `storeId` | string | Yes | Target store ID |
| `sourceVariantId` | string | No | Variant ID if known |
| `quantity` | number | No | Quantity if known |
| `hasToken` | boolean | Yes | Whether the request carried a token |
| `context.agentName` | string | No | Name of the observed agent |
| `context.reason` | string | No | Why this observation was logged |

### Response (200)

```json
{
  "observed": true,
  "traceId": "trc_obs001"
}
```

---

## POST /agents/v1/exec/introspect

Inspect a token without consuming it. Useful for debugging during integration.

### Request

```bash
curl -X POST https://api.sandbox.permyx.io/agents/v1/exec/introspect \
  -H "Content-Type: application/json" \
  -H "X-API-Key: permyx_agent_xxx" \
  -d '{
    "executionToken": "eyJhbGciOiJSUzI1NiIs..."
  }'
```

### Request Body

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `executionToken` | string | Yes | The JWT token to inspect |

### Response (200)

```json
{
  "valid": true,
  "expired": false,
  "claims": {
    "storeId": "store-123",
    "variantId": "shopify:variant:123456",
    "qty": 1,
    "priceAmount": 120.00,
    "currency": "USD",
    "scope": "agent_exec",
    "intentHash": "sha256:abc..."
  },
  "replayStatus": "unused",
  "policyDecision": "allowed",
  "expiresIn": 85
}
```

| Field | Type | Description |
|-------|------|-------------|
| `valid` | boolean | Whether the token signature and structure are valid |
| `expired` | boolean | Whether the token has expired |
| `claims` | object | Decoded token claims (null if invalid) |
| `replayStatus` | string | `"unused"`, `"consumed"`, or `"unknown"` |
| `policyDecision` | string | `"allowed"`, `"denied"`, or `"unknown"` |
| `expiresIn` | number | Seconds until expiry (null if expired) |

---

## Execution Token (JWT)

Tokens are RS256-signed JWTs with a 120-second TTL. They are bound to a specific store, variant, quantity, price, and scope via the `execution_intent_hash` claim.

### Claims

| Claim | Type | Description |
|-------|------|-------------|
| `iss` | string | Always `"permyx"` |
| `sub` | string | Agent key ID |
| `storeId` | string | Target store |
| `variantId` | string | Target variant |
| `qty` | number | Authorized quantity |
| `price_amount` | number | Authorized price |
| `currency` | string | ISO 4217 currency code |
| `sku_hash` | string | SHA-256 of `storeId\|variantId` |
| `execution_intent_hash` | string | SHA-256 binding token to exact intent parameters |
| `scope` | string | Always `"agent_exec"` |
| `jti` | string | Unique token ID (UUID v4) — used for replay protection |
| `iat` | number | Issued-at timestamp (Unix) |
| `exp` | number | Expiry timestamp (iat + 120 seconds) |
| `ver` | string | Protocol version |

---

## Reason Codes

When a request is denied, the `reasonCode` field contains one of these standardized codes:

| Code | Description |
|------|-------------|
| `NO_TOKEN` | No execution token provided |
| `TOKEN_EXPIRED` | Token has expired (>120s) |
| `INVALID_SIGNATURE` | JWT signature verification failed |
| `STORE_MISMATCH` | storeId in request doesn't match token |
| `REPLAY_DETECTED` | Token jti already consumed (single-use) |
| `INTENT_MISMATCH` | Recomputed intent hash doesn't match token |
| `SCOPE_RESTRICTED` | Scope must be `agent_exec` |
| `STALE_PRICE` | Price data older than 6 hours |
| `VARIANT_NOT_FOUND` | Variant not found in store catalog |
| `RATE_LIMITED` | Agent key rate limit exceeded |
| `POLICY_DENIED` | General policy denial |
| `ENFORCE_NOT_AVAILABLE` | Enforcement mode not yet available (V1 is log-only) |

---

## Error Responses

All error responses follow this format:

```json
{
  "error": {
    "code": "REASON_CODE",
    "message": "Human-readable description."
  }
}
```

### HTTP Status Codes

| Status | Meaning |
|--------|---------|
| 200 | Success |
| 400 | Invalid request (missing/malformed fields) |
| 401 | Invalid or missing API key |
| 403 | Authorization denied (see reason code) |
| 429 | Rate limited |
| 500 | Internal server error |

---

## Rate Limits

Agent API keys have per-minute and monthly request limits based on tier. Rate limit headers are included in every response:

| Header | Description |
|--------|-------------|
| `X-RateLimit-Limit` | Requests per minute allowed |
| `X-RateLimit-Remaining` | Requests remaining in current window |
| `X-RateLimit-Reset` | Unix timestamp when the window resets |

---

## Changelog

| Date | Change |
|------|--------|
| 2026-03-30 | docs: Architecture, data contract, platform overview, feed adapters, catalog API |
| 2026-03-02 | Initial beta documentation published |
