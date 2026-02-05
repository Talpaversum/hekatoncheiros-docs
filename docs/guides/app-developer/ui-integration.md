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
