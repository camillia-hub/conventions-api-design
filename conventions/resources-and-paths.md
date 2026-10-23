# Resources and Paths

URLs name things, not actions. A path identifies a resource; the HTTP method supplies the verb. This keeps the API predictable and lets caching, routing, and permissions key off the path.

## Rules

- Plural nouns for collections: `/customers`, `/orders`, `/invoices`.
- Kebab-case for multi-word segments: `/billing-accounts`, `/payment-methods`.
- Lowercase always; path casing differs across stacks, so we remove the ambiguity.
- No verbs in resource paths: `/orders/{id}/cancellation`, not `/orders/{id}/cancel`.
- IDs are the final segment: `/orders/{orderId}`. Nesting deeper than one level is a smell; prefer a flat path with a filter, `/lines?orderId={id}`, over `/orders/{id}/lines/{lineId}`.

## Layout

```
GET    /customers
POST   /customers
GET    /customers/{customerId}
PATCH  /customers/{customerId}
DELETE /customers/{customerId}

GET    /customers/{customerId}/orders     # one level of nesting, fine
GET    /orders?customerId={customerId}    # equivalent flat form, preferred at depth 2+
```

## Actions that are not CRUD

Model the action as a sub-resource or a state transition, never as a verb in the path:

```
POST  /orders/{orderId}/cancellation      # creates a cancellation resource
PATCH /orders/{orderId}                   # { "status": "cancelled" } is also fine
POST  /reports/{reportId}/regeneration    # async re-run
```

## Trailing slashes and duplicates

No trailing slashes: `/customers/` and `/customers` are not two resources. We do not add redirects to paper over either form; clients send the canonical form.

## Why

Noun-only, flat, kebab-case paths make URLs guessable in the good sense: a developer who knows `/orders` can predict `/orders/{id}/cancellation` without reading the docs.
