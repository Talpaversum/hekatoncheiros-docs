# UI kit / Design system

The UI kit is a shared component library for the Core Console and other applications. The goal is consistency, fast iteration, and easy branding transfer.

## Design principles

- **Consistency**: unified spacing, colors, typography
- **Composition**: small parts build into larger units
- **Accessibility**: keyboard navigation, contrast, aria attributes
- **Theming**: theme tokens (light/dark + tenant branding)

## Tokens

- `color.primary`, `color.secondary`, `color.danger`
- `space.xs/sm/md/lg/xl`
- `radius.sm/md/lg`
- `shadow.sm/md/lg`

## Base components (atoms)

### Button

- variants: `primary`, `secondary`, `danger`, `ghost`
- states: `default`, `hover`, `disabled`, `loading`

### Input

- text, password, email
- validation, error state

### Select

- single and multi select

### Badge

- status (active/disabled/pending)

### Avatar

- user avatar with initials fallback

## Composite components (molecules)

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

## Layout components

- `PageLayout` – main page layout
- `SidebarNav` – navigation
- `TopBar` – context, profile, tenant
- `SectionHeader` – title + actions

## Icons and states

- consistent icon set (e.g., lucide/react)
- unified empty states and loading skeletons

## Suggested structure

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
