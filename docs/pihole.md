# Pi-hole

Pi-hole provides DNS-level ad blocking for the homelab.

## Design

Pi-hole is deployed through Argo CD as the `pihole` application.

Current choices:

- Namespace: `pihole`
- Image: `pihole/pihole:latest`
- LoadBalancer IP: `192.168.0.205`
- Web UI: `http://192.168.0.205/admin`
- Persistent config path: `/etc/pihole`
- Storage class: `longhorn`
- Persistent volume size: `2Gi`
- Node placement: `k3s-04`

## Secrets

The Pi-hole web password is stored in a Kubernetes Secret named `pihole-web-password`.

The password is intentionally not committed to Git.

## Rollout approach

Pi-hole should be deployed and tested before changing router or client DNS settings.

Recommended order:

1. Deploy Pi-hole.
2. Confirm the web UI works.
3. Confirm DNS works with manual `nslookup` tests.
4. Point one test client at Pi-hole.
5. If successful, update router DHCP DNS settings.

## Manual DNS test

From Windows:

```powershell
nslookup example.com 192.168.0.205
```

From Linux:

```bash
dig @192.168.0.205 example.com
```
