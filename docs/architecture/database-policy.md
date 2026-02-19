# Database Policy – Hekatoncheiros

## 1. Tenancy model

- Default: DB-per-tenant.
- V každé tenant DB existují:
  - `core` schema
  - `app_<app_id>` schema pro každou instalovanou aplikaci

## 2. Provisioning

- Databázi pro tenant vytváří core (ne aplikace).
- Aplikace nikdy nesmí provádět `CREATE DATABASE`.
- Core může běžet s elevated DB právy pro provisioning (kompromis pro UX).

## 3. Schema-per-app

- Každá aplikace vlastní jedno schema: `app_<app_id>`.
- Žádná aplikace nemá přístup do jiného app schema ani do `core` schema.

## 4. Migrace

- Migrace dodává aplikace přes endpointy:
  - `GET /.well-known/hc/migrations`
  - `GET /.well-known/hc/migrations/{id}`
- Core:
  - stáhne migrace
  - ověří SHA-256 hash každé migrace (MVP)
  - aplikuje migrace do příslušného schema
- Aplikace migrace nikdy nespouští sama proti DB.

## 5. Runtime DB Access

- Core vytvoří DB roli pro aplikaci:
  - přístup pouze do `app_<app_id>` schema
- Aplikace obdrží DB credentials až po úspěšné instalaci.

## 6. Instalace aplikace – životní cyklus

Stavy:

- `registered`
- `installing`
- `migrating`
- `ready`
- `failed`
- `disabled`

Aplikace:

- před `ready` musí odmítat business API
- po `installation/complete` přechází do runtime režimu

## 7. Remote Apps (future)

- Core2Core integrace
- žádná sdílená databáze
- roadmap item

## MVP poznámka

V aktuálním MVP běží platforma nad jednou fyzickou PostgreSQL databází (logická tenancy), ale pravidla výše platí beze změny:

- schema-per-app,
- core-managed migrace,
- žádné `CREATE DATABASE` v aplikacích.
