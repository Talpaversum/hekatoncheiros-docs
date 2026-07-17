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
- [ ] Connect approved hosted author apps to a build worker that checks out the
  selected Git revision, produces an immutable runtime package, and hands its
  reviewed deployment plan to the existing Core runtime manager.
- [ ] Materialize approved author catalog submissions into the public catalog
  feed, including immutable manifest/package references for hosted apps and
  verified external manifest/feed plus issuer discovery for trusted
  self-hosted apps.
- [ ] Provision and route Talpaversum-hosted Licensing as an isolated managed
  issuer namespace per `author_id`; keep author-scoped authorization through
  products, customers, grants, licenses, activations, and audit.

### Later

- [x] Add feed/author-signed update signals for sources outside the running
  application's identity.
- [x] Add optional automatic refresh for trusted/official sources.
- [x] Add audit logging and policy guards for automatic runtime UI changes.
- [x] Add HTTPS through proxy or ingress termination.
- [x] Expand and verify bare-metal/VM and Kubernetes deployments.
- [x] Integrate `hc-author-registry` into author onboarding: issue `author_id`
  and `author_cert_jws`, manage public keys, and publish JWKS/revocation data.

### Licensing and Registry Handoff

- [x] Add a safe, explicit development/test PKI bootstrap with ignored private
  material, environment-marked trust anchors, fail-closed Registry and issuer
  configuration validation, and full-chain rejection tests.
- [ ] Provision non-development cryptographic material for the Author Registry
  root, author signing keys and certificates, Core delegation keys, and issuer
  signing keys; replace all Compose development keys before production use.
- [x] Add independently deployable development Compose stacks for
  `hc-author-registry` and `hc-app-licensing`, with persistent PostgreSQL,
  explicit development trust configuration, Core network integration, and
  health checks. Production deployment and trust material remain open.
- [ ] Deploy production `hc-author-registry` with persistent storage and its
  offline-provisioned root trust, connect Core to it, and verify scheduled
  trust and revocation synchronization.
- [ ] Deploy production `hc-app-licensing` inside the author's HC instance,
  configure its author certificate, Registry trust roots, Core delegation
  JWKS, and trusted Core instance JWKS, then install its manifest and
  management UI plugin.
- [ ] Exercise the complete online lifecycle end to end: author onboarding,
  product and customer setup, Core instance registration, grant approval,
  license issue, activation, renewal or replacement, suspension, and
  revocation enforcement.
- [ ] Exercise the signed offline activation-request and license-response flow,
  including stale, revoked, malformed, and mismatched artifacts.
- [ ] Add scheduled Registry and issuer revocation synchronization in Core;
  retain manual refresh as an administrative recovery action.
- [ ] Replace temporary administration-token compatibility paths with scoped,
  rotatable service identities for Core-to-Registry, Core-to-Issuer, and
  Registry-to-Issuer machine communication.
- [ ] Expand issuer and Registry integration tests around database-backed
  lifecycle transitions, authorization denials, and audit failure paths.
- [ ] Review reported npm dependency advisories and upgrade affected packages
  without forced or behavior-breaking updates.

## Cross-Cutting Platform Work

### Core Console UI

- [x] Separate User, Tenant, and Platform settings with privilege-aware
  Administration navigation while retaining contextual sidebars and
  application-provided `nav_entries`.
- [x] Separate application installation from runtime availability with
  hysteretic health checks, safe registry status, proxy-level 503 blocking,
  disabled Web navigation, direct-URL availability UI, polling, and a runtime
  dashboard widget.
- [x] Add the Author Portal with explicit hosted, trusted self-hosted, and
  private self-hosted workflows; author requests and scoped memberships;
  encrypted GitHub connections and private-repository manifest validation;
  app/runtime/catalog review states; trusted-origin connectivity checks; and
  permission-aware EN/CS UI with documented English fallback.

### AAA Accounting and Audit Log

- [x] Extend the existing `core.audit_log` and backward-compatible
  `recordAudit` with structured events, sanitization, request correlation,
  scope, visibility, actor/application identity, and query indexes.
- [x] Add own, tenant, and platform read privileges with backend-enforced
  tenant isolation, multi-value filters, filter options, detail lookup, and
  stable cursor pagination.
- [x] Add the localized Core Console Audit log with URL-persisted filters and a
  structured detail drawer; keep technical event identifiers language-neutral.
- [x] Add explicit batched retention maintenance and a non-blocking Inventory
  reference client using the existing Core app runtime token.
- [x] Add a compact operational Dashboard with live and explicitly planned
  widgets based on currently available APIs.
- [x] Replace the fixed dashboard with one permission-aware dashboard per
  user, backed by generic user preferences, a shared widget registry,
  automatic drag-and-drop persistence, inline widget settings, and Account
  Dashboard administration.
- [x] Publish the dashboard widget contract for installable applications and
  validate it with an Inventory-owned summary widget.
- [x] Make dashboard density content-aware with compact KPI cards, summary and
  list presentations, per-widget supported sizes, isolated loading/error/empty
  states, and size-dependent recent audit event lists.
- [x] Separate tenant license activation, offline import, entitlement review,
  and active selection from app-scoped license binding.
- [x] Define the author-operated Issuer Admin workflow separately from Core and
  document the authentication and administration APIs required before its UI
  can safely issue licenses.
- [x] Implement the author-operated licensing issuer with products, customers,
  registered Core instances, commercial grants, activation approval, issued
  licenses, renewal/replacement foundations, revocation, offline exchange,
  attributable audit, and a localized Core-hosted management plugin.
- [x] Complete the central Author Registry lifecycle with delegated Core RBAC,
  author approval/suspension/revocation, public-key and certificate lifecycle,
  cryptographic trust-anchor metadata, public revocation snapshots, audit, and
  platform administration UI.
- [x] Enforce cached registry and issuer revocations in Core and add signed
  offline activation-request export.
- [x] Consolidate responsive enterprise UI conventions for page actions,
  cards, tables, forms, status feedback, empty states, and navigation.

### Localization

Localization is platform-wide and must remain consistent across Core, the Web
shell, and installable applications. The work is ordered so that applications
do not implement incompatible resource or fallback conventions before the
platform contract exists.

#### Contract and Platform Foundation

- [x] Define the localization contract: translation-key naming, resource file
  format, canonical locale identifiers, interpolation and placeholder rules,
  fallback behavior, resource validation, and contract/resource versioning.
- [x] Implement the contract in Core and the Web application with required
  support for `en`, `cs`, `sk`, `de`, `fr`, and `es`.
- [x] Use `en` as the platform default locale and the final fallback for every
  platform-owned or application-owned translation lookup.
- [x] Add a per-user preferred display language within each HC instance,
  including persistence, account API exposure, and Web-shell selection.

#### Application Integration

- [x] Extend application manifests to declare supported locales, require every
  application to support at least `en`, and identify the bundled translation
  resources and their localization-contract version.
- [x] Resolve application translations using the user's selected locale when
  the application declares support for it.
- [x] Fall back to `en` when an application does not support the selected
  locale, and fall back per key when a selected-locale resource is incomplete.
- [x] Define installation and runtime handling for missing, invalid,
  incomplete, incompatible, and outdated application translation resources,
  including which conditions reject installation and which produce warnings.

#### Quality and Developer Workflow

- [x] Add shared validation tooling for duplicate translation keys, missing
  required English translations, unsupported or non-canonical locale
  identifiers, and placeholder mismatches between locales.
- [x] Apply localization validation in Core/Web CI and during application
  manifest or artifact validation where appropriate.
- [x] Document the localization contract, fallback resolution, versioning, and
  translation workflow for Core, Web, and third-party application developers.

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
