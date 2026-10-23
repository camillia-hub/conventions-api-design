# Idempotency for Unsafe Methods

Networks fail after the server has committed. Clients must be able to retry POST safely, so every POST that creates or mutates accepts an idempotency key.

## Client contract

```
POST /orders
Idempotency-Key: 01J9Z8K6WQ3F4N2R0T5Y7X9ABC
```

- The key is a client-generated unique value (UUIDv7 or ULID recommended) per logical operation, reused across retries of that operation.
- Keys are scoped per endpoint and per authenticated principal.
- Safe methods (GET, PUT and PATCH with preconditions, DELETE) need no key: they are idempotent by semantics.

## Server behavior

- The first request with a key executes; the key, a hash of the request, and the response are stored for 24 hours.
- A retry with the same key and an identical request body replays the stored response, including its status code, with the header `Idempotency-Replayed: true`.
- A retry with the same key but a different body is rejected: 409 with code `idempotency_key_reuse`.
- Concurrent requests with the same key: exactly one executes; the others wait and replay its result.

## Rejection example

```json
{
  "error": {
    "code": "idempotency_key_reuse",
    "message": "Idempotency-Key 01J9Z8... was already used with a different request body.",
    "details": [
      { "field": "Idempotency-Key", "issue": "reuse with mismatched payload" }
    ]
  }
}
```

## Implementation notes

- Store the key in the same transaction that commits the mutation, so a crash mid-write cannot split key and effect.
- Scope expiry by business need: payments 24 hours, imports 7 days. Expired keys are treated as new operations, so clients must not retry after expiry anyway.
- Document per endpoint whether the key is required or optional; new money-moving endpoints must require it.

## Why

Idempotency keys turn "retry and pray" into a protocol: clients can auto-retry through timeouts and 5xx responses without double-charging, double-booking, or double-sending.
