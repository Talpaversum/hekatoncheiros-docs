# Hekatoncheiros Roadmap

Task status is marked consistently throughout this document:

- `[x]` completed,
- `[ ]` planned,
- open design decisions are separated from implementation tasks.

## Phase 1 - Local Installable Apps

### Completed

- [x] DB-per-tenant model (logically separated within one physical database for
  the MVP)
- [x] Schema-per-app (`app_<app_id>`)
- [x] Migrations through Core (`/.well-known/hc/migrations*`)
- [x] SHA-256 migration verification
- [x] License enforcement for tenant runtime navigation and API access
- [x] Signed manifests
- [x] Installation lifecycle
- [x] Local application catalog and catalog feeds
- [x] Installation modes: `external`, `stage_only`, and opt-in Core-managed
  Compose
- [x] Manual feed source synchronization and public instance feed export
- [x] Publish/unpublish management for installed applications
- [x] Core storage and manual refresh of UI plugin artifacts
- [x] Core Console for applications, tenants, users, RBAC, licensing, and
  configuration
- [x] Help entries provided by application manifests

### Completed Runtime Foundation

- [x] Deployment plan validation including `package_url`, SHA-256, Compose file,
  service name, and internal URL
- [x] Safe download and extraction of `tar.gz` packages with rejection of unsafe
  paths, links, and unsupported file types
- [x] Compose policy guard prohibiting published ports, host mounts, privileged
  mode, additional capabilities, and host network/PID/IPC
- [x] Opt-in Docker Compose adapter and a verified `hc-app-inventory` runtime
  package with a shared network and health check
- [x] Persistent ownership of Core-managed runtimes and runtime-aware uninstall
- [x] Manual manifest, catalog, and UI artifact checks and refreshes, including
  stored `update_signal` data and catalog stale/update summaries
- [x] App-auth update signal webhook and manual issuance of short-lived app
  runtime JWTs
- [x] Audited administrator approval, bound to the deployment plan plus manifest
  and package hashes, before starting a Core-managed runtime

## Active Work

### Next

- [x] Add `stop` and `update` lifecycle actions for Core-managed Compose
  runtimes.
- [x] Securely deliver and rotate app runtime tokens in Core-managed
  applications without manual copying.

### Later

- [x] Add feed/author-signed update signals for sources outside the running
  application's identity.
- [x] Add optional automatic refresh for trusted/official sources.
- [x] Add audit logging and policy guards for automatic runtime UI changes.
- [x] Add HTTPS through proxy or ingress termination.
- [x] Expand and verify bare-metal/VM and Kubernetes deployments.
- [x] Integrate `hc-author-registry` into author onboarding: issue `author_id`
  and `author_cert_jws`, manage public keys, and publish JWKS/revocation data.

## Phase 2 - Core2Core Integration

Core2Core is deferred and is not part of the current implementation scope.

- [ ] Remote applications
- [ ] Scheduled catalog feed synchronization
- [ ] `suggest app for feed` workflow with pending publication requests
- [ ] Publish tokens for pre-approved namespaces, applications, and CI pipelines
- [ ] Namespace trust and protected namespace policy
- [ ] OAuth trust
- [ ] Separate databases with no shared DB
- [ ] Signed cross-core tokens
- [ ] Signed migration bundles

## Open Decisions

These items require design decisions rather than implementation alone.

- [ ] Resolve database ownership for standalone applications. Define ownership
  of database credentials, migrations, backup/restore boundaries, and lifecycle
  state when an application is not operated by the target HC instance but uses
  its tenant/app database model.
- [ ] Reconsider the `hc-author-registry` boundary: a standalone service or an
  optional authority mode in Core.
- [ ] Decide the product boundary of `hc-app-licensing`: a standalone
  author/vendor service or an installable HC application with a manifest and
  administration UI.
- [ ] If `hc-app-licensing` becomes an installable application, add its manifest,
  runtime package, UI plugin, and management of customers, grants, licenses, and
  revocations.

### Application-Owned Mobile Clients

- [ ] Define the client boundary. A mobile client is an optional presentation
  channel for an installed application's backend, not a Core UI or a replacement
  for the Core web shell. Decide whether HC supports web-only, web-and-mobile,
  mobile-only, and future client combinations; mobile-only support would require
  changing the current UI-artifact installation contract. Business logic must
  remain owned by the application backend.
- [ ] Define mobile authentication and instance/tenant discovery using
  Core-owned identity, tenant, privilege, licensing, and application access
  context. Specify client registration, login, token refresh and revocation,
  logout, and selection of the correct HC instance and tenant.
- [ ] Define mobile metadata in the application manifest: supported platforms,
  bundle/package identifiers, store references, deep links, redirect URIs,
  minimum client versions, and requested API scopes and permissions.
- [ ] Decide whether mobile clients access application backends directly or
  through Core-managed ingress. Every request must preserve identity, tenant,
  privilege, licensing, and audit context without bypassing Core security
  boundaries. Client-specific API façades may exist, but business rules and
  authorization must remain centralized in the backend.
- [ ] Define multi-client lifecycle and compatibility policy across the backend,
  web UI plugin, and mobile clients, including updates, backward compatibility,
  and minimum supported versions.
- [ ] Define optional platform capabilities and their security contracts: push
  notifications, deep links, device registration, offline synchronization,
  background processing, and secure local storage.
- [ ] Define tenant administration for mobile access, client-specific policies,
  device revocation, remote logout, and audit of mobile authentication and
  device activity.
- [ ] Decide whether vendors distribute mobile clients independently or expose
  trusted store metadata through the HC catalog, including the integrity and
  publisher-verification model available on each platform.

A mobile client must not become an alternative Core administration interface
unless a dedicated Core administration API and security model are explicitly
designed and approved.

## Current Deployment Principles

- Docker Compose is the currently verified path for local self-hosting.
- Development installations build images locally; no public image registry is
  used yet.
- Development deployments default to HTTP; optional Compose ingress termination
  provides HTTPS with operator-supplied certificates.
- Bare-metal/VM and Kubernetes have validated reference configurations; they
  are not yet packaged installers or Helm releases.
