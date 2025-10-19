# Advanced Topics

This section covers advanced Kubernetes topics that involve extending or deeply customizing the platform's behavior.

## Topics

*   [API Groups](./README.md#api-groups)
*   [Custom Resource Definitions (CRDs) and Operators](./crd/README.md)
*   [Helm](./helm/README.md)
*   [Kustomize](./kustomize/README.md)
*   [JSONPath](./jsonpath/README.md)

---

## API Groups

The Kubernetes API is organized into "API groups" for easier extension.

*   The core, original API is at the `/api/v1` path and is not part of any group. This includes fundamental objects like `pods`, `services`, and `namespaces`.
*   Newer, grouped APIs are at the `/apis/$GROUP_NAME/$VERSION` path (e.g., `/apis/apps/v1`, `/apis/networking.k8s.io/v1`).
*   You can explore the API from within the cluster by using `kubectl proxy`, which creates a proxy service to the API server, making it accessible on `localhost`.
