# Scheduling

*   **Manual Scheduling:** Use the `nodeName` field in the pod spec.
*   **Labels and Selectors:** Group resources using labels and select them with selectors.
*   **Taints and Tolerations:**
    *   **Taint a node:** `kubectl taint node <node-name> key=value:effect`
        *   Effects: `NoSchedule`, `NoExecute`, `PreferNoSchedule`
    *   **Add a toleration to a pod:**
        ```yaml
        tolerations:
        - key: "key"
          operator: "Equal"
          value: "value"
          effect: "NoSchedule"
        ```
*   **Node Selector:** Use `nodeSelector` in the pod spec to target nodes with specific labels.
*   **Node Affinity/Anti-Affinity:** More advanced scheduling based on node labels.
