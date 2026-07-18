# Author workflow architecture

Hekatoncheiros separates three independent decisions: an instance capability says whether a service exists, RBAC says whether a user may operate that service, and author scope says which `author_id` the operation may access. Passing one check never implies passing either of the others.

## Instance capabilities

Core exposes the authenticated `GET /api/v1/platform/capabilities` endpoint. Capabilities are configured through `INSTANCE_CAPABILITIES_JSON`; the value is a JSON object whose keys are capability names and whose values are booleans. Existing private installations have safe defaults: `privateAppDevelopment`, `trustedOrigins`, and `privateCatalogs` are enabled, while official registry, catalog, and hosted services are disabled. Registry capabilities also remain unavailable until `AUTHOR_REGISTRY_URL` is configured.

Capability-dependent backend operations return a stable error code in the response details. Disabled official registry and catalog operations use `official_registry_not_available` and `official_catalog_not_available`; an enabled but incomplete service uses `capability_not_configured`.

## Separate user interfaces

- **Developer Tools** is the private-instance entry point. Local applications, feeds/private catalogs, trusted origins, runtime, and local licensing do not require an official author identity or Author Registry.
- **Author Workspace** is available to members of approved author profiles. It contains author applications, Git connections, team membership, licensing, submissions, and author activity. A user with several memberships selects an active author; the selection is persisted in the browser and validated again by Core for every scoped request.
- **Registry Administration** is a separate operator interface. It requires both the official registry capability and platform RBAC. Author approval, catalog approval, and runtime approval remain separate workflows and audit events.

Private self-hosted development is not an official author mode and is never offered by the “Become an Author” form. That form is only for Talpaversum-hosted and trusted self-hosted author identities.

Opening a disabled module by a direct URL shows an unavailable-service explanation; hiding navigation is not the security boundary.

## Author-scoped RBAC

Author-owned Git connections, applications, catalog submissions, memberships, and workflow events carry an `author_id`. Core derives access from an active `core.author_memberships` row and the permission set for that membership. Workspace list queries join the membership table and optional active-author filters are checked before the query runs. Platform privileges do not silently convert a workspace query into a global registry query.

Git credentials stay in Core, are encrypted at rest, and are never returned by the API. Disconnecting a connection removes the stored ciphertext. Registry and licensing services receive public trust/licensing data, not repository credentials.

## Trust, catalogs, and outages

The Author Registry is an official Talpaversum service, not a required Core dependency. Private installations continue to install direct manifests and use configured private catalog feeds while the registry is absent or unavailable. Official catalog publishing is distinct from trusted private catalogs and direct, unverified sources.

Existing trust and revocation snapshots remain locally usable during an upstream outage. Operations that require current registry state must report their failure without disabling private development or already stored author workspace data.

Suspending or revoking an author blocks new trusted publication; it does not implicitly stop every existing local installation. Operators must evaluate installed applications, catalog records, issued licenses, and snapshot age separately. New application versions are validated from their manifest and changes to runtime, permissions, origins, identity, or licensing trust require their corresponding review again.

## Migration

No registry URL or official capability is introduced automatically by migration. Therefore upgrading an existing private installation does not add an Author Registry dependency or expose registry administration. Official Talpaversum deployments must explicitly enable and configure the capabilities they operate.

The production Author Registry deployment requires an external `DATABASE_URL`, Core delegated authentication, and root public/private JWKS mounted as protected files. It never provisions PostgreSQL or generates root keys. The separate development Compose file is explicitly non-production. Registry migrations are discovered by filename and recorded transactionally in `registry_schema_migrations`.
