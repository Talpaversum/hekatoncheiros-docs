# Hekatoncheiros documentation

This index separates current status, future work, architecture, operations, guides and reference material.

## Project

- [Roadmap](./ROADMAP.md)
- [Platform status](../STATUS.md)
- [Changelog](../CHANGELOG.md)

## ADR

- [ADR-00X: Application Installation Registry](./adr/ADR-00X-app-installation-registry.md)
- [ADR-00Y: Core-triggered App-Managed Migrations](./adr/ADR-00Y-core-triggered-app-managed-migrations.md)
- [ADR-00Z: Application Catalog Feeds and Namespace Trust](./adr/ADR-00Z-app-catalog-feeds-and-namespace-trust.md)

## Architecture

- [Overview](./architecture/overview.md)
- [Core modules](./architecture/core.md)
- [Web shell](./architecture/web-shell.md)
- [AAA accounting and audit log](./architecture/audit-log.md)
- [Application execution model](./architecture/app-execution-model.md)
- [Application runtime health](./architecture/application-runtime-health.md)
- [Author workflows](./architecture/author-workflows.md)
- [Author and developer UI boundaries](./architecture/author-ui-boundaries.md)
- [Author Registry](./architecture/author-registry.md)
- [Licensing boundaries](./architecture/licensing.md)
- [Database policy](./architecture/database-policy.md)
- [Application database ownership](./architecture/application-database-ownership.md)
- [Production signing-key management](./architecture/signing-key-management.md)
- [Application-owned mobile clients](./architecture/mobile-clients.md)
- [Core2Core (deferred)](./architecture/core2core.md)

## Operations

- [Deployment support](./operations/deployment.md)
- [Production PKI](./operations/production-pki.md)
- [Author Registry](./operations/author-registry.md)
- [Licensing issuer](./operations/licensing-issuer.md)
- [Database migrations](./operations/database-migrations.md)
- [Backup and recovery](./operations/backup-and-recovery.md)

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
- [Using Registry Administration](./guides/platform-operator/author-registry.md)

### Deployment

- [Deployment overview](./deployment/overview.md)
- [Baremetal / VM deployment](./deployment/baremetal.md)
- [Docker deployment](./deployment/docker.md)
- [Kubernetes deployment](./deployment/kubernetes.md)

### App developer

- [Quick Start](./guides/app-developer/quickstart.md)
- [Manifest](./guides/app-developer/manifest.md)
- [UI integration](./guides/app-developer/ui-integration.md)
- [Private application development](./guides/app-developer/private-app-development.md)
- [Author workflows](./guides/app-developer/author-workflows.md)

## Testing

- [Author workflow acceptance matrix](./testing/acceptance-matrix.md)

## Reference

### API

- [Core API](./reference/api/README.md)
- [OpenAPI](./reference/api/openapi.yml)
- [Author Portal API](./reference/api/author-portal.md)

### Database

- [Schema outline](./reference/db/schema-outline.md)
- [DB-per-tenant provisioning](./reference/db/per-tenant-provisioning.md)
