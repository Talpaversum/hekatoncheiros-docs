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

## UI Plugin Distribution Model

The UI plugin distribution model is normative.

- UI application code MUST be distributed as a **UI plugin artifact**
  (typically a JavaScript ESM bundle such as `plugin.js`).
- The web shell MUST NOT install, build, store, or copy UI plugin artifacts.
- The web shell repository MUST NOT be used as plugin artifact storage.
- Core MUST be the single owner of UI plugin artifact installation.
- Core MUST own artifact download, integrity verification (at least checksum),
  storage, and runtime exposure.
- Core MUST expose installed artifacts through stable core-owned URLs.
- The web shell MUST dynamically load plugin code only from `ui_url` provided
  by core at runtime.

### Development mode vs Production / Marketplace mode

- **Development mode** is a developer convenience workflow and MUST NOT be
  treated as the reference production model.
- In development mode, developers MAY build artifacts manually and install
  them afterward for local testing.
- **Production / Marketplace mode** MUST use prebuilt UI plugin artifacts.
- In production, build-on-install MUST NOT be used.

### Security model (summary)

- Core-hosted plugin artifacts are safer than loading third-party URLs directly
  in the web shell.
- Core MAY enforce checksum validation and artifact signing policies.
- Core MAY enforce CSP restrictions to core-owned origins for plugin loading.
- Plugin installation MUST be an admin-only operation.

## Not Supported

The following models are NOT supported:

- Standalone Single Page Applications (SPA)
- Applications with their own login screens
- User-facing UI served on a separate port
- Auto-mounting UI into the DOM
- Global window registration (e.g. window.hcApps)
