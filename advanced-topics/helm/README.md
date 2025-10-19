# Helm

Helm is a package manager for Kubernetes that helps you manage Kubernetes applications. Helm uses a packaging format called "charts." A chart is a collection of files that describe a related set of Kubernetes resources.

## Common Helm Commands

*   **`helm list`**: Lists all of the releases in the current namespace.
    ```bash
    helm list -n <namespace>
    ```

*   **`helm history`**: Displays the release history for a named release.
    ```bash
    helm history <release-name> -n <namespace>
    ```

*   **`helm upgrade`**: Upgrades a release to a new version of a chart.
    ```bash
    helm upgrade <release-name> <chart> --version <chart-version> -n <namespace>
    ```

*   **`helm rollback`**: Rolls back a release to a previous version.
    ```bash
    # Roll back to the previous version
    helm rollback <release-name> -n <namespace>

    # Roll back to a specific version from 'helm history'
    helm rollback <release-name> <revision-number> -n <namespace>
    ```

*   **`helm install`**: Installs a chart.
    ```bash
    helm install <release-name> <chart> -n <namespace>
    ```
