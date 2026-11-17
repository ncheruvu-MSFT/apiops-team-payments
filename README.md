# apiops-team-payments

APIOps pipeline for **Team Payments**' slice of the shared Crowe demo API Management instance
(`apim-y3ownlt7uuuqu`, resource group `apim-infra-simple-apim-2`). One of 3 sibling repos
(`apiops-team-orders`, `apiops-team-payments`, `apiops-team-inventory`) demonstrating the
**Polyrepo** multi-team APIOps pattern with a live RBAC guardrail against cross-team deletion.

## What this repo owns

| Resource | Name |
|---|---|
| API | `team-payments-api` (path `/team-payments`) |
| Named value | `team-payments-backend-url` |
| Subscription | `sub-team-payments-api` |

See [`NAMING-CONVENTIONS.md`](./NAMING-CONVENTIONS.md) for the full naming policy all 3 team repos
follow, and the Spectral ruleset (`.spectral.yaml`) that enforces it as a real CI lint gate.

## How this repo authenticates to Azure

GitHub OIDC -> the **`id-apiops-team-payments`** user-assigned managed identity (no stored client
secret). That identity is granted the custom **"APIOps Team Publisher (No Delete)"** RBAC role,
scoped to the `apim-y3ownlt7uuuqu` resource only -- it can create/update any APIOps-managed
resource but **cannot delete anything**, including its own `team-payments-api`.

## Pipelines

- **`run-extractor.yml`** (manual `workflow_dispatch`) -- pulls `team-payments-api` config from
  the live APIM into `apimartifacts/`, opens a PR with the diff.
- **`run-publisher.yml`** (auto on push to `main`) -- lints the OpenAPI spec against
  `.spectral.yaml` (**blocking**), then publishes via the real `Azure/apiops` v7.0.4 publisher
  binary. **Never uses `--delete-unmatched`**.

## Why this doesn't collide with Team Orders or Team Inventory

1. `apimartifacts/` here only ever contains `team-payments-*` resources.
2. The managed identity's RBAC role has no delete permission on *anything*.
3. The Spectral ruleset blocks any path or `operationId` that doesn't start with the
   `team-payments` prefix before the pipeline ever reaches the publish step.
