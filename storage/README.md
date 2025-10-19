# Storage

*   **PersistentVolume (PV):** A piece of storage in the cluster that has been provisioned by an administrator. It is a resource in the cluster, just like a node is a cluster resource.
*   **PersistentVolumeClaim (PVC):** A request for storage by a user. It is similar to a Pod. Pods consume node resources, and PVCs consume PV resources.
*   **StorageClass:** Provides a way for administrators to describe the "classes" of storage they offer. Different classes might map to quality-of-service levels, or to backup policies, or to arbitrary policies determined by the cluster administrators.

## PV and PVC Interaction

*   A `PersistentVolumeClaim` will bind to an available `PersistentVolume` that meets the claim's requirements (e.g., `accessModes`, `storageClassName`, size).
*   If a suitable PV exists, the PVC will be bound to it. If no other volume is available and claimed, the PV will be the size of the Claim.
*   The PV and PVC must match on `accessMode`, `storageClassName`, etc.

## Using PVCs in Pods

*   Pods use a `PersistentVolumeClaim` as a volume.
*   The cluster finds the bound PV for the PVC and mounts that volume for the Pod.

### Example Pod with a PVC

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-pod
spec:
  containers:
    - name: my-container
      image: nginx
      volumeMounts:
        - mountPath: "/var/www/html"
          name: my-storage
  volumes:
    - name: my-storage
      persistentVolumeClaim:
        claimName: my-pvc
```

*   A `volume` can also be a `hostPath`, which mounts a file or directory from the host node's filesystem into your Pod. However, this is not recommended for most applications.