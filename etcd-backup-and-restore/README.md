# ETCD Backup and Restore

## Online Backup

*   **Backup Command:**
    ```bash
    ETCDCTL_API=3 etcdctl snapshot save snapshot.db
    ```

*   **Backup with TLS:**
    ```bash
    ETCDCTL_API=3 etcdctl \
      --endpoints=https://127.0.0.1:2379 \
      --cacert=/etc/kubernetes/pki/etcd/ca.crt \
      --cert=/etc/kubernetes/pki/etcd/server.crt \
      --key=/etc/kubernetes/pki/etcd/server.key \
      snapshot save /backup/etcd-snapshot.db
    ```

*   **Check Status:**
    ```bash
    ETCDCTL_API=3 etcdctl snapshot status snapshot.db
    ```

## Online Restore

1.  Stop the `kube-apiserver` service:
    ```bash
    systemctl stop kube-apiserver
    ```
2.  Run the restore command:
    ```bash
    ETCDCTL_API=3 etcdctl snapshot restore snapshot.db --data-dir /var/lib/etcd-from-backup
    ```
    *Note: Upon restore, you can either delete the current etcd data directory and replace it with the restored one, or update the etcd configuration to point to the new data directory. This may take some time to take effect.*
3.  Reload the systemd manager configuration:
    ```bash
    systemctl daemon-reload
    ```
4.  Restart the `etcd` service:
    ```bash
    systemctl restart etcd
    ```
5.  Start the `kube-apiserver` service:
    ```bash
    systemctl start kube-apiserver
    ```

## Offline Backup

*   For an offline backup, you can copy the etcd data directory:
    ```bash
    etcdctl backup \
      --data-dir /var/lib/etcd \
      --backup-dir /backup/etcd-backup
    ```

## Notes

*   To find the etcd version, check the image tag of the etcd pod.
*   The default etcd client port is `2379`.
*   You can check and update the `etcd.service` file for configuration details.