# Web frontend documentation

This folder describes the frontend (TypeScript/React) for managing **Hekatoncheiros Core**. The documentation is written to serve as a **foundation for other applications**, especially the UI kit and shared components.

## Goals

- provide a baseline console for core/kernel
- define shared UI components (buttons, forms, layout)
- document the base architecture and data flows

## Runtime dependencies

- Backend Core API: `http://127.0.0.1:3000`
- Database (PostgreSQL) runs in Docker on port `5432`

## Documentation structure

- [Frontend architecture](./architecture.md)
- [UI kit / Design system](./ui-kit.md)
- [Core/Kernel console](./core-kernel-console.md)

## Conventions

- The docs prefer a **component-first approach** and **reusable UI primitives**.
- Components are designed to be **reusable across applications**.
- UI routes align with the core API surface (`/api/v1/...`).
