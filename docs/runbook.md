# Runbook

## Check cluster health

```powershell
kubectl get nodes -o wide
kubectl get pods -A
```

## Check MetalLB

```powershell
kubectl get pods -n metallb-system -o wide
kubectl get ipaddresspools -n metallb-system
kubectl get l2advertisements -n metallb-system
```

## Check Argo CD

```powershell
kubectl get pods -n argocd
kubectl get svc -n argocd argocd-server
```

## Create Grafana admin secret

Grafana credentials are not committed to Git. Create the secret before syncing the monitoring application:

```powershell
kubectl create namespace monitoring
kubectl create secret generic grafana-admin -n monitoring --from-literal=admin-user=admin --from-literal=admin-password='<CHOOSE_A_STRONG_PASSWORD>'
```

## Check monitoring

```powershell
kubectl get application monitoring -n argocd
kubectl get pods -n monitoring
kubectl get svc -n monitoring
```

Grafana should be available at:

```text
http://192.168.0.204
```

## Prepare nodes for Longhorn

Run this before syncing the Longhorn application. Longhorn requires iSCSI support on K3s nodes, and `nfs-common` prepares the nodes for future RWX volumes and backups.

```powershell
$nodes = @(
  "192.168.0.121",
  "192.168.0.122",
  "192.168.0.123",
  "192.168.0.124",
  "192.168.0.125",
  "192.168.0.126"
)

foreach ($node in $nodes) {
  ssh kevin@$node "sudo apt update && sudo apt install -y open-iscsi nfs-common cryptsetup dmsetup && sudo systemctl enable --now iscsid"
}
```

## Check Longhorn

```powershell
kubectl get application longhorn -n argocd
kubectl get pods -n longhorn-system
kubectl get storageclass
```

If Argo CD gets stuck waiting for `longhorn-pre-upgrade`, remove the stuck hook and refresh the application:

```powershell
kubectl delete job longhorn-pre-upgrade -n longhorn-system --ignore-not-found
kubectl patch application longhorn -n argocd --type merge -p '{"operation":null}'
kubectl annotate application longhorn -n argocd argocd.argoproj.io/refresh=hard --overwrite
```

## Access Longhorn UI

The Longhorn UI is intentionally not exposed by MetalLB. Use a local port-forward:

```powershell
kubectl -n longhorn-system port-forward svc/longhorn-frontend 8080:80
```

Then open:

```text
http://localhost:8080
```

## Create Pi-hole web password secret

Pi-hole credentials are not committed to Git. Create the secret before syncing the Pi-hole application:

```powershell
kubectl create namespace pihole
kubectl create secret generic pihole-web-password -n pihole --from-literal=password='<CHOOSE_A_STRONG_PASSWORD>'
```

## Check Pi-hole

```powershell
kubectl get application pihole -n argocd
kubectl get pods -n pihole
kubectl get svc -n pihole
kubectl get pvc -n pihole
```

Pi-hole should be available at:

```text
http://192.168.0.205/admin
```

Test DNS manually before changing router or client DNS settings:

```powershell
nslookup example.com 192.168.0.205
```

## Access Argo CD

Open the Argo CD LoadBalancer IP in a browser:

```text
https://<ARGOCD_LOADBALANCER_IP>
```

## Deploy changes

```powershell
git add .
git commit -m "Update homelab platform"
git push
```

Then sync in Argo CD, or wait for auto-sync if enabled.
