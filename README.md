# One-Node K8s Cluster (Talos + Omni + Incus)

Single-node production-ready Kubernetes cluster running as an Incus VM, managed by Omni with automatic provisioning.

## Stack

- **Talos Linux** v1.12.4 — immutable, API-driven Kubernetes OS
- **Kubernetes** v1.34.5
- **Omni** — cluster lifecycle management
- **Incus** — VM provisioning via the [Omni Incus Infrastructure Provider](https://github.com/aredan/omni-incus-infra-provider)
- **Cilium** — CNI (installed separately after cluster bootstrap)

## Structure

```
machine-class.yaml                        # Incus VM specs (CPU, memory, disks)
cluster.yaml                              # Omni cluster template
patches/
  allow-scheduling-on-controlplanes.yaml  # Enable workloads on control plane
  disable-default-cni.yaml               # Disable default CNI for Cilium
  machine-extra-disk.yaml                 # Mount second disk at /var/mnt/extra
```

## Machine Class

The `incus-single-node` machine class defines the VM:

| Resource | Value |
|----------|-------|
| CPU | 4 |
| Memory | 8192 MB |
| Root Disk | 40 GB |
| Data Disk | 100 GiB (mounted at `/var/mnt/extra`) |
| Type | virtual-machine |

## Prerequisites

- Omni instance with the Incus infrastructure provider running
- Talos image imported into Incus (`talos` alias)
- `omnictl` CLI configured

## Deploy

```bash
# Apply the machine class
omnictl apply -f machine-class.yaml

# Create the cluster
omnictl cluster template sync -f cluster.yaml
```

## Install Cilium

The default CNI is disabled. After the cluster is up, install Cilium via Helm:

```bash
helm repo add cilium https://helm.cilium.io/
helm repo update
helm install cilium cilium/cilium --namespace kube-system
```

The node will remain `NotReady` until Cilium is installed.

## Customizing

- Adjust CPU, memory, and disk sizes in `machine-class.yaml`
- Change the storage pool name if not using `default`
- Modify the extra disk mount point in `patches/machine-extra-disk.yaml`
