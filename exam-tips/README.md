# Exam Tips

### Pods

*   **Create an NGINX Pod:**
    ```bash
    kubectl run nginx --image=nginx
    ```

*   **Generate Pod Manifest (YAML):**
    ```bash
    kubectl run nginx --image=nginx --dry-run=client -o yaml
    ```

### Deployments

*   **Create a Deployment:**
    ```bash
    kubectl create deployment --image=nginx nginx
    ```

*   **Generate Deployment Manifest (YAML):**
    ```bash
    kubectl create deployment --image=nginx nginx --dry-run=client -o yaml
    ```

*   **Create a Deployment with 4 Replicas:**
    ```bash
    kubectl create deployment nginx --image=nginx --replicas=4
    ```

*   **Scale a Deployment:**
    ```bash
    kubectl scale deployment nginx --replicas=4
    ```

*   **Save Deployment to a File:**
    ```bash
    kubectl create deployment nginx --image=nginx --dry-run=client -o yaml > nginx-deployment.yaml
    ```

### Services

*   **Expose a Pod as a ClusterIP Service:**
    ```bash
    kubectl expose pod redis --port=6379 --name redis-service --dry-run=client -o yaml
    ```

*   **Create a ClusterIP Service:**
    ```bash
    kubectl create service clusterip redis --tcp=6379:6379 --dry-run=client -o yaml
    ```

*   **Expose a Pod as a NodePort Service:**
    ```bash
    kubectl expose pod nginx --type=NodePort --port=80 --name=nginx-service --dry-run=client -o yaml
    ```

*   **Create a NodePort Service:**
    ```bash
    kubectl create service nodeport nginx --tcp=80:80 --node-port=30080 --dry-run=client -o yaml
    ```
