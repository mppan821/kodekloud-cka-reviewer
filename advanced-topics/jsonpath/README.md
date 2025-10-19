# Advanced Inspection with JSONPath

JSONPath is a powerful way to format the output of `kubectl get` to extract specific pieces of information from the JSON response.

## Usage

1.  **Get the full output in JSON** to inspect the structure of the object:
    ```bash
    kubectl get pods <pod-name> -o json
    ```
2.  **Formulate a JSONPath query** to extract the desired fields.
    *   The query is wrapped in `'{}'`.
    *   `$` represents the root object (this is optional in `kubectl`).
    *   `.items` is often used to access the list of objects when getting multiple resources. `[*]` selects all items in the list.

### Examples

*   **Get the image of the first container in the first pod:**
    ```bash
    kubectl get pods -o=jsonpath='{.items[0].spec.containers[0].image}'
    ```
*   **Loop through items with `range` to create formatted output:**
    ```bash
    kubectl get nodes -o=jsonpath='{range .items[*]}{.metadata.name}{"\t"}{.status.capacity.cpu}{"\n"}{end}'
    ```
*   **Filter results:** Find the name of a context that uses a specific user.
    ```bash
    kubectl config view -o jsonpath="{.contexts[?(@.context.user=='aws-user')].name}"
    ```

## Custom Columns

A simpler alternative to JSONPath for creating custom tabular output.

*   **Get node name and CPU capacity:**
    ```bash
    kubectl get nodes -o=custom-columns=NODE:.metadata.name,CPU:.status.capacity.cpu
    ```
*   **Sort the output:**
    ```bash
    kubectl get nodes --sort-by=.status.capacity.cpu
    ```
