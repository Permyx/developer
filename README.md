# Permyx Developer Documentation

Execution authorization for AI commerce. Permyx sits between AI agents and merchant checkouts — issuing, validating, and logging execution tokens so every machine-initiated transaction is visible, auditable, and policy-governed.

## Quickstart: Your First Execution Token in 5 Minutes

### Prerequisites

- A Permyx agent API key (`permyx_agent_xxx`) — provided during onboarding
- Sandbox base URL: `https://api.sandbox.permyx.io`
- A store ID and variant ID (provided with your API key)

### 1. Request an execution token

```bash
curl -X POST https://api.sandbox.permyx.io/agents/v1/exec/authorize \
  -H "Content-Type: application/json" \
  -H "X-API-Key: permyx_agent_YOUR_KEY_HERE" \
  -d '{
    "storeId": "store-YOUR_STORE_ID",
    "permyxProductId": "PRODUCT_UUID",
    "sourceVariantId": "shopify:variant:123456",
    "quantity": 1,
    "price": { "amount": 29.99, "currency": "USD" },
    "scope": "agent_exec"
  }'
```

Response:

```json
{
  "executionToken": "eyJhbGciOiJSUzI1NiIs...",
  "expiresAt": "2026-03-02T20:15:45Z",
  "traceId": "trc_abc123",
  "decision": "allowed"
}
```

The token is a short-lived JWT (120 second TTL) bound to the exact store, variant, quantity, price, and scope.

### 2. Validate the token at checkout

Pass the `executionToken` to your checkout surface. Before processing the order, validate it:

```bash
curl -X POST https://api.sandbox.permyx.io/agents/v1/exec/validate \
  -H "Content-Type: application/json" \
  -H "X-API-Key: permyx_agent_YOUR_KEY_HERE" \
  -d '{
    "storeId": "store-YOUR_STORE_ID",
    "executionToken": "eyJhbGciOiJSUzI1NiIs..."
  }'
```

Response:

```json
{
  "allowed": true,
  "reasonCode": null,
  "traceId": "trc_xyz789",
  "tokenConsumed": true
}
```

Tokens are single-use. A second validate call with the same token returns `REPLAY_DETECTED`.

### 3. Log an observation (optional)

If you detect an agent-initiated checkout that didn't carry a token, log it:

```bash
curl -X POST https://api.sandbox.permyx.io/agents/v1/exec/observe \
  -H "Content-Type: application/json" \
  -H "X-API-Key: permyx_agent_YOUR_KEY_HERE" \
  -d '{
    "storeId": "store-YOUR_STORE_ID",
    "sourceVariantId": "shopify:variant:123456",
    "quantity": 1,
    "hasToken": false
  }'
```

### 4. View events in the dashboard

All execution events — authorized, denied, observed — appear in the merchant dashboard at `https://portal.sandbox.permyx.io`.

## API Reference

See [docs/execution-auth-api.md](docs/execution-auth-api.md) for the complete endpoint reference including request/response schemas, JWT claims, and reason codes.

## Environments

| Environment | API Base URL | Portal |
|---|---|---|
| Sandbox | `https://api.sandbox.permyx.io` | `https://portal.sandbox.permyx.io` |
| Production | `https://api.permyx.io` | `https://portal.permyx.io` |

All beta users start in the sandbox environment.

## V1 — Log-Only Mode

V1 operates in **log-only mode**. No checkout is blocked. Authorization decisions are logged for merchant visibility. This lets you integrate and observe real execution patterns before enforcement activates in V1.1.

## Support

- **Email:** permyx-support@permyx.com
- **Engineering calls:** Weekly call with the founding team (scheduled during onboarding)
- **Website:** [permyx.io](https://permyx.io)

## License

Copyright 2026 Permyx. All rights reserved.
