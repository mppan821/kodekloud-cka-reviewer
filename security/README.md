# Security

## Authentication & Authorization

### 1. Certificates (TLS)

*   Kubernetes components use TLS certificates for secure communication.
*   The `kube-apiserver` manifest (`/etc/kubernetes/manifests/kube-apiserver.yaml`) specifies the paths to its TLS key and certificate files.
*   You can inspect a certificate using `openssl x509 -in <cert-file> -text -noout`.
*   Troubleshoot certificate issues by checking the logs of the relevant component (e.g., `journalctl -u kubelet` or `crictl logs <container-id>`).

#### Certificate API

*   Kubernetes can manage certificate signing requests (CSRs).
*   Create a `CertificateSigningRequest` object. The request itself is base64 encoded.
*   `kubectl get csr`: View pending requests.
*   `kubectl certificate approve <csr-name>`: Approve a request.
*   `kubectl certificate deny <csr-name>`: Deny a request.
*   The `kube-controller-manager` is responsible for handling these requests.

### 2. Kubeconfig

*   `kubeconfig` files organize access to clusters, users, and contexts.
*   `kubectl config view`: View the current configuration.
*   `kubectl config use-context <context-name>`: Switch to a different context.
*   You can specify a namespace within a context for convenience.
*   Certificates can be included as a file path (`certificate-authority`) or as base64-encoded data (`certificate-authority-data`).

### 3. Authorization

Once authenticated, the request is authorized.

#### Authorization Modes

The `kube-apiserver` can be configured with different authorization modes via the `--authorization-mode` flag (e.g., `Node,RBAC`). It checks them in order.

*   **Node:** Authorizes requests made by kubelets.
*   **ABAC (Attribute-Based Access Control):** Difficult to manage.
*   **RBAC (Role-Based Access Control):** The standard, most common method.
*   **Webhook:** Delegates authorization to an external service.
*   **AlwaysAllow / AlwaysDeny:** Self-explanatory.

#### RBAC (Role-Based Access Control)

RBAC uses `Roles` or `ClusterRoles` to define permissions, and `RoleBindings` or `ClusterRoleBindings` to grant those permissions to users, groups, or `ServiceAccounts`.

*   **Role:** A set of permissions within a specific namespace.
    *   Defines rules for verbs (`get`, `list`, `watch`, `create`, `update`, `patch`, `delete`) on resources (`pods`, `deployments`, `services`).
*   **RoleBinding:** Grants a `Role` to a subject (user, group, or `ServiceAccount`) within that same namespace.

*   **ClusterRole:** A set of permissions that can be cluster-scoped or namespace-scoped.
    *   Can grant access to cluster-scoped resources (e.g., `nodes`, `persistentvolumes`, `namespaces`).
    *   Can also be used to grant the same permissions across all namespaces.
*   **ClusterRoleBinding:** Grants a `ClusterRole` to a subject, either cluster-wide or within a specific namespace.

*   **Commands:**
    *   `kubectl get roles`, `kubectl get rolebindings`
    *   `kubectl describe role <role-name>`
    *   `kubectl auth can-i create deployments --as <user-name> --namespace <ns>`: Check permissions for a user.

### 4. Admission Controllers

After a request is authorized, it goes to Admission Controllers before being persisted in etcd. They can mutate or validate requests.

*   The list of enabled plugins is set by the `--enable-admission-plugins` flag on the `kube-apiserver`.
*   **MutatingAdmissionWebhook:** Modifies the object before it's stored.
*   **ValidatingAdmissionWebhook:** Validates the object. The request is rejected if it fails validation.
*   The flow is: Mutating webhooks -> Schema Validation -> Validating webhooks.

## Pod and Container Security

### Service Accounts

*   Provide an identity for processes that run in a Pod.
*   A default `ServiceAccount` is created in every namespace.
*   Pods are automatically assigned the `default` service account if not specified otherwise with `spec.serviceAccountName`.
*   The service account token is mounted as a volume at `/var/run/secrets/kubernetes.io/serviceaccount`.

### ImagePullSecrets

*   To pull images from a private registry, you need to provide credentials.
*   Create a secret of type `kubernetes.io/dockerconfigjson`:
    ```bash
    kubectl create secret docker-registry <secret-name> --docker-server=<registry> --docker-username=<user> --docker-password=<pass>
    ```
*   Specify the secret in the pod spec:
    ```yaml
    spec:
      imagePullSecrets:
        - name: <secret-name>
    ```

### Security Context

*   A `securityContext` defines privilege and access control settings for a Pod or Container.
*   `spec.securityContext` (Pod level): Settings apply to all containers in the pod.
*   `spec.containers[].securityContext` (Container level): Settings apply only to that container and override pod-level settings.
*   **`runAsUser: <UID>`**: Specifies the User ID to run the container's entrypoint.
*   This is the Kubernetes equivalent of the `USER` instruction in a Dockerfile.

## Secret Encryption at Rest

*   By default, Kubernetes Secrets are base64 encoded, not encrypted, in etcd.
*   To encrypt them, you need to create an `EncryptionConfiguration` object and configure the `kube-apiserver` to use it.

1.  **Create an `EncryptionConfiguration` manifest:** This YAML file specifies which resources to encrypt and which provider (`aescbc`, `kms`, etc.) to use.
2.  **Update `kube-apiserver`:** Modify the `kube-apiserver.yaml` manifest to point to the encryption configuration file using the `--encryption-provider-config` flag.
3.  **Mount the config file** into the `kube-apiserver` pod as a volume.
4.  **Restart the `kube-apiserver`.**
5.  **Re-encrypt existing secrets:** Only new secrets created after this change will be encrypted. To encrypt existing secrets, you need to run `kubectl get secrets --all-namespaces -o json | kubectl replace -f -`.