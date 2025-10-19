# Troubleshooting

## Monitoring and Logging

### Metrics

*   The Kubernetes **Metrics Server** provides resource usage metrics for pods and nodes. It's a cluster addon that needs to be installed.
    ```bash
    # Example installation
    kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml
    ```
*   Once installed (it may take a minute or two to gather data), you can use `kubectl top`:
    ```bash
    # View resource usage for nodes
    kubectl top node

    # View resource usage for pods
    kubectl top pod
    ```

### Logging

*   Fetch logs from a running pod:
    ```bash
    kubectl logs <pod-name>
    ```
*   Stream logs in real-time:
    ```bash
    kubectl logs -f <pod-name>
    ```
*   If the pod has multiple containers, you must specify the container name:
    ```bash
    kubectl logs <pod-name> -c <container-name>
    ```

## Troubleshooting Checklists

### Application Issues

1.  **Pod Status:** Check if the pod is `Running`. Use `kubectl get pods` and `kubectl describe pod <pod-name>`. Look at the `Events` section for errors.
2.  **Logs:** Check the application logs with `kubectl logs <pod-name>`.
3.  **Service & Endpoints:**
    *   Check if the service exists: `kubectl get service`.
    *   Describe the service and check its `Selector`: `kubectl describe service <service-name>`. Does it match the pod's labels?
    *   Check if the service has `Endpoints`: `kubectl get endpoints <service-name>`. The IPs should match the pod IPs. If there are no endpoints, the selector is likely wrong.
4.  **Ports:** Ensure the `containerPort` in the pod spec matches the port your application is listening on. Check that the service's `port` and `targetPort` are correct.
5.  **Configuration:** Check that `ConfigMaps` and `Secrets` are mounted correctly and have the right values.
6.  **Network Policy:** Check if a `NetworkPolicy` is blocking traffic to or from the pod.

### Control Plane Issues

1.  **Nodes:** Are all nodes `Ready`? Use `kubectl get nodes`.
2.  **System Pods:** Are all pods in the `kube-system` namespace `Running`? Use `kubectl get pods -n kube-system`.
3.  **Component Status:** Check the status of the main control plane components: `kube-apiserver`, `kube-controller-manager`, `kube-scheduler`.
4.  **Component Logs:**
    *   If running as systemd services, use `journalctl -u <service-name>` (e.g., `journalctl -u kube-apiserver`).
    *   If running as static pods, use `kubectl logs <pod-name> -n kube-system`.
    *   Editing manifests in `/etc/kubernetes/manifests` on the control plane node will cause the `kubelet` to restart the component. This can be a way to fix a broken configuration, but expect a brief outage of that component.

### Node Issues

1.  **Node Status:** If a node is `NotReady`, use `kubectl describe node <node-name>` to look for events and conditions.
2.  **Kubelet:** SSH into the node and check the `kubelet` service:
    ```bash
    systemctl status kubelet
    journalctl -u kubelet
    ```
3.  **Kubelet Config:** Check the kubelet configuration files, typically `/etc/kubernetes/kubelet.conf` or `/var/lib/kubelet/config.yaml`.
4.  Restart the `kubelet` after fixing any issues: `systemctl restart kubelet`.
