# Application Lifecycle Management

*   **Rollouts:**
    *   `kubectl rollout status <deployment>`
    *   `kubectl rollout history <deployment>`
    *   `kubectl rollout undo <deployment>`
*   **Deployment Strategies:**
    *   **Recreate:** Terminates all old pods before creating new ones.
    *   **RollingUpdate:** Replaces pods one by one (default).
*   **Updating Deployments:**
    *   `kubectl apply -f <deployment-file.yaml>`
    *   `kubectl set image deployment/<deployment-name> <container-name>=<new-image>`
*   **Environment Variables:**
    *   `env`: Plain key-value pairs.
    *   `valueFrom`: Reference ConfigMaps or Secrets.
*   **ConfigMaps:** Store non-confidential configuration data.
*   **Secrets:** Store sensitive data like passwords and API keys.
*   **Multi-container Pods:**
    *   **Init Containers:** Run to completion before the main containers start.
    *   **Sidecar Containers:** Run alongside the main containers.
