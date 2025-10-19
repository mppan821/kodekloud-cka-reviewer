# Cluster Maintenance

*   **OS Upgrades:**
    *   `kubectl drain <node-name>`: Safely evict pods from a node.
    *   `kubectl cordon <node-name>`: Mark a node as unschedulable.
    *   `kubectl uncordon <node-name>`: Mark a node as schedulable again.
*   **Cluster Upgrade:**
    *   Upgrade one minor version at a time.
    *   Use `kubeadm upgrade plan` and `kubeadm upgrade apply`.
    *   Upgrade control plane nodes first, then worker nodes.
