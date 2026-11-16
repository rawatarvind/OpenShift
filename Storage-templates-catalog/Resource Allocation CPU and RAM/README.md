# Resource Allocation CPU and RAM

> Explains how Kubernetes uses resource requests and limits for CPU and memory to drive scheduling, runtime enforcement, QoS classes, and cluster stability, with examples and best practices

This guide explains how Kubernetes allocates CPU and memory using resource *requests* and *limits*. These settings drive scheduling, runtime enforcement, Quality of Service (QoS) classification, and cluster stability. Understanding how requests and limits interact with the scheduler, kubelet, and cgroups helps you avoid unscheduled pods, OOMKills, or unexpected CPU throttling.

Key terms:

* Request — the resource amount the pod asks the scheduler to reserve for placement.
* Limit — the maximum resource a container is allowed to consume at runtime.
* kube-scheduler — places pods onto nodes based on requests.
* kubelet & cgroups — enforce limits and report OOM or throttling events.

## Requests

A request is the amount of CPU or memory a pod declares so the scheduler can reserve capacity on a node. For example, a pod needing 500Mi memory should set `requests.memory: "500Mi"`. The scheduler sums requests of already scheduled pods and the new pod, then chooses a node that has the available capacity.

Scheduling is strictly based on requests — not limits. If your cluster has fixed node resources (for example, a node with 24Gi RAM), many requests can exhaust available node capacity. Without cluster autoscaling, exhausted capacity leads to unschedulable pods and degraded services.

![A stylized presentation slide titled "Requests" showing a request flow with red icons, a server stack and arrows. Text labels show "24 GIGs" and a red speech bubble saying "Hey! We are running out of space...".](https://mintcdn.com/kodekloud-c4ac6d9a/_-d-o-x8WFPLT_DD/images/OpenShift-4/Storage-Templates-and-Catalog/Resource-Allocation-CPU-and-RAM/requests-flow-server-stack-24gigs.jpg?fit=max\&auto=format\&n=_-d-o-x8WFPLT_DD\&q=85\&s=130d2c3ec502a0faf7310f2575c94df3)

## Limits

A limit is the maximum a container can consume at runtime.

* Memory: If a container exceeds its memory limit, the kernel will usually terminate the process (reported as `OOMKilled`). Memory limits are enforced by cgroups and managed by the kubelet.
* CPU: If a CPU limit is set, Linux CFS quota mechanisms throttle the container when it tries to use more than the allowed CPU. If no CPU limit is set, the container can use spare CPU cycles on the node when available (subject to contention and QoS).

At runtime kubelet enforces limits via cgroups; at scheduling time the scheduler uses requests only.

![A minimalist slide titled "Limits" showing two stacked red arrow buttons and the text "Kubelet Ensures that the Limit is Met." To the right is a green-bordered box containing two red service/hierarchy icons with a green checkmark indicating the limit is satisfied.](https://mintcdn.com/kodekloud-c4ac6d9a/_-d-o-x8WFPLT_DD/images/OpenShift-4/Storage-Templates-and-Catalog/Resource-Allocation-CPU-and-RAM/kubelet-limits-stacked-arrows-green-check.jpg?fit=max\&auto=format\&n=_-d-o-x8WFPLT_DD\&q=85\&s=5eacaf41e0a01fd1fe62b43e69fe2177)

## How requests and limits work together

* Scheduler uses requests to select a node that has enough available capacity.
* kubelet (via cgroups) enforces limits at runtime.
* Memory limit violation → container is typically OOMKilled.
* CPU limit violation → container is throttled (not killed).

## ResourceQuota and namespace-level controls

A `ResourceQuota` restricts aggregate resource consumption across all pods in a namespace. It counts the sum of requests and/or limits across the namespace.

Example: the following manifests create a `test` namespace and a `ResourceQuota` that caps total requested memory to `512Mi` and total memory limits to `1000Mi` for that namespace.

```yaml theme={null}
apiVersion: v1
kind: Namespace
metadata:
  name: test
---
apiVersion: v1
kind: ResourceQuota
metadata:
  name: memorylimit
  namespace: test
spec:
  hard:
    requests.memory: 512Mi
    limits.memory: 1000Mi
```

Important: these values are totals for the namespace. They do not apply per-pod unless you design them that way. If any new pod would cause the namespace totals to exceed the quota, the pod will be rejected.

## Example container resource configuration

Below is a container spec fragment (part of a Pod or Deployment) showing per-container requests and limits. Use this to control scheduling and runtime behavior.

```yaml theme={null}
metadata:
  namespace: webapp
  labels:
    app: nginxdeployment
spec:
  containers:
  - name: nginxdeployment
    image: nginx:latest
    resources:
      requests:
        memory: "64Mi"
        cpu: "250m"
      limits:
        memory: "128Mi"
    ports:
    - containerPort: 80
```

Notes on this example:

* The scheduler accounts for `64Mi` memory and `250m` CPU when placing the pod.
* At runtime the container may consume up to `128Mi` memory; exceeding this will likely cause an `OOMKilled` event.
* No CPU `limits` are specified, so Kubernetes will not install a CPU quota for this container; it may use spare CPU cycles if available but is not guaranteed guaranteed throughput beyond the `requests`.

## CPU vs memory: differences and behavior

The two resources behave differently in practice:

| Aspect | CPU | Memory |
| - | - | - |
| Scheduling uses | `requests` for both CPU and memory | `requests` |
| Runtime enforcement | Throttling when `limits.cpu` exceeded (CFS quota) | Hard enforcement: exceeding `limits.memory` usually causes `OOMKilled` |
| Burst behavior | Can burst beyond `requests` if no `limits` set and CPU spare exists | Cannot “burst” beyond memory limits — process killed on OOM |
| Compressible? | Yes — CPU is compressible (can be throttled) | No — memory is incompressible (kernel OOM) |
| Common impact if misconfigured | Performance degradation due to throttling | Pod termination and potential node instability |

![A minimalist diagram showing two square icons side-by-side: a red tile with a CPU/chip symbol and a blue tile with a memory card symbol. Below them are red labels that read "Doesn't give CPU" and "Gives back Memory."](https://mintcdn.com/kodekloud-c4ac6d9a/_-d-o-x8WFPLT_DD/images/OpenShift-4/Storage-Templates-and-Catalog/Resource-Allocation-CPU-and-RAM/red-cpu-blue-memory-tiles-labels.jpg?fit=max\&auto=format\&n=_-d-o-x8WFPLT_DD\&q=85\&s=8856357c7edbbe00e23444aa45a116bf)

## QoS classes (Guaranteed, Burstable, BestEffort)

QoS is derived from how you set requests and limits for containers in a Pod. It influences eviction order and scheduling behavior under resource pressure.

| QoS class | How to achieve it | Behavior under pressure |
| - | - | - |
| Guaranteed | All containers set equal `requests` and `limits` for every resource (CPU & memory) | Last to be evicted |
| Burstable | Containers set `requests` but either omit `limits` or set `limits` > `requests` | Evicted before Guaranteed, after BestEffort |
| BestEffort | No `requests` and no `limits` set | First to be evicted under pressure |

## Best practices

* Always set `requests` for both CPU and memory so the scheduler can make accurate placement decisions.
* Set memory `limits` to protect nodes from runaway containers (memory leaks) that can destabilize a node.
* Consider `limits.cpu` carefully: setting CPU limits prevents bursting but can cause throttling. If you want burstable CPU behavior, rely on `requests` and omit CPU `limits` for those workloads.
* Use `ResourceQuota` to control aggregate resource consumption in a namespace and prevent noisy neighbors.
* Measure and iterate: start with conservative `requests` and `limits`, monitor (Prometheus, metrics-server, or cloud metrics), then tune based on actual usage and SLOs.

Do not rely solely on heuristics. Omitted memory limits can allow a container to consume node memory leading to node instability. Conversely, overly-tight CPU limits can cause excessive throttling and poor performance. Monitor and test under realistic load.

## Links and references

* Kubernetes scheduling and resource management: [https://kubernetes.io/docs/concepts/configuration/manage-resources-containers/](https://kubernetes.io/docs/concepts/configuration/manage-resources-containers/)
* ResourceQuota docs: [https://kubernetes.io/docs/concepts/policy/resource-quotas/](https://kubernetes.io/docs/concepts/policy/resource-quotas/)
* QoS classes: [https://kubernetes.io/docs/concepts/configuration/pod-priority-preemption/#quality-of-service](https://kubernetes.io/docs/concepts/configuration/pod-priority-preemption/#quality-of-service)

## Monitoring and tuning

Use real-world telemetry (CPU/memory usage, throttling metrics, OOM events) from tools like Prometheus, metrics-server, or cloud provider monitoring to tune `requests` and `limits`. Iterate based on latency, throughput, and SLOs rather than static guesses.

