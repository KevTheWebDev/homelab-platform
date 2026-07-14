# Network Monitoring

Network monitoring is provided by Prometheus Blackbox Exporter, Prometheus alert rules, and Grafana.

## What is monitored

### ICMP reachability

| Target | IP |
|---|---:|
| Gateway | 192.168.0.1 |
| Proxmox pve1 | 192.168.0.50 |
| Proxmox masternode | 192.168.0.52 |
| k3s-01 | 192.168.0.121 |
| k3s-02 | 192.168.0.122 |
| k3s-03 | 192.168.0.123 |
| k3s-04 | 192.168.0.124 |
| k3s-05 | 192.168.0.125 |
| k3s-06 | 192.168.0.126 |
| Traefik ingress | 192.168.0.200 |
| Argo CD | 192.168.0.201 |
| Pi-hole | 192.168.0.205 |

### TCP reachability

| Target | Endpoint |
|---|---|
| Proxmox pve1 web | `192.168.0.50:8006` |
| Proxmox masternode web | `192.168.0.52:8006` |
| Proxmox masternode SSH | `192.168.0.52:22` |
| K3s API | `192.168.0.121:6443` |
| Traefik web | `192.168.0.200:80` |
| Argo CD HTTPS | `192.168.0.201:443` |
| Pi-hole DNS TCP | `192.168.0.205:53` |
| k3s node SSH | `192.168.0.121-192.168.0.126:22` |

## Grafana queries

Use these PromQL queries in Grafana panels.

### Probe status

```promql
probe_success{job=~"homelab-icmp|homelab-tcp"}
```

### Probe latency

```promql
probe_duration_seconds{job=~"homelab-icmp|homelab-tcp"}
```

### Down targets only

```promql
probe_success{job=~"homelab-icmp|homelab-tcp"} == 0
```

### ICMP latency by target

```promql
probe_duration_seconds{job="homelab-icmp"}
```

## Alerts

Prometheus alert rules are managed in the `monitoring` application.

| Alert | Meaning |
|---|---|
| `HomelabNetworkProbeFailed` | An ICMP or TCP probe has failed for at least 2 minutes. |
| `HomelabNetworkLatencyHigh` | An ICMP probe has been above 100ms for at least 5 minutes. |

## Validate deployment

```powershell
kubectl get application blackbox-exporter -n argocd
kubectl get pods -n monitoring -l app=blackbox-exporter
kubectl get svc -n monitoring blackbox-exporter
kubectl get prometheusrules -n monitoring
```
