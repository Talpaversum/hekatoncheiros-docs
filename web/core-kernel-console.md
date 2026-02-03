# Core/Kernel console

This document defines the baseline UI for managing **Core/Kernel**. It is intentionally designed to serve as a reference console for other applications.

## Key UI concepts

- **Context banner**: shows tenant, current user, impersonation/delegation
- **Core navigation**: stable menu for key modules
- **Privilege gating**: menu and actions depend on `privileges`
- **Audit visibility**: mandatory audit trail for sensitive actions

## Primary sections

### 1) Dashboard

- tenant status overview
- license summary
- recent audit events

### 2) Tenants

- tenant list (for superadmin)
- tenant detail
- create tenant option (install-time / platform admin)

**API**

- `GET /api/v1/tenants`
- `GET /api/v1/tenants/me`
- `POST /api/v1/tenants`

### 3) Users

- user list
- user detail
- impersonation start/stop

**API**

- `GET /api/v1/users/me`
- `GET /api/v1/users/{id}`
- `POST /api/v1/access/impersonation/start`
- `POST /api/v1/access/impersonation/stop`

### 4) Access

- privilege viewer
- delegation list
- create delegation

**API**

- `GET /api/v1/access/privileges/me`
- `GET /api/v1/access/delegations/me`
- `POST /api/v1/access/delegations`

### 5) Apps

- app registry
- enable/disable app per tenant

**API**

- `POST /api/v1/apps/register`
- `POST /api/v1/tenants/apps/{app_id}/enable`
- `POST /api/v1/tenants/apps/{app_id}/disable`

### 6) Licensing

- license overview
- offline license activation

**API**

- `GET /api/v1/licensing/apps/{app_id}`
- `POST /api/v1/licensing/apps/{app_id}/activate-offline`

### 7) Audit

- audit feed
- filter by user/tenant/action

**API**

- `POST /api/v1/audit/record`

## Base UI layout

```
┌───────────────────────────────────────────────────┐
│ TopBar: tenant | user | impersonation | actions   │
├───────────────┬───────────────────────────────────┤
│ SidebarNav    │ Page content                       │
│ - Dashboard   │ - SectionHeader                    │
│ - Tenants     │ - DataTable / Forms / Widgets      │
│ - Users       │                                     │
│ - Access      │                                     │
│ - Apps        │                                     │
│ - Licensing   │                                     │
│ - Audit       │                                     │
└───────────────┴───────────────────────────────────┘
```

## UI components used

- `PageLayout`, `SidebarNav`, `TopBar`
- `SectionHeader`
- `Table`, `Card`, `FormField`, `Button`
- `Modal` for confirming sensitive actions
- `Notification` for feedback

## Mandatory UX rules

- Impersonation is always visually highlighted
- Sensitive actions require confirmation
- Empty states must include guidance text
- All actions log audit events (server-side)
