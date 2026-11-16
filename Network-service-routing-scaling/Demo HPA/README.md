# Demo HPA

> Guide to enabling and managing Kubernetes/OpenShift Horizontal Pod Autoscalers using oc autoscale or declarative HorizontalPodAutoscaler manifests, including metrics, examples, troubleshooting and removal

This guide covers two primary ways to enable Horizontal Pod Autoscaling (HPA) in an OpenShift / Kubernetes cluster:

* Quick one-off setup from the command line using `oc autoscale`.
* Declarative control by creating a `HorizontalPodAutoscaler` YAML manifest.

Both approaches use Kubernetes' autoscaling API (not OpenShift-specific) and rely on the cluster exposing resource metrics (for example via `metrics-server` or another metrics provider). Without metrics the HPA controller cannot make scaling decisions.

CPU-based autoscaling requires metrics to be exposed in the cluster (e.g., `metrics-server`). If resource metrics are unavailable, the HPA controller will not be able to scale pods.

## Quick comparison

| Method | Use case | Command / Example |
| - | - | - |
| CLI | Fast, ad-hoc autoscaling for existing Deployment | `oc autoscale deployment/<name> --min=<min> --max=<max> --cpu-percent=<n>` |
| Declarative YAML | Repeatable, version-controlled HPA resource | Apply a `HorizontalPodAutoscaler` manifest with `oc apply -f` |

## Autoscaling with the CLI

Use `oc autoscale` to rapidly attach an HPA to an existing Deployment. The general form:

```bash theme={null}
oc autoscale deployment/<deployment-name> --min=<min-replicas> --max=<max-replicas> [--cpu-percent=<target-cpu-percent>]
```

Example: ensure the `front-end` Deployment never drops below two replicas and may scale up to seven replicas, targeting 75% CPU:

```bash theme={null}
oc autoscale deployment/front-end --min=2 --max=7 --cpu-percent=75
```

What this does:

* Creates a `HorizontalPodAutoscaler` resource in the same namespace that targets the named Deployment.
* The HPA controller enforces the configured `minReplicas` and scales up/down (within `maxReplicas`) based on the metric (CPU in the example).

If you refresh the OpenShift Console and view the Pods for the `front-end` Deployment, you should see at least the minimum number of pods running.

![A screenshot of the Red Hat OpenShift Container Platform web console showing the "Pods" tab in the default project, listing front-end pods with status "Running," ready/restarts counts, and owner info. The left navigation shows Administrator workloads (Deployments, Pods, etc.) and a red banner warns the cluster is for development/testing.](https://mintcdn.com/kodekloud-c4ac6d9a/Xa5GerWBw0vsOVlC/images/OpenShift-4/Networks-Services-Routes-and-Scaling/Demo-HPA/red-hat-openshift-pods-default.jpg?fit=max\&auto=format\&n=Xa5GerWBw0vsOVlC\&q=85\&s=9d0fde7045ef804a482d95b520e3d4c8)

## Creating a HorizontalPodAutoscaler YAML

For reproducibility and source control, create an HPA manifest. HPAs live in the `autoscaling` API group. Use:

* `autoscaling/v1` for simple CPU-based autoscaling with `targetCPUUtilizationPercentage`.
* `autoscaling/v2` for advanced metric types, multiple metrics, and custom metrics formats.

Example `autoscaling/v1` manifest that targets the `front-end` Deployment:

```yaml theme={null}
apiVersion: autoscaling/v1
kind: HorizontalPodAutoscaler
metadata:
  name: front-end
  namespace: default
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: front-end
  minReplicas: 3
  maxReplicas: 7
  targetCPUUtilizationPercentage: 75
```

Important notes:

* Do not include a `status` block in your manifest — the controller populates `status` automatically.
* For memory-based autoscaling, custom metrics, or multiple metric types, use `autoscaling/v2` and the `metrics` array described in the Kubernetes docs.

Apply the manifest:

```bash theme={null}
oc apply -f hpa-front-end.yaml
```

Or paste the YAML into the OpenShift Console using the Developer or Administrator perspective: Import YAML → Create Resource.

![A screenshot of the Red Hat OpenShift Container Platform web console showing the "Deployments" view for the default project, listing deployments like carts, catalogue, front-end and orders with their pod counts and labels. The left sidebar shows Administrator navigation (Workloads → Deployments) and a red banner warns CodeReady Containers is for development and testing.](https://mintcdn.com/kodekloud-c4ac6d9a/Xa5GerWBw0vsOVlC/images/OpenShift-4/Networks-Services-Routes-and-Scaling/Demo-HPA/openshift-console-deployments-default-project.jpg?fit=max\&auto=format\&n=Xa5GerWBw0vsOVlC\&q=85\&s=3256f6c413283199bebe07cce4c37c26)

## Removing or replacing an existing HPA

HPA resource names are unique within a namespace. If an HPA already exists for the Deployment, attempting to create another with the same name will fail. To replace or remove an existing HPA:

* CLI: `oc delete hpa <name>`
* Console: open the Deployment, click the actions (hamburger) menu, choose to remove the HorizontalPodAutoscaler, and confirm the removal.

Deleting an HPA removes the autoscaling rules but does not delete the Deployment. The Deployment's replica count will remain at whatever value it currently has until manually changed or another automation adjusts it.

![A screenshot of the Red Hat OpenShift web console showing a confirmation dialog asking to remove the HorizontalPodAutoscaler for the "front-end" deployment, with Cancel and Remove buttons. The deployments list and left-hand navigation are visible in the background.](https://mintcdn.com/kodekloud-c4ac6d9a/Xa5GerWBw0vsOVlC/images/OpenShift-4/Networks-Services-Routes-and-Scaling/Demo-HPA/redhat-openshift-confirm-remove-hpa-frontend.jpg?fit=max\&auto=format\&n=Xa5GerWBw0vsOVlC\&q=85\&s=6dcb0db65c38de97d3e61e256330bc58)

After removal, reapply your YAML manifest or recreate the HPA in the Console. The HPA controller will reconcile and resume adjusting the Deployment's replica count according to the configured targets.

## Troubleshooting tips

* Verify metrics availability:
  * Check that `metrics-server` or another metrics provider is deployed and healthy.
  * Use `kubectl top pods` (or `oc adm top pods`) to confirm CPU/memory metrics are visible.
* Validate HPA resource:
  * `oc get hpa` and `oc describe hpa <name>` show current metrics, observed values, and events.
* Common causes for no scaling:
  * No metrics available.
  * Resource requests are not set on pods (HPA uses requests to calculate utilization).
  * Incorrect `scaleTargetRef` (wrong name, kind, or API group).

## Links and references

* [Kubernetes Horizontal Pod Autoscaler concepts](https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale/)
* [OpenShift Documentation — Autoscaling and HPA](https://docs.openshift.com/)
* [metrics-server GitHub](https://github.com/kubernetes-sigs/metrics-server)

