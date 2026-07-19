# Application availability

Application availability is evaluated by Core through the installed application's configured `base_url` and declared health endpoint.

A running Docker container does not by itself mean that the application is reachable or healthy from the Core container.

Application availability consists of separate states:

- installation status,
- runtime health,
- license status,
- UI integration status.

The Web interface must display the individual states and the exact reason when an application cannot be opened.

## Docker networking

When Core and an application run in different containers, `localhost` inside Core refers to the Core container itself.

The application's internal Docker network hostname must be used as its `base_url`, for example:

`http://inventory:4010`

Core and the application must be connected to a shared Docker network.

Verify the following when Core reports a DNS or connection error:

- both services are attached to the same Docker network,
- the hostname in `base_url` matches the application's service name or network alias,
- the application listens on the configured container port,
- the deployment stores the internal URL rather than a host-published URL.

Use **Run diagnostics** on the installed application detail to check DNS resolution, HTTP connectivity, the health and manifest endpoints, the stored UI artifact, license binding, and trusted origin. Diagnostics report findings and recommendations without changing configuration.

## Inventory troubleshooting

Inventory may be reported as degraded even when its container is running. Its health endpoint also verifies database availability. If the database check fails, Inventory returns a degraded or HTTP 503 health response.

Core maps a valid `degraded` response to degraded runtime health even when the endpoint uses HTTP 503. Check Inventory database connectivity and dependent services before restarting the application container.

## Diagnostic outcomes

- `localhost_misconfiguration`: replace `localhost` with the Docker network hostname.
- `dns_failed`: verify the shared Docker network and service alias.
- `connection_refused`: verify the container port and the address on which the process listens.
- `connection_timeout`: verify routing, firewall rules, and application startup.
- `tls_error`: verify the certificate chain and hostname.
- `health_endpoint_missing`: verify the health path declared by the manifest.
- `health_degraded`: inspect the reported message and application dependencies.
- `license_missing`: bind an active tenant license when the application requires one.
