# Cluster Maintenance

## OS Upgrades and Node Maintenance

*   If a node is down for more than 5 minutes (default), pods are terminated and recreated on other nodes. This is controlled by the `pod-eviction-timeout` setting in the `kube-controller-manager`.
*   **`kubectl drain <node-name>`**: Gracefully evicts all pods from a node and marks it as unschedulable.
    *   This won't work if there are pods on the node that are not managed by a ReplicaSet, Deployment, etc.
    *   Forcing a drain on a node with such pods will cause them to be lost permanently. Ensure workloads are managed by controllers.
*   **`kubectl cordon <node-name>`**: Marks a node as unschedulable without evicting existing pods.
*   **`kubectl uncordon <node-name>`**: Marks a node as schedulable again.

## Kubernetes Releases

*   Use `kubectl get nodes` to check the version (`Major.minor.patch`) of each node.
*   New features are often disabled by default in alpha releases and need to be enabled via feature flags.
*   Component versions (etcd, CoreDNS) can differ.
*   The main Kubernetes components (`kube-apiserver`, `kube-controller-manager`, `kube-scheduler`, `kubelet`, `kube-proxy`, `kubectl`) generally share the same version.
*   The `kube-apiserver` is the primary component. Other components have some version skew tolerance:
    *   `kube-controller-manager` and `kube-scheduler` can be one minor version behind the `kube-apiserver` (X-1).
    *   `kubelet` and `kube-proxy` can be up to two minor versions behind (X-2).

## Cluster Upgrade Process

*   **Best Practice:** Upgrade one minor version at a time. Only the last 3 minor versions are typically supported.
*   Managed Kubernetes services (like GKE, EKS, AKS) often provide a simplified, one-click upgrade process.
*   For self-managed clusters using `kubeadm`, the process is more manual.

### `kubeadm` Upgrade

1.  **Upgrade the primary control plane node:**
    *   `kubeadm upgrade plan`: Check for available upgrades.
    *   `apt-get upgrade -y kubeadm=<version>`: Upgrade the `kubeadm` package.
    *   `kubeadm upgrade apply <version>`: Apply the cluster upgrade.
    *   `apt-get upgrade -y kubelet=<version> kubectl=<version>`: Upgrade the `kubelet` and `kubectl`.
    *   `systemctl restart kubelet`: Restart the kubelet service.
2.  **Upgrade other control plane nodes.**
3.  **Upgrade worker nodes.**

### Worker Node Upgrade Strategies

*   **All at once:** Upgrade all worker nodes simultaneously. This will cause downtime.
*   **One node at a time:** Drain and upgrade each node sequentially. Workloads will be rescheduled to other nodes.
*   **Add new nodes:** Add new, upgraded nodes to the cluster, migrate workloads to them, and then decommission the old nodes.

### Example Worker Node Upgrade Workflow

1.  **Drain the node:**
    ```bash
    kubectl drain <node-name> --ignore-daemonsets
    ```
2.  **On the worker node, upgrade packages:**
    ```bash
    apt-get upgrade -y kubeadm=<version>
    apt-get upgrade -y kubelet=<version>
    ```
3.  **Run `kubeadm upgrade`:**
    ```bash
    kubeadm upgrade node
    ```
4.  **Restart the kubelet:**
    ```bash
    systemctl restart kubelet
    ```
5.  **Uncordon the node:**
    ```bash
    kubectl uncordon <node-name>
    ```