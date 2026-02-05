# Application Execution Model

Hekatoncheiros supports applications as PLUGINS, not standalone web applications.

An application provides:
- backend services (optional)
- UI components
- declarative metadata (manifest)

The application UI is rendered INSIDE the core web shell.
Applications must not expose their own user-facing web frontend.

## Supported: Plugin UI Model

- Applications export UI components as a module.
- Core imports the module and renders components using its own router.
- Authentication and session handling are owned by core.
- Applications receive API clients and auth context from core.

## Not Supported

The following models are NOT supported:

- Standalone Single Page Applications (SPA)
- Applications with their own login screens
- User-facing UI served on a separate port
- Auto-mounting UI into the DOM
- Global window registration (e.g. window.hcApps)
