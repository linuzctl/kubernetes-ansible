# Getting Started with kubernetes-ansible

This guide will help you quickly set up a Kubernetes cluster using the kubernetes-ansible playbooks.

## Prerequisites

To create a Kubernetes cluster:

- ansible
- ssh access (with root privileges)

To finalize the Kubernetes cluster:

- kubectl
- helm
- helmfile

## Environment

In this guide, we use the following environment:

| Node            | Hostname      | IP Address       | Role           |
|-----------------|---------------|------------------|----------------|
| K8s master 01   | k8s-demo-m-01 | 192.168.188.50   | control-plane  |
| K8s master 02   | k8s-demo-m-02 | 192.168.188.51   | control-plane  |
| K8s master 03   | k8s-demo-m-03 | 192.168.188.52   | control-plane  |
| K8s worker 01   | k8s-demo-w-01 | 192.168.188.53   | worker         |
| K8s worker 02   | k8s-demo-w-02 | 192.168.188.54   | worker         |
| K8s worker 03   | k8s-demo-w-03 | 192.168.188.55   | worker         |

| Service         | Address                | Description                |
|-----------------|------------------------|----------------------------|
| KubeAPI         | 192.168.188.56:6443    | Kubernetes API endpoint    |

The Kubernetes API should be available at 192.168.188.56:6443 with the DNS name `demo.k8s.example.com`.

## Create Kubernetes Cluster

### 1. Clone the Repository

```bash
git clone git@github.com:linuzctl/kubernetes-ansible.git
cd kubernetes-ansible
```

### 2. Install Ansible and Dependencies

```bash
ansible-galaxy install -r requirements.yaml
```

### 3. Prepare Demo Inventory

Copy the example inventory and customize it for your environment:

```bash
cp -R inventory/example inventory/demo
```

Adjust the values in your hosts.yaml files as needed.

```yaml
# vim inventory/demo/hosts.yaml
---
k8s_control_plane:
  hosts:
    k8s-demo-m-01:
      ansible_host: 192.168.188.50
    k8s-demo-m-02:
      ansible_host: 192.168.188.51
    k8s-demo-m-03:
      ansible_host: 192.168.188.52
k8s_worker:
  hosts:
    k8s-demo-w-01:
      ansible_host: 192.168.188.53
    k8s-demo-w-02:
      ansible_host: 192.168.188.54
    k8s-demo-w-03:
      ansible_host: 192.168.188.55

k8s:
  children:
    k8s_control_plane:
    k8s_worker:
  vars:
    # add if needed
    ansible_connection: ssh
    ansible_user: demo
    ansible_ssh_private_key_file: /path/to/ssh_key
```

Adjust the values in your group_vars/all.yaml files.

```yaml
# vim inventory/demo/group_vars/all.yaml
kubernetes_cluster_name: demo
kubernetes_dns_domain: k8s.example.com
kubernetes_server_ip: 192.168.188.56
```

Ensure the kube-vip interface default value is correct.

### 4. Check Connectivity

Test Ansible connectivity to your nodes:

```bash
ansible -i inventory/demo all -m ping
```

Check if you have root access:

```bash
ansible -i inventory/demo all -m command -a "whoami" -b
```

expected output:

```bash
k8s-demo-m-01 | CHANGED | rc=0 >>
root
k8s-demo-m-03 | CHANGED | rc=0 >>
root
k8s-demo-w-02 | CHANGED | rc=0 >>
root
k8s-demo-w-01 | CHANGED | rc=0 >>
root
k8s-demo-w-03 | CHANGED | rc=0 >>
root
k8s-demo-m-02 | CHANGED | rc=0 >>
root
```

### 5. Deploy the Cluster

Run the main playbook to install Kubernetes:

```bash
ansible-playbook playbooks/main.yaml -i inventory/demo
```

After deployment, the Kubernetes cluster will be up but initially in the NotReady state.

```bash
> kubectl get nodes
NAME            STATUS      ROLES           AGE     VERSION
k8s-demo-m-01   NotReady    control-plane   1m53s   v1.36.1
k8s-demo-m-02   NotReady    control-plane   1m25s   v1.36.1
k8s-demo-m-03   NotReady    control-plane   1m25s   v1.36.1
k8s-demo-w-01   NotReady    <none>          1m11s   v1.36.1
k8s-demo-w-02   NotReady    <none>          1m11s   v1.36.1
k8s-demo-w-03   NotReady    <none>          1m11s   v1.36.1
```

## Finalize Kubernetes Cluster

By default, the kubernetes-ansible playbook disables CoreDNS and kube-proxy. To make the cluster fully functional, you need to install CoreDNS and Cilium. The easiest way to do this is with Helmfile.

### 1. Copy kubeconfig File

```bash
scp k8s-demo-m-01:/etc/kubernetes/admin.conf ~/.kube/config
```

### 2. Approve Pending CSRs

Before installing Cilium and CoreDNS, you must approve any pending certificate signing requests (CSRs). When new nodes join the cluster, they generate CSRs that must be approved by a cluster administrator before the nodes can communicate securely with the control plane. For more information see this [issue](https://github.com/kubernetes/kubernetes/issues/73356)

To approve all pending CSRs, run:

```bash
kubectl get csr --no-headers | awk '$NF=="Pending" {print $1}' | xargs kubectl certificate approve
```

### 3. Install Cilium and CoreDNS via Helmfile

```bash
helmfile apply docs/helmfile.yaml
```

### 4. Check Kubernetes Nodes After Cilium and CoreDNS Are Installed

```bash
> kubectl get nodes
NAME            STATUS   ROLES           AGE     VERSION
k8s-demo-m-01   Ready    control-plane   2m47s   v1.36.1
k8s-demo-m-02   Ready    control-plane   2m19s   v1.36.1
k8s-demo-m-03   Ready    control-plane   2m19s   v1.36.1
k8s-demo-w-01   Ready    <none>          2m5s    v1.36.1
k8s-demo-w-02   Ready    <none>          2m5s    v1.36.1
k8s-demo-w-03   Ready    <none>          2m5s    v1.36.1
```
