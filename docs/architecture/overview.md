# Architecture Overview

Status: draft, early development.

Hekatoncheiros Core is a platform kernel, not a business application.

Core owns:

- identity and access control
- tenant context
- app registry and lifecycle
- licensing state
- audit/event boundaries
- UI plugin runtime exposure

Core does not own:

- app business logic
- app data semantics
- app query optimization
- app-specific limits beyond reporting license state

## Current MVP Shape

The current implementation is intentionally small:

- Fastify API under `/api/v1`
- PostgreSQL
- row-level/logical tenancy as the practical MVP mode
- React web shell in a separate repository
- app installation and UI plugin storage still evolving

DB-per-tenant and stronger installer automation are target architecture, not
fully implemented runtime behavior yet.

## Request Pipeline

Every request should follow the same conceptual path:

1. Ingress
2. Authentication
3. Tenant resolution
4. Privilege evaluation
5. License context resolution
6. Route to Core or app integration
7. Audit where appropriate

The important rule is that apps should not bypass Core-owned identity, tenant,
privilege, or licensing context.

## Tenancy and Data

Target model:

- single-tenant self-host: one DB, app schemas
- DB-per-tenant: one DB per tenant, `core` schema plus app schemas
- row-level tenancy: shared DB with tenant scoping

Current MVP:

- shared PostgreSQL database
- Core schema
- app schema isolation where app support exists

Apps must not access the Core schema directly.

## App Model

Applications integrate through:

- manifest metadata
- optional backend service
- UI plugin artifact
- declared API/event/privilege contracts

The web shell renders app UI. Apps must not provide their own user-facing login
or standalone SPA as the primary platform UI.

Core must own installed UI plugin artifacts and expose runtime `ui_url` values.
The web shell loads from Core-provided URLs.

The application catalog is discovery and acquisition state. Installing from the
catalog creates or stages local runtime state:

- `external`: Core stores the installed app and connects to an already running
  app base URL.
- `stage_only`: Core records the selected plan without enabling runtime use.
- `compose`: target mode where Core starts an app-owned compose bundle after
  admin approval; currently planned, not fully implemented.

Licensing does not block installation. It blocks tenant runtime access when the
manifest declares `licensing.required=true` and no selected active license
exists.

## Licensing

Core validates and stores license/entitlement state.

Apps consume license context and enforce app-specific behavior such as:

- read-only mode
- feature availability
- usage limits

License expiry must be non-destructive.

## Open Questions

- final app migration authority for all installation modes
- standalone app database ownership
- Core-managed compose runtime manager and deployment policy
- production installer flow
- production packaging for Docker and Kubernetes
- custom app hostnames and tenant resolution

Keep detailed decisions in ADRs and focused reference docs rather than growing
this overview.
