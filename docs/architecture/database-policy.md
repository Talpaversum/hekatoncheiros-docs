# Database Policy - Hekatoncheiros

## 1. Tenancy model

- Default: DB-per-tenant.
- Each tenant database contains:
  - the `core` schema
  - one `app_<app_id>` schema for each installed application

## 2. Provisioning

- Core creates tenant databases; applications do not.
- Applications MUST NOT execute `CREATE DATABASE`.
- Core may use elevated database privileges for provisioning as an operational
  compromise that simplifies installation.

## 3. Schema-per-app

- Each application owns one schema: `app_<app_id>`.
- An application cannot access another application's schema or the `core`
  schema.

## 4. Migrations

- The application provides migrations through these endpoints:
  - `GET /.well-known/hc/migrations`
  - `GET /.well-known/hc/migrations/{id}`
- Core:
  - downloads the migrations
  - verifies the SHA-256 hash of every migration (MVP)
  - applies migrations to the corresponding application schema
- Applications MUST NOT run migrations directly against the database.

## 5. Runtime DB Access

- Core creates a database role for the application with access limited to its
  `app_<app_id>` schema.
- The application receives database credentials only after successful
  installation.

## 6. Application Installation Lifecycle

States:

- `registered`
- `installing`
- `migrating`
- `ready`
- `failed`
- `disabled`

Application behavior:

- Before reaching `ready`, the application must reject business API requests.
- After `installation/complete`, the application enters runtime mode.

## 7. Remote Apps (future)

- Core2Core integration
- no shared database
- tracked as a roadmap item

## MVP Note

The current MVP runs on one physical PostgreSQL database with logical tenancy,
but the rules above still apply without modification:

- schema-per-app
- Core-managed migrations
- no `CREATE DATABASE` issued by applications
