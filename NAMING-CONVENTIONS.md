# APIM Naming Conventions & OpenAPI Spec Guidance

Shared policy across `apiops-team-orders`, `apiops-team-payments`, `apiops-team-inventory`.

## The core rule

> Every resource a team creates must be prefixed with that team's name. Every shared/platform
> resource must have NO team prefix, and is owned exclusively by the platform team.

## Naming table

| Resource type | Convention | Team Payments example | Platform (shared) example |
|---|---|---|---|
| API id / name | `<team>-<resource>-api` | `team-payments-api` | n/a |
| API path | `/<team>-<resource>` | `/team-payments` | n/a |
| API display name | `"<Team> <Resource> API"` | `"Team Payments API"` | n/a |
| Operation id | `<verb>-<resource>[-<detail>]`, kebab-case | `post-refund` | n/a |
| Named value (team-owned) | `<team>-<purpose>` | `team-payments-backend-url` | -- |
| Named value (platform-owned) | `<purpose>` -- no team prefix | -- | `jwt-signing-key` |
| Policy fragment (team-specific) | `<team>-<purpose>` | `team-payments-fraud-check` | -- |
| Policy fragment (shared) | `<purpose>` -- no team prefix | -- | `standard-response-headers` |
| Product | `product-<team>` | `product-team-payments` | `product-starter` |
| Subscription | `sub-<team>-<api>` | `sub-team-payments-api` | -- |
| Backend | `<team>-<resource>-backend` | `team-payments-backend` | -- |
| Tags | `team:<team>`, `domain:<resource>` | `team:payments`, `domain:billing` | -- |

## OpenAPI spec (`specification.yaml`) rules

1. `info.title` must exactly equal the API display name -- `"Team Payments API"`.
2. Every path must be scoped under `/<team>-<resource>`.
3. `operationId` must be kebab-case (`^[a-z][a-z0-9]*(-[a-z0-9]+)*$`) and unique within the API.
4. `info.version` uses semantic-ish strings; breaking changes get a new major version.
5. `servers[0].url` points at the shared gateway's hostname; env overrides live in
   `configuration.<env>.yaml`, never hardcoded per-team in the spec.

## Enforcement: Spectral ruleset (real lint gate, not just a style guide)

See `.spectral.yaml` in this repo. This ruleset is a **real blocking gate** in `run-publisher.yml`.

## Why this, and not just "be careful"

- RBAC enforces *who* can delete what (the "No Delete" custom role).
- Naming + Spectral enforces *what resources look like* -- so a team's config can't silently
  collide with another team's resource name or path.
