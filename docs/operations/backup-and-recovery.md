# Backup and recovery

Status: Required operational baseline; production procedures must be validated per deployment.

Backups must cover all state required to restore identity, authorization and application behavior consistently.

## Backup scope

- Core PostgreSQL data, including tenants, installed applications, licensing bindings, audit data and migration history;
- Core data storage, including stored UI artifacts and managed deployment state;
- Author Registry database and public trust/revocation history;
- licensing issuer database and audit history;
- production key references and recoverable private material according to the approved custody model;
- deployment configuration needed to reconnect services without changing their identities.

Development keys and databases are disposable and are not substitutes for production backup validation.

## Recovery order

1. Recover databases and verify migration histories.
2. Recover production identity and trust material without generating replacements.
3. Recover Core data and application artifacts.
4. Start Registry and issuer dependencies and verify readiness.
5. Start Core and Web, then synchronize public trust and revocation snapshots.
6. Verify tenant isolation, license enforcement and application health before reopening traffic.

Recovery tests must document recovery point and recovery time objectives, integrity checks and the handling of a lost or compromised key. Application data owned by external runtimes remains subject to the [database ownership open decision](../architecture/application-database-ownership.md).
