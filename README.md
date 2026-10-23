# HTTP API Design

Conventions pack for Camillia workspaces: opinionated HTTP API conventions covering paths, errors, pagination, versioning, and safe retries.

## Conventions at a glance

| File | Convention |
| --- | --- |
| `conventions/resources-and-paths.md` | Plural kebab-case nouns, no verbs in paths, nesting limits. |
| `conventions/errors.md` | The single error envelope and the RFC 9457 problem-details alternative. |
| `conventions/pagination-and-filtering.md` | Cursor pagination, filter and sort query parameters. |
| `conventions/versioning.md` | Version via Accept header, breaking-change policy, sunsets. |
| `conventions/idempotency.md` | `Idempotency-Key` contract for safe POST retries. |

## Install

Install from the Camillia Community Hub. The pack's files are placed under `.workspace/conventions/` in your workspace, where agents and teammates pick them up as the default way of working.

## License

MIT - see [LICENSE](LICENSE). Copyright 2026 Camillia contributors.
