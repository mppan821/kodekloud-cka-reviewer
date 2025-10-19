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
    You can then update the YAML file with the replicas or any other field before creating the deployment.

### Services

*   **Create a Service named `redis-service` of type ClusterIP to expose pod `redis` on port 6379:**
    ```bash
    kubectl expose pod redis --port=6379 --name redis-service --dry-run=client -o yaml
    ```
    (This will automatically use the pod's labels as selectors)

    *Or:*
    ```bash
    kubectl create service clusterip redis --tcp=6379:6379 --dry-run=client -o yaml
    ```
    (This will not use the pod's labels as selectors; instead, it will assume selectors as `app=redis`. You cannot pass in selectors as an option, so it does not work very well if your pod has a different label set. So, generate the file and modify the selectors before creating the service).

*   **Create a Service named `nginx` of type NodePort to expose pod `nginx`'s port 80 on port 30080 on the nodes:**
    ```bash
    kubectl expose pod nginx --type=NodePort --port=80 --name=nginx-service --dry-run=client -o yaml
    ```
    (This will automatically use the pod's labels as selectors, but you cannot specify the node port. You have to generate a definition file and then add the node port in manually before creating the service with the pod.)

    *Or:*
    ```bash
    kubectl create service nodeport nginx --tcp=80:80 --node-port=30080 --dry-run=client -o yaml
    ```
    (This will not use the pod's labels as selectors).

    **Note:** Both of the above commands have their own challenges. While one of them cannot accept a selector, the other cannot accept a node port. I would recommend going with the `kubectl expose` command. If you need to specify a node port, generate a definition file using the same command and manually input the `nodePort` before creating the service.

### General Tips

*   Familiarize yourself with `vim` or `nano` for editing manifest files quickly.
*   Use `kubectl explain` to understand the structure of Kubernetes objects and their fields.
*   Understand how `kubeconfig` files are structured and how to switch between contexts.

### References

*   [Kubernetes Documentation - kubectl Commands](https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands)
*   [Kubernetes Documentation - kubectl Conventions](https://kubernetes.io/docs/reference/kubectl/conventions/)

---

## General Certification Tips (from the Linux Foundation)

*   **Environment**: The exam is online, proctored, and consists of 15-20 performance-based tasks to be completed in 2 hours. You will be working in a Linux command-line environment.
*   **System Requirements**: You must provide your own computer with a supported OS, a single monitor, a reliable internet connection, a microphone, and a webcam.
*   **No Other Applications**: You are not allowed to have other applications or browser windows open besides the one for the exam.
*   **SSH and `sudo`**: Tasks must be completed on designated hosts, which you will access via SSH. You can assume elevated privileges using `sudo -i`.

---

## Kubectl Quick Reference

This is a summary of essential commands from the official Kubernetes documentation.

### Context and Configuration

*   **View current context**: `kubectl config current-context`
*   **List all contexts**: `kubectl config get-contexts`
*   **Switch context**: `kubectl config use-context <context-name>`
*   **Set the namespace for the current context**: `kubectl config set-context --current --namespace <namespace-name>`

### Creating Objects

*   **Create from file**: `kubectl apply -f ./my-manifest.yaml`
*   **Create from directory**: `kubectl apply -f ./my-directory/`
*   **Create a deployment**: `kubectl create deployment nginx --image=nginx`
*   **Create a job**: `kubectl create job my-job --image=busybox -- echo "Hello"`
*   **Get documentation for a resource type**: `kubectl explain pods`

### Viewing and Finding Resources

*   **List all pods in all namespaces**: `kubectl get pods -A` or `kubectl get pods --all-namespaces`
*   **List pods with more detail**: `kubectl get pods -o wide`
*   **Get a resource's YAML**: `kubectl get pod my-pod -o yaml`
*   **Describe a resource for verbose output**: `kubectl describe node my-node`
*   **Sort results**: `kubectl get services --sort-by=.metadata.name`
*   **Show labels**: `kubectl get pods --show-labels`
*   **List events, filtering for warnings**: `kubectl events --types=Warning`

### Updating and Rolling Back

*   **Update a container's image**: `kubectl set image deployment/frontend www=image:v2`
*   **Check rollout history**: `kubectl rollout history deployment/frontend`
*   **Roll back to the previous version**: `kubectl rollout undo deployment/frontend`
*   **Watch rollout status**: `kubectl rollout status -w deployment/frontend`
*   **Trigger a rolling restart**: `kubectl rollout restart deployment/frontend`
*   **Add a label**: `kubectl label pods my-pod new-label=awesome`
*   **Add an annotation**: `kubectl annotate pods my-pod icon-url=http://example.com/icon.png`

### Editing and Scaling

*   **Edit a resource interactively**: `kubectl edit svc/my-service`
*   **Use a different editor**: `KUBE_EDITOR="nano" kubectl edit svc/my-service`
*   **Scale a deployment**: `kubectl scale --replicas=5 deployment/my-deployment`
*   **Autoscale a deployment**: `kubectl autoscale deployment my-deployment --min=2 --max=10 --cpu-percent=80`

### Deleting Resources

*   **Delete from file**: `kubectl delete -f ./pod.json`
*   **Delete by name**: `kubectl delete pod,service my-pod my-service`
*   **Delete by label**: `kubectl delete pods,services -l name=myLabel`
*   **Delete all in namespace**: `kubectl -n my-ns delete pod,svc --all`
*   **Force delete (no grace period)**: `kubectl delete pod my-pod --now`

### Interacting with Pods

*   **Get logs**: `kubectl logs my-pod`
*   **Stream logs**: `kubectl logs -f my-pod`
*   **Run a command in a pod**: `kubectl exec my-pod -- ls /`
*   **Get an interactive shell**: `kubectl exec -it my-pod -- /bin/sh`
*   **Forward a port**: `kubectl port-forward my-pod 5000:6000`

### Interacting with Nodes

*   **Mark node as unschedulable**: `kubectl cordon my-node`
*   **Drain node for maintenance**: `kubectl drain my-node --ignore-daemonsets`
*   **Mark node as schedulable**: `kubectl uncordon my-node`
*   **Add a taint**: `kubectl taint nodes my-node key=value:NoSchedule`

### Autocomplete

Setting up shell autocomplete is highly recommended to save time.

*   **Bash**:
    ```bash
    source <(kubectl completion bash)
    echo "source <(kubectl completion bash)" >> ~/.bashrc
    ```
*   **Zsh**:
    ```bash
    source <(kubectl completion zsh)
    echo '[[ $commands[kubectl] ]] && source <(kubectl completion zsh)' >> ~/.zshrc
    ```
