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

| Service | IP | Purpose |
|---|---:|---|
| Argo CD | 192.168.0.200 | GitOps web UI |
| Test Nginx | 192.168.0.202 | Test application |
| Homepage | 192.168.0.203 | Homelab dashboard |
| Grafana | 192.168.0.204 | Monitoring dashboards |
| Pi-hole | 192.168.0.205 | DNS and ad blocking |

Future services should use an unused address from the MetalLB range.
