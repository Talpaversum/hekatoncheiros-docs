# UI kit / Design system

UI kit je sdílená knihovna komponent pro Core Console. Cíl je konzistence, rychlá iterace a možnost přenést branding. Aktuální implementace odpovídá **Material‑inspired** stylu s **light/dark** schématem.

## Design principles

- **Consistency**: sjednocené spacingy, typografie a stavy
- **Composition**: malé prvky skládají větší celky
- **Accessibility**: aria atributy, kontrast, focus ringy
- **Theming**: light/dark přes CSS tokeny + Tailwind v4 `@theme`

## Tokens (CSS custom properties)

Hlavní tokeny jsou v `hekatoncheiros-web/src/index.css` a mapují se do Tailwind utilit přes `@theme`:

- **Surface vrstvy**
  - `--hc-bg` – pozadí aplikace
  - `--hc-surface` – hlavní content (karty, menu)
  - `--hc-surface-variant` – zvýraznění (hover/active)
  - `--hc-rail` – sidebar rail (tmavší)
  - `--hc-topbar` – topbar (nejtmavší)
- **Text**: `--hc-text`, `--hc-muted`
- **Akcent**: `--hc-primary`, `--hc-on-primary`
- **Danger**: `--hc-danger`, `--hc-on-danger`
- **Topbar gradient**: `--hc-topbar-glow`, `--hc-topbar-depth`

Tailwind utilitám odpovídají například:

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

- použito v nastavení (dark mode toggle)

### Avatar (`ui-kit/components/Avatar.tsx`)

- fallback na iniciály

## Composite components (molecules)

### Card (`ui-kit/components/Card.tsx`)

- surface container + shadow

### Table (`ui-kit/components/Table.tsx`)

- jednoduchá tabulka s povrchem

### Menu (`ui-kit/components/Menu.tsx`)

- dropdown pro Apps/Settings/Profile
- zavírání na klik mimo / ESC

## Layout components

- `AppTopBar` – sticky topbar s globální navigací, settings dropdownem a profile dropdownem
- `SidebarNav` – kontextový sidebar podle route
- `PageLayout` – základní page wrapper
- `TopBar` – page header (sekundární)

### AppTopBar (layout)

- **Left:** Dashboard, Apps (dropdown), Licensing
- **Right:** messaging icon, settings dropdown (theme toggle), user dropdown (Odhlásit)

### SidebarNav (layout)

- mění obsah dle `/core/dashboard`, `/core/apps`, `/core/licensing`

## Theming (light/dark)

- switch je v dropdownu nastavení (ozubené kolo)
- stav se ukládá do `localStorage` (klíč `hc_theme`)
- `useTheme` aplikuje `document.documentElement.dataset.theme`

## Doporučená struktura složek

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
