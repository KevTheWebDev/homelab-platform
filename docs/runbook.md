# Runbook

## Check cluster health

```powershell
kubectl get nodes -o wide
kubectl get pods -A
```

## Check MetalLB

```powershell
kubectl get pods -n metallb-system -o wide
kubectl get ipaddresspools -n metallb-system
kubectl get l2advertisements -n metallb-system
```

## Check Argo CD

```powershell
kubectl get pods -n argocd
kubectl get svc -n argocd argocd-server
```

Argo CD should use `externalTrafficPolicy: Local` in this environment. That setting is managed by the `argocd-config` Argo CD application.

```powershell
kubectl get application argocd-config -n argocd
kubectl describe svc -n argocd argocd-server
curl.exe -k -I https://argocd.homelab.local
```

## Create Grafana admin secret

Grafana credentials are not committed to Git. Create the secret before syncing the monitoring application:

```powershell
kubectl create namespace monitoring
kubectl create secret generic grafana-admin -n monitoring --from-literal=admin-user=admin --from-literal=admin-password='<CHOOSE_A_STRONG_PASSWORD>'
```

## Check monitoring

```powershell
kubectl get application monitoring -n argocd
kubectl get application blackbox-exporter -n argocd
kubectl get pods -n monitoring
kubectl get svc -n monitoring
kubectl get prometheusrules -n monitoring
```

Grafana should be available at:

```text
http://grafana.homelab.local
```

## Check active alerts

```powershell
kubectl get prometheusrules -n monitoring
kubectl port-forward -n monitoring svc/kube-prometheus-stack-alertmanager 9093:9093
```

Then open:

```text
http://localhost:9093
```

Prometheus alert rules are managed by the `monitoring` Argo CD application.

## Check network probes

Blackbox Exporter provides ICMP and TCP network probes for Grafana and Prometheus alerts.

```powershell
kubectl get application blackbox-exporter -n argocd
kubectl get pods -n monitoring -l app=blackbox-exporter
kubectl get svc -n monitoring blackbox-exporter
kubectl port-forward -n monitoring svc/blackbox-exporter 9115:9115
```

Then test a probe locally:

```text
http://localhost:9115/probe?module=tcp_connect&target=192.168.0.52:8006
```

Useful Grafana PromQL queries:

```promql
probe_success{job=~"homelab-icmp|homelab-tcp"}
probe_duration_seconds{job=~"homelab-icmp|homelab-tcp"}
```

## Check SSH login monitoring

```powershell
kubectl get application loki -n argocd
kubectl get application alloy -n argocd
kubectl get application grafana-dashboards -n argocd
kubectl get pods -n monitoring -l app=loki
kubectl get pods -n monitoring -l app=alloy
kubectl get configmap -n monitoring grafana-dashboard-ssh-login-attempts
```

In Grafana, use the Loki datasource and open the `SSH Login Attempts` dashboard.

## Incident response

Use [incident-response.md](./incident-response.md) whenever a service, VM, K3s node, Proxmox host, or network endpoint becomes unreachable.

Start with:

```powershell
kubectl get nodes -o wide
kubectl get pods -A -o wide
kubectl get events -A --sort-by=.lastTimestamp
kubectl get applications -n argocd
```

## Check ingress

```powershell
kubectl get svc -n kube-system traefik
kubectl get ingress -A
```

After Pi-hole DNS records point ingress-hosted apps to Traefik's external IP, test from a client:

```powershell
nslookup homepage.homelab.local
curl.exe -I http://homepage.homelab.local
curl.exe -I http://homarr.homelab.local
curl.exe -I http://nginx.homelab.local
curl.exe -I http://grafana.homelab.local
curl.exe -I http://uptime.homelab.local
curl.exe -I http://n8n.homelab.local
```

If an ingress route fails, first verify Traefik's LoadBalancer IP, service policy, and endpoints before changing individual applications.

Traefik should use `externalTrafficPolicy: Local` in this environment. That setting is managed by the `traefik-config` Argo CD application.

```powershell
kubectl get application traefik-config -n argocd
kubectl describe svc -n kube-system traefik
```

## Prepare nodes for Longhorn

Run this before syncing the Longhorn application. Longhorn requires iSCSI support on K3s nodes, and `nfs-common` prepares the nodes for future RWX volumes and backups.

```powershell
$nodes = @(
  "192.168.0.121",
  "192.168.0.122",
  "192.168.0.123",
  "192.168.0.124",
  "192.168.0.125",
  "192.168.0.126"
)

foreach ($node in $nodes) {
  ssh kevin@$node "sudo apt update && sudo apt install -y open-iscsi nfs-common cryptsetup dmsetup && sudo systemctl enable --now iscsid"
}
```

## Check Longhorn

```powershell
kubectl get application longhorn -n argocd
kubectl get pods -n longhorn-system
kubectl get storageclass
```

If Argo CD gets stuck waiting for `longhorn-pre-upgrade`, remove the stuck hook and refresh the application:

```powershell
kubectl delete job longhorn-pre-upgrade -n longhorn-system --ignore-not-found
kubectl patch application longhorn -n argocd --type merge -p '{"operation":null}'
kubectl annotate application longhorn -n argocd argocd.argoproj.io/refresh=hard --overwrite
```

## Access Longhorn UI

The Longhorn UI is intentionally not exposed by MetalLB. Use a local port-forward:

```powershell
kubectl -n longhorn-system port-forward svc/longhorn-frontend 8080:80
```

Then open:

```text
http://localhost:8080
```

## Create Pi-hole web password secret

Pi-hole credentials are not committed to Git. Create the secret before syncing the Pi-hole application:

```powershell
kubectl create namespace pihole
kubectl create secret generic pihole-web-password -n pihole --from-literal=password='<CHOOSE_A_STRONG_PASSWORD>'
```

## Check Pi-hole

```powershell
kubectl get application pihole -n argocd
kubectl get pods -n pihole
kubectl get svc -n pihole
kubectl get pvc -n pihole
```

Pi-hole should be available at:

```text
http://pihole.homelab.local/admin
```

Test DNS manually before changing router or client DNS settings:

```powershell
nslookup example.com 192.168.0.205
```

## Configure local DNS records

In Pi-hole, go to **Local DNS > DNS Records** and add:

| Domain | IP |
|---|---:|
| `argocd.homelab.local` | 192.168.0.201 |
| `nginx.homelab.local` | Traefik external IP |
| `homepage.homelab.local` | Traefik external IP |
| `homarr.homelab.local` | Traefik external IP |
| `grafana.homelab.local` | Traefik external IP |
| `uptime.homelab.local` | Traefik external IP |
| `n8n.homelab.local` | Traefik external IP |
| `pihole.homelab.local` | 192.168.0.205 |

Then test from a client using Pi-hole DNS:

```powershell
nslookup homepage.homelab.local
nslookup homarr.homelab.local
nslookup grafana.homelab.local
nslookup uptime.homelab.local
nslookup n8n.homelab.local
```

## Create Homarr encryption secret

Homarr requires a 64-character hexadecimal encryption key. Do not commit this key to Git.

Create it before syncing the Homarr application:

```powershell
kubectl create namespace homarr
$key = -join ((1..64) | ForEach-Object { "{0:x}" -f (Get-Random -Minimum 0 -Maximum 16) })
kubectl create secret generic homarr-secret -n homarr --from-literal=SECRET_ENCRYPTION_KEY=$key
```

## Check Homarr

```powershell
kubectl get application homarr -n argocd
kubectl get pods -n homarr
kubectl get svc -n homarr
kubectl get pvc -n homarr
kubectl get ingress -n homarr
```

Homarr should be available at:

```text
http://homarr.homelab.local
```

## Create n8n secrets

n8n credentials and encryption keys are not committed to Git. Create the secret before syncing the n8n application:

```powershell
kubectl create namespace n8n

$postgresPassword = -join ((1..48) | ForEach-Object { "{0:x}" -f (Get-Random -Minimum 0 -Maximum 16) })
$encryptionKey = -join ((1..64) | ForEach-Object { "{0:x}" -f (Get-Random -Minimum 0 -Maximum 16) })

kubectl create secret generic n8n-secret -n n8n `
  --from-literal=POSTGRES_PASSWORD=$postgresPassword `
  --from-literal=N8N_ENCRYPTION_KEY=$encryptionKey
```

## Check n8n

```powershell
kubectl get application n8n -n argocd
kubectl get pods -n n8n -o wide
kubectl get svc -n n8n
kubectl get pvc -n n8n
kubectl get ingress -n n8n
```

n8n should be available at:

```text
http://n8n.homelab.local
```

## Check Uptime Kuma

```powershell
kubectl get application uptime-kuma -n argocd
kubectl get pods -n uptime-kuma
kubectl get svc -n uptime-kuma
kubectl get pvc -n uptime-kuma
kubectl get ingress -n uptime-kuma
```

Uptime Kuma should be available at:

```text
http://uptime.homelab.local
```

Suggested monitors to create after first login:

| Monitor | Type | Target |
|---|---|---|
| Homepage | HTTP(s) | `http://homepage.homelab.local` |
| Grafana | HTTP(s) | `http://grafana.homelab.local/login` |
| Argo CD | HTTP(s) | `https://argocd.homelab.local` |
| Pi-hole Web | HTTP(s) | `http://pihole.homelab.local/admin` |
| Pi-hole DNS | DNS | `example.com` using resolver `192.168.0.205` |
| Homarr | HTTP(s) | `http://homarr.homelab.local` |
| n8n | HTTP(s) | `http://n8n.homelab.local` |

## Access Argo CD

Open the Argo CD LoadBalancer IP in a browser:

```text
https://<ARGOCD_LOADBALANCER_IP>
```

## Deploy changes

```powershell
git add .
git commit -m "Update homelab platform"
git push
```

Then sync in Argo CD, or wait for auto-sync if enabled.
