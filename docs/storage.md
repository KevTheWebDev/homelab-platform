# Storage

Persistent storage is provided by Longhorn, a distributed block storage system for Kubernetes.

## Design

Longhorn is installed with Argo CD as the `longhorn` application.

The default Longhorn storage class is intentionally not made the cluster default yet. This keeps storage adoption explicit while the platform is still being built.

Current storage choices:

- Chart: `longhorn`
- Chart repository: `https://charts.longhorn.io`
- Chart version: `1.12.0`
- Namespace: `longhorn-system`
- Default replica count: `2`
- Default reclaim policy: `Retain`
- Pre-upgrade checker job: disabled for Argo CD/GitOps installs
- Longhorn UI exposure: `ClusterIP` only

## Why Longhorn

Longhorn gives this homelab a production-style storage layer:

- Persistent volumes for stateful workloads.
- Replicated volumes across nodes.
- Snapshots and backup support.
- A dedicated storage UI for operations.

## Security note

The Longhorn UI is not exposed with MetalLB because it does not include authentication by default. Access it with `kubectl port-forward` until an authenticated ingress is added.

## Future work

- Test dynamic PVC provisioning.
- Decide whether Longhorn should become the default storage class.
- Add recurring snapshots.
- Add a backup target.
- Use Longhorn-backed persistent storage for Pi-hole.
