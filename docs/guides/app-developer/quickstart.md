# App Developer Quickstart

This guide shows how to implement an application for the Hekatoncheiros platform.

It assumes you have read and accepted the App Manifest Specification.

## Execution model (mandatory)

Hekatoncheiros applications run as **plugins**.

The execution model is defined by the App Manifest and enforced by the platform kernel.

In particular:

- Applications contribute UI components
- UI is rendered inside the platform web shell
- Applications do not run standalone web frontends
- Applications do not own routing or authentication

If your application requires a standalone SPA or its own login flow, it is not compatible with this platform.

## What you build

A typical application repository contains:

- `manifest/app-manifest.json`  
  Declarative description of identity, privileges, APIs, and UI contribution.
- `src/plugin.ts`  
  UI plugin entrypoint exporting components and a registration function.
- `src/server/*` (optional)  
  Backend services exposed via HTTP APIs.

The manifest is the authoritative contract.
Your code must conform to it.

## UI plugin structure

Your UI code is an importable module.

It must export a registration function that returns UI contributions declared in the manifest.

Conceptually:

- pages (React components)
- optional widgets or panels
- navigation metadata (aligned with the manifest)

The platform provides an `appContext` containing:
- authenticated API access
- user and tenant context
- configuration and feature flags

Applications must not implement their own authentication or session handling.

## Local development

Recommended workflow:

1. Run the core platform locally.
2. Run the app backend locally (if applicable).
3. Build or watch the UI plugin module.

A local dev server may be used for faster iteration, but:
- it is a developer tool only
- it must not be treated as a user-facing frontend
- production builds must be consumable as a module by the core shell

## Packaging and installation

Applications are installed through platform tooling (installer or marketplace).

During installation:
- the manifest is validated
- privileges are approved
- UI contributions are registered

At runtime:
- the core loads the UI module
- routes and navigation are owned by the core
- privileges are enforced centrally

## Next steps

- App Manifest Specification
- UI Integration Guide
- Privileges and Authorization Guide
