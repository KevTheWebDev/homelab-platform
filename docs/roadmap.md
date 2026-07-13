# Roadmap

This roadmap tracks planned platform additions that make the homelab more useful and more resume-friendly.

## Near term

- Ingress consolidation with Traefik host-based routes.
- Keep Pi-hole as the local DNS source of truth for `homelab.local`.
- Keep dedicated LoadBalancer IPs during ingress testing, then remove them once routes are stable.

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
