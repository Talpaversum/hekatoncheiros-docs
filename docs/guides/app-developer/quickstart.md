# App Developer Quickstart

Status: draft.

HC apps are plugins integrated into the platform shell. They are not standalone
user-facing SPAs with their own login.

## Minimal App Repository

```text
manifest/app-manifest.json
src/plugin.ts
src/server/*            # optional backend
```

The manifest is the integration contract. It declares identity, privileges,
API/UI integration, and licensing expectations.

## UI Plugin

The UI plugin exports a registration function, for example:

```ts
export function register(appContext) {
  return {
    routes: [],
    nav_entries: [],
  };
}
```

The platform provides context and API access. Apps must not implement their own
platform authentication.

## Local Development

1. Run Core.
2. Run the app backend if the app has one.
3. Build or watch the UI plugin artifact.
4. Install/register the app through Core tooling.

Development servers are allowed for iteration only. Runtime UI must still be
loaded by the platform shell.

## Production Direction

Production app distribution should use prebuilt UI plugin artifacts. Core owns
artifact download, validation, storage, and runtime `ui_url` generation.

Build-on-install is acceptable only as a development convenience for now.

## See Also

- [Manifest](./manifest.md)
- [UI integration](./ui-integration.md)
- [Application execution model](../../architecture/app-execution-model.md)
