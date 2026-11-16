# Networking Overview

> Overview of OpenShift networking, explaining OVS overlay SDN, pod IPs versus Services and DNS, SDN/CNI plugin modes, exposing applications and related best practices

Welcome. This lesson explains how OpenShift networking works and how it builds on Kubernetes networking concepts. We'll cover:

* A quick Kubernetes networking recap
* How OpenShift implements an overlay network using Open vSwitch (OVS)
* Pod IPs versus stable Service DNS names
* OpenShift SDN plugin modes and CNI alternatives
* How to expose applications externally

## Kubernetes networking recap

A Kubernetes cluster has control plane and worker nodes. Each node (physical or virtual) has its own IP address (for example, `192.168.1.10`, `192.168.1.11`, `192.168.1.12`, `192.168.1.13`).

Applications run inside containers within pods. Each pod receives its own IP address. Because workloads typically communicate (for example, a web frontend talking to a database), the network must provide unique pod IPs and routing between nodes so pods can reach each other reliably.

OpenShift implements a software-defined network (SDN) to provide a single virtual network that spans

![A slide titled "Challenge" showing a Kubernetes cluster diagram with a Master and three Worker nodes hosting cloud-networked pods, each labeled with an IP address. An arrow on the right points to "OpenShift SDN," indicating the network overlay.](https://mintcdn.com/kodekloud-c4ac6d9a/Xa5GerWBw0vsOVlC/images/OpenShift-4/Networks-Services-Routes-and-Scaling/Networking-Overview/kubernetes-cluster-pods-ips-openshift-sdn.jpg?fit=max\&auto=format\&n=Xa5GerWBw0vsOVlC\&q=85\&s=c42b8519b1fcde66b2f5208861a0ff34)

across all nodes in the cluster. This virtual overlay network is implemented using [Open vSwitch](https://www.openvswitch.org/) (OVS), a distributed virtual switch that supports VLAN tagging, trunking, LACP, port mirroring, and other advanced networking features.

A common default overlay CIDR in OpenShift is `10.128.0.0/14`. The platform divides that range into per-node subnets (for example `10.128.0.0`, `10.128.2.0`, `10.128.4.0`), and all pods scheduled onto a node receive IP addresses from that node’s subnet. In a sample deployment, a web pod might get `10.128.0.5` while a database pod gets `10.128.2.2`.

You can inspect pod IPs with:

```console theme={null}
$ oc get pods -o wide
NAME                        READY   STATUS    RESTARTS   AGE    IP           NODE
my-web-app                  1/1     Running   0          2d     10.128.0.5   worker-01
my-sql-db                   1/1     Running   0          1d     10.128.2.2   worker-02
```

## Pod IPs vs Services and DNS

Although pods can reach each other by IP, pod IPs are ephemeral — they can change after a restart or reschedule. To provide stable connectivity, OpenShift exposes the Service abstraction and an internal DNS so clients talk to stable names instead of pod IPs.

Bad (discouraged) example — connecting directly to a pod IP:

```python theme={null}
mysql.connect("10.128.2.2")
```

Preferred examples — connect via a Service name or the full cluster DNS name:

```python theme={null}
# Using a Service name (service in the same namespace)
mysql.connect("my-sql-db")

# Or using the full cluster DNS name
mysql.connect("my-sql-db.default.svc.cluster.local")
```

OpenShift’s DNS implementation currently uses [CoreDNS](https://coredns.io/) in modern releases (earlier versions used SkyDNS/kube-dns). CoreDNS is backed by the cluster datastore so Service names resolve reliably across the cluster.

Always prefer Services + DNS names to reach applications inside the cluster rather than using pod IPs. Services provide a stable endpoint and decouple clients from pod lifecycle events.

## OpenShift SDN plugin modes

OpenShift SDN supports multiple plugin modes. A common default is `ovs-subnet`, which provides cross-node pod connectivity by allocating subnets and programming OVS. To enforce isolation between projects (namespaces), OpenShift also offers `ovs-multitenant`, which assigns each project a virtual network ID (VNID) to prevent unintended pod-to-pod communication across projects.

![A slide titled "SDN Plugins" showing four rounded host boxes labeled "ovs-multitenant" linked by an "ovs-subnet" with rows of red server/container icons and network connection symbols beneath them. The diagram illustrates multitenant Open vSwitch networking connecting many servers/containers across hosts.](https://mintcdn.com/kodekloud-c4ac6d9a/Xa5GerWBw0vsOVlC/images/OpenShift-4/Networks-Services-Routes-and-Scaling/Networking-Overview/sdn-ovs-multitenant-subnet-diagram.jpg?fit=max\&auto=format\&n=Xa5GerWBw0vsOVlC\&q=85\&s=c7f6a0d17e01929b587b4ab8fa6419b3)

A quick comparison:

| Plugin | Purpose | Typical use case |
| - | - | - |
| `ovs-subnet` | Provides a flat overlay across all nodes | Simple clusters needing basic pod networking |
| `ovs-multitenant` | Adds VNID-based isolation between projects | Multitenant environments requiring namespace isolation |
| CNI alternatives (e.g., Nuage, Contiv, `flannel`) | Different data plane and policy capabilities | Large-scale, policy-driven, or vendor-specific networking needs |

Beyond OVS-based plugins, OpenShift can integrate with other CNI providers such as Nuage, Contiv, and [Flannel](https://github.com/flannel-io/flannel). Each solution has different operational characteristics and features — evaluate them based on throughput, policy, observability, and operational workflow.

## Exposing applications outside the cluster

So far we covered overlay networking that enables internal pod-to-pod and Service connectivity. To expose applications to users outside the cluster, OpenShift provides:

* Services with `NodePort`, `LoadBalancer`, or `ClusterIP` types
* Routes (OpenShift’s higher-level routing resource) that map external hostnames to Services and provide TLS termination, path routing, and host-based routing

Choose Services for internal or L4 exposure and Routes when you need hostname-based L7 routing and OpenShift-managed ingress.

## Best practices

* Use Services + DNS for in-cluster communication to avoid brittle pod IP dependencies.
* Choose an SDN/CNI plugin that matches your isolation, performance, and policy requirements.
* Monitor overlay CIDR utilization to avoid IP exhaustion during scaling.
* Use Routes for external HTTP(S) traffic; use LoadBalancer or Ingress depending on your infrastructure provider.

## References

* [Open vSwitch](https://www.openvswitch.org/)
* [CoreDNS](https://coredns.io/)
* [Flannel on GitHub](https://github.com/flannel-io/flannel)
* [Kubernetes Networking Concepts](https://kubernetes.io/docs/concepts/cluster-administration/networking/)



