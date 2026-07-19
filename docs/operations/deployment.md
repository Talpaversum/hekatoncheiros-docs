# Deployment support and operations

Status: Current support statement with production targets still open.

Docker Compose is the currently verified path for local self-hosting.

Bare-metal/VM and Kubernetes configurations are validated reference deployments. They are not currently distributed as packaged installers or Helm releases.

Development deployments may use locally built images and development trust material. This does not make them production deployments.

## Deployment levels

### Local Docker Compose

The Core Compose project builds Core and Web locally and runs PostgreSQL for development and local self-hosted validation. Optional application, Registry and issuer Compose projects are separate and must join explicitly configured networks where integration requires it.

Built-in passwords, HTTP endpoints, disposable PKI and published database ports are development conveniences. Operators must not carry these defaults into a production environment.

See the [Docker procedure](../deployment/docker.md).

### Bare-metal or VM reference

The validated reference uses an operator-managed PostgreSQL service, Node.js, systemd and nginx TLS termination. Installation automation, supported packaging, upgrade orchestration and recovery certification remain operator responsibilities.

See the [bare-metal/VM reference](../deployment/baremetal.md).

### Kubernetes reference

The validated Kustomize reference describes Core, Web, persistent Core data, external PostgreSQL credentials and ingress. It is not a Helm release, does not define a production image channel and does not establish a supported clustered Core topology.

See the [Kubernetes reference](../deployment/kubernetes.md).

## Production requirements

A production deployment requires, at minimum:

- externally managed secrets and rotation;
- operator-supplied TLS and ingress policy;
- persistent database and Core data storage;
- migration and rollback procedures;
- tested backup and recovery;
- production trust material provisioned outside application startup;
- restricted network exposure and service identities;
- readiness, liveness and operational monitoring;
- reviewed dependency and image provenance.

These requirements describe a target. Their presence in documentation does not mean every reference deployment has completed production validation.

See [database migrations](./database-migrations.md), [backup and recovery](./backup-and-recovery.md) and [production PKI](./production-pki.md).
