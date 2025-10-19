# Kustomize

Kustomize is a template-free tool to customize application configuration. It's built into `kubectl` and allows you to manage Kubernetes configurations for different environments without modifying the original YAML files.

You use a `kustomization.yaml` file to define a base configuration and then apply overlays or "patches" for different environments (e.g., development, staging, production).

## Applying Kustomize Configurations

To apply the configuration for a specific environment (overlay):
```bash
kubectl apply -k /path/to/overlays/environment
```

## Transformers

Transformers modify all resources included in the kustomization.

*   **`commonLabels`**: Adds a set of labels to every resource.
*   **`commonAnnotations`**: Adds a set of annotations to every resource.
*   **`namePrefix`**: Adds a prefix to the name of every resource.
*   **`nameSuffix`**: Adds a suffix to the name of every resource.

### Example `kustomization.yaml` with Transformers

```yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization

# List of resources to include
resources:
  - deployment.yaml
  - service.yaml

# Add a prefix to all resource names
namePrefix: dev-

# Add common labels to all resources
commonLabels:
  env: development
```

## Patches

Patches allow for more targeted modifications to resources.

### Strategic Merge Patch

You provide a fragment of YAML that gets merged into the base resource. Kustomize uses strategic merge directives (like `$patch: delete`) to control the merge behavior.

**Example: Deleting a container**
```yaml
# In your overlay's kustomization.yaml
patches:
  - path: deployment-patch.yaml
```
```yaml
# deployment-patch.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app
spec:
  template:
    spec:
      containers:
        - name: sidecar-container
          $patch: delete
```

### JSON 6902 Patch

This method is more explicit and uses a syntax similar to filesystem paths to target specific fields.

*   **`op`**: The operation to perform (`add`, `remove`, `replace`).
*   **`path`**: The path to the target field (e.g., `/spec/replicas`).
*   **`value`**: The new value for `add` or `replace` operations.

**Example: Replacing an image**
```yaml
# In your overlay's kustomization.yaml
patches:
  - target:
      kind: Deployment
      name: api-deployment
    patch: |-
      - op: replace
        path: /spec/template/spec/containers/0/image
        value: caddy:latest
```

**Example: Adding a toleration**
*To add an item to the end of a list, you can use `-` as the index.*
```yaml
# patch.yaml
- op: add
  path: /spec/template/spec/tolerations/-
  value:
    key: "example-key"
    operator: "Exists"
    effect: "NoSchedule"
```

## Components

Components are like modules for Kustomize. They are reusable pieces of configuration that can be included in a `kustomization.yaml` file. This is useful for composing configurations from shared, common parts.

```yaml
# kustomization.yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization

resources:
  - base/

components:
  - ../components/logging
  - ../components/monitoring
```
