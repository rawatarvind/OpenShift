# Network Policies InterPod Security

> Explains Kubernetes NetworkPolicies and how they restrict pod ingress and egress using layer 3 and 4 IP and port rules, with examples and CNI enforcement notes.

Kubernetes NetworkPolicies control pod-to-pod and pod-to-external traffic by specifying rules based on IP addresses and ports. At a technical level, NetworkPolicies operate on OSI layers 3 (Network — IP/CIDR) and 4 (Transport — TCP/UDP ports), allowing you to enforce pod network access using familiar firewall-like constructs.

Key SEO phrases: Kubernetes NetworkPolicy, pod-to-pod security, ingress egress rules, CNI NetworkPolicy support.

Before diving into examples, a quick OSI refresher to place NetworkPolicy primitives:

* Application layer: what applications use (e.g., Ingress controllers).
* Presentation layer: formats and encryption.
* Session layer: connection/session control (ports/sessions).
* Transport layer: TCP/UDP — where NetworkPolicy ports apply.
* Network layer: routing and IP — where NetworkPolicy ipBlock/CIDR applies.
* Data link layer: framing on the network.
* Physical layer: raw bit transmission (switches, NICs).

In Kubernetes terms: an Ingress controller lives at the application layer, ports referenced in NetworkPolicies are at the transport layer, and CIDRs/IP addresses referenced are at the network layer. NetworkPolicies therefore enforce access at the IP and port level using layer 3/4 primitives.

![A minimalist graphic with two red hexagon outlines on the left. On the right is a red rounded panel showing a computer icon and the text "how Pod communicates with network" with a blue "Inside/ Outside" label.](https://mintcdn.com/kodekloud-c4ac6d9a/_-d-o-x8WFPLT_DD/images/OpenShift-4/OpenShift-Security/Network-Policies-InterPod-Security/pod-network-communication-inside-outside.jpg?fit=max\&auto=format\&n=_-d-o-x8WFPLT_DD\&q=85\&s=94b81ee22e3372eb5a13259410ce779e)

Think of NetworkPolicies like firewall rules scoped to pods: they control ingress (incoming) and egress (outgoing) connectivity similar to a router or firewall.

![A simple slide illustration with two red hexagon network icons and a blue label reading "Control Traffic" with the note "like router/firewall...". To the right is a red rounded-square icon showing a computer monitor and a shield/document symbol representing security or monitoring.](https://mintcdn.com/kodekloud-c4ac6d9a/_-d-o-x8WFPLT_DD/images/OpenShift-4/OpenShift-Security/Network-Policies-InterPod-Security/control-traffic-network-security-monitoring.jpg?fit=max\&auto=format\&n=_-d-o-x8WFPLT_DD\&q=85\&s=b3773c6f16ad677f8d694c7f25f5620b)

Core characteristics of NetworkPolicies

* Namespaced and applied to selected pods via `podSelector`.
* Additive: multiple NetworkPolicies can select the same pod; the effective allowed traffic is the union of all rules.
* Default behavior:
  * If no NetworkPolicy selects a pod, that pod is open for both ingress and egress.
  * If a NetworkPolicy selects a pod and includes `Ingress` rules, ingress is restricted to those rules; similarly for `Egress`.
  * To lock both directions, ensure policies cover both `Ingress` and `Egress`.
* Enforcement depends on the cluster CNI: your plugin must support NetworkPolicy (examples: Calico, Cilium, Canal). Without CNI support the rules are not enforced.

Example NetworkPolicy manifest

```yaml theme={null}
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: example-pod-policy
  namespace: default
spec:
  podSelector:
    matchLabels:
      app: myapp
  policyTypes:
    - Ingress
    - Egress
  ingress:
    - from:
        - ipBlock:
            cidr: 172.17.0.0/16
            except:
              - 172.17.1.0/24
        - namespaceSelector:
            matchLabels:
              project: myproject
        - podSelector:
            matchLabels:
              role: frontend
      ports:
        - protocol: TCP
          port: 6379
  egress:
    - to:
        - ipBlock:
            cidr: 10.0.0.0/24
```

What the manifest does (concise):

* Targets pods labeled `app: myapp` in the `default` namespace.
* Restricts both ingress and egress (`policyTypes` includes both).
* Ingress allows TCP traffic on port 6379 from:
  * IPs in `172.17.0.0/16` excluding `172.17.1.0/24`.
  * Pods in namespaces labeled `project: myproject`.
  * Pods labeled `role: frontend` (in the same namespace unless a `namespaceSelector` is used).
* Egress allows outbound traffic to `10.0.0.0/24`.

Table — common NetworkPolicy fields and usage

| Field | Purpose | Example / Notes |
| - | - | - |
| `podSelector` | Selects pods the policy applies to | `matchLabels: app: myapp` |
| `policyTypes` | Which directions the policy controls | `["Ingress", "Egress"]` |
| `ingress.from` | Allowed sources for ingress | `ipBlock`, `namespaceSelector`, `podSelector` |
| `ingress.ports` | Allowed transport ports/protocols | `protocol: TCP`, `port: 6379` |
| `egress.to` | Allowed destinations for egress | `ipBlock` with CIDR or other selectors |

Behavior summary

* If a pod is selected by one or more policies that include `Ingress`, only traffic matching those `ingress` rules is allowed in.
* If a pod is selected by one or more policies that include `Egress`, only traffic matching those `egress` rules is allowed out.
* Traffic in directions not covered by any selecting policy remains unrestricted.

NetworkPolicies operate at layers 3 and 4 (IP and port). They do not filter by HTTP paths or application-layer data — use an application-aware proxy, Ingress controller, or a WAF for L7 filtering.

NetworkPolicies are enforced only if your cluster's CNI plugin implements them. Verify your CNI (e.g., Calico, Cilium) supports NetworkPolicy before relying on these rules for security.

Links and references

* Kubernetes NetworkPolicy docs: [https://kubernetes.io/docs/concepts/services-networking/network-policies/](https://kubernetes.io/docs/concepts/services-networking/network-policies/)
* Calico: [https://projectcalico.org/](https://projectcalico.org/)
* Cilium: [https://cilium.io/](https://cilium.io/)
* Redis (port reference): [https://redis.io/](https://redis.io/)

