# Incident Response

This document defines how the homelab should detect, record, and investigate outages.

The goal is not only to know that something is down, but to preserve enough evidence to understand why it happened and prevent repeat incidents.

## Monitoring layers

| Layer | Tool | Purpose |
|---|---|---|
| Synthetic checks | Uptime Kuma | Detect whether services, DNS, Proxmox hosts, and K3s nodes are reachable from the cluster network. |
| Kubernetes metrics | Prometheus / kube-prometheus-stack | Detect node readiness, pod crashes, unavailable deployments, PVC failures, and scrape failures. |
| Network probes | Blackbox Exporter | Detect LAN, Proxmox, K3s node, ingress, DNS, and API endpoint reachability. |
| Dashboards | Grafana | Visualize node, pod, storage, and service trends before/during/after incidents. |
| GitOps state | Argo CD | Detect whether live cluster state matches Git. |
| Storage state | Longhorn | Detect unhealthy volumes, detached volumes, and replica issues. |

## Required Uptime Kuma monitors

Create monitors for both user-facing services and infrastructure endpoints.

### HTTP monitors

| Monitor | Target | Notes |
|---|---|---|
| Homepage | `http://homepage.homelab.local` | Primary dashboard. |
| Homarr | `http://homarr.homelab.local` | Alternate dashboard. |
| Grafana | `http://grafana.homelab.local/login` | Monitoring UI. |
| Uptime Kuma | `http://uptime.homelab.local` | Alerting UI should monitor itself. |
| Argo CD | `https://argocd.homelab.local` | Enable TLS/certificate ignore if using the default self-signed certificate. |
| Pi-hole Web | `http://pihole.homelab.local/admin` | DNS/admin UI. |
| Test Nginx | `http://nginx.homelab.local` | Simple ingress smoke test. |

### DNS monitors

| Monitor | Resolver | Query |
|---|---|---|
| Pi-hole DNS Resolution | `192.168.0.205` | `example.com` A record |
| Local DNS: Homepage | `192.168.0.205` | `homepage.homelab.local` A record |
| Local DNS: Argo CD | `192.168.0.205` | `argocd.homelab.local` A record |

### TCP / ping monitors

| Monitor | Target | Port / Type |
|---|---|---|
| Proxmox pve1 | `192.168.0.50` | TCP 8006 |
| Proxmox masternode | `192.168.0.52` | TCP 8006 |
| k3s-01 VM | `192.168.0.121` | Ping |
| k3s-02 VM | `192.168.0.122` | Ping |
| k3s-03 VM | `192.168.0.123` | Ping |
| k3s-04 VM | `192.168.0.124` | Ping |
| k3s-05 VM | `192.168.0.125` | Ping |
| k3s-06 VM | `192.168.0.126` | Ping |
| K3s API | `192.168.0.121` | TCP 6443 |
| Pi-hole DNS | `192.168.0.205` | TCP 53 and UDP/DNS monitor |

## Alert severity

| Severity | Examples | Expected response |
|---|---|---|
| Critical | Proxmox host down, K3s node NotReady, Argo CD unreachable, Pi-hole DNS down, PVC Lost | Investigate immediately. |
| Warning | Pod crash loop, deployment unavailable, app out of sync, local DNS record failing | Investigate same day. |
| Info | Planned reboot, app deployment, known maintenance | Record as maintenance. |

## First-response checklist

When an alert fires:

1. Write down the alert name, time, affected target, and current symptoms.
2. Check Uptime Kuma for the first failed check and failure duration.
3. Check Grafana for node CPU, memory, disk, network, and pod restart spikes.
4. Check Blackbox Exporter probe status in Grafana:

```promql
probe_success{job=~"homelab-icmp|homelab-tcp"}
probe_duration_seconds{job=~"homelab-icmp|homelab-tcp"}
```

5. Check Argo CD for app sync/health status.
6. Check Kubernetes state:

```powershell
kubectl get nodes -o wide
kubectl get pods -A -o wide
kubectl get events -A --sort-by=.lastTimestamp
kubectl get applications -n argocd
```

7. If a node is affected, check the node directly:

```bash
hostname
ip addr
ip route
ping -c 4 192.168.0.1
ping -c 4 8.8.8.8
systemctl status k3s --no-pager
journalctl -u k3s -n 100 --no-pager
```

8. If a Proxmox host is affected, check from its console:

```bash
hostname
ip addr
ip route
cat /etc/network/interfaces
ping -c 4 192.168.0.1
ping -c 4 8.8.8.8
```

## Evidence to preserve

For each incident, save:

- Uptime Kuma screenshot or event history.
- Alert name and time fired/resolved.
- `kubectl get nodes -o wide`.
- `kubectl get pods -A -o wide`.
- `kubectl get events -A --sort-by=.lastTimestamp`.
- Relevant `journalctl` output.
- Any Proxmox host networking output.
- Root cause and prevention action.

## Incident record template

```markdown
# Incident: <short title>

## Summary

- Start:
- End:
- Duration:
- Severity:
- Affected services:

## Detection

- Alert source:
- First failed check:
- Who noticed:

## Impact

- What was unreachable?
- What still worked?

## Timeline

- HH:MM - Event
- HH:MM - Event

## Root cause

What failed and why?

## Resolution

What fixed the issue?

## Prevention

What monitoring, alert, documentation, or architecture change prevents this from repeating?

## Evidence

Commands, screenshots, logs, and links.
```
