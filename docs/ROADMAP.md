# Hekatoncheiros – Platform Roadmap & TODO

> **Living document**  
> This roadmap captures platform priorities, architectural milestones,
> and a parking lot for ideas.  
> It is intentionally high-level and evolves over time.

---

## P0 — Core platform flow (MUST HAVE)
*Without this, the platform is not usable.*

### Core
- [ ] Core-hosted UI plugin assets (download, storage, serve)
- [ ] Installer-only auth for UI artifacts (core-signed token)
- [ ] Install-time enforcement (401/403 without installer auth)
- [ ] Endpoint `/api/v1/apps/:slug/ui/plugin.js`
- [ ] Clear installer error reporting (misconfigured apps)

### Web / UI
- [ ] **Manage Apps page**
  - list installed apps
  - install (base_url + manifest)
  - uninstall
- [ ] Apps dropdown driven by app registry
- [ ] RBAC in UI (install/manage only for admins)
- [ ] Dashboard as permanent entry point

### App model
- [ ] Consistent `slug` usage across UI and API
- [ ] Manifest snapshot stored in core
- [ ] Per-app DB schema provisioning (keep consistent)

---

## P1 — Usability & Developer Experience
*The platform becomes a usable product.*

### UI / UX
- [ ] Search in Apps dropdown
- [ ] Recently used apps
- [ ] Favorites / pinned apps (user preferences)
- [ ] App detail page (read-only: app_id, slug, ui_url, integrity)
- [ ] Clear empty states (no apps, no permissions)

### Developer Experience
- [ ] Manifest discovery (`/.well-known/hekatoncheiros/manifest.json`)
- [ ] Install without manual manifest JSON input
- [ ] Dev helpers:
  - build + install tooling
  - actionable error messages
- [ ] App health indicators in UI

---

## P2 — Operations, stability, security
*Long-term maintainability and safety.*

### Core
- [ ] Persistent AppInstallationStore (DB or file-based)
- [ ] Reinstall / refresh UI plugin artifact
- [ ] App enable / disable per tenant
- [ ] Audit log (install, uninstall, upgrade)
- [ ] Graceful handling of missing artifacts

### Security
- [ ] Mandatory artifact integrity checks (sha256)
- [ ] Signed artifacts (publisher trust)
- [ ] Allowlist of external artifact sources
- [ ] CSP restricted to core-hosted assets

---

## P3 — Marketplace & scale
*From platform to ecosystem.*

### Marketplace
- [ ] App catalog (metadata, screenshots, versions)
- [ ] Install wizard generated from manifest
- [ ] Publisher identity & verification
- [ ] Versioned app releases
- [ ] App upgrade / rollback

### UX at scale
- [ ] Apps dropdown grouping (categories)
- [ ] Tenant-level default apps
- [ ] Bulk install / rollout
- [ ] User onboarding flows

---

## P4 — Nice-to-have / long-term
*Strategic, not required for launch.*

- [ ] App permissions editor (UI)
- [ ] App usage analytics
- [ ] Cross-app navigation contracts
- [ ] Theme / branding per tenant
- [ ] Plugin sandboxing / stricter isolation

---

## Parking lot
*Unprioritized ideas and notes.*

- [ ] …
- [ ] …
- [ ] …

---

## Working rules
- P0 / P1 items are architecture-critical.
- Larger items should be backed by an ADR.
- This document is not a sprint plan.
