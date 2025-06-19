# Bycur Kubernetes Cluster

This inventory deploys a single-node Kubernetes cluster using Kubespray with GPU support and essential infrastructure components.

## Overview

The `bycur` inventory configures a Kubernetes cluster with:
- Single node acting as both control plane and worker node
- NVIDIA GPU support for AI/ML workloads
- Core infrastructure components for production-ready deployment
- Service mesh capabilities with Istio in ambient mode

## Prerequisites

Before running the deployment, install the following packages on the Kubernetes nodes:

```bash
sudo dnf install ansible
sudo dnf install sshpass
sudo dnf install python3-kubernetes
```

## Cluster Deployment

### 1. Reset Existing Cluster (Optional)

If you need to clean up an existing cluster installation:

```bash
ansible-playbook -i inventory/bycur/hosts.yaml -u $USERNAME -b -K -v reset.yml
```

### 2. Deploy Kubernetes Cluster

Deploy the base Kubernetes cluster using Kubespray:

```bash
ansible-playbook -i inventory/bycur/hosts.yaml -u $USERNAME -b -K -v cluster.yml
```

### 3. Install Add-on Components

After the cluster is running, install the add-on components:

```bash
ansible-playbook -i inventory/bycur/hosts.yaml -v inventory/bycur/playbooks/install.yaml
```

This playbook installs the following components in order:
1. **NVIDIA GPU Operator** - Enables GPU support for Kubernetes workloads
2. **cert-manager** - Manages TLS certificates automatically
3. **TopoLVM** - Provides dynamic local storage provisioning
4. **MetalLB** - Load balancer implementation for bare metal clusters
5. **Istio** - Service mesh for advanced traffic management (configured in ambient mode)

## Helm Values Generation

To generate or update Helm values for the components:

```bash
# Generate GPU Operator values
helm show values --repo https://helm.ngc.nvidia.com/nvidia gpu-operator > inventory/bycur/playbooks/values/gpu-operator.yaml

# Generate Istio values (for ambient mode)
helm template istiod --repo https://istio-release.storage.googleapis.com/charts istiod --set profile=ambient --dry-run > inventory/bycur/playbooks/values/istiod.yaml
```

## Cluster Configuration

- **Node**: Single node at IP `10.3.3.128` serving as control plane, worker, and etcd node
- **Network Plugin**: Calico (default Kubespray configuration)
- **Container Runtime**: Configured through Kubespray defaults

## Directory Structure

```
inventory/bycur/
├── hosts.yaml              # Ansible inventory file
├── README.md               # This file
└── playbooks/              # Component installation playbooks
    ├── install.yaml        # Main installation orchestrator
    ├── nvidia.yaml         # GPU Operator deployment
    ├── cert.yaml           # cert-manager deployment
    ├── topolvm.yaml        # TopoLVM deployment
    ├── metallb.yaml        # MetalLB deployment
    ├── istio.yaml          # Istio deployment
    ├── gateway/            # Gateway configurations
    └── values/             # Helm chart values
```

## Notes

- Ensure the `$USERNAME` environment variable is set to your SSH username before running commands
- The `-K` flag will prompt for your sudo password
- The `-v` flag enables verbose output for troubleshooting
- All components are installed using Helm charts with customized values stored in the `values/` directory
