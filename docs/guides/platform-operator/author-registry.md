# Author Registry operations

The official Author Registry runs only as part of the Talpaversum-operated platform.

## Production database

Production deployment requires an externally supplied `DATABASE_URL`.

The production Registry compose file must not create or start PostgreSQL.

## Development database

`docker-compose.dev.yml` may contain a disposable local PostgreSQL service.

This compose file is optional and development-only. It must never be started automatically by production startup, tests or migrations.

## Authentication

Human management operations use short-lived Core user delegation tokens and explicit Registry permissions.

`ADMIN_TOKEN` is not part of the production authentication model.

## Root keys

The production Registry root key is provisioned outside normal application startup.

Production startup must never generate a root key.

## Migrations

The production container applies numbered migrations before starting the Registry server.

Migration tests must validate the resulting database schema against an explicitly supplied test database. Tests must not create a database container automatically.
