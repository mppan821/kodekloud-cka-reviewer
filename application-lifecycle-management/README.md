# Application Lifecycle Management

## Deployments and Rollouts

*   **Deployment Strategies:**
    *   **Recreate:** All old pods are killed before any new ones are created. This results in downtime.
    *   **RollingUpdate:** Pods are replaced one by one, ensuring the application remains available during the update. This is the default strategy.

*   **Updating Deployments:**
    *   **Declarative:** Modify the deployment manifest and run `kubectl apply -f <file.yaml>`.
    *   **Imperative:** Use `kubectl set image deployment/<name> <container>=<new-image>`.
        *   *Note:* This can cause a drift between your manifest file and the live state of the object.

*   **Rollout Commands:**
    *   `kubectl rollout status <deployment>`: Monitor the status of a rollout.
    *   `kubectl rollout history <deployment>`: View the history of rollouts.
    *   `kubectl rollout undo <deployment>`: Roll back to the previous version.

## Configuring Pods and Containers

### Overriding Docker CMD/Entrypoint

*   **`command`** in Kubernetes overrides the Docker `ENTRYPOINT`.
*   **`args`** in Kubernetes overrides the Docker `CMD`.
*   All arguments must be strings (e.g., `args: ["10"]`).

### Environment Variables

*   **`env`**: A list of plain key-value pairs.
    ```yaml
    env:
      - name: MY_VARIABLE
        value: "my-value"
    ```
*   **`valueFrom`**: Reference keys from a `ConfigMap` or `Secret`.
    ```yaml
    env:
      - name: MY_SECRET_VAR
        valueFrom:
          secretKeyRef:
            name: my-secret
            key: password
    ```
*   **`envFrom`**: Expose all key-value pairs from a `ConfigMap` or `Secret` as environment variables.
    ```yaml
    envFrom:
      - configMapRef:
          name: app-config
    ```

### ConfigMaps

Store non-confidential configuration data as key-value pairs.

*   **Imperative Creation:**
    ```bash
    kubectl create configmap <name> --from-literal=KEY=VALUE --from-file=path/to/file
    ```
*   **Declarative Creation:**
    ```yaml
    apiVersion: v1
    kind: ConfigMap
    metadata:
      name: app-config
    data:
      APP_COLOR: blue
      APP_MODE: prod
    ```
*   **As a Volume:** Mount a `ConfigMap` as a volume inside a pod.
    ```yaml
    volumes:
      - name: config-volume
        configMap:
          name: app-config
    ```

### Secrets

Store sensitive data. Secrets are base64 encoded, but not encrypted by default in etcd.

*   To view a secret's data, you must decode it from base64.
*   Can be injected as environment variables (`secretKeyRef`) or mounted as volumes, similar to `ConfigMaps`.

## Pod Patterns

### Multi-container Pods

*   Containers in the same pod are created and destroyed together.
*   They share the same network space (can communicate via `localhost`) and storage volumes.

### Init Containers

*   Run and complete their tasks before the main application containers are started.
*   Defined in the `initContainers` block in the pod spec.
*   They run sequentially. If any `initContainer` fails, the pod will be restarted.
*   Useful for pre-start tasks like waiting for a database to be ready or setting up permissions.

### Sidecar Containers

*   A container that runs alongside the main application container to provide supporting functionality.
*   Examples: log shipping, service mesh proxies, monitoring agents.
*   In modern Kubernetes, this is often implemented as a regular container. For ordering, you can use `initContainers` with `restartPolicy: Always` (a feature that has been evolving). The key difference from a simple co-located container is the lifecycle dependency and purpose.

## Workload Controllers

### DaemonSets

*   Ensures that a copy of a pod runs on all (or some) nodes in the cluster.
*   When a new node is added, a pod is automatically scheduled to it.
*   Perfect for cluster-level agents like monitoring (e.g., Prometheus Node Exporter) or logging (e.g., Fluentd).
*   The spec is similar to a Deployment, but you specify `kind: DaemonSet` and there are no replicas.

### Static Pods

*   Managed directly by the `kubelet` daemon on a specific node, without the `kube-apiserver` observing them.
*   The `kubelet` watches a specific directory (e.g., `/etc/kubernetes/manifests`, configured by `--pod-manifest-path`) for pod manifests.
*   If a manifest is added/removed, the `kubelet` creates/deletes the pod.
*   They are ignored by the scheduler.
*   Often used to run control plane components (like `etcd`, `kube-apiserver`) as pods on the control plane nodes themselves.

## Autoscaling

### Horizontal Pod Autoscaler (HPA)

*   Automatically scales the number of pods in a ReplicaSet, Deployment, or StatefulSet based on observed CPU utilization or other select metrics.
*   Requires a metrics server to be installed in the cluster.
*   Requires resource `requests` to be set on the containers for CPU/memory-based scaling.
*   **Imperative Command:**
    ```bash
    kubectl autoscale deployment <name> --cpu-percent=80 --min=1 --max=10
    ```
*   **Declarative Object:** `kind: HorizontalPodAutoscaler`

### Vertical Pod Autoscaler (VPA)

*   Automatically adjusts the CPU and memory `requests` and `limits` for pods to match actual usage.
*   Not a built-in component; must be installed separately.
*   **Update Modes:**
    *   `Off`: Only provides recommendations without applying them.
    *   `Initial`: Only sets resources when a pod is first created.
    *   `Recreate` / `Auto`: Evicts pods and recreates them with the new resource values.
*   Useful for stateful workloads or applications with heavy resource needs (databases, ML). It is generally not recommended to use VPA with HPA on CPU or memory.
