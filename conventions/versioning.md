# API Versioning

APIs evolve without breaking existing clients. The version travels in a header, not a path segment, so URLs stay stable and cache-friendly across versions.

## Version header

Clients send the version as a vendor media type:

```
Accept: application/vnd.example.v2+json
```

- Requests without the header receive the oldest still-supported version.
- Responses state the served version: `Content-Type: application/vnd.example.v2+json`.
- An unknown version is rejected with 406 and code `unsupported_version`, listing the supported versions in `details`.

## What needs a new version

Only breaking changes: removing or renaming fields, changing types or semantics, tightening validation, changing defaults, removing endpoints.

Additive changes ship in the current version: new endpoints, new optional fields, new enum values. Clients must tolerate unknown enum values; that requirement is part of the client guide.

## Sunset policy

- A version is supported for at least 12 months after its successor ships.
- Deprecation is announced with response headers (RFC 8594 style) and the developer changelog:

```
Deprecation: Fri, 01 May 2026 00:00:00 GMT
Sunset: Sat, 01 May 2027 00:00:00 GMT
```

- After the sunset date the version returns 410 Gone with code `version_sunset`.

## Within a version

Additive changes are dated in the changelog only; there are no per-request date headers. If a behavior fix is not safe for all clients of the version, it is not a fix; it is the next version.

## Why

Header versioning keeps one canonical URL per resource, lets clients upgrade on their own schedule, and makes the supported surface explicit in every response.
