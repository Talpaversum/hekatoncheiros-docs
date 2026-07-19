# Licensing issuer operations

Status: Single-author operation is implemented; managed hosted operation is not production-ready.

## Single-author self-hosted issuer

A trusted self-hosted author operates one issuer identity in `single_author` mode. Production configuration requires:

- an external persistent `DATABASE_URL`;
- an explicit production author certificate chained to a pinned Registry root;
- the matching issuer private signing key through a protected injection boundary;
- Core delegation JWKS, issuer and audience configuration;
- stable public issuer URL and health endpoints;
- database migrations before server start;
- backup, rotation and revocation procedures.

The service must reject missing, placeholder, development or mismatched identity material in production mode.

## Managed multi-author hosted issuer

The source contains an implemented `managed_multi_author` foundation and author-scoped data access. It is not currently a production-operated Talpaversum service.

Production availability requires separate signing identities for every author, secure provisioning, cross-author isolation validation, scoped service identities, durable deployment, monitoring and tested recovery. Configuration of the mode alone does not satisfy those requirements.

## Health and persistence

Liveness reports that the process is running. Readiness and health must include database access and required identity configuration. Issuer data and audit records are persistent business data and must be included in backup and restore validation.

See [licensing architecture](../architecture/licensing.md), [production PKI](./production-pki.md) and [backup and recovery](./backup-and-recovery.md).
