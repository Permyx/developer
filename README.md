# Permyx Developer Documentation

**The Execution Authorization Layer for AI Commerce**

Permyx sits between AI agents and merchant checkouts — issuing, validating, and logging execution tokens so every machine-initiated transaction is visible, auditable, and policy-governed.

> Payment authorization ≠ execution permission.  
> Permyx governs what AI agents are allowed to **do** in commerce — before they act.

---

## The Problem

Commerce infrastructure was built for humans. Every checkout flow assumes a human is present, authentication equals intent, and payment approval equals permission to act.

AI agents break all three assumptions. They act at machine speed, without a human present, with intent that may be ambiguous.

| What Exists Today | What's Missing |
|-------------------|----------------|
| Card networks authorize payments | No system authorizes agent **execution** |
| Fraud tools score transaction risk | No policy enforcement for agent **scope** |
| Merchants set pricing and availability | No token-gated execution for agents |
| Platforms optimize the buyer experience | No replay protection or burn-on-use tokens |

**No system governs what an AI agent is actually permitted to execute.** Permyx fills that gap.

---

## How It Works

```
┌─────────────────┐     ┌───────────────────────────┐     ┌─────────────────────┐
│   AI AGENTS     │     │        Permyx EAM         │     │     CHECKOUT        │
├─────────────────┤     ├───────────────────────────┤     ├─────────────────────┤
│ GPT Shopping    │     │ • Token issuance          │     │ Shopify             │
│ Perplexity Shop │ ──► │ • Policy enforcement      │ ──► │ BigCommerce         │
│ Custom Agents   │     │ • Velocity controls       │     │ WooCommerce         │
│ Klarna AI       │     │ • Replay protection       │     │ Headless/Custom     │
│ Enterprise Bots │     │ • Burn-on-use validation  │     │ Any checkout        │
└─────────────────┘     └───────────────────────────┘     └─────────────────────┘
```

### Authorization Flow

| Step | What Happens |
|------|--------------|
| 1. Agent requests auth | Agent calls `POST /authorize` with identity, cart value, SKU list, merchant ID, and scope |
| 2. Policy evaluation | Permyx validates against merchant-defined policies. Result: ISSUED or DENIED with reason code |
| 3. Token issued | Signed JWT execution token returned — cryptographically bound to agent, merchant, cart, SKU, and TTL |
| 4. Token carried to checkout | Agent presents token at checkout. Merchant validates via `POST /validate` |
| 5. Burn-on-use + audit | Token consumed on validation — single-use, replay-protected. Every step logged with trace ID |

---

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
  "expiresAt": "2026-03-30T20:15:45Z",
  "traceId": "trc_abc123",
  "decision": "allowed"
}
```

The token is a short-lived JWT (120-second TTL) bound to the exact store, variant, quantity, price, and scope.

### 2. Validate the token at checkout

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

---

## API Reference

See [docs/execution-auth-api.md](docs/execution-auth-api.md) for the complete endpoint reference including:
- Request/response schemas for all 4 EAM endpoints
- JWT token claims and structure
- Reason codes and error handling
- Rate limit headers

## Architecture

See [docs/architecture.md](docs/architecture.md) for:
- The 5-layer control model (Ingest → Transform → Govern → Authorize → Distribute)
- Protocol integrations (OpenAI ACP, Google UCP, Microsoft MCP, Meta Commerce Graph)
- Connector architecture and data flow
- Enforcement model (V1 log-only → V1.1 active blocking)

## Data Contract

See [docs/data-contract.md](docs/data-contract.md) for:
- Protocol capability discovery
- Verification semantics (verified / inferred / stale / withheld)
- Scope-aware enforcement across chat, cart, compare, and agent_exec contexts
- Event streaming and compliance surface

## Platform Overview

See [docs/platform-overview.md](docs/platform-overview.md) for:
- Why Permyx exists — the execution gap in AI commerce
- The authorization flow in detail
- V1 API surface and enforcement model
- Feed adapters (Google Merchant, OpenAI ACP)
- Public catalog API for agent discovery

---

## Environments

| Environment | API Base URL | Portal |
|-------------|--------------|--------|
| Sandbox | `https://api.sandbox.permyx.io` | `https://portal.sandbox.permyx.io` |
| Production | `https://api.permyx.io` | `https://portal.permyx.io` |

All beta users start in the sandbox environment.

---

## Tech Stack

| Layer | Technology |
|-------|------------|
| API | TypeScript, Hono, Node.js |
| Portal | Next.js 14, React, Tailwind CSS |
| Functions | Azure Functions (TypeScript) |
| Database | Azure Cosmos DB (NoSQL) |
| Messaging | Azure Service Bus |
| Auth | RS256-signed JWTs, OAuth 2.0 |
| Infrastructure | Azure (App Service, Static Web Apps, Key Vault) |
| IaC | Bicep |
| Monorepo | pnpm workspaces, TypeScript |

---

## V1 — Log-Only Mode

V1 operates in **log-only mode**. No checkout is blocked. Authorization decisions are logged for merchant visibility. This lets merchants integrate and observe real execution patterns before enforcement activates in V1.1.

### V1.1 — Coming Next

- Checkout blocking on invalid/missing token
- Shopify native Checkout Function integration
- Token burn-on-use enforced at validation
- Policy enforcement (SKU whitelist, cart value caps, velocity limits)

---

## Contributing

See [CONTRIBUTING.md](CONTRIBUTING.md) for how to get involved.

## Support

- **Email:** developers@permyx.io
- **Website:** [permyx.io](https://permyx.io)
- **Beta Access:** [permyx.io/beta](https://permyx.io/beta)

## License

Copyright 2026 Permyx. All rights reserved.
