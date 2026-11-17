# Demo Network Policies

> Demonstrates a Kubernetes NetworkPolicy restricting database pod ingress to pods with a specific label, plus deployment manifest, application commands, and OpenShift console verification

This article demonstrates a Kubernetes NetworkPolicy that restricts ingress to a database pod (CartsDB) by using pod labels. You will see the NetworkPolicy manifest, the Carts deployment that gets the required label, how to apply them with `oc`, and what to expect in the OpenShift web console.

## NetworkPolicy: allow ingress only from labeled pods

Below is a simple NetworkPolicy that selects the CartsDB pods and permits ingress only from pods that carry the label `cartsdbaccess: "true"`:

```yaml theme={null}
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: access-cartsdb
spec:
  podSelector:
    matchLabels:
      name: carts-db
  ingress:
  - from:
    - podSelector:
        matchLabels:
          cartsdbaccess: "true"
```

What this policy does:

* `podSelector.matchLabels.name: carts-db` targets the CartsDB pods in the same namespace as the NetworkPolicy.
* The `ingress` block allows traffic only from pods with the label `cartsdbaccess: "true"`.
* Any pod without that label cannot initiate inbound connections to the selected CartsDB pods.

| Field | Purpose | Example |
| - | - | - |
| `podSelector` | Selects which pods the policy applies to (namespace-scoped) | `matchLabels: { name: carts-db }` |
| `ingress.from.podSelector` | Restricts allowed source pods (only same namespace) | `matchLabels: { cartsdbaccess: "true" }` |

Labels are case-sensitive and must match exactly between the NetworkPolicy and pod metadata. When a NetworkPolicy selects a pod and defines ingress rules, all ingress not explicitly allowed by any matching NetworkPolicy will be denied for that pod (assuming your CNI implements NetworkPolicy).

## Carts deployment: grant database access via label

This Deployment creates the `carts` application pods and adds the `cartsdbaccess: "true"` label so they match the NetworkPolicy's `from.podSelector` and can reach the CartsDB pods:

```yaml theme={null}
apiVersion: apps/v1
kind: Deployment
metadata:
  name: carts
  labels:
    name: carts
spec:
  replicas: 1
  selector:
    matchLabels:
      name: carts
  template:
    metadata:
      labels:
        name: carts
        cartsdbaccess: "true"
    spec:
      containers:
      - name: carts
        image: weaveworksdemos/carts:0.4.8
```

Because the pods created by this Deployment include `cartsdbaccess: "true"`, they match the NetworkPolicy's `from.podSelector` and are allowed inbound access to pods labeled `name: carts-db`.

Apply the Carts deployment:

```bash theme={null}
oc apply -f carts.yaml
```

Example output:

```bash theme={null}
PS C:\Users\mike\Desktop> oc apply -f carts.yaml
Warning: spec.template.spec.nodeSelector[beta.kubernetes.io/os]: deprecated since v1.14; use "kubernetes.io/os" instead
Warning: would violate PodSecurity "restricted:latest": allowPrivilegeEscalation != false (container "carts" must set securityContext.allowPrivilegeEscalation=false), unrestricted capabilities (container "carts" must set securityContext.capabilities.drop=["ALL"]), runAsNonRoot != true (pod or container "carts" must set securityContext.runAsNonRoot=true), seccompProfile (pod or container "carts" must set securityContext.seccompProfile.type to "RuntimeDefault" or "Localhost")
deployment.apps/carts created
PS C:\Users\mike\Desktop>
```

After applying, the Carts pod should be running and will include the `cartsdbaccess: "true"` label that allows it to access the CartsDB pods.

![A screenshot of the Red Hat OpenShift web console showing the "Pod details" page for pod carts-6945f489b-l2k78 (namespace: default) with its status listed as Running. The left sidebar displays the cluster navigation (Workloads → Pods, Deployments, etc.).](https://mintcdn.com/kodekloud-c4ac6d9a/_-d-o-x8WFPLT_DD/images/OpenShift-4/OpenShift-Security/Demo-Network-Policies/openshift-web-console-pod-carts-running.jpg?fit=max\&auto=format\&n=_-d-o-x8WFPLT_DD\&q=85\&s=04a57f4efda0b542e4d0081a3258261f)

## Apply the NetworkPolicy

Once your application pods have the correct label, apply the NetworkPolicy manifest:

```bash theme={null}
oc apply -f networkpolicy.yaml
```

Example output:

```bash theme={null}
PS C:\Users\mike\Desktop> oc apply -f .\networkpolicy.yaml
networkpolicy.networking.k8s.io/access-cartsdb created
PS C:\Users\mike\Desktop>
```

View the policy in the OpenShift (or Kubernetes) web console under Networking. The policy should show:

* The pod selector targeting pods labeled `name: carts-db`.
* The ingress rule that allows traffic only from pods labeled `cartsdbaccess: "true"`.

## Namespace scope and cross-namespace access

NetworkPolicy selectors are namespace-scoped by default. That means:

* `podSelector` and `from.podSelector` only match pods in the same namespace as the NetworkPolicy.
* To allow access from pods in other namespaces, include a `namespaceSelector` or use a combination of selectors.

| Use case | How to permit |
| - | - |
| Allow pods in another namespace | Add `from: - namespaceSelector: ...` to the ingress rule |
| Allow specific pods across namespaces | Combine `namespaceSelector` and `podSelector` inside `from` |

## Links and references

* [OpenShift 4 course](https://learn.kodekloud.com/user/courses/openshift-4)
* [Kubernetes for the Absolute Beginners - Hands-on Tutorial](https://learn.kodekloud.com/user/courses/kubernetes-for-the-absolute-beginners-hands-on-tutorial)
* Kubernetes NetworkPolicy docs: [https://kubernetes.io/docs/concepts/services-networking/network-policies/](https://kubernetes.io/docs/concepts/services-networking/network-policies/)
