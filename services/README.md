# Services

*   **NodePort:** Exposes a service on a static port on each node.
    *   `targetPort`: Port on the pod.
    *   `port`: Port on the service.
    *   `nodePort`: Port on the node (valid range: 30000-32767).
*   **ClusterIP:** Creates a virtual IP inside the cluster to enable communication between services (e.g., frontend to backend).
*   **LoadBalancer:** Distributes traffic to pods, usually requiring cloud provider integration.
