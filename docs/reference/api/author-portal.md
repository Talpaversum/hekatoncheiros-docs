# Author Portal API

All routes are below `/api/v1/author-portal` and require a Core user token.
Author resources enforce membership and `author.*` permissions. Administrative
routes enforce the corresponding `platform.*` privilege.

- `GET /overview` returns the current user's requests, profiles, apps,
  submissions, operating-mode policies, and review capabilities.
- `POST /requests`, `PUT /requests/:id`, and
  `POST /requests/:id/submit` manage author request drafts.
- `GET /admin/requests` and `POST /admin/requests/:id/action` implement review,
  approval, suspension, and revocation.
- `GET|PUT /profiles/:id` and `GET|PUT /profiles/:id/members` manage an
  author-scoped profile and RBAC membership.
- `/git-connections` routes connect/revoke GitHub credentials, list accessible
  repositories, and inspect manifests without exposing stored tokens.
- `/apps` routes create app drafts from Git and apply app/runtime workflow
  actions.
- `/catalog-submissions` routes submit, review, publish, and unpublish official
  catalog candidates.
- `GET /activity` returns author workflow and security events visible to the
  current user.

Trusted-origin connectivity is tested with
`POST /api/v1/platform/trusted-origins/:id/test`; it requires platform
configuration permission.
