# Sample Questions

---

## 1. Cluster Upgrade

**Task:** Upgrade the Kubernetes cluster from version `1.32.0` to `1.33.0` using `kubeadm`. Upgrade one node at a time, starting with the controlplane. To minimize downtime, the deployment `gold-nginx` should be rescheduled on an alternate node before upgrading each node.

### Solution

**On the `controlplane` node:**

1.  **Update apt repository source to point to the new version.**
    *   Edit `/etc/apt/sources.list.d/kubernetes.list` and change the version in the URL to `v1.33`.

2.  **Drain the controlplane node.**
    ```bash
    kubectl drain controlplane --ignore-daemonsets
    ```

3.  **Update packages and install the correct `kubeadm` version.**
    ```bash
    apt update
    apt-cache madison kubeadm # Find the available 1.33.0 version string
    apt-get install kubeadm=1.33.0-1.1
    ```

4.  **Plan and apply the upgrade.**
    ```bash
    kubeadm upgrade plan v1.33.0
    kubeadm upgrade apply v1.33.0
    ```

5.  **Upgrade `kubelet` and uncordon the node.**
    ```bash
    apt-get install kubelet=1.33.0-1.1
    systemctl daemon-reload
    systemctl restart kubelet
    kubectl uncordon controlplane
    ```

6.  **Remove control-plane taint (if necessary) to allow workloads to be scheduled.**
    ```bash
    kubectl taint node controlplane node-role.kubernetes.io/control-plane:NoSchedule-
    ```

**On the worker node (`node01`):**

1.  **Drain the worker node.**
    ```bash
    kubectl drain node01 --ignore-daemonsets
    ```

2.  **SSH to the worker node.**

3.  **Update apt repository source** (same as step 1 for controlplane).

4.  **Update packages and install `kubeadm`.**
    ```bash
    apt update
    apt-get install kubeadm=1.33.0-1.1
    ```

5.  **Upgrade the node.**
    ```bash
    kubeadm upgrade node
    ```

6.  **Upgrade `kubelet`.**
    ```bash
    apt-get install kubelet=1.33.0-1.1
    systemctl daemon-reload
    systemctl restart kubelet
    ```

7.  **Exit the SSH session.**

8.  **Uncordon the worker node.**
    ```bash
    kubectl uncordon node01
    ```

---

## 2. Custom Columns and Sorting

**Task:** Print the names of all deployments in the `admin2406` namespace in the following format, sorted by deployment name. Write the result to `/opt/admin2406_data`.

`DEPLOYMENT CONTAINER_IMAGE READY_REPLICAS NAMESPACE`

### Solution

```bash
kubectl -n admin2406 get deployment \
  -o custom-columns=DEPLOYMENT:.metadata.name,CONTAINER_IMAGE:.spec.template.spec.containers[].image,READY_REPLICAS:.status.readyReplicas,NAMESPACE:.metadata.namespace \
  --sort-by=.metadata.name > /opt/admin2406_data
```

---

## 3. Kubeconfig Troubleshooting

**Task:** A kubeconfig file at `/root/CKA/admin.kubeconfig` is not working. Troubleshoot and fix it.

### Solution

The issue is likely an incorrect `server` address or port for the `kube-apiserver`.

1.  **Inspect the kubeconfig file.**
2.  **Verify the cluster endpoint.** The port should typically be `6443`. In this case, it was incorrectly set to `4380`.
3.  **Correct the port** in the `admin.kubeconfig` file.
4.  **Test the configuration:**
    ```bash
    kubectl cluster-info --kubeconfig /root/CKA/admin.kubeconfig
    ```

---

## 4. Deployment Update and Annotation

**Task:** Create a new deployment called `nginx-deploy` with image `nginx:1.16` and 1 replica. Then, upgrade the deployment to version `1.17` and record the reason for the change in an annotation.

### Solution

```bash
# 1. Create the deployment
kubectl create deployment nginx-deploy --image=nginx:1.16

# 2. Update the image
kubectl set image deployment/nginx-deploy nginx=nginx:1.17

# 3. Annotate the deployment to record the change cause
kubectl annotate deployment nginx-deploy kubernetes.io/change-cause="Updated nginx image to 1.17"
```

---

## 5. Troubleshoot Pending Pod (PVC Issue)

**Task:** A deployment `alpha-mysql` in the `alpha` namespace has pods that are not running. Fix the issue. The deployment needs to mount the `PersistentVolume` `alpha-pv` at `/var/lib/mysql` and set the environment variable `MYSQL_ALLOW_EMPTY_PASSWORD=1`. Do not alter the PV.

### Solution

Describing the pod (`kubectl describe pod <pod-name> -n alpha`) would show an event indicating that it's waiting for a `PersistentVolumeClaim` to be bound. The deployment is trying to use a PVC that doesn't exist.

The solution is to create the required `PersistentVolumeClaim`.

```yaml
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: mysql-alpha-pvc
  namespace: alpha
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
  storageClassName: slow # This must match the PV's storage class
```

---

## 6. ETCD Backup

**Task:** Take a backup of the etcd cluster and save it to `/opt/etcd-backup.db` on the controlplane node.

### Solution

```bash
export ETCDCTL_API=3
etcdctl snapshot save \
  --endpoints=127.0.0.1:2379 \
  --cacert=/etc/kubernetes/pki/etcd/ca.crt \
  --cert=/etc/kubernetes/pki/etcd/server.crt \
  --key=/etc/kubernetes/pki/etcd/server.key \
  /opt/etcd-backup.db
```

---

## 7. Pod with Secret Volume

**Task:** Create a pod named `secret-1401` in the `admin1401` namespace using the `busybox` image. The container, named `secret-admin`, should sleep for `4800` seconds. Mount the secret `dotfile-secret` as a `read-only` volume at `/etc/secret-volume`.

### Solution

First, generate a base manifest:
```bash
kubectl run secret-1401 -n admin1401 --image=busybox --dry-run=client -o yaml --command -- sleep 4800 > admin.yaml
```

Then, edit `admin.yaml` to add the volume mount and volume definition, and correct the container name.

**Final Manifest:**
```yaml
---
apiVersion: v1
kind: Pod
metadata:
  name: secret-1401
  namespace: admin1401
spec:
  containers:
  - name: secret-admin
    image: busybox
    command:
      - sleep
      - "4800"
    volumeMounts:
    - name: secret-volume
      mountPath: "/etc/secret-volume"
      readOnly: true
  volumes:
  - name: secret-volume
    secret:
      secretName: dotfile-secret
```