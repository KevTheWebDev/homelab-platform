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
