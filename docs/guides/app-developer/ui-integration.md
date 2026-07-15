# UI integration and design system

This document describes the UI kit and developer conventions for integrating
into the platform shell. It is a practical baseline for applications that want
to use shared components and the Core Console visual language.

## Goals

- define shared UI components (buttons, forms, layout)
- describe the baseline architecture and data flow
- keep a consistent visual language across applications

## Design principles

- **Consistency**: unified spacing, typography, and states
- **Composition**: smaller parts compose larger structures
- **Accessibility**: ARIA attributes, contrast, focus rings
- **Theming**: light/dark via CSS tokens + Tailwind v4 `@theme`

## UI Plugin Contract

An application UI module must export:

- a registration function (e.g. register or createPlugin)
- React components used as pages or widgets

The registration function returns:
- route definitions
- navigation entries
- optional widgets

## Dashboard widget contract

Applications may return `dashboard_widgets` from the same `register(appContext)`
function used for routes and navigation. Core owns the dashboard layout; the
application continues to own the widget component and its business logic.

Each widget declares:

- a globally stable `id` (use reverse-domain naming)
- localized `title`, optional `description`, and `category`
- `requiredPrivileges` and `supportedScopes`
- `defaultVisible`, `defaultSize`, and `defaultPosition`
- `presentation` (`kpi`, `summary`, or `list`) and the meaningful
  `supportedSizes`
- `defaultSettings` and a React `component`
- optionally, a `settingsComponent` and `refresh` callback

The component receives `{ settings }`. A settings component receives
`{ value, onChange }`; Core renders it in a modal and persists the resulting
settings with the user's dashboard preference. Applications must query data
only inside their component and must use `appContext.api.request` so existing
authentication and tenant isolation remain in effect.

Example:

```tsx
return {
  routes,
  nav_entries,
  dashboard_widgets: [{
    id: "com.example.inventory.asset-summary",
    title: labels.assetSummary,
    category: labels.inventory,
    requiredPrivileges: ["inventory.read"],
    supportedScopes: ["tenant"],
    defaultVisible: false,
    defaultSize: "small",
    supportedSizes: ["small", "medium"],
    presentation: "summary",
    defaultPosition: 1000,
    defaultSettings: {},
    component: AssetSummaryWidget,
  }],
};
```

Widget IDs and defaults form persisted configuration and should therefore be
treated as a versioned public contract. New application widgets default to
hidden for users who already customized their dashboard, preventing upgrades
from unexpectedly changing an existing layout.

Widget content must use its `size` prop, not only the card width. Compact KPI
widgets should avoid fixed minimum heights; list widgets should increase the
number and detail of rows for larger supported sizes. Every data-backed widget
owns its loading, empty, and error UI, and hidden or unauthorized widgets are
not mounted and therefore must not issue requests.

## Tokens (CSS custom properties)

Primary tokens live in `hekatoncheiros-web/src/index.css` and map to Tailwind utilities via `@theme`:

- **Surface layers**
  - `--hc-bg` – application background
  - `--hc-surface` – main content (cards, menus)
  - `--hc-surface-variant` – highlight (hover/active)
  - `--hc-rail` – sidebar rail (darker)
  - `--hc-topbar` – topbar (darkest)
- **Text**: `--hc-text`, `--hc-muted`
- **Accent**: `--hc-primary`, `--hc-on-primary`
- **Danger**: `--hc-danger`, `--hc-on-danger`
- **Topbar gradient**: `--hc-topbar-glow`, `--hc-topbar-depth`

Tailwind equivalents include:

- `bg-hc-rail`, `bg-hc-topbar`, `bg-hc-surface`
- `text-hc-muted`, `text-hc-text`
- `from-hc-topbar-glow`, `to-hc-topbar-depth`

## Base components (atoms)

### Button (`ui-kit/components/Button.tsx`)

- variants: `filled`, `tonal`, `outlined`, `ghost`, `danger`
- states: hover/focus/disabled

### Input (`ui-kit/components/Input.tsx`)

- text, password
- error state

### IconButton (`ui-kit/components/IconButton.tsx`)

- variants: `default`, `tonal`, `ghost`

### Switch (`ui-kit/components/Switch.tsx`)

- used in settings (dark mode toggle)

### Avatar (`ui-kit/components/Avatar.tsx`)

- fallback to initials

## Composite components (molecules)

### Card (`ui-kit/components/Card.tsx`)

- surface container + shadow

### Table (`ui-kit/components/Table.tsx`)

- simple surfaced table

### Menu (`ui-kit/components/Menu.tsx`)

- dropdown for Apps/Settings/Profile
- closes on outside click / ESC

## Layout components

- `AppTopBar` – sticky topbar with global nav, settings dropdown, and profile dropdown
- `SidebarNav` – contextual sidebar based on route
- `PageLayout` – base page wrapper
- `TopBar` – page header (secondary)

### AppTopBar (layout)

- **Left:** Dashboard, Apps (dropdown), Licensing
- **Right:** messaging icon, settings dropdown (theme toggle), user dropdown (Sign out)

### SidebarNav (layout)

- changes content based on `/core/dashboard`, `/core/apps`, `/core/licensing`

## Theming (light/dark)

- switch lives in settings dropdown (gear icon)
- state is stored in `localStorage` (key `hc_theme`)
- `useTheme` applies `document.documentElement.dataset.theme`

## Recommended folder structure

```
src/ui-kit/
  components/
    Avatar.tsx
    Button.tsx
    Card.tsx
    IconButton.tsx
    Input.tsx
    Menu.tsx
    Switch.tsx
    Table.tsx
  layout/
    AppTopBar.tsx
    SidebarNav.tsx
    PageLayout.tsx
    TopBar.tsx
  theme/
    theme-storage.ts
    useTheme.ts
```

## Runtime dependencies

- Backend Core API: `http://127.0.0.1:3000`
- Database (PostgreSQL) runs in Docker on port `5432`

## Conventions

- Documentation prefers a **component-first approach** and **reusable UI primitives**.
- Components are designed for **reuse across applications**.
- UI routes and data flow follow the core API surface (`/api/v1/...`).
