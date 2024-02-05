## 📖 Overview

This is a monorepository implementing infrastructure-as-code (IaC) and GitOps practices for my home Kubernetes cluster using tools such as [ArgoCD](https://github.com/argoproj/argo-cd) and [Renovate](https://github.com/renovatebot/renovate).

This project is forked directly from [jdmcmahan/home-ops](https://github.com/jdmcmahan/home-ops) with modifications to suit my environment, which in turn is largely inspired by [onedr0p/flux-cluster-template](https://github.com/onedr0p/flux-cluster-template) 

## ⛵ Kubernetes

k8s cluster is running on virtualized Talos servers on Proxmox. The infrastructure is somewhat hyperconverged with nodes handling both application workloads and block storage via [Rook Ceph](https://github.com/rook/rook).

### Installation

[Sidero Omni](https://www.siderolabs.com/platform/saas-for-kubernetes/) is the management plane for the cluster. The nodes are [automatically bootstraped and provisioned over PXE](https://omni.siderolabs.com/docs/how-to-guides/how-to-register-a-bare-metal-machine-pxe/) based on the resources defined in [/infrastructure/sidero/omni](https://github.com/jdmcmahan/home-ops/tree/main/infrastructure/sidero/omni).

### Core Components

- [cert-manager](https://github.com/cert-manager/cert-manager) - creates and manages SSL certificates for services in the cluster.
- [external-dns](https://github.com/kubernetes-sigs/external-dns) - automatically syncs DNS records from services to my local DNS provider.
- [external-secrets](https://github.com/external-secrets/external-secrets/) - creates Kubernetes secrets from 1Password via [1Password Connect](https://github.com/1Password/connect).
- [metallb](https://github.com/metallb/metallb) - provides external IP addresses and load balancing functionality for services.
- [rook-ceph](https://github.com/rook/rook) - provides distributed block storage for peristent volumes.
- [traefik](https://github.com/traefik/traefik) - exposes HTTP traffic to external clients over DNS.

## :octocat: GitOps

This repository uses [ArgoCD](https://github.com/argoproj/argo-cd) and its [`ApplicationSet`](https://argo-cd.readthedocs.io/en/stable/user-guide/application-set) concept to deploy and manage all resources in the cluster (including itself). Installing ArgoCD and deploying the inital `ApplicationSet` (and, in turn, everything else in the cluster) is as easy as:

```bash
$ kustomize build --enable-helm apps/management/argocd | kubectl apply -f -
```

This ArgoCD `ApplicationSet` is currently configured to discover all `kustomization.yaml` files at any level under the `apps/` directory. These files may reference other resources which will be applied along with the application. One advantage of this approach is that all new `kustomization.yaml` files added to this Git repository will be discovered and deployed to the cluster by ArgoCD. Conversely, removal of a resource definition in the repository will cause that resource to be uninstalled from the cluster. Such changes are automated and instantaneous thanks to GitHub's webhook capabilities.

The full `ApplicationSet` configuration is defined in [`apps/management/argocd/applications.yaml`](apps/management/argocd/applications.yaml).

[Renovate](https://github.com/renovatebot/renovate) (running as a [GitHub App](https://github.com/apps/renovate)) monitors the entire repository for application updates. These updates are automatically applied to the cluster by merging the resulting Renovate pull requests.

## 🖧 Networking

| Name                    | CIDR              |
|-------------------------|-------------------|
| Kubernetes nodes (VLAN) | `192.168.8.0/24`  |
| Kubernetes pods         | `10.244.0.0/16`   |
| Kubernetes services     | `10.96.0.0/12`    |

## 🔧 Hardware

Many of my hardware components and parts have been purchased in cheap marketplace/ebay offers or were things I already had from previous workstation upgrades. This has resulted in a very cost-effective but capable lab thanks to federated software like Proxmox and k8s.

| Device                      | Count | CPU              | RAM   | Operating System | Purpose                                      |
|-----------------------------|-------|------------------|-------|------------------|----------------------------------------------|
| Lenovo Thinkcentre tiny m600| 3     | Intel J3710      | 8 GB  | Proxmox          | Ceph block storage & Talos nodes             |
| Kingnovy firewall host      | 1     | Intel N5100      | 16 GB | Proxmox          | Vyos router (HA), Talos nodes                |
| Rackmount server PC         | 1     | Ryzen 2700x      | 64 GB | Proxmox          | VyOS router (HA), NAS, other VMs             |


