# Frontend architecture

## Goal

The frontend (TypeScript/React) provides a unified UI for managing **Hekatoncheiros Core** and also serves as a **design system** for other applications. The architecture is designed to:

- separate the **core console** from **shared UI components**
- be easy to extend with additional applications
- respect the core API surface `/api/v1/...`

## High-level overview

- **App Shell**: layout, routing, navigation, theming
- **Core Console**: kernel pages (tenants, users, access, apps, licensing)
- **UI Kit**: shared components (buttons, forms, tables)
- **Data Layer**: API client, caching, auth

## Suggested modules

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

1. **Auth** via `/api/v1/auth/login`
2. After login, fetch `/api/v1/context` for tenant, actor, privileges, licenses
3. UI uses the context for:
   - menu visibility (based on privileges)
   - license visibility
   - feature gating
4. Core changes (e.g., impersonation) refresh the context

## Component conventions

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

## Backend integration

- API base: `http://127.0.0.1:3000/api/v1`
- JWT token transmission: `Authorization: Bearer <token>`

## Ready for sharing

The UI kit is designed to be published as a standalone package (e.g., `@hc/ui-kit`) and reused across applications.
