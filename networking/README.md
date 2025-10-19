# Networking

## Network Policies

Network Policies provide a way to control the traffic flow at the IP address or port level (OSI layer 3 or 4).

*   **Ingress:** Incoming traffic to a pod.
*   **Egress:** Outgoing traffic from a pod.

By default, pods are non-isolated; they accept traffic from any source. Pods become isolated by having a `NetworkPolicy` that selects them.

*   Network policies are implemented by a network plugin (e.g., Calico, Cilium). If the CNI plugin doesn't support network policies, creating a `NetworkPolicy` resource will have no effect.
*   Policies are defined using a `NetworkPolicy` kind.
*   They use labels to select pods and define rules.
*   Rules can be specified for `ingress` and `egress` traffic.
*   You can filter traffic based on:
    *   `podSelector`: Selects pods in the same namespace.
    *   `namespaceSelector`: Selects all pods in namespaces with matching labels.
    *   `ipBlock`: Selects a range of IPs (CIDR).

### Default Behaviors

*   If an `ingress` rule is defined, it automatically allows the corresponding `egress` traffic for the response.
*   By default, traffic is allowed from any pod in any namespace that has the same label. To allow traffic from other namespaces, a `namespaceSelector` must be used.

## Ingress

An Ingress manages external access to the services in a cluster, typically HTTP. It can provide load balancing, SSL termination, and name-based virtual hosting.

*   You need an Ingress controller to satisfy an Ingress. Only creating an Ingress resource has no effect.

## Linux Networking Basics

### Routing

*   `ip link`: Show network interfaces on the host.
*   `ip addr`: See IP addresses assigned to interfaces.
*   `ip addr add <ip>/<netmask> dev <interface>`: Assign an IP address to an interface (temporary).
    *   To make it persistent, add it to `/etc/network/interfaces`.
*   To enable IP forwarding, set the value in `/proc/sys/net/ipv4/ip_forward` to `1`.
*   `ip route`: Show or manipulate the IP routing table. Use this to define gateways to reach other networks.

### DNS

*   `/etc/hosts`: A local file for static hostname-to-IP mappings.
*   `/etc/resolv.conf`: Specifies the DNS servers to use for name resolution (e.g., `nameserver 192.168.1.100`).
*   `/etc/nsswitch.conf`: Configures the name-service switch, which controls the order of lookup (e.g., `hosts: files dns` means check `/etc/hosts` first, then query DNS).
*   When a local host can't resolve a name, it forwards the request to the configured DNS server.