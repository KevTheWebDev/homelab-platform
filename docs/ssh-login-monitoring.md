# SSH Login Monitoring

SSH login monitoring is provided by Grafana Loki, Grafana Alloy, and a Grafana dashboard.

## Scope

Phase 1 collects SSH authentication logs from the K3s VM nodes.

The logs are collected from:

```text
/var/log/auth.log
```

on each K3s VM.

Proxmox host SSH logs are not collected by this Kubernetes DaemonSet because Proxmox runs outside the K3s cluster. Add Proxmox host log shipping later with Grafana Alloy installed directly on each Proxmox host or by forwarding syslog to Loki.

## Components

| Component | Purpose |
|---|---|
| Loki | Stores logs. |
| Alloy | Runs as a DaemonSet and ships K3s VM auth logs to Loki. |
| Grafana Loki datasource | Lets Grafana query logs with LogQL. |
| SSH Login Attempts dashboard | Shows failed logins, accepted logins, and raw auth events. |

## Validate

```powershell
kubectl get application loki -n argocd
kubectl get application alloy -n argocd
kubectl get application grafana-dashboards -n argocd
kubectl get pods -n monitoring -l app=loki
kubectl get pods -n monitoring -l app=alloy
kubectl get svc -n monitoring loki
```

## Grafana queries

Use the Loki datasource.

### Failed SSH attempts

```logql
sum by (host) (count_over_time({job="ssh-auth"} |~ "Failed password|Invalid user|authentication failure" [5m]))
```

### Accepted SSH logins

```logql
sum by (host) (count_over_time({job="ssh-auth"} |~ "Accepted password|Accepted publickey" [5m]))
```

### Raw SSH events

```logql
{job="ssh-auth"} |~ "sshd|Failed password|Invalid user|Accepted password|Accepted publickey|authentication failure"
```

## Dashboard

In Grafana, open:

```text
Dashboards -> SSH Login Attempts
```

If the dashboard does not appear, verify the dashboard ConfigMap exists:

```powershell
kubectl get configmap -n monitoring grafana-dashboard-ssh-login-attempts
```

Then restart Grafana:

```powershell
kubectl rollout restart deployment/kube-prometheus-stack-grafana -n monitoring
```
