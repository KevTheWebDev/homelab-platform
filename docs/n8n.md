# n8n

n8n is deployed as the homelab workflow automation platform.

## Access

```text
http://n8n.homelab.local
```

Create this Pi-hole local DNS record:

```text
n8n.homelab.local -> 192.168.0.200
```

`192.168.0.200` is the shared Traefik ingress LoadBalancer IP.

## Deployment

n8n is deployed by Argo CD as the `n8n` application.

The deployment includes:

- Namespace: `n8n`
- n8n image: `docker.n8n.io/n8nio/n8n:latest`
- PostgreSQL image: `postgres:16-alpine`
- n8n data PVC: `n8n-data`
- PostgreSQL data PVC: `n8n-postgres-data`
- Storage class: `longhorn`
- Node placement: `k3s-05`
- Ingress hostname: `n8n.homelab.local`

## Secrets

n8n requires secrets that are intentionally not committed to Git:

- `POSTGRES_PASSWORD`
- `N8N_ENCRYPTION_KEY`

Create the secret before syncing the n8n application:

```powershell
kubectl create namespace n8n

$postgresPassword = -join ((1..48) | ForEach-Object { "{0:x}" -f (Get-Random -Minimum 0 -Maximum 16) })
$encryptionKey = -join ((1..64) | ForEach-Object { "{0:x}" -f (Get-Random -Minimum 0 -Maximum 16) })

kubectl create secret generic n8n-secret -n n8n `
  --from-literal=POSTGRES_PASSWORD=$postgresPassword `
  --from-literal=N8N_ENCRYPTION_KEY=$encryptionKey
```

Do not lose the encryption key. n8n uses it to encrypt credentials.

## Check n8n

```powershell
kubectl get application n8n -n argocd
kubectl get pods -n n8n -o wide
kubectl get svc -n n8n
kubectl get pvc -n n8n
kubectl get ingress -n n8n
kubectl logs -n n8n deployment/n8n --tail=100
```

## Notes

This first deployment runs n8n as a single instance backed by PostgreSQL.

Queue mode, Redis, and separate workers can be added later if workflows become CPU-heavy or need more concurrency.
