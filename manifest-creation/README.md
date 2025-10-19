# Manifest Creation

*   **Create Pod Manifest:**
    ```bash
    kubectl run nginx --image=nginx --dry-run=client -o yaml
    ```

*   **Create Deployment Manifest:**
    ```bash
    kubectl create deployment nginx --image=nginx --replicas=4 --dry-run=client -o yaml
    ```

### ReplicaSet vs. ReplicationController

*   **ReplicaSet:** Requires a `selector` to manage pods based on matching labels. If pods with the same labels exist, the ReplicaSet will manage them.
*   **ReplicationController:** Does not require a `selector`.
