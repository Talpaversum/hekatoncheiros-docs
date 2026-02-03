# UI kit / Design systém

UI kit slouží jako sdílená knihovna komponent pro Core Console i další aplikace. Cílem je konzistence, rychlá iterace a jednoduché přenesení brandingu.

## Design principy

- **Konzistence**: jednotné spacing, barvy, typografie
- **Kompozice**: malé prvky skládáme do větších
- **Přístupnost**: klávesové ovládání, kontrast, aria atributy
- **Tematizace**: theme tokens (light/dark + tenant branding)

## Tokens

- `color.primary`, `color.secondary`, `color.danger`
- `space.xs/sm/md/lg/xl`
- `radius.sm/md/lg`
- `shadow.sm/md/lg`

## Základní komponenty (atomy)

### Button

- varianty: `primary`, `secondary`, `danger`, `ghost`
- stavy: `default`, `hover`, `disabled`, `loading`

### Input

- text, password, email
- validace, error stav

### Select

- single i multi select

### Badge

- status (active/disabled/pending)

### Avatar

- uživatel, inicála fallback

## Kompozitní komponenty (molekuly)

### FormField

- label + input + error

### Card

- header + body + footer

### Table

- sortable header, row actions, empty state

### Modal

- confirm / destructive modals

### Notification

- toast / inline alert

## Layout komponenty

- `PageLayout` – hlavní layout stránky
- `SidebarNav` – navigace
- `TopBar` – kontext, profil, tenant
- `SectionHeader` – nadpis + akce

## Ikony a stavy

- konzistentní sada ikon (např. lucide/react)
- jednotné empty states a loading skeletons

## Doporučená struktura

```
src/ui-kit/
  components/
    Button/
    Input/
    FormField/
    Table/
  tokens/
    colors.ts
    spacing.ts
  styles/
    global.css
```
