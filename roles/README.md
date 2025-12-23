# roles

- **container_engine** Installs different container engines along with required tools.
- **crictl** Installed only when needed, primarily for debugging purposes.
- **k8s** Configures Kubernetes and creates the cluster.
- **k8s_tools** Installs Kubernetes tools like kubeadm, kubectl, and kubelet.
- **k8s_upgrade** Upgrades Kubernetes master and worker nodes.
- **kube-vip** Sets up kube-vip for virtual IP management.
- **prerequisites** Prepares nodes with necessary packages and settings for Kubernetes.
- **reset** Deletes the Kubernetes cluster and cleans up nodes.
