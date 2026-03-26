# Homelab GitOps (k3s on Proxmox VE)

This repository contains the GitOps configuration for my on-premises k3s cluster, managed via FluxCD.

## 🏗️ Cluster Architecture

My cluster is hosted on **Proxmox VE** and follows a high-availability (HA) configuration:

- **6 Nodes total:**
  - **3 Control Plane (Masters):** Handles the API server, scheduler, and controller manager.
  - **3 Worker Nodes:** Runs the application workloads.
- **k3s distribution:** Lightweight Kubernetes distribution.

## 🛠️ Tooling & Stack

- **[FluxCD](https://fluxcd.io/):** Handles Continuous Delivery (CD). It synchronizes the cluster state with this repository.
- **[SOPS](https://github.com/mozilla/sops):** For secret management, ensuring sensitive data is encrypted before being committed.
- **[age](https://github.com/FiloSottile/age):** A modern encryption tool used as the SOPS backend.
- **[Kustomize](https://kustomize.io/):** Used to customize and template Kubernetes manifests.

## 📂 Repository Structure

```text
/homelab-gitops/
├── clusters/
│   └── on-prem-k3s-cluster/     # Main cluster entry point
│       ├── apps/                # Application manifests (Ingress, Deployments, etc.)
│       └── flux-system/         # FluxCD internal components and sync config
├── .sops.yaml                   # SOPS encryption rules
└── README.md                    # Project documentation
```

## 🔐 Secret Management (SOPS + age)

Secrets are encrypted using SOPS and an `age` key. The encryption rules are defined in `.sops.yaml`. Only `data` and `stringData` fields in YAML files are encrypted.

### Encrypting a Secret
To encrypt a secret, create a regular Kubernetes Secret manifest and run:
```bash
sops --encrypt --in-place secret.yaml
```

### Decrypting for local editing
```bash
sops --decrypt --in-place secret.yaml
```

## 🚀 Bootstrap & Synchronization

The cluster is bootstrapped with Flux pointing to this repository:

```bash
flux bootstrap github \
  --owner=alexgrajdan \
  --repository=homelab-gitops \
  --branch=main \
  --path=./clusters/on-prem-k3s-cluster \
  --personal
```

Flux will automatically reconcile any changes pushed to the `main` branch.
