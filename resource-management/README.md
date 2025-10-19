# Resource Management

## Namespaces

Namespaces are a way to divide cluster resources between multiple users.

*   They can be used to allocate resources, and define policies.
*   To create an object in a specific namespace, add the `metadata.namespace` field to its manifest.
*   To switch your current context to a different namespace:
    ```bash
    kubectl config set-context $(kubectl config current-context) --namespace=<target-namespace>
    ```

## Resource Requests and Limits

You can specify resource `requests` and `limits` for containers.

*   **`requests`**: The amount of resources the container is guaranteed to get. The scheduler uses this value to find a suitable node.
*   **`limits`**: The maximum amount of resources the container can use.

### CPU

*   CPU is a compressible resource. If a container hits its CPU limit, it will be throttled.
*   **Best Practice:** It's often better to set a CPU request but no limit. This allows workloads to burst and use available CPU on the node without being unnecessarily throttled, leading to better node utilization.

### Memory

*   Memory is an incompressible resource. There is no throttling.
*   If a container exceeds its memory limit, it will be terminated with an Out of Memory (OOM) error.

## LimitRange

*   A `LimitRange` is a policy object for a namespace that provides constraints for resource allocations.
*   It can set default `request` and `limit` values for containers in pods that don't specify their own.
*   It only takes effect for pods created after the `LimitRange` was applied.

## ResourceQuota

*   A `ResourceQuota` is a namespace-level object that provides constraints that limit aggregate resource consumption per namespace.
*   It can limit the quantity of objects that can be created in a namespace by type, as well as the total amount of compute resources that may be consumed by resources in that namespace.

### Example ResourceQuota

```yaml
apiVersion: v1
kind: ResourceQuota
metadata:
  name: compute-quota
  namespace: dev
spec:
  hard:
    pods: "10"
    requests.cpu: "4"
    requests.memory: "5Gi"
    limits.cpu: "10"
    limits.memory: "10Gi"
```

*   You can't change the resource limits/requests directly on a running pod. However, if the pod is managed by a Deployment, you can update the Deployment's pod template, which will trigger a rollout that replaces the old pods with new ones that have the updated resource settings.