# Changelog

This document records notable completed platform changes.

Detailed implementation commits remain available in the individual repositories.

## Unreleased

### Added

- Nothing recorded yet.

### Changed

- Nothing recorded yet.

### Fixed

- Nothing recorded yet.

### Security

- Nothing recorded yet.

## 2026-07

### Added

- Added the Developer Tools project lifecycle with source connections, repository discovery, validation, deployments, updates, rollback and logs.
- Added opt-in Core-managed Compose and Dockerfile deployment with reviewed plans, immutable source revisions and runtime actions.
- Added official author onboarding with scoped author memberships, Author Workspace and separate administration surfaces.
- Added Author Registry key, certificate, revocation and trust-snapshot lifecycles.
- Added the author-operated licensing issuer and its Core-hosted delegated management UI.
- Added runtime-aware application availability, persisted health diagnostics and on-demand diagnostic checks.
- Added customizable per-user dashboards, a widget registry and application-provided widgets.
- Added the platform localization contract and six supported platform locales.

### Changed

- Separated private application development from the two official author operating modes.
- Separated User, Tenant and Platform settings and made navigation privilege- and capability-aware.
- Extended managed application deployment with explicit approval, token delivery, stop, update and last-working rollback behavior.
- Distinguished local catalogs and instance feeds from the future official Talpaversum public catalog.

### Fixed

- Preserved healthy runtime state separately from missing licenses and missing UI integration.
- Mapped dependency-aware HTTP 503 health responses to `degraded` instead of a generic unavailable state.
- Diagnosed container-local `localhost`, Docker DNS and connection failures with stable reason codes.

### Security

- Added fail-closed validation for development, test and production trust material.
- Added Core delegation and author-scoped authorization boundaries for Registry and issuer management.
- Added deployment package path validation, Compose policy restrictions and approval binding to manifest, package and plan hashes.
- Added structured audit sanitization, tenant isolation coverage and batched retention support.
