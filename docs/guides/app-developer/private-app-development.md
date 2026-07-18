# Private application development

Developer Tools is a project lifecycle workspace. It manages source connections, application discovery, validation, deployment, updates, runtime operations and logs. It is not an alternative label for Apps Management and it is not limited to one-time application installation.

Private application development is a local platform workflow. It does not use the official Talpaversum Author Registry.

A private developer does not:

- submit an author request,
- receive an `author_id`,
- receive an author certificate,
- use the official author workspace,
- publish to the official Talpaversum catalog.

## Workflow

1. Start Hekatoncheiros Core and Web.
2. Develop the application according to the application and manifest specifications.
3. Choose a runtime. Dockerfile and Docker Compose projects are built and
   started by Core; only an external-service project must already be running.
4. Open Developer Tools in Hekatoncheiros Web.
5. Select **Add project** and save a draft from the source-type step.
6. Select a source connection, repository, server workspace, manifest URL, or
   private feed URL.
7. Discover the application manifest and review its application and runtime
   configuration.
8. Test connectivity and explicitly approve the exact trusted origin.
9. Validate the manifest and review the manifest/runtime diff.
10. Deploy and install the application.
11. Continue managing updates, runtime state, deployment history, and sanitized
    logs from the project detail.

Developer Tools must execute this workflow directly. It must not be only a page containing links to Apps Management, Trusted Origins and feed settings.

## Trust

A trusted origin allows Core to retrieve application resources from an explicitly approved origin. It does not create an official author identity and does not make the application trusted by the Talpaversum Author Registry.

A private application must be visibly marked as local or unverified.

## Project and installed application

A developer project represents source, configuration, validation, deployment,
and its link to an installed application. The installed application is the
runtime/catalog record produced by a successful deployment. Removing or
stopping an installation does not turn it into the source project, and Apps
Management remains a lower-level technical view.

Projects are tenant-scoped and require `developer.*` privileges. They never
require `author_id`, an author certificate, or Author Registry access.

## Sources and connections

Supported project source types are GitHub, GitLab, a generic Git repository, a
server-side local workspace, a direct manifest URL, and a private feed.
Credentials belong to tenant-scoped connection records, are encrypted at rest,
are never returned to Web, and are erased when a connection is revoked. A
personal connection is visible only to its owner. A tenant-shared connection
requires the shared-connection privilege and is visible only inside its tenant.
A project stores only the connection reference.

GitHub App installation is the preferred GitHub connection. Token-based GitHub
and GitLab connections, HTTPS credentials, SSH deploy keys, and private-feed
credentials remain explicit connection types with minimal scopes.

GitHub App and GitLab connections discover the repositories available to the
installation or token and then list branches and tags. Generic Git validates an
explicit credential-free HTTPS or SSH repository URL and loads its refs. GitHub
installation tokens are short-lived, cached only until shortly before expiry,
and are never returned to Web or copied into a project.

## Local workspace

A local workspace is a directory available to the Core server and located under an explicitly configured workspace root. It is not an arbitrary directory selected from the user's browser.

A browser cannot provide a persistent server-side deployment connection to an arbitrary directory on the user's computer.

Core canonicalizes the path and rejects traversal or a path outside
`DEVELOPER_WORKSPACE_ROOTS`. In Docker, the workspace root must be mounted into
the Core container. A future ZIP upload, CLI, or local agent may bridge files
from a user's computer; the browser workflow does not claim that capability.

## Synchronization and updates

Synchronization records the source revision, reads the manifest, computes its
hash, and compares it with the validated and deployed revisions. The project
distinguishes unchanged source, an available update, required or failed
validation, required deployment, security-significant runtime review, and a
failed deployment.

Before validation or deployment, Developer Tools shows manifest and runtime
changes, including routes, privileges, origin, runtime configuration, and
licensing metadata.

Changes to permissions, capabilities, origin, licensing, managed-runtime
source, ports, volumes, image, or environment-variable names require an
explicit runtime approval. Approval is bound to the current diff hash and is
invalidated by the next synchronization. Environment values and secrets are
not included in the diff.

## Deployments, runtime, and logs

Every deployment records its revision, manifest hash, initiator, timestamps,
installation/runtime results, predecessor, and failure. Rollback is offered
only when the active runtime provider can restore a stored previous deployment.
Unsupported runtimes report that limitation instead of simulating success.

Dockerfile and Docker Compose are supported runtime providers. They are
configured only when `APP_RUNTIME_DOCKER_ENABLED` is true, Core has access to a
Docker daemon, and `DEVELOPER_DOCKER_NETWORK` names a network shared with the
application. The development Compose stack configures these requirements. An
external service has health validation but no Core-managed runtime operations;
direct manifests and feeds do not imply a managed runtime.

Managed runtimes expose Start, Stop, Restart, and Rebuild. Each action is
audited and recorded in the runtime log. During an update, Core restores the
previous active runtime when build, start, health, or installation fails. A
successful rollback rebuilds the stored previous source, validates its health
and manifest, and atomically makes it active.

Logs are categorized as source sync, build, validation, installation, runtime,
or deployment. They can be filtered by tenant project and deployment and
downloaded. Authorization headers, tokens, passwords, secrets, API keys, and
private key blocks are redacted before storage.

Stored developer logs are limited to 100,000 characters per entry, 10,000
entries per project, and 30 days. List responses are paginated; managed runtime
logs can be read live with a bounded tail.
