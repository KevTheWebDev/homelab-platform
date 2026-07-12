# Roadmap

This roadmap tracks planned platform additions that make the homelab more useful and more resume-friendly.

## Near term

- Ingress consolidation with Traefik host-based routes.
- Keep Pi-hole as the local DNS source of truth for `homelab.local`.
- Keep dedicated LoadBalancer IPs during ingress testing, then remove them once routes are stable.

## Planned apps

| App | Purpose | Notes |
|---|---|---|
| Homarr | Alternate dashboard and service launcher | Compare with Homepage before deciding which becomes primary. |
| Uptime Kuma | Uptime checks and status visibility | Good candidate for Longhorn-backed persistent storage. |
| Grafana network monitoring | Router, DNS, node, and service visibility | Add dashboards after metrics sources are chosen. |

## Platform improvements

- Add TLS for internal hostnames.
- Add Longhorn backup targets.
- Add external-dns or a GitOps-managed DNS strategy if Pi-hole records become tedious to maintain manually.
- Add alerting rules for node health, disk pressure, failed pods, and DNS availability.
