# Author Registry operations

Status: Development operation is implemented; production deployment is pending.

The production Author Registry uses an externally supplied `DATABASE_URL`. PostgreSQL is not provisioned by the production Registry Compose definition. A bundled PostgreSQL service is permitted only in an explicitly selected development Compose configuration.

## Startup order

1. Provision the external database and least-privilege credentials.
2. Provision the approved Registry identity and root material outside startup.
3. Run Registry migrations before starting the server.
4. Start the Registry and verify liveness and readiness.
5. Configure Core delegation verification and the permitted Registry audience.
6. Configure Core with the Registry endpoint and pinned public trust anchor.
7. Perform and verify trust synchronization.

Migration discovery is filename-based and recorded transactionally. A missing or invalid production root, placeholder material or private key exposed through public JWKS must prevent readiness.

Human administration uses Core-issued delegation and platform RBAC. Production machine-to-machine credentials still require scoped service identities; a shared administrator token is not the production target.

Trust synchronization imports public anchors and revocation snapshots. It never transfers private Registry signing material into Core.

See [Registry architecture](../architecture/author-registry.md), [production PKI](./production-pki.md) and [database migrations](./database-migrations.md).
