# Network Plan

## LAN

| Item | Value |
|---|---:|
| Gateway/router | 192.168.0.1 |
| DHCP range | 192.168.0.2-192.168.0.100 |
| K3s node range | 192.168.0.121-192.168.0.126 |
| MetalLB range | 192.168.0.200-192.168.0.220 |

## K3s nodes

| Hostname | IP | Role |
|---|---:|---|
| k3s-01 | 192.168.0.121 | control-plane, etcd |
| k3s-02 | 192.168.0.122 | control-plane, etcd |
| k3s-03 | 192.168.0.123 | control-plane, etcd, heavy-worker |
| k3s-04 | 192.168.0.124 | worker |
| k3s-05 | 192.168.0.125 | worker |
| k3s-06 | 192.168.0.126 | worker |

## MetalLB

MetalLB provides LoadBalancer IPs from:

```text
192.168.0.200-192.168.0.220
```

These addresses must remain outside the DHCP pool.

## Assigned LoadBalancer IPs

| Service | IP | DNS name | Purpose |
|---|---:|---|---|
| Argo CD | 192.168.0.201 | `argocd.homelab.local` | GitOps web UI |
| Test Nginx | 192.168.0.202 | `nginx.homelab.local` | Test application |
| Homepage | 192.168.0.203 | `homepage.homelab.local` | Homelab dashboard |
| Grafana | 192.168.0.204 | `grafana.homelab.local` | Monitoring dashboards |
| Pi-hole | 192.168.0.205 | `pihole.homelab.local` | DNS and ad blocking |

Future services should use an unused address from the MetalLB range.

## Ingress

Traefik is the cluster ingress controller included with K3s.

Ingress is being introduced in phases:

1. Keep existing per-app LoadBalancer services in place as a rollback path.
2. Add host-based routes through Traefik.
3. After routes are verified, move most HTTP apps behind one shared ingress IP.

Pi-hole should keep its dedicated LoadBalancer IP because it serves DNS on port 53.

Argo CD should stay on its dedicated LoadBalancer IP for now because its HTTPS and API behavior is more sensitive than simple HTTP apps.

Check the current Traefik ingress IP with:

```powershell
kubectl get svc -n kube-system traefik
```

During ingress testing, point these Pi-hole DNS records to Traefik's external IP:

| DNS name | Routed service |
|---|---|
| `homepage.homelab.local` | Homepage |
| `nginx.homelab.local` | Test Nginx |
| `grafana.homelab.local` | Grafana |

Keep these records on their dedicated service IPs:

| DNS name | IP | Reason |
|---|---:|---|
| `argocd.homelab.local` | 192.168.0.201 | Dedicated Argo CD endpoint |
| `pihole.homelab.local` | 192.168.0.205 | DNS service endpoint |

## Local DNS

Pi-hole provides local DNS records for homelab services under `homelab.local`.

These records make the platform easier to use while preserving the underlying MetalLB IP plan.
