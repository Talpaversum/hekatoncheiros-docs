# Author Registry operations

Status: Local production-style operation is implemented; production deployment is pending.

The Registry has a single production-oriented `docker-compose.yml`. It uses an external PostgreSQL database, Docker secret files, the external Core network, and production root validation. It does not bundle PostgreSQL.

## Local production-style startup

Start Core first, then initialize and start Registry:

```bash
cd hc-author-registry
npm ci
npm run pki:bootstrap:local
docker compose up -d --build
curl --fail http://localhost:4020/health/ready
```

The bootstrap verifies the Core network, writes ignored local root files and a mode-`0600` `.env`, and validates `docker compose config`. If `DATABASE_URL` is absent, it provisions a dedicated Registry database and role in the running Core PostgreSQL container. If it is already configured, its value is preserved.

Normal reruns preserve the root identity. `npm run pki:bootstrap:local -- --force` deliberately rotates it and therefore invalidates locally pinned trust and certificates. The generated `local-registry-root-*` material is strictly local and must not be committed or promoted. `REGISTRY_ROOT_MODE=production` means that the local stack exercises production loading and validation; it does not make the generated key an approved production trust anchor.

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
