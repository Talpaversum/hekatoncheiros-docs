# Database migration operations

Status: Implemented migration runners; production rollback procedures require deployment validation.

Core, Author Registry and the licensing issuer own separate migration histories. Run each product's migrations against its own configured database before exposing the new server version to traffic.

## Rules

- Use an explicit environment-specific `DATABASE_URL`.
- Back up the database before a migration that cannot be safely reconstructed.
- Run only the migration runner belonging to that product.
- Record applied filenames and checksums where the product supports them.
- Treat a checksum mismatch, missing prerequisite or partially applied migration as a deployment failure.
- Do not use development databases or test bootstrap scripts as production migration tooling.
- Verify readiness and representative reads after migration.

Core startup applies Core migrations under a PostgreSQL advisory lock. Application migrations are downloaded and SHA-256 verified by Core before being applied to the application's schema. Applications must not migrate `core` or another application's schema directly.

Registry migrations run before Registry server startup and are recorded transactionally. Issuer migrations run against the issuer database before the issuer accepts management or activation traffic.

Rollback is deployment-specific. Prefer restoring a tested backup or deploying a forward corrective migration; never assume every SQL migration is mechanically reversible.
