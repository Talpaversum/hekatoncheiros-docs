# Kubernetes Deployment

Status: future work.

Kubernetes is a target deployment mode, but there are no maintained manifests or
Helm chart yet.

## Expected first version

- Namespace
- Core Deployment
- web Deployment
- PostgreSQL StatefulSet or external DB Secret
- Services
- Ingress
- ConfigMap and Secret
- PVC for Core-owned data if needed

## Defaults

- HTTP ingress for development
- one host per HC instance
- app-specific hostnames later
- local/dev image handling first
- production image registry later

## Probes

Core:

```text
readiness: /api/v1/readyz
liveness: /api/v1/healthz
```

App backends:

```text
/health
```

## Packaging decision

Helm is the likely packaging format once Kubernetes work starts, but this is not
decided yet.
