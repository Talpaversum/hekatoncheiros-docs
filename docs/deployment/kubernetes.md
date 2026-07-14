# Kubernetes Deployment

Status: validated Kustomize reference manifests; not yet a Helm distribution.

`hekatoncheiros-core/deploy/kubernetes` provides a namespace, Core and web
Deployments and Services, Core data PVC, ConfigMap, and TLS Ingress. PostgreSQL
is external and credentials come from a separately created Secret.

## Prerequisites and images

- Kubernetes with a default StorageClass and nginx Ingress Controller
- externally reachable PostgreSQL and a TLS certificate
- Core/web images loaded on every node, or immutable registry tags

The reference uses `imagePullPolicy: Never`. Build Core normally. From the
directory containing both repositories, build the non-root Kubernetes web image:

```bash
docker build -f hekatoncheiros-core/deploy/kubernetes/web.Dockerfile \
  -t hekatoncheiros-web:k8s-local .
```

## Secrets and apply

```bash
kubectl create namespace hekatoncheiros
kubectl -n hekatoncheiros create secret generic hekatoncheiros-secrets \
  --from-literal=DATABASE_URL='postgres://hc_user:replace-me@db.example.com:5432/hc_core' \
  --from-literal=JWT_SECRET='replace-with-generated-secret' \
  --from-literal=INSTALLER_TOKEN_SECRET='replace-with-different-generated-secret'
kubectl -n hekatoncheiros create secret tls hekatoncheiros-tls \
  --cert=tls.crt --key=tls.key
kubectl kustomize deploy/kubernetes
kubectl apply --dry-run=client -k deploy/kubernetes
kubectl apply -k deploy/kubernetes
```

Change `hc.example.com` in `ingress.yaml`. For licensing OAuth, add its HTTPS
callback URL to the ConfigMap.

Core startup applies migrations under an advisory lock. The reference uses one
Core replica and a `ReadWriteOnce` volume; scaling needs a storage and runtime
ownership review.

```bash
kubectl -n hekatoncheiros rollout status deployment/core
kubectl -n hekatoncheiros rollout status deployment/web
kubectl -n hekatoncheiros get ingress,pods,svc,pvc
curl --fail https://hc.example.com/api/v1/readyz
```

Core readiness is `/api/v1/readyz`; liveness is `/api/v1/healthz`.
`APP_RUNTIME_DOCKER_ENABLED=false`, so Compose runtimes are unavailable here.
Remaining production work includes signed registry images, PostgreSQL HA and
backup policy, NetworkPolicies, and a Helm packaging decision.
