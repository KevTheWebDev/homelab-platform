# homelab-platform

Public GitOps repository for my Proxmox + K3s homelab platform.

## Current platform

- 6 mini PCs running Proxmox
- 6 Ubuntu Server VMs
- K3s Kubernetes cluster
- 3 server/control-plane nodes with embedded etcd
- 3 worker nodes
- MetalLB for LAN LoadBalancer IPs
- Argo CD for GitOps

## Node layout

| Node | IP | Role | Notes |
|---|---:|---|---|
| k3s-01 | 192.168.0.121 | control-plane, etcd | K3s server |
| k3s-02 | 192.168.0.122 | control-plane, etcd | K3s server |
| k3s-03 | 192.168.0.123 | control-plane, etcd, heavy-worker | Intel i7 node |
| k3s-04 | 192.168.0.124 | worker | K3s agent |
| k3s-05 | 192.168.0.125 | worker | K3s agent |
| k3s-06 | 192.168.0.126 | worker | K3s agent |

## Network

| Purpose | Range |
|---|---:|
| Router/gateway | 192.168.0.1 |
| DHCP range | 192.168.0.2-192.168.0.100 |
| K3s nodes | 192.168.0.121-192.168.0.126 |
| MetalLB LoadBalancer IPs | 192.168.0.200-192.168.0.220 |

## GitOps model

Argo CD watches this repository and reconciles Kubernetes resources into the homelab cluster.

Initial flow:

1. Push this repo to GitHub.
2. Apply `clusters/homelab/root-app.yaml`.
3. Argo CD deploys infrastructure and apps from Git.

## Resume summary

Built and operated a six-node Kubernetes homelab on Proxmox using K3s, MetalLB, Argo CD, and GitOps to manage containerized workloads, networking, and platform documentation.
