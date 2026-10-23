# Pagination, Filtering, Sorting

List endpoints are cursor-paginated and expose filtering and sorting through documented query parameters.

## Cursor pagination

Request:

```
GET /orders?limit=50&cursor=eyJpZCI6Im9yZF8xMjMifQ
```

Response:

```json
{
  "data": [ { "id": "ord_124" } ],
  "page": {
    "nextCursor": "eyJpZCI6Im9yZF8xNzMifQ",
    "hasMore": true
  }
}
```

- `limit`: default 50, maximum 200; out-of-range values are clamped, not rejected.
- `cursor`: opaque, URL-safe, treated as write-only by clients. Cursors expire after 24 hours; an expired cursor returns 400 with code `cursor_expired`.
- Iterate until `hasMore` is false. An absent `nextCursor` also means the end.
- The ordering behind the cursor is stable (see sorting), so pages never skip or duplicate rows while data changes.

No offset pagination: offsets drift under concurrent writes and degrade at depth.

## Filtering

Filters are query parameters named after the field:

```
GET /orders?status=open&createdAfter=2026-01-01T00:00:00Z
```

- Multi-value filters repeat the parameter: `?status=open&status=pending` (OR within a field, AND across fields).
- Unknown filter parameters are rejected with 400 `unknown_parameter`; silently ignoring them hides client typos.
- Date filters use ISO 8601 UTC. `createdAfter` is exclusive, `createdBefore` inclusive.

## Sorting

```
GET /orders?sort=-createdAt
```

- Comma-separated for tie-breakers: `?sort=status,-createdAt`.
- The `-` prefix means descending.
- The final implicit tie-breaker is always the resource ID, ascending; that is what keeps cursor pages stable.
- Sortable fields are whitelisted per endpoint and documented; anything else is 400 `invalid_sort`.

## Why

Cursor pagination plus a deterministic sort gives clients exactly-once page semantics during live writes, which offset and limit cannot promise.
