# Scheduling

The Kubernetes scheduler is responsible for assigning pods to nodes. If the `kube-scheduler` component is not running, pods will remain in a `Pending` state.

## Manual Scheduling

You can bypass the scheduler by manually assigning a pod to a specific node using the `spec.nodeName` field in the pod's manifest.

## Labels and Selectors

*   **Labels:** Key-value pairs attached to objects (e.g., pods, nodes).
*   **Selectors:** Used by controllers (e.g., ReplicaSet, Service) to select a set of objects based on their labels.

### Example: ReplicaSet with Selector

```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: replicaset-1
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx # This must match the pod template's labels
  template:
    metadata:
      labels:
        app: nginx # Pod label
```

## Taints and Tolerations

*   **Taints** are applied to nodes, and **Tolerations** are applied to pods.
*   Taints repel pods from a node unless they have a matching toleration.

### Tainting a Node

```bash
kubectl taint node <node-name> key=value:effect
```

*   **Effects:**
    *   `NoSchedule`: New pods won't be scheduled on the node unless they have a matching toleration.
    *   `PreferNoSchedule`: The scheduler will try to avoid placing pods on the node.
    *   `NoExecute`: New pods won't be scheduled, and existing pods without a matching toleration will be evicted.

### Adding a Toleration to a Pod

```yaml
spec:
  tolerations:
    - key: "key"
      operator: "Equal"
      value: "value"
      effect: "NoSchedule"
```

### Removing a Taint

```bash
# The key and effect must match the taint to be removed.
kubectl taint node <node-name> key:effect-
```

## Node Selector

A simple way to constrain pods to nodes with specific labels.

*   **Label a node:**
    ```bash
    kubectl label nodes <node-name> size=Large
    ```
*   **Add `nodeSelector` to the pod spec:**
    ```yaml
    spec:
      nodeSelector:
        size: Large
    ```
    *Note: `nodeSelector` does not support complex expressions like `OR` or `NOT`.*

## Node Affinity and Anti-Affinity

Provides more expressive rules than `nodeSelector`.

*   **Types:**
    *   `requiredDuringSchedulingIgnoredDuringExecution`: The rule *must* be met for the pod to be scheduled.
    *   `preferredDuringSchedulingIgnoredDuringExecution`: The scheduler will *try* to enforce the rule, but it's not mandatory.
*   The `IgnoredDuringExecution` part means that if the node's labels change after the pod is scheduled, the pod will continue to run on that node.

### Example: Node Affinity

```yaml
spec:
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
          - matchExpressions:
              - key: size
                operator: In
                values:
                  - Large
                  - Medium
```

## Priority and Preemption

*   **PriorityClasses** are cluster-scoped objects that define a priority level.
*   Pods with higher priority can preempt (evict) pods with lower priority to make room for themselves if the cluster is out of resources.

### Creating a PriorityClass

```yaml
apiVersion: scheduling.k8s.io/v1
kind: PriorityClass
metadata:
  name: high-priority
value: 1000000
globalDefault: false
description: "This priority class should be used for critical pods only."
preemptionPolicy: PreemptLowerPriority # or Never
```

### Assigning a PriorityClass to a Pod

Add the `priorityClassName` to the pod spec:

```yaml
spec:
  priorityClassName: high-priority
```

## Multiple Schedulers

*   You can run multiple custom schedulers in a cluster.
*   To assign a pod to a specific scheduler, use the `spec.schedulerName` field.
*   You can see which scheduler placed a pod by looking at the `kubectl get events` output.

### Scheduler Profiles

The scheduling process consists of several stages (plugins):

1.  **Queueing:** The `PrioritySort` plugin sorts pending pods.
2.  **Filtering:** Plugins like `NodeResourcesFit` and `TaintToleration` filter out nodes that cannot run the pod.
3.  **Scoring:** Plugins score the remaining nodes to find the best fit.
4.  **Binding:** The `DefaultBinder` plugin binds the pod to the selected node.