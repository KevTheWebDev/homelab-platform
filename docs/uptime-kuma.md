# Uptime Kuma

Uptime Kuma provides service uptime checks and status visibility for the homelab.

## Access

```text
http://uptime.homelab.local
```

The hostname should point to the shared Traefik ingress IP:

```text
uptime.homelab.local -> 192.168.0.200
```

## Deployment

Uptime Kuma is deployed by Argo CD as the `uptime-kuma` application.

It uses:

- `louislam/uptime-kuma:2`
- Port `3001` inside the container
- `/app/data` for persistent application data
- Longhorn-backed PVC named `uptime-kuma-data`
- Traefik ingress for `uptime.homelab.local`

## Recommended monitors

| Monitor | Type | Target |
|---|---|---|
| Homepage | HTTP(s) | `http://homepage.homelab.local` |
| Grafana | HTTP(s) | `http://grafana.homelab.local/login` |
| Argo CD | HTTP(s) | `https://argocd.homelab.local` |
| Pi-hole Web | HTTP(s) | `http://pihole.homelab.local/admin` |
| Pi-hole DNS | DNS | `example.com` using resolver `192.168.0.205` |
| n8n | HTTP(s) | `http://n8n.homelab.local` |
| Test Nginx | HTTP(s) | `http://nginx.homelab.local` |

## Operations

```powershell
kubectl get application uptime-kuma -n argocd
kubectl get pods -n uptime-kuma
kubectl get pvc -n uptime-kuma
kubectl logs -n uptime-kuma deployment/uptime-kuma
```
