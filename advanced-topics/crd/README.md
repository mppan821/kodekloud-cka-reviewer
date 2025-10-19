# Custom Resource Definitions (CRDs) and Operators

## Custom Resource Definitions (CRDs)

CRDs allow you to define your own object types, extending the Kubernetes API to meet your needs. Once you create a CRD, you can create and manage objects of that type just like built-in objects like `Pods` or `Deployments`.

A CRD object defines a new resource with a name and a schema that you specify. The Kubernetes API server then serves a new RESTful endpoint for your custom resource.

## Custom Controllers (Operators)

A CRD on its own is just data. To make it useful, you need a **custom controller** (also known as an **Operator**) that understands what to do with those custom objects.

*   A controller is a control loop that watches the state of the cluster for changes to specific objects and makes changes to move the current state towards the desired state.
*   The **Operator pattern** combines a CRD with a custom controller to manage a specific application or piece of software. For example, a `Prometheus` operator would watch for `Prometheus` custom objects and automatically configure and manage the underlying `Pods`, `Services`, and `ConfigMaps` needed to run a Prometheus monitoring instance.
*   [Operator Hub](https://operatorhub.io/) is a registry for finding and sharing community-built operators.
