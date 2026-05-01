# DataGerry Helm Chart

Starter Helm chart for DataGerry on Kubernetes with:
- Backend and frontend Deployments
- Ingress for `/`, `/rest`, and `/docs`
- Liveness/Readiness probes (`/rest/healthz` and `/rest/readyz`)
- Cloud-mode runtime key material via Kubernetes Secret

## 1) Generate cloud key material

The backend expects base64-encoded key values for cloud mode.

```bash
# RSA keys
openssl genrsa -out private.pem 4096
openssl rsa -in private.pem -pubout -out public.pem

# AES-256 symmetric key (32 random bytes, base64)
openssl rand -base64 32 > symmetric.b64

# Convert RSA PEM files to single-line base64
export DG_RSA_PRIVATE_KEY="$(base64 -w 0 private.pem)"
export DG_RSA_PUBLIC_KEY="$(base64 -w 0 public.pem)"
export DG_SYMMETRIC_KEY="$(cat symmetric.b64)"
```

## 2) Create values override

```yaml
secrets:
  create: true
  rsaPrivateKey: "<base64-private-pem>"
  rsaPublicKey: "<base64-public-pem>"
  symmetricKey: "<base64-32-byte-key>"

backend:
  env:
    connectionString: "mongodb://user:pass@mongo:27017"
  corsOrigins:
    - "https://tenant-a.example.com"
    - "https://tenant-b.example.com"
```

## 3) Install

```bash
helm upgrade --install datagerry ./contrib/helm/datagerry -f values.yaml -n datagerry --create-namespace
```

## Multi-tenant notes

- This chart enables backend cloud mode by default (`--cloud` argument).
- Tenant isolation is expected at the application level via tenant database mapping from authenticated user claims.
- Prefer tenant hostnames (or wildcard host) in ingress to keep clean separation between tenant UX endpoints.
- For production, use an external managed MongoDB and set backend.env.connectionString.
