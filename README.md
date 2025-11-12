# Payment Gateway

> ⚠️ **WORK IN PROGRESS. DO NOT IMPLEMENT YET**
>
> The mock bank service is still being built. This spec will be updated.

## What You'll Build

A payment gateway service for FicMart (an e-commerce platform) that handles card payments through a banking partner's API.

The gateway must support the standard e-commerce payment flow:

- **Authorize**: Reserve funds when an order is placed
- **Capture**: Charge the customer when goods ship
- **Void**: Cancel an authorization before capture
- **Refund**: Return money after capture

## System Overview

```

FicMart Order Service → Your Payment Gateway → Mock Bank API (provided)

```

## Mock Bank API

The bank service exposes these endpoints:

### Authorize Payment

```

POST /api/v1/authorizations
Idempotency-Key: <unique-id>

Request:
{
  "card_number": "4532015112830366",
  "cvv": "123",
  "expiry_month": "12",
  "expiry_year": "2025",
  "amount": 9999,
  "currency": "USD"
}

Response 200: { authorization_id, status, amount, expires_at }
Response 400: { error: "invalid_card" }
Response 402: { error: "insufficient_funds" }

```

### Capture Payment

```

POST /api/v1/captures
Idempotency-Key: <unique-id>

Request: { authorization_id, amount }
Response 200: { capture_id, status, captured_at }
Response 400: { error: "authorization_expired" }

```

### Void Authorization

```

POST /api/v1/voids
Idempotency-Key: <unique-id>

Request: { authorization_id }
Response 200: { void_id, status, voided_at }
Response 400: { error: "already_captured" }

```

### Refund Payment

```

POST /api/v1/refunds
Idempotency-Key: <unique-id>

Request: { capture_id, amount }
Response 200: { refund_id, status, refunded_at }
Response 400: { error: "not_captured" }

```

### Query Operations

```

GET /api/v1/authorizations/{id}
GET /api/v1/captures/{id}
GET /api/v1/refunds/{id}

```

### Bank Behaviour

The mock bank simulates real-world APIs:

- Validates cards using Luhn algorithm
- Enforces state transitions (can't capture after void, etc.)
- Respects idempotency keys (duplicate requests return original response)
- Random failures: ~5% of requests timeout or return 500
- Random latency: 100ms-2000ms per request
- Authorizations expire after 7 days

## Gateway Requirements

Your gateway must:

1. **Expose an interface** for FicMart to perform payment operations (authorize, capture, void, refund)
2. **Communicate with the bank API** to execute operations
3. **Handle all failures**: timeouts, transient errors, invalid states, expired authorizations
4. **Ensure idempotency**: duplicate requests don't create duplicate charges
5. **Track payment state**: maintain enough data to reconcile with bank records
6. **Enforce state machine**:

   ```

   PENDING → AUTHORIZED → CAPTURED → REFUNDED
                ↓
              VOIDED

   ```

Invalid transitions must be rejected (e.g., can't void after capture).

*Full specification details and resources will be added when the mock bank service is complete.*
