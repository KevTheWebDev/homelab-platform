# Architecture

This homelab is designed as a small production-style platform for learning cloud infrastructure, Kubernetes operations, and GitOps.

## Layers

```text
Physical mini PCs
  -> Proxmox
    -> Ubuntu Server VMs
      -> K3s Kubernetes
        -> MetalLB, Traefik, Argo CD, Prometheus, Grafana
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
- Grafana provides dashboards at `http://192.168.0.204`.
- Grafana credentials are stored in a Kubernetes Secret named `grafana-admin` in the `monitoring` namespace. The secret is intentionally not committed to Git.

## Future services

Planned platform additions:

- Persistent storage for stateful workloads.
- Pi-hole for homelab DNS/ad-blocking.
- Additional internal apps exposed through MetalLB and tracked in Homepage.
