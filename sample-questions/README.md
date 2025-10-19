# Sample Questions

## Cluster Upgrade to 1.33.0

Upgrade the current version of kubernetes from 1.32.0 to 1.33.0 exactly using the kubeadm utility. Make sure that the upgrade is carried out one node at a time starting with the controlplane node. To minimize downtime, the deployment gold-nginx should be rescheduled on an alternate node before upgrading each node.

### Solution

**On the controlplane node:**

1.  **Update apt repository:**
    ```bash
    vim /etc/apt/sources.list.d/kubernetes.list
    # Update the version in the URL to the next available minor release, i.e v1.33.
    # deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.33/deb/ /
    ```

2.  **Drain the controlplane node:**
    ```bash
    kubectl drain controlplane --ignore-daemonsets
    ```

3.  **Update and install kubeadm:**
    ```bash
    apt update
    apt-cache madison kubeadm
    # Based on the version information displayed by apt-cache madison, it indicates that for Kubernetes version 1.33.0, the available package version is 1.33.0-1.1. Therefore, to install kubeadm for Kubernetes v1.33.0, use the following command:
    apt-get install kubeadm=1.33.0-1.1
    ```

4.  **Upgrade the cluster:**
    ```bash
    kubeadm upgrade plan v1.33.0
    kubeadm upgrade apply v1.33.0
    ```

5.  **Upgrade kubelet and uncordon:**
    ```bash
    apt-get install kubelet=1.33.0-1.1
    systemctl daemon-reload
    systemctl restart kubelet
    kubectl uncordon controlplane
    ```

6.  **Remove taint (if any):**
    ```bash
    kubectl describe node controlplane | grep -i taint
    kubectl taint node controlplane node-role.kubernetes.io/control-plane:NoSchedule-
    kubectl describe node controlplane | grep -i taint
    ```

**On the worker node (node01):**

1.  **Drain the worker node:**
    ```bash
    kubectl drain node01 --ignore-daemonsets
    ```

2.  **SSH to the worker node and update apt repository:**
    ```bash
    # ssh node01
    vim /etc/apt/sources.list.d/kubernetes.list
    # Update the version in the URL to the next available minor release, i.e v1.33.
    # deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.33/deb/ /
    ```

3.  **Update and install kubeadm:**
    ```bash
    apt update
    apt-cache madison kubeadm
    apt-get install kubeadm=1.33.0-1.1
    ```

4.  **Upgrade the node:**
    ```bash
    kubeadm upgrade node
    ```

5.  **Upgrade kubelet and uncordon:**
    ```bash
    apt-get install kubelet=1.33.0-1.1
    systemctl daemon-reload
    systemctl restart kubelet
    # exit from ssh
    ```

6.  **Uncordon the worker node:**
    ```bash
    kubectl uncordon node01
    ```

## JSONPath

Print the names of all deployments in the `admin2406` namespace in the following format:

`DEPLOYMENT CONTAINER_IMAGE READY_REPLICAS NAMESPACE`

`<deployment name> <container image used> <ready replica count> <Namespace>`

. The data should be sorted by the increasing order of the deployment name.

Write the result to the file `/opt/admin2406_data`.

### Solution

```bash
kubectl -n admin2406 get deployment -o custom-columns=DEPLOYMENT:.metadata.name,CONTAINER_IMAGE:.spec.template.spec.containers[].image,READY_REPLICAS:.status.readyReplicas,NAMESPACE:.metadata.namespace --sort-by=.metadata.name > /opt/admin2406_data
```

## Kubeconfig Troubleshooting

A kubeconfig file called `admin.kubeconfig` has been created in `/root/CKA`. There is something wrong with the configuration. Troubleshoot and fix it.

### Solution

Make sure the port for the `kube-apiserver` is correct. So for this change port from `4380` to `6443`.

```bash
kubectl cluster-info --kubeconfig /root/CKA/admin.kubeconfig
```

## Deployment and Annotation

Create a new deployment called `nginx-deploy`, with image `nginx:1.16` and `1` replica.
Next, upgrade the deployment to version `1.17` using `rolling update` and add the annotation message `Updated nginx image to 1.17`.

### Solution

```bash
kubectl create deployment nginx-deploy --image=nginx:1.16
kubectl set image deployment/nginx-deploy nginx=nginx:1.17
kubectl annotate deployment nginx-deploy kubernetes.io/change-cause="Updated nginx image to 1.17"
```

## Troubleshoot Pending Pod

A new deployment called `alpha-mysql` has been deployed in the `alpha` namespace. However, the pods are not running. Troubleshoot and fix the issue. The deployment should make use of the persistent volume `alpha-pv` to be mounted at `/var/lib/mysql` and should use the environment variable `MYSQL_ALLOW_EMPTY_PASSWORD=1` to make use of an empty root password.

**Important:** Do not alter the persistent volume.

### Solution

Use the command `kubectl describe` and try to fix the issue.

Solution manifest file to create a pvc called `mysql-alpha-pvc` as follows:

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
  storageClassName: slow
```

## ETCD Backup

Take the backup of ETCD at the location `/opt/etcd-backup.db` on the `controlplane` node.

### Solution

```bash
export ETCDCTL_API=3
etcdctl snapshot save --cacert=/etc/kubernetes/pki/etcd/ca.crt --cert=/etc/kubernetes/pki/etcd/server.crt --key=/etc/kubernetes/pki/etcd/server.key --endpoints=127.0.0.1:2379 /opt/etcd-backup.db
```

## Secret Volume Mount

Create a pod called `secret-1401` in the `admin1401` namespace using the `busybox` image. The container within the pod should be called `secret-admin` and should sleep for `4800` seconds.

The container should mount a `read-only` secret volume called `secret-volume` at the path `/etc/secret-volume`. The secret being mounted has already been created for you and is called `dotfile-secret`.

### Solution

Use the command `kubectl run` to create a pod definition file. Add secret volume and update container name in it.

Alternatively, run the following command:

```bash
kubectl run secret-1401 -n admin1401 --image=busybox --dry-run=client -oyaml --command -- sleep 4800 > admin.yaml
```

Add the `secret` volume and mount path to create a pod called `secret-1401` in the `admin1401` namespace as follows:

```yaml
---
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: secret-1401
  name: secret-1401
  namespace: admin1401
spec:
  volumes:
  - name: secret-volume
    # secret volume
    secret:
      secretName: dotfile-secret
  containers:
  - command:
    - sleep
    - "4800"
    image: busybox
    name: secret-admin
    # volumes' mount path
    volumeMounts:
    - name: secret-volume
      readOnly: true
      mountPath: "/etc/secret-volume"
```
