# Hekatoncheiros documentation

This index separates architecture, ADR decisions, guides, and reference material.

## ADR

- [ADR-00X: Application Installation Registry](./adr/ADR-00X-app-installation-registry.md)
- [ADR-00Y: Core-triggered App-Managed Migrations](./adr/ADR-00Y-core-triggered-app-managed-migrations.md)
- [ADR-00Z: Application Catalog Feeds and Namespace Trust](./adr/ADR-00Z-app-catalog-feeds-and-namespace-trust.md)

## Architecture

- [Overview](./architecture/overview.md)
- [Core modules](./architecture/core.md)
- [Web shell](./architecture/web-shell.md)
- [AAA accounting and audit log](./architecture/audit-log.md)

## Licensing

- [Licensing overview](./licensing/overview.md)
- [Trust model](./licensing/trust-model.md)
- [Identity model](./licensing/identity-model.md)
- [Token structures](./licensing/token-structures.md)
- [OAuth flow](./licensing/oauth-flow.md)
- [Offline flow](./licensing/offline-flow.md)
- [Core integration](./licensing/core-integration.md)
- [Issuer administration workflow](./licensing/issuer-admin-workflow.md)
- [API: author registry](./licensing/api-author-registry.md)
- [API: licensing server](./licensing/api-licensing-server.md)
- [API: core](./licensing/api-core.md)
- [ERD: author registry](./licensing/erd-author-registry.md)
- [ERD: licensing server](./licensing/erd-licensing.md)
- [ERD: core](./licensing/erd-core.md)

## Guides

- [Localization contract and workflow](./guides/localization.md)

### Platform operator

- [Installer configuration](./guides/platform-operator/installer-configuration.md)

### Deployment

- [Deployment overview](./deployment/overview.md)
- [Baremetal / VM deployment](./deployment/baremetal.md)
- [Docker deployment](./deployment/docker.md)
- [Kubernetes deployment](./deployment/kubernetes.md)

### App developer

- [Quick Start](./guides/app-developer/quickstart.md)
- [Manifest](./guides/app-developer/manifest.md)
- [UI integration](./guides/app-developer/ui-integration.md)

## Reference

### API

- [Core API](./reference/api/README.md)
- [OpenAPI](./reference/api/openapi.yml)

### Database

- [Schema outline](./reference/db/schema-outline.md)
- [DB-per-tenant provisioning](./reference/db/per-tenant-provisioning.md)
