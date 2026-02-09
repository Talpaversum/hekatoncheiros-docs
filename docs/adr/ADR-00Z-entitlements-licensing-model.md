# ADR-00Z: Entitlements & Licensing Model

## Status
Accepted

## Context

Platform supports multi-tenant runtime, multiple entitlement sources, and offline-capable operation.
Historically, licensing state was represented as a single license row per `(tenant, app)` and mixed
installation and licensing semantics.

We need a final model where:
- apps can be installed without license,
- DB is runtime source of truth,
- online/offline are just entitlement sources,
- deterministic resolution applies when multiple entitlements are active.

## Decision

### 1) DB-first runtime truth

Runtime authorization and app visibility are resolved from Core DB tables:
- `core.tenant_app_entitlements`
- `core.tenant_app_selection`
- `core.offline_license_tokens`

Online licensing and offline token ingest are writers into this model.

### 2) Installed vs entitled

Installed app (`core.installed_apps`) does not imply entitlement.

`/apps/registry` visibility is:
`installed ∩ enabled ∩ entitled(active, resolved) ∩ privilege checks`.

### 3) Selection + deterministic fallback

Selection is explicit per `(tenant, app)` via `core.tenant_app_selection`.

If selected entitlement is valid, it wins.
If missing/invalid, resolver uses deterministic fallback over valid entitlements:
1. `source`: `OFFLINE > ONLINE`
2. `tier`: `enterprise > standard > trial > free` (`3 > 2 > 1 > 0`, unknown `-1`)
3. `valid_to`: later wins
4. `created_at`: newer wins
5. `id`: final tie-breaker

### 4) Offline portability boundary

Offline tokens are non-transferable by requiring:
- `aud == platform_instance_id`

`platform_instance_id` is stored in `core.platform_instance` and is stable.

### 5) Clock skew and offline time behavior

Resolver and token verification support configurable clock-skew tolerance.

When entitlement is outside strict window but within configured soft grace window,
Core may continue operation and logs warning/audit instead of hard-blocking.

## Consequences

- Multiple active entitlements per tenant/app are supported.
- App runtime always gets a resolved entitlement (`tier/limits/source/window`).
- Offline entitlement ingest works without online dependency after token delivery.
- Core behavior is deterministic and reproducible.
