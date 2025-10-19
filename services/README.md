# Services

*   **NodePort:** Exposes a service on a static port on each node.
    *   `targetPort`: Port on the pod.
    *   `port`: Port on the service.
    *   `nodePort`: Port on the node (valid range: 30000-32767).
*   **ClusterIP:** Creates a virtual IP inside the cluster to enable communication between services (e.g., frontend to backend).
*   **LoadBalancer:** Distributes traffic to pods, usually requiring cloud provider integration.

## NodePort

*   The `port` is the only mandatory field. If `targetPort` is not provided, it defaults to the same value as `port`. The `nodePort` will be automatically assigned a free port if not specified.
*   The selector for the service is derived from the pod's labels.
*   Load balancing is automatically configured based on the labels.
*   The default load balancing algorithm is random. Session affinity can be enabled.

### Example Spec

```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-app
spec:
  type: NodePort
  ports:
    - targetPort: 8080
      port: 80
      nodePort: 30000
  selector:
    app: my-app
    type: front-end
```

## ClusterIP

*   This is the default service type.
*   It uses `targetPort` and `port` (no `nodePort`).
*   It also uses a selector to map to the correct pods.
*   Pods can be accessed via the ClusterIP or the service name.

## LoadBalancer

*   Integrates with the native load balancer of a cloud provider.
*   Set the service `type` to `LoadBalancer`.
*   If not supported by the environment (e.g., a local VirtualBox setup), it will behave like a NodePort service.

## Imperative Commands

*   Create a NodePort service:
    ```bash
    kubectl create service nodeport webapp-service --node-port=30080 --tcp=8080:8080 --dry-run=client -o yaml
    ```

## Cross-Namespace Communication

*   To connect to a service in a different namespace, use the following format:
    `service-name.namespace.svc.cluster.local`
    *   `cluster.local` is the default domain name.
    *   `svc` is the subdomain for services.