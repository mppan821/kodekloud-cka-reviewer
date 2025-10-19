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

### Imperative vs. Declarative

*   **Imperative:** A step-by-step approach.
    *   Example: Provision a VM, then install software.
    *   Uses `kubectl` commands like `run`, `expose`, `scale`, `create`, `edit`, `set image`.
    *   Changes are tracked in the user's session history, which can be hard to manage.
    *   For updates, `kubectl edit` can be used, but it sets a specific image. A better approach is to modify the manifest and use `kubectl replace -f manifest.yaml`.
    *   `kubectl replace` will error if the object doesn't exist.
    *   `kubectl create` will error if the object already exists.

*   **Declarative:** Specifies the desired final state.
    *   The system determines if the declared requirement already exists.
    *   Uses `kubectl apply`.
    *   **EXAM TIP:** Use imperative commands for quick operations. `kubectl edit` is useful for making direct changes. Imperative commands are generally best for the exam.

### `kubectl apply`

*   Creates a `last-applied-configuration` annotation to compare the live object's state with the local file's new state.
*   **Why is the last-applied configuration needed?**
    *   To detect deleted fields and items like labels.
    *   This annotation is only created when `kubectl apply` is used; `kubectl create` does not create it.