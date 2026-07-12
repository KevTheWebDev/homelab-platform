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
