# Homarr

Homarr is deployed as an alternate dashboard and service launcher for the homelab.

Homepage remains the current primary dashboard while Homarr is evaluated side-by-side.

## Access

```text
http://homarr.homelab.local
```

The hostname should point to the shared Traefik ingress IP:

```text
homarr.homelab.local -> 192.168.0.200
```

## Deployment

Homarr is deployed by Argo CD as the `homarr` application.

It uses:

- `ghcr.io/homarr-labs/homarr:latest`
- Port `7575` inside the container
- `/appdata` for persistent application data
- Longhorn-backed PVC named `homarr-appdata`
- Traefik ingress for `homarr.homelab.local`

## Secret

Homarr requires a `SECRET_ENCRYPTION_KEY`.

The key is stored in a Kubernetes Secret named `homarr-secret` in the `homarr` namespace and is intentionally not committed to Git.

Create it with:

```powershell
kubectl create namespace homarr
$key = -join ((1..64) | ForEach-Object { "{0:x}" -f (Get-Random -Minimum 0 -Maximum 16) })
kubectl create secret generic homarr-secret -n homarr --from-literal=SECRET_ENCRYPTION_KEY=$key
```

## Operations

```powershell
kubectl get application homarr -n argocd
kubectl get pods -n homarr
kubectl get pvc -n homarr
kubectl logs -n homarr deployment/homarr
```
