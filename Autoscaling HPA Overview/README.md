# Autoscaling HPA Overview

> Explains vertical versus horizontal scaling and how Kubernetes OpenShift Horizontal Pod Autoscaler automates replica scaling using metrics, formulas, manifests, examples, and best practices.

Let's take a moment to clarify vertical vs horizontal scaling and how Kubernetes/OpenShift implements autoscaling with the Horizontal Pod Autoscaler (HPA).

Vertical scaling increases the CPU and memory allocated to a single pod when that pod is under-provisioned for a heavier load. This approach is sometimes required for specific workloads (e.g., stateful apps that can't easily be replicated), but in cloud-native deployments horizontal scaling is generally preferred.

![An infographic titled "Vertical Scaling / Horizontal Scaling" showing two rounded cards underneath: a red card with a CPU chip icon and a blue card with a storage/memory icon.](https://mintcdn.com/kodekloud-c4ac6d9a/Xa5GerWBw0vsOVlC/images/OpenShift-4/Networks-Services-Routes-and-Scaling/Autoscaling-HPA-Overview/vertical-horizontal-scaling-cpu-memory-infographic.jpg?fit=max\&auto=format\&n=Xa5GerWBw0vsOVlC\&q=85\&s=814772bffdc9dc69588d67b562f8ff5b)

Horizontal scaling adjusts the number of pod replicas to match demand. For example, an application might run a single pod during low traffic, scale to multiple replicas under load, and then scale back down. The Horizontal Pod Autoscaler automates this process by observing resource usage (CPU, memory) or custom metrics and altering the replica count within a configured minimum and maximum.

How the HPA works

* The HPA controller periodically queries metrics for the target pods (CPU, memory, or external/custom metrics).
* It compares the observed utilization against the target value you specify.
* Using that comparison it computes a desired replica count and scales the target resource up or down, bounded by `minReplicas` and `maxReplicas`.
* The HPA relies on a metrics provider in the cluster (for example, `metrics-server` or a Prometheus adapter) to supply those metrics.

Ensure your cluster exposes the required metrics to the HPA (for example via the Prometheus adapter or `metrics-server`). Without a metrics provider, the HPA cannot make scaling decisions.

HPA replica calculation (common formula)
A commonly used formula to determine desired replicas is:

desiredReplicas = ceil(currentReplicas \* (currentUtilization / targetUtilization))

Example:

* If current replicas = 2, current CPU utilization = 80%, target utilization = 50%:
  desiredReplicas = ceil(2 \* 80/50) = ceil(3.2) = 4

HPA manifest example (autoscaling/v2)
This example configures the HPA to target CPU utilization for a Deployment named `my-app`:

```yaml theme={null}
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: my-app-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: my-app
  minReplicas: 1
  maxReplicas: 5
  metrics:
    - type: Resource
      resource:
        name: cpu
        target:
          type: Utilization
          averageUtilization: 50
```

Quick CLI creation (oc compatible)
You can create a basic autoscaler from the command line (OpenShift `oc` or `kubectl` compatible):

```bash theme={null}
oc autoscale deployment/my-app --min=1 --max=5 --cpu-percent=50
```

Vertical vs Horizontal scaling — quick comparison

| Resource Type | What changes | Best for | Example fields |
| - | - | - | - |
| Vertical scaling | Increases CPU/memory of a single pod | Workloads that cannot be replicated easily (legacy/stateful apps) | `resources.requests`, `resources.limits` |
| Horizontal scaling | Changes the number of pod replicas | Microservices and stateless apps that scale out/in | `minReplicas`, `maxReplicas`, `metrics` |

Best practices and considerations

* Prefer horizontal scaling for stateless workloads; it offers better availability and resiliency.
* Tune `minReplicas` and `maxReplicas` to prevent constant scaling (flapping) or under-provisioning.
* Use appropriate metrics: CPU is common, but custom or external metrics (latency, queue length) can provide better scaling behavior for some apps.
* Use `autoscaling/v2` when you need multiple metric types or fine-grained controls. `autoscaling/v2beta2` was previously common but check your cluster's supported API versions.

If your cluster lacks a metrics provider (for example `metrics-server` or a Prometheus adapter), the HPA will not receive the metrics it needs and scaling will not occur. Verify metrics are available before relying on HPA.

Useful links and references

* [Kubernetes Horizontal Pod Autoscaler](https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale/)
* [OpenShift Documentation](https://docs.openshift.com/)
* [prometheus-adapter (GitHub)](https://github.com/kubernetes-sigs/prometheus-adapter)
* [metrics-server (GitHub)](https://github.com/kubernetes-sigs/metrics-server)

Summary

* Vertical scaling changes the resources allocated to a single pod; horizontal scaling changes the number of replicas.
* HPA automates horizontal scaling by reading metrics, comparing current vs. target utilization, computing desired replicas with a formula like the one shown above, and scaling within `minReplicas` and `maxReplicas`.
* Ensure your cluster exposes metrics (via `metrics-server` or a Prometheus adapter) so HPA can function.