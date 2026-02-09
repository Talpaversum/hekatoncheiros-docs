# Hekatoncheiros App Manifest Specification
> Draft v0.1

## Purpose

The App Manifest is the single, authoritative declaration of how an application integrates with the Hekatoncheiros Platform Kernel. The manifest declares UI contribution, not UI execution.

It exists to:

- enforce isolation
- enable safe multi-tenancy
- support licensing and marketplaces
- enable automated validation (by core and by agents such as Cline)
- prevent implicit or undocumented coupling

An app cannot be installed or enabled without a valid manifest.

## Manifest Scope and Guarantees

The manifest declares:

- identity and ownership
- data ownership
- permissions and privileges
- licensing model
- integration points (API, events, UI)
- collaboration constraints

The Platform Kernel guarantees that:

- the manifest is validated before app activation
- declared contracts are enforced
- undeclared behavior is rejected

## App Identity

Each app must declare:

- `app_id`
  - MUST be globally unique across all vendors and marketplaces
  - MUST be immutable for the lifetime of the application
  - MUST be stable across releases
  - SHOULD use reverse-domain notation
  - MUST NOT be assigned by the core
  - random UUIDs are discouraged
  - hash-only identifiers are discouraged
  - examples:
    - `com.talpaversum.inventory`
    - `io.acme.warehouse`
    - `cz.example.asset_registry`
- `app_name`
  - human-readable
  - not required to be unique
- `version`
  - semantic version
  - used for compatibility checks
- `vendor`
  - legal or organizational identity
  - used for marketplace attribution

## Tenancy Awareness

Apps are tenant-agnostic by design.

The manifest must declare:

- whether the app is:
  - tenant-scoped (default)
  - supports cross-tenant collaboration (explicit opt-in)

### Cross-tenant collaboration rules

Cross-tenant collaboration is mediated exclusively by the core.

Apps never resolve foreign tenant identities directly.

Sharing scope is defined by:

- inviting tenant policy
- user privileges
- app-declared collaboration surface

Apps must explicitly declare:

- what objects (if any) are shareable
- what operations are allowed in shared context (read, comment, act)

## Data Ownership Declaration

Each app must declare:

- the schemas it owns
- that it does not access any other schema

Rules:

- one app → one or more schemas
- no cross-app foreign keys
- no shared tables
- no implicit joins

This declaration allows:

- schema provisioning
- permission enforcement
- migration isolation
- AGPL boundary clarity

### Schema derivation from app_id

Application database schema is derived from `app_id`.

- Schema format: `app_${sanitized(app_id)}`
- Sanitization rules:
  - lowercase
  - allowed characters: `a–z`, `0–9`, `_`
  - all other characters are replaced with `_`

### PostgreSQL identifier length limit

- PostgreSQL identifiers are limited to 63 characters
- If the derived schema name exceeds this limit:
  - it is truncated
  - a short, stable hash suffix is appended
  - the hash is used only to preserve uniqueness
- the hash is not the application identity
- the hash is not part of the manifest

## Permissions and Privileges

Apps must declare:

- required privileges
- optional privileges
- privilege scopes (tenant / department / group)

Privileges are:

- defined by the core
- evaluated by the core
- consumed by the app

Apps may never:

- define new privilege semantics
- bypass privilege evaluation
- escalate privileges

### Privilege namespaces

Reserved namespaces (core/platform only):

- `core.*`
- `platform.*`
- `tenant.*`

Applications MUST NOT declare these namespaces in manifest `required_privileges`.

Canonical app-scoped privilege format:

- `app:<app_id>:<priv>`

Examples:

- `app:vendor.crm:contacts.read`
- `app:vendor.crm:contacts.write`

### Privilege scopes (grant scope)

Privilege identifiers remain strings, but grants are scoped by `tenant_id`:

- platform-scope grant: `tenant_id = NULL`
- tenant-scope grant: `tenant_id = <tenant-id>`

`/context` returns effective privileges for the current tenant context:

- all platform-scope grants
- plus tenant-scope grants for the current tenant

Examples:

- `platform.superadmin` is platform-scope (`tenant_id = NULL`)
- `tenant.config.manage` can be granted only for tenant `A` and absent for tenant `B`

### Superadmin wildcard

`platform.superadmin` acts as wildcard for any `platform.*` privilege checks.

It is NOT a wildcard for:

- `core.*`
- `tenant.*`
- app-scoped privileges (e.g. `app:<app_id>:...`)

## Impersonation and Delegation

### Impersonation

Declared support must specify:

- whether impersonation is allowed
- which roles may impersonate
- scope of impersonation (tenant / department / group)

Rules:

- full impersonation is allowed only for admins of the relevant scope
- no cross-tenant impersonation
- platform superadmin is the only exception
- all impersonation is auditable and explicit

### Delegation

Apps may opt into delegation support.

Delegation means:

- User A authorizes User B to perform specific actions on their behalf

Scope is:

- action-limited
- time-limited
- revocable

Delegation is:

- granted by the user
- evaluated by the core
- enforced by the app

## Licensing Declaration

Each app must declare its license model, independently of the core.

The manifest must specify:

- license types supported:
  - perpetual
  - time-limited
  - scale-limited (users, workload, entities)
- feature flags tied to license state
- behavior on license expiry

### Offline licensing model (clarified)

- license activation is explicit and manual
- license is cryptographically validated by the core
- no online checks are required after activation
- no early revocation exists

### On license expiration

- features are disabled non-destructively
- app enters read-only mode

Existing data remains:

- accessible
- queryable
- available to other apps via APIs

No automatic deletion or mutation is allowed.

Core responsibility:

- validate license
- report license state and limits

App responsibility:

- enforce limits
- enforce feature availability

## API Integration

Apps must declare:

- APIs they expose
- APIs they consume from the core

Rules:

- all core interaction happens via API
- no direct DB access to core data
- APIs are versioned
- backward compatibility rules are enforced by the core

## Event Integration

Apps must declare:

- events they emit
- events they consume

### Delivery semantics (clarified)

- exactly-once delivery is a design goal
- implementation may internally use retries, deduplication, or persistence
- apps must be idempotent
- apps must tolerate retries

From the app’s perspective:

- an event is processed once
- duplicates must not cause side effects

## UI Integration

Apps may declare:

- navigation entries
- UI surfaces
- configuration panels

Rules:

- UI is hosted and rendered inside the platform shell
- apps do not own routing, rendering, or authentication
- apps provide UI components, not standalone web applications
- no user-facing UI may be served directly by the app
- visibility is privilege-based and enforced by the core

### UI artifact source declaration and runtime URL

- The manifest MAY declare a reference to the UI plugin artifact source.
- The manifest MUST NOT define where or how the artifact is hosted at runtime.
- The manifest MUST NOT depend on any specific location inside the web shell.
- Runtime `ui_url` in the app registry MUST be generated by core after
  installation.

Relationship model:

- manifest → declaration of artifact source reference
- installer → artifact download and validation
- app registry → runtime `ui_url` generated and owned by core

## Prohibited Behavior (Hard Enforcement)

Apps may never:

- manage users, tenants, or privileges
- modify another app’s data
- access undeclared schemas
- bypass licensing information
- bypass audit logging
- perform cross-tenant actions without explicit core mediation

Violations result in:

- app disablement
- marketplace rejection
- administrative intervention

## Validation and Enforcement

Manifests are validated at:

- install time
- upgrade time

Incompatible changes require explicit admin approval.

Core may refuse to load an app with:

- missing declarations
- privilege violations
- unsafe collaboration scopes

### Manifest validation: reserved required_privileges

Any manifest `required_privileges` entry that starts with a reserved prefix is rejected.

Rejected examples:

- `platform.apps.manage`
- `core.something`
- `tenant.config.manage`

Allowed examples:

- `app:vendor.crm:contacts.read`
- `hc-app-inventory.items.read`

Error message:

`Invalid required_privilege "<X>": reserved namespaces core./platform./tenant. are not allowed in app manifests.`

Seed baseline used by core (default admin):

- `platform.superadmin` (platform-scope)
- required `core.*` privileges (platform-scope)

## Status

This manifest specification is intentionally strict.

It trades:

- flexibility → safety
- convenience → long-term maintainability
- implicit behavior → explicit contracts

This is aligned with:

- AGPL core
- marketplace goals
- multi-tenant self-hosting
- agent-assisted development
