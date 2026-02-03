# Ovládací rozhraní Core/Kernel

Tento dokument definuje základní UI pro správu **Core/Kernel**. Je cíleně navržen tak, aby byl použitelný jako referenční konzole pro další aplikace.

## Hlavní koncepty UI

- **Context banner**: zobrazuje tenant, aktuálního uživatele, impersonation/delegation
- **Core navigation**: stabilní menu pro klíčové moduly
- **Privilege gating**: menu i akce závisí na `privileges`
- **Audit visibility**: povinná audit stopa pro citlivé akce

## Primární sekce

### 1) Dashboard

- přehled tenant stavu
- info o licencích
- poslední audit eventy

### 2) Tenants

- seznam tenantů (pro superadmin)
- detail tenanta
- možnost vytvořit tenant (install-time / platform admin)

**API**

- `GET /api/v1/tenants`
- `GET /api/v1/tenants/me`
- `POST /api/v1/tenants`

### 3) Users

- seznam uživatelů
- detail uživatele
- impersonation start/stop

**API**

- `GET /api/v1/users/me`
- `GET /api/v1/users/{id}`
- `POST /api/v1/access/impersonation/start`
- `POST /api/v1/access/impersonation/stop`

### 4) Access

- privileges viewer
- delegation seznam
- vytvoření delegace

**API**

- `GET /api/v1/access/privileges/me`
- `GET /api/v1/access/delegations/me`
- `POST /api/v1/access/delegations`

### 5) Apps

- registry aplikací
- enable/disable app per tenant

**API**

- `POST /api/v1/apps/register`
- `POST /api/v1/tenants/apps/{app_id}/enable`
- `POST /api/v1/tenants/apps/{app_id}/disable`

### 6) Licensing

- přehled licencí
- aktivace offline licencí

**API**

- `GET /api/v1/licensing/apps/{app_id}`
- `POST /api/v1/licensing/apps/{app_id}/activate-offline`

### 7) Audit

- audit feed
- filtr podle user/tenant/action

**API**

- `POST /api/v1/audit/record`

## Základní UI layout

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

## Používané UI komponenty

- `PageLayout`, `SidebarNav`, `TopBar`
- `SectionHeader`
- `Table`, `Card`, `FormField`, `Button`
- `Modal` pro potvrzení citlivých akcí
- `Notification` pro feedback

## Povinné UX pravidla

- Impersonation je vždy vizuálně zvýrazněna
- Citlivé akce vyžadují potvrzení
- Prázdné stavy mají guidance text
- Všechny akce logují audit event (server-side)
