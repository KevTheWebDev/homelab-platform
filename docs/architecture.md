# Architecture

This homelab is designed as a small production-style platform for learning cloud infrastructure, Kubernetes operations, and GitOps.

## Layers

```text
Physical mini PCs
  -> Proxmox
    -> Ubuntu Server VMs
      -> K3s Kubernetes
        -> MetalLB, Traefik Ingress, Argo CD, Prometheus, Grafana, Longhorn
          -> Homelab apps
```

## Kubernetes

K3s runs across six Ubuntu Server VMs:

- `k3s-01`, `k3s-02`, and `k3s-03` are server/control-plane nodes.
- `k3s-04`, `k3s-05`, and `k3s-06` are worker nodes.
- `k3s-03` is labeled as the high-CPU/heavy-worker node.

## GitOps

Argo CD is the desired-state controller. Changes should be made in Git, committed, pushed, and then reconciled by Argo CD into the cluster.

The intended workflow is:

```text
Edit manifests
  -> git commit
  -> git push
  -> Argo CD syncs
  -> Kubernetes updates the cluster
```

## Observability

Monitoring is provided by the `kube-prometheus-stack` Helm chart, managed by Argo CD as the `monitoring` application.

- Prometheus collects Kubernetes and node metrics.
- Grafana provides dashboards at `http://grafana.homelab.local`.
- Grafana credentials are stored in a Kubernetes Secret named `grafana-admin` in the `monitoring` namespace. The secret is intentionally not committed to Git.

## Ingress

Traefik provides host-based HTTP routing for homelab services.

Ingress routes currently include:

- `homepage.homelab.local`
- `homarr.homelab.local`
- `nginx.homelab.local`
- `grafana.homelab.local`
- `uptime.homelab.local`

Most browser-based apps use internal ClusterIP services behind Traefik instead of consuming their own MetalLB addresses.

## Storage

Persistent storage is provided by Longhorn, managed by Argo CD as the `longhorn` application.

- Longhorn provides replicated Kubernetes volumes for stateful apps.
- The Longhorn UI is kept internal-only because it does not include authentication by default.
- The default Longhorn replica count is set to `2` to balance resilience with the cluster's limited local storage.

## Future services

Planned platform additions:

- Network monitoring dashboards in Grafana.
- TLS certificates for ingress-hosted apps.
- Longhorn backup targets for persistent volumes.
