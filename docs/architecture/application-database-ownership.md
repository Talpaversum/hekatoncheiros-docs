# Application database ownership

Status: Open decision.

Core-managed applications currently use a Core-controlled application schema and Core-managed migrations. The ownership model for standalone or externally operated application runtimes is unresolved.

The decision must define:

- **Credentials:** who creates, stores, rotates and revokes database credentials;
- **Migrations:** whether Core or the external operator applies and verifies migrations;
- **Backup and restore:** consistency boundaries, schedules, recovery order and validation ownership;
- **Uninstall:** retention, export and deletion behavior when an application is removed;
- **Ownership transfer:** how data and credentials move between operators without crossing tenant or app boundaries;
- **External runtime:** network access and responsibility when Core does not manage the process;
- **Tenant/app data boundary:** isolation from `core`, other applications and other tenants.

Until this decision is approved, deployment references must not promise direct database integration for arbitrary standalone applications.

Related current policy: [database policy](./database-policy.md).
