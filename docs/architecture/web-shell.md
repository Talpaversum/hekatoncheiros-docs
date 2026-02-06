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

## Core/Kernel console (reference)

Baseline UI for managing **Core/Kernel**. It serves as a reference console for other applications as well.

### Key UI concepts

- **Context banner**: tenant, current user, impersonation/delegation
- **Core navigation**: stable menu for key modules
- **Privilege gating**: menu and actions depend on `privileges`
- **Audit visibility**: audit trail for sensitive actions

### Primary sections

#### 1) Dashboard

- tenant status overview
- license summary
- recent audit events

#### 2) Tenants

- tenant list (for superadmin)
- tenant detail
- create tenant option (install-time / platform admin)

**API**

- `GET /api/v1/tenants`
- `GET /api/v1/tenants/me`
- `POST /api/v1/tenants`

#### 3) Users

- user list
- user detail
- impersonation start/stop

**API**

- `GET /api/v1/users/me`
- `GET /api/v1/users/{id}`
- `POST /api/v1/access/impersonation/start`
- `POST /api/v1/access/impersonation/stop`

#### 4) Access

- privilege viewer
- delegation list
- create delegation

**API**

- `GET /api/v1/access/privileges/me`
- `GET /api/v1/access/delegations/me`
- `POST /api/v1/access/delegations`

#### 5) Apps

- app registry
- enable/disable app per tenant

**Installed apps (temporary dev installer)**

- thin admin UI for install/uninstall (no wizard)
- in-memory `AppInstallationStore`
- requires privilege `platform.apps.manage`

The web shell is not a plugin installer and not a plugin artifact repository.
The web shell MUST NOT build, install, or store UI plugin artifacts.
UI plugin artifacts are installed and hosted by core, and the web shell MUST
load them dynamically from core-provided `ui_url` values.

**API**

- `POST /api/v1/apps/register`
- `POST /api/v1/tenants/apps/{app_id}/enable`
- `POST /api/v1/tenants/apps/{app_id}/disable`
- `GET /api/v1/apps/installed`
- `POST /api/v1/apps/installed`
- `DELETE /api/v1/apps/installed/{app_id}`

#### 6) Licensing

- license overview
- offline license activation

**API**

- `GET /api/v1/licensing/apps/{app_id}`
- `POST /api/v1/licensing/apps/{app_id}/activate-offline`

#### 7) Audit

- audit feed
- filter by user/tenant/action

**API**

- `POST /api/v1/audit/record`

### Base UI layout

The shell layout is stable, but **navigation content is context-driven**.

- **SidebarNav** changes based on the current route:
  - `/core/*` and `/admin/*` use static section configs (Dashboard, Apps, Licensing, Admin, ...)
  - `/app/:appId/*` uses **app registry** `nav_entries` for the active app
- **TopBar** provides global navigation + dropdowns:
  - left: Dashboard, Apps dropdown (registry + app entries), Licensing
  - right: messaging icon, Settings dropdown (theme toggle), User dropdown (logout)

```
┌──────────────────────────────────────────────────────────────┐
│ TopBar: Dashboard | Apps ▾ | Licensing | Settings ▾ | User ▾ │
├──────────────────┬───────────────────────────────────────────┤
│ SidebarNav (ctx) │ Page content                              │
│ - core/admin OR  │ - SectionHeader                           │
│ - app nav        │ - DataTable / Forms / Widgets             │
└──────────────────┴───────────────────────────────────────────┘
```

### UI components used

- `PageLayout`, `SidebarNav`, `TopBar`
- `SectionHeader`
- `Table`, `Card`, `FormField`, `Button`
- `Modal` for confirming sensitive actions
- `Notification` for feedback

### Mandatory UX rules

- Impersonation is always visually highlighted
- Sensitive actions require confirmation
- Empty states must include guidance text
- All actions log audit events (server-side)

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
