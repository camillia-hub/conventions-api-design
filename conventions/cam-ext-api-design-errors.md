# Error Responses

Every non-2xx response uses the same JSON envelope, so clients write one error-handling path for the whole API.

## Envelope

```json
{
  "error": {
    "code": "order_not_found",
    "message": "Order ord_9f1e2ab does not exist.",
    "details": [
      { "field": "orderId", "issue": "unknown identifier" }
    ]
  }
}
```

- `code`: stable, machine-readable, snake_case, namespaced by domain when helpful, such as `payment_declined`. Codes are additive only; a code is never reused for a different meaning.
- `message`: human-readable, present tense, safe to show to an integrator's support team. Never includes stack traces, SQL, or internal hostnames.
- `details`: optional array; for validation errors, one entry per field.

## Status codes

| Code | When |
| --- | --- |
| 400 | Malformed JSON, schema violations |
| 401 | Missing or invalid credentials |
| 403 | Authenticated but not allowed |
| 404 | Unknown resource (also used to avoid leaking existence across tenants) |
| 409 | State conflict: duplicates, optimistic-lock failures |
| 422 | Well-formed but semantically invalid (business rules) |
| 429 | Rate limited; include `Retry-After` |
| 5xx | Our fault; `code` is `internal_error`, details stay server-side |

## Alternative: RFC 9457 problem details

New external APIs may instead use `application/problem+json`:

```json
{
  "type": "https://api.example.com/problems/order-not-found",
  "title": "Order not found",
  "status": 404,
  "detail": "Order ord_9f1e2ab does not exist.",
  "instance": "/orders/ord_9f1e2ab"
}
```

Pick one format per API and document the choice in the API's README; never mix both in one API. The envelope above remains the default for existing services.

## Why

A single error shape turns error handling from per-endpoint archaeology into shared client code, and stable codes let integrators build alerting that survives our wording tweaks.
