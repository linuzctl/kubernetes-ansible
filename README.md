# kubernetes-ansible

An Ansible playbook to create and manage Kubernetes clusters using kubeadm. Designed for my homelab or testing environments.

Feature:

- Install Kubernetes via binary or package manager
- Support for multiple CPU architectures
- Create, upgrade, and reset clusters

## Prerequisites

Before running this playbook, ensure the following:

- SSH access to all nodes in the inventory
- Sudo privileges on all nodes
- Basic understanding of Kubernetes and kubeadm

## Supported OS

- Debian 12/13
- (Other Linux distributions may work, but not tested)

## Usage

1. Copy the example inventory:

   ```bash
   cp -R inventory/example inventory/dev
   ```

2. Edit the inventory and adjust variables according to your cluster setup.

   ### Create Kubernetes cluster

   ```bash
   ansible-playbook playbooks/main.yaml -i inventory/dev
   ```

   ### Upgrade Kubernetes cluster

   ```bash
   ansible-playbook playbooks/upgrade.yaml -i inventory/dev
   ```

   ### Delete Kubernetes cluster

   ```bash
   ansible-playbook playbooks/reset.yaml -i inventory/dev
   ```

## Disclaimer

This playbook is intended for personal or homelab use only. Use it only if you understand Kubernetes and kubeadm. Production stability is not guaranteed.
