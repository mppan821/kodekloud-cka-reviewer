# ETCD Backup and Restore

*   **Backup:**
    ```bash
    ETCDCTL_API=3 etcdctl snapshot save snapshot.db
    ```

*   **Check Status:**
    ```bash
    ETCDCTL_API=3 etcdctl snapshot status snapshot.db
    ```

*   **Restore:**
    1.  Stop the kube-apiserver.
    2.  Run the restore command:
        ```bash
        ETCDCTL_API=3 etcdctl snapshot restore snapshot.db --data-dir /var/lib/etcd-from-backup
        ```
    3.  Reload and restart services.
