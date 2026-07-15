# Core modules and boundaries

A concise overview of internal Core modules, their responsibilities, and dependency rules.
For the overall architectural context, see [overview](./overview.md).

## Dependency principle

Modules may depend only “downward” in the hierarchy. Example: `core.apps` can call
`core.tenancy` and `core.access`, but `core.access` must not import `core.apps`.

## core.platform (kernel „spine“)

- Config: configuration loading, validation, read-only runtime config
- Context: `RequestContext`, `TenantContext`, `ActorContext`
- Policy pipeline: middleware ordering (non-bypassable)

## core.identity

- Users
- Credentials (password, SSO identities, API tokens)
- Sessions / token issuance
- Password policies, MFA hooks (if any)

## core.tenancy

- Tenants
- Tenant settings
- Departments (tenant-scoped)
- Tenant resolver (domain/header/claim)
- DB routing strategy (single-tenant, db-per-tenant, row-level)

## core.access

- Privileges / roles
- Permission evaluation
- Policy engine
- Delegation (action-scoped)
- Impersonation (admin-scoped, no cross-tenant unless superadmin)

## core.licensing

- License store
- License signature validation
- Entitlement model (features + limits)
- License context builder (per-tenant)
- License API (apps query here)

Detailed target architecture and contracts:

- [Licensing overview](../licensing/overview.md)

## core.apps

- App registry (install/upgrade/enable/disable)
- Manifest parser + validator
- Route registration for app APIs
- UI integration metadata (menu items, surfaces)
- Schema provisioning trigger (per tenant)

## core.events

- Event store (durable)
- Delivery coordinator
- Consumer state (processed IDs / checkpoints)
- Retry policy
- Event API for apps

## core.audit

- Structured, backward-compatible audit event recording
- Tamper-evident strategy (at minimum: append-only, signed hashes optional)
- Tenant-safe query/report endpoints with cursor pagination
- “actor metadata” injection (impersonation/delegation/license state)
- See [AAA accounting and audit log](./audit-log.md).

## core.messaging

- Internal messages
- Notifications
- Mail abstraction (SMTP/provider adapters)
- Templates (optional)

## core.secrets

- App credential manager
- Secret storage abstraction
- Per-app identity and access control to secrets

## core.installer

- One-time install workflow
- Database provisioning
- Admin bootstrap
- Disable/self-destruct logic (installer lock)

## core.web

- API server
- Routing, middleware, auth
- Health checks, readiness
- Static UI shell serving (if the core hosts UI)
