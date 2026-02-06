# ADR-00Y: Core-triggered App-Managed Migrations

## Status
Proposed

## Context

The Core provisions application schemas and manages app lifecycle state.
Database migrations are currently executed by application backends.

We need a safe, auditable mechanism to trigger migrations per tenant without:

- granting apps direct access to the Core schema
- running third-party SQL inside the Core process
- weakening isolation guarantees

The mechanism must preserve tenant scoping, be repeatable, and provide strong observability.

## Decision

The Core will act as an orchestrator only. The application backend will execute migrations.

### 1) Interface

- Each app that supports migrations MUST expose an internal endpoint
  (e.g. `POST /internal/migrate`).
- The endpoint MUST be protected by mTLS or a Core-signed token.
- The Core MUST include tenant scope in each migration request.
- The endpoint MUST NOT be publicly routable.

### 2) Idempotence

- Repeated migration requests MUST NOT corrupt state.
- The app MUST report the latest applied migration version.
- The app MUST ignore already applied migrations.

### 3) Observability

The Core MUST log and/or audit the following states:

- migration requested
- migration started
- migration finished
- migration failed

### 4) Failure modes

- The Core MUST apply a bounded retry policy on timeouts.
- Partial failures MUST be reported with clear status.
- Automatic rollback is **not** performed.
- Any rollback strategy is the responsibility of the app.

### 5) Security boundaries

- The endpoint is internal only.
- The Core MUST NOT run app SQL.
- The app MUST NOT access the Core schema.

### 6) Manifest declaration

Apps MUST declare migration support in the manifest:

- `integration.migrations.mode = "app-managed"`
- `integration.migrations.endpoint = "/internal/migrate"`
- `integration.migrations.version_endpoint = "/internal/migrations/version"`

The Core uses this declaration to validate compatibility and orchestration.

### 7) Alternatives

- **Core runs app SQL**: rejected (breaks isolation, unsafe).
- **Artifact runner service**: deferred (requires additional infra).

## Consequences

- Higher coordination complexity between Core and app backends.
- Stronger security and isolation guarantees.
- Marketplace-compatible behavior with explicit contracts.

## Rejected Alternatives

- Core executes third-party SQL directly.
- External runner that pulls and executes app artifacts (deferred).
