# Hekatoncheiros Platform Status

This document describes capabilities currently implemented in the main development branches.

It does not guarantee that every capability is enabled or configured in every deployment.

The following terms are distinct:

- **Implemented** — the capability exists in the source code.
- **Supported** — the platform defines and validates the capability.
- **Configured** — the required infrastructure and credentials are present.
- **Enabled** — the instance administrator allows the capability to be used.
- **Production-ready** — the capability has completed the required production deployment and operational validation.

Documentation distinguishes between implemented, supported, configured, enabled and production-ready capabilities. A capability being present in the source code does not mean that it is enabled or production-configured on a particular instance.

## Application platform

Core implements a tenant-aware platform on PostgreSQL. The current development deployment uses logical tenant separation in one physical database and creates an application-owned schema for each installed application. Core discovers, verifies and applies application migrations.

The platform validates application manifests, signed manifests and namespace trust. It implements the installation lifecycle, Core-hosted application UI plugins, permission-aware navigation entries and application-provided help entries. These capabilities remain subject to instance configuration, trust policy and RBAC.

## Developer Tools

Developer Tools implements a project lifecycle workspace with draft configuration, source validation, deployment records, runtime actions, logs, update detection, approval and rollback support.

Implemented project sources include:

- GitHub repositories through tokens or a configured GitHub App;
- GitLab repositories through a configured token and base URL;
- generic Git repositories supported by the Git checkout adapter;
- local workspaces mounted into Core;
- direct manifest URLs and private feed URLs.

Provider credentials, network access and runtime support must be configured before the corresponding path is usable. Private application development is not official authorship and does not grant an `author_id`.

## Application installation and runtime

Installation supports `external`, `stage_only` and opt-in Core-managed Compose modes. Managed deployment verifies the deployment plan, package SHA-256, Compose policy and administrator approval before starting a runtime.

Core implements managed runtime token delivery and rotation, health monitoring, diagnostic results, stop and update actions, deployment history, rollback to the last working deployment and runtime-aware uninstall. Runtime, installation, license, UI integration and runtime-management states are evaluated independently. A running container alone is not considered a healthy application.

See [application availability](./docs/guides/app-operator/application-availability.md) and [deployment operations](./docs/operations/deployment.md).

## Catalogs and application sources

Core implements a local catalog, trusted origins, private catalog feeds, manual feed refresh, optional automatic refresh for configured sources and publication or unpublication of installed applications through an instance feed.

The local catalog and an instance's exported feed are not the official Talpaversum public catalog. Materialization of approved author submissions into the official public catalog is not complete and remains on the [roadmap](./docs/ROADMAP.md).

## Licensing

Core implements license consumption and enforcement, tenant activation, signed offline import, application license binding and revocation-aware validation.

The licensing application implements a single-author issuer for a trusted self-hosted author, including products, customers, grants, instances, activation and license operations. Core hosts the delegated human management interface.

Foundations for managed multi-author operation and author-scoped isolation exist in the issuer source and tests. A production Talpaversum-hosted multi-author licensing service, per-author production signing identities and its operational provisioning are not complete and are not production-ready.

See [licensing architecture](./docs/architecture/licensing.md) and [issuer operations](./docs/operations/licensing-issuer.md).

## Authors and Author Registry

There are exactly two official author operating modes: `talpaversum_hosted` and `trusted_self_hosted`. Private application development is a separate local workflow; a private developer is not an official author.

Core implements author requests, scoped memberships, Author Workspace, Author Administration, Registry Administration, Catalog Review and Runtime Review boundaries. The official Author Registry implements author identities, public keys, author certificates, revocations and trust snapshots. Human management uses Core delegation and RBAC.

The Registry is an official Talpaversum trust authority, not a service automatically available on every Core instance. Production deployment and production root provisioning remain incomplete.

See [author workflows](./docs/architecture/author-workflows.md), [UI boundaries](./docs/architecture/author-ui-boundaries.md) and [Author Registry architecture](./docs/architecture/author-registry.md).

## Authentication, authorization and audit

Core implements user and application authentication, tenant and platform RBAC, author-scoped authorization and delegated user identities for application and service calls. Authorization is enforced by APIs; hidden navigation is not a security boundary.

Structured audit records include actor, scope, outcome and correlation data. Tenant-isolation checks, correlation IDs, audit sanitization and configurable batched retention maintenance are implemented.

## Core Console and dashboard

Core Console separates User, Tenant and Platform settings and builds navigation from privileges and instance capabilities. It includes application, licensing, author and platform administration surfaces.

The dashboard is customizable per user through a widget registry. Core and installed applications can provide widgets. Installed-application management displays independent lifecycle states, the exact availability reason and on-demand diagnostics.

## Localization

The platform UI supports `en`, `cs`, `sk`, `de`, `fr` and `es`, with a per-user locale and English fallback. Application manifests can declare localized resources through the manifest localization contract. Repository tooling validates key parity and prevents untranslated raw keys in registered components.

## Deployment support

Docker Compose is the currently verified path for local self-hosting.

Bare-metal/VM and Kubernetes configurations are validated reference deployments. They are not currently distributed as packaged installers or Helm releases.

Development deployments may use locally built images and development trust material. This does not make them production deployments. Production secret storage, trust provisioning, recovery and operational hardening are separate requirements described in [operations](./docs/operations/deployment.md).

## Known production gaps

- Production Registry root ceremony, PKI provisioning and signing-key custody are not approved or deployed.
- The official Author Registry does not yet have a completed production deployment and recovery validation.
- Managed multi-author hosted licensing is not a production-operated service.
- The hosted author build worker and controlled artifact store are not implemented end to end.
- Approved author submissions are not materialized into the official public catalog.
- Machine communication still needs scoped, independently rotatable service identities for production.
- Production deployment hardening, packaged installers and supported upgrade/recovery procedures remain incomplete.
- Core clustering and coordinated runtime ownership are not implemented.
- Standalone and externally operated application database ownership remains an [open decision](./docs/architecture/application-database-ownership.md).
