# Task: Author Workflows, Catalog Publishing, Git Integration and UI Completion

## Context

Hekatoncheiros must support three distinct application authoring and operating modes.

Current implementation already contains partial backend functionality:

- Core has admin-oriented author onboarding and registry proxy routes under `/platform/authors*` and `/platform/author-registry/*`.
- Core can issue delegated user tokens toward the Author Registry.
- `hc-app-licensing` already uses delegated Core identity instead of local admin accounts.
- `hc-app-licensing` has management APIs for products, customers, grants, activations, licenses, audit and issuer identity validation.
- Web already contains Apps, Licensing and Authors-related pages.
- Runtime startup already has an approval/security concept.

But the end-to-end user-facing workflows are not complete.

The goal is to make all three authoring modes possible through usable UI, not only through backend APIs.

---

# Required Authoring Modes

## Mode 1: Talpaversum-hosted official author

The author:

- registers on the official Talpaversum instance,
- requests author status,
- is approved in the official Author Registry,
- connects a Git repository,
- develops the application in their repository,
- uses Talpaversum-hosted build/runtime,
- uses Talpaversum-hosted licensing,
- can publish the application into the official catalog.

This is the default low-friction author experience.

The author does not operate infrastructure.

Talpaversum operates:

- Author Registry,
- app hosting/runtime,
- build/deploy pipeline,
- licensing issuer,
- signing key storage,
- official catalog,
- approval workflow,
- audit.

---

## Mode 2: Trusted self-hosted author

The author:

- runs their own Hekatoncheiros instance,
- registers on the official Talpaversum instance,
- is approved in the official Author Registry,
- receives an `author_id` and author certificate,
- configures their own `hc-app-licensing`,
- develops and operates applications on their own infrastructure,
- may publish applications into the official Talpaversum catalog,
- issues licenses through their own licensing server.

Talpaversum provides trust and catalog publishing.

The author operates:

- their own Core/Web,
- their own licensing server,
- their own app runtime,
- their own signing key,
- their own customers/licenses.

---

## Mode 3: Private self-hosted developer

The developer:

- runs their own Core and Web,
- does not register with Talpaversum,
- does not use the official Author Registry,
- does not publish to the official catalog,
- writes applications for themselves or for a closed private ecosystem,
- installs applications manually or through a private catalog.

Important: since the official Author Registry only runs on the Talpaversum instance, this mode must not depend on author registration.

This mode is not “author registry authoring”.

It is private local application development.

Typical workflow:

1. run Core and Web,
2. create an application according to the app specification,
3. run the app runtime, for example using Docker,
4. configure the app origin / trusted origin,
5. add or import the app manifest/feed,
6. install the app into the local instance,
7. use it locally.

---

# Main Requirement

Implement these three modes as explicit product workflows in Core/Web and related repositories.

Do not leave this as backend-only functionality.

There must be visible UI actions, forms, status screens, help text and documentation.

---

# Repository Scope

At minimum update:

- `hekatoncheiros-core`
- `hekatoncheiros-web`
- `hc-app-licensing`
- `hc-author-registry`
- `hekatoncheiros-docs`

If needed, also update example app repositories or app manifest documentation.

---

# Core Requirements

## 1. Model author operating modes

Add explicit support for author/app operating modes.

Suggested enum:

- `talpaversum_hosted`
- `trusted_self_hosted`
- `private_self_hosted`

The system must use this mode to decide:

- whether official registry approval is required,
- whether an author certificate is required,
- whether the app may be submitted to the official catalog,
- whether licensing is Talpaversum-hosted or external,
- whether the runtime is Talpaversum-managed or external,
- what UI actions are available.

---

## 2. Author application workflow

Add a first-class author request workflow.

Current Core author onboarding is admin-oriented and uses `platform.authors.manage`.

Add user-facing author applications.

Required states:

- `draft`
- `submitted`
- `pending_review`
- `needs_changes`
- `approved`
- `rejected`
- `suspended`
- `revoked`

Required actions:

- create author request,
- edit draft request,
- submit request,
- admin review,
- request changes,
- approve,
- reject,
- suspend,
- revoke,
- view audit/history.

Required fields:

- requested display name,
- legal/company name if applicable,
- contact email,
- website,
- Git provider profile,
- short description,
- requested operating mode,
- intended app distribution,
- agreement/terms confirmation,
- review notes.

When approved:

- create or link `author_id`,
- create author profile,
- create membership for requesting user,
- assign author-scoped permissions,
- if official/trusted mode requires registry identity, call Author Registry onboarding.

---

## 3. Author memberships and scoped RBAC

Add author-scoped access.

A platform user may manage one author but not another.

Required model:

- `author`
- `author_membership`
- `author_role`
- `author_permission`
- optional `author_invitation`

Suggested permissions:

- `author.profile.manage`
- `author.members.manage`
- `author.git.manage`
- `author.apps.create`
- `author.apps.manage`
- `author.apps.submit`
- `author.apps.publish`
- `author.licensing.manage`
- `author.licensing.issue`
- `author.licensing.revoke`
- `author.audit.read`

Keep existing platform-wide permissions for Talpaversum operators:

- `platform.authors.manage`
- `platform.catalog.manage`
- `platform.apps.runtime.manage`
- `platform.author_registry.manage`

Author-scoped permissions must never give global platform admin rights.

---

## 4. Git integration

Implement Git connection as a first-class workflow.

Support at least a provider abstraction with initial implementation for GitHub.

Do not assume repositories are public.

Private repositories must be supported.

Required capabilities:

- connect Git provider account,
- store provider connection securely,
- support GitHub App installation or token-based access,
- select repository,
- verify repository access,
- verify ownership or granted access,
- select branch,
- locate app manifest,
- read manifest from repository,
- validate manifest,
- show validation errors in UI,
- refresh/sync repository metadata,
- disconnect repository.

Security requirements:

- access tokens must be encrypted at rest,
- tokens must never be shown after creation,
- token usage must be audited,
- support token revocation/disconnect,
- private repository access must not leak repository contents to unauthorized users,
- Git connection must be scoped to the author.

The UI must make clear whether the repository is:

- public,
- private,
- inaccessible,
- missing manifest,
- manifest invalid,
- ready.

---

## 5. App creation from Git

Add app creation flow from connected repository.

Required flow:

1. author selects connected Git repository,
2. Core reads app manifest,
3. Core validates app identity, integration slug, runtime metadata, UI metadata and licensing requirements,
4. Core creates app draft,
5. author reviews app metadata,
6. author submits app for build/publish/runtime approval.

Required app states:

- `draft`
- `manifest_invalid`
- `ready_for_review`
- `submitted`
- `approved`
- `rejected`
- `published`
- `runtime_pending`
- `runtime_approved`
- `running`
- `disabled`

For Mode 1:

- app can be built/deployed by Talpaversum,
- runtime start requires operator approval unless policy explicitly allows auto-start for safe runtimes.

For Mode 2:

- app is external/self-hosted,
- official catalog entry points to external manifest/feed and external licensing issuer,
- Talpaversum does not run the app.

For Mode 3:

- app is local/private,
- no official catalog publishing,
- no Talpaversum registry requirement.

---

## 6. Runtime policy

Make runtime policy explicit in UI and backend.

Suggested policy:

### Talpaversum-hosted app

Author may request deployment/start.

Only users with `platform.apps.runtime.manage` can approve first runtime start or dangerous runtime changes.

Runtime approval must show:

- image/build source,
- compose/runtime plan,
- exposed ports,
- volumes,
- environment variables,
- Docker socket usage,
- network access,
- declared health check,
- declared origins,
- app permissions.

### Trusted self-hosted app

Talpaversum does not run the app.

The author’s own instance controls runtime policy.

Official catalog may only reference the app and licensing issuer.

### Private self-hosted app

Local instance admin controls runtime policy.

No official registry or catalog is involved.

---

## 7. Trusted origins / app origins

Complete the user-facing flow for private self-hosted apps.

There must be UI and help for:

- adding trusted origin,
- reviewing origin risk,
- attaching origin to app manifest/feed,
- testing connectivity,
- installing app from local/external origin,
- showing origin status.

Private app installation should not require Talpaversum author registry.

It may require local admin approval.

---

## 8. Licensing per mode

### Mode 1: Talpaversum-hosted official author

Licensing is hosted on the Talpaversum instance.

Required:

- managed licensing namespace per `author_id`,
- author can manage only their own products/customers/grants/licenses,
- signing key is managed by Talpaversum by default,
- author does not need to copy private keys,
- UI must expose products, grants, licenses and activations scoped to author.

### Mode 2: Trusted self-hosted author

Licensing is external.

Required:

- author gets certificate from Talpaversum registry,
- author configures their own `hc-app-licensing`,
- official catalog entry stores external issuer discovery URL,
- catalog validation verifies issuer identity and author certificate chain,
- customer Core validates license chain against Talpaversum root.

### Mode 3: Private self-hosted developer

Licensing is optional/private.

Required:

- app may be unlicensed if local policy allows it,
- app may use a local/private issuer,
- no Talpaversum registry dependency,
- UI must clearly show that the app is private/unverified.

---

# Web/UI Requirements

## 1. Add Author Portal

Add a visible Author Portal in the web UI.

Suggested navigation:

- Author Portal
  - Overview
  - Become an Author
  - My Author Profiles
  - Git Connections
  - Applications
  - Licensing
  - Catalog Submissions
  - Audit / Activity

Only show sections allowed by permissions and mode.

---

## 2. Become an Author UI

For ordinary users:

- button: “Become an Author”
- form for author request,
- mode selection:
  - Hosted on Talpaversum
  - Trusted self-hosted
  - Private self-hosted / local only

Important: for private self-hosted mode, explain that no Talpaversum author registration is needed.

If user selects private self-hosted, UI should guide them to local app development documentation instead of creating official author request.

---

## 3. Admin review UI

For Talpaversum operators:

- pending author requests,
- request detail,
- approve,
- reject,
- request changes,
- suspend,
- revoke,
- audit trail.

When approving, show exactly what will be created:

- author profile,
- registry author,
- certificate/key workflow if applicable,
- memberships,
- permissions.

---

## 4. Git UI

Add pages/components for:

- Connect Git provider,
- list connected providers,
- select repository,
- private repo status,
- manifest detection,
- manifest validation results,
- create app from repo.

Every error must be visible and actionable.

---

## 5. App workflow UI

Author must be able to click through:

- create app from Git,
- review manifest,
- fix manifest errors,
- submit app,
- request runtime approval if hosted,
- submit to official catalog if eligible,
- see status and next action.

---

## 6. Catalog publishing UI

Add catalog submission workflow.

Required:

- submit app to official catalog,
- show eligibility by mode,
- show validation errors,
- operator review,
- approve/reject,
- publish/unpublish.

Catalog entries must distinguish:

- Talpaversum-hosted official app,
- trusted external app,
- private/local app.

Private/local apps must not appear in official catalog.

---

## 7. Help and empty states

Every new UI page must have useful empty states and inline help.

At minimum add help text for:

- what an author is,
- difference between the three modes,
- why private self-hosted does not need official registry,
- why private repos require a Git connection,
- what runtime approval means,
- what trusted origin means,
- what official catalog publishing means,
- what licensing issuer means.

Do not leave blank tables without explanation.

---

# Translation Requirements

Update i18n resources for all supported platform languages:

- EN
- CS
- SK
- DE
- FR
- ES

Add translations for:

- Author Portal
- Become an Author
- operating modes
- Git connection
- repository states
- manifest validation
- app submission
- runtime approval
- catalog publishing
- licensing states
- trusted origin help
- private self-hosted help
- registry trust warnings
- audit labels
- error messages

Fallback language remains EN.

No new user-visible string should be hardcoded only in English.

---

# Documentation Requirements

Update `hekatoncheiros-docs`.

Add or update docs for:

## 1. Authoring modes

Document all three modes:

- Talpaversum-hosted official author,
- trusted self-hosted author,
- private self-hosted developer.

Include:

- who operates what,
- where registry is used,
- where licensing runs,
- whether official catalog publishing is allowed,
- required setup,
- security implications.

## 2. Author onboarding

Document:

- user request,
- admin approval,
- author membership,
- registry certificate,
- suspension/revocation.

## 3. Git integration

Document:

- public repo,
- private repo,
- access token/GitHub App,
- manifest detection,
- token security.

## 4. Private self-hosted app development

Document the local flow:

1. run Core and Web,
2. create app according to spec,
3. run app backend/runtime,
4. configure trusted origin,
5. expose app manifest/feed,
6. install app locally,
7. optionally configure local/private licensing.

## 5. Runtime approval

Document:

- why approval exists,
- Docker socket risk,
- hosted vs self-hosted policy,
- what operators must review.

## 6. Catalog publishing

Document:

- official hosted apps,
- trusted external apps,
- private apps,
- review process,
- unpublishing/revocation.

---

# Security Requirements

- Do not create a production Registry Root Key automatically.
- Dev/test roots must remain clearly marked as non-production.
- Official registry trust must be based on cryptographic root key, not URL.
- Private self-hosted mode must not depend on Talpaversum registry.
- Git tokens must be encrypted at rest.
- Runtime approval must prevent unreviewed dangerous deployments.
- All administrative actions must be audited with real user identity.
- Machine-to-machine operations must not be logged as anonymous `ADMIN_TOKEN`.
- Author-scoped users must not access other authors’ data.

---

# Acceptance Criteria

## Mode 1

A user can:

1. register/login,
2. click “Become an Author”,
3. request Talpaversum-hosted author mode,
4. be approved by an operator,
5. connect a Git repo,
6. create an app draft from repo manifest,
7. submit it,
8. request runtime/catalog approval,
9. manage hosted licensing,
10. publish to official catalog after approval.

## Mode 2

A user can:

1. request trusted self-hosted author mode,
2. be approved by Talpaversum,
3. receive/obtain author certificate flow,
4. configure external licensing issuer,
5. submit external app metadata to official catalog,
6. pass validation,
7. publish trusted external app.

## Mode 3

A private instance admin/developer can:

1. run Core/Web locally,
2. create/run an app outside Talpaversum registry,
3. configure trusted origin,
4. install app from local/private manifest/feed,
5. use the app without official author registration,
6. see that the app is private/unverified.

## UI

- All workflows have visible navigation.
- All actions are clickable.
- All important states are visible.
- Empty states explain next steps.
- Errors are actionable.

## Translations

- No new user-facing text lacks i18n keys.
- EN and CS must be complete.
- SK, DE, FR and ES must have at least complete non-empty translations or documented fallback handling.

## Documentation

- Docs explain all three workflows.
- Docs explain security/trust implications.
- Docs explain private self-hosted workflow without official registry.

---

# Notes

Do not collapse the three modes into two.

Important distinction:

- trusted self-hosted author is registered with Talpaversum and may publish to official catalog,
- private self-hosted developer is not registered with Talpaversum and cannot use official author registry features.

The official Author Registry exists only on the Talpaversum instance.

Private self-hosted ecosystems may implement their own local trust model, but that is not the official Talpaversum author registry.
