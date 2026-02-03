# Architektura frontendu

## Cíl

Frontend (TypeScript/React) poskytuje jednotné UI pro správu **Hekatoncheiros Core** a zároveň slouží jako **design systém** pro další aplikace. Architektura je navržena tak, aby:

- oddělila **core konzoli** od **sdílených UI komponent**
- byla snadno rozšiřitelná o další aplikace
- respektovala core API surface `/api/v1/...`

## Vysoká úroveň

- **App Shell**: layout, routing, navigace, theming
- **Core Console**: stránky pro kernel (tenants, users, access, apps, licensing)
- **UI Kit**: sdílené komponenty (buttony, formuláře, tabulky)
- **Data Layer**: klient pro API, caching, auth

## Doporučené moduly

```
src/
  app/
    AppShell.tsx
    routes.tsx
    layout/
  core-console/
    pages/
    widgets/
    core-api/
  ui-kit/
    components/
    tokens/
    layout/
  data/
    api-client/
    auth/
    cache/
```

## Data flow

1. **Auth** přes `/api/v1/auth/login`
2. Po loginu načíst `/api/v1/context` pro tenant, actor, privileges, licenses
3. UI používá kontext pro:
   - zobrazení menu (podle oprávnění)
   - zobrazení licencí
   - gating funkcí
4. Změny v core (např. impersonation) aktualizují kontext

## Konvence komponent

- **Atomic UI** (Button, Input, Badge)
- **Composable** (FormField, Card, Table)
- **Page-level** (CoreTenantsPage, CoreUsersPage)

## Routing

- `/core/tenants`
- `/core/users`
- `/core/access`
- `/core/apps`
- `/core/licensing`
- `/core/audit`

## Integrace backendu

- API base: `http://127.0.0.1:3000/api/v1`
- Přenos JWT tokenu v `Authorization: Bearer <token>`

## Připraveno pro sdílení

UI kit je navržený tak, aby šel publikovat jako samostatný balík (např. `@hc/ui-kit`) a používat v dalších aplikacích.
