# Roadmap

This roadmap tracks planned platform additions that make the homelab more useful and more resume-friendly.

## Near term

- Ingress consolidation with Traefik host-based routes.
- Keep Pi-hole as the local DNS source of truth for `homelab.local`.
- Keep dedicated LoadBalancer IPs during ingress testing, then remove them once routes are stable.
- Configure Uptime Kuma notifications for critical service, VM, DNS, and Proxmox host failures.
- Add persistent log aggregation for root-cause analysis.

## Planned apps

| App | Purpose | Notes |
|---|---|---|
| Grafana network monitoring | Router, DNS, node, and service visibility | Add dashboards after metrics sources are chosen. |

## Completed apps

| App | Purpose | Notes |
|---|---|---|
| Uptime Kuma | Uptime checks and status visibility | Longhorn-backed and exposed through Traefik ingress. |
| Homarr | Alternate dashboard and service launcher | Longhorn-backed and exposed through Traefik ingress. |

## Platform improvements

- Add TLS for internal hostnames.
- Add Longhorn backup targets.
- Add external-dns or a GitOps-managed DNS strategy if Pi-hole records become tedious to maintain manually.
- Add alerting rules for node health, disk pressure, failed pods, and DNS availability.
- Add Loki/Grafana Alloy or another log pipeline so incident evidence survives pod and node restarts.
