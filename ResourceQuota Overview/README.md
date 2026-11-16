# ResourceQuota Overview

> Explains Kubernetes ResourceQuota usage to limit namespace aggregate CPU memory storage and object counts, with examples, best practices, and troubleshooting guidance

At this point we understand pod resource requests and limits, but there is another important cluster-level control: ResourceQuota. A ResourceQuota is applied at the namespace level (in OpenShift this is called a project) and constrains the total resources a namespace/project can consume so that a single team or workload cannot exhaust cluster-wide resources.

![A Red Hat OpenShift diagram showing a user icon connected to a red "Name Space" container that contains two simplified service/node hierarchy icons. The Red Hat OpenShift logo and the label "Project" appear at the top right.](https://mintcdn.com/kodekloud-c4ac6d9a/_-d-o-x8WFPLT_DD/images/OpenShift-4/Storage-Templates-and-Catalog/ResourceQuota-Overview/redhat-openshift-namespace-user-services.jpg?fit=max\&auto=format\&n=_-d-o-x8WFPLT_DD\&q=85\&s=706f88241162d80fc0c7b2e92b6dfb89)

In [Kubernetes](https://learn.kodekloud.com/user/courses/kubernetes-for-the-absolute-beginners-hands-on-tutorial) terminology this is called a namespace; in [OpenShift](https://learn.kodekloud.com/user/courses/openshift-4) it’s called a project. They serve the same purpose—logical isolation and scoping for resources.

Inside a namespace/project you can configure ResourceQuota objects to limit aggregated compute (CPU, memory), storage-related resources (PersistentVolumeClaims), and counts of Kubernetes objects (Secrets, ConfigMaps, Services, etc.). When usage in a namespace exceeds the configured quota, attempts to create objects that would increase usage beyond the quota will be blocked or leave resources in a Pending state until the quota is changed or usage drops.

![A slide titled "Quota" that says "Provide Constraints for Resources" and shows a red panel labeled "Total Compute Resources" with CPU and storage icons. An arrow points to a red hexagon with the note "Status: Pending...".](https://mintcdn.com/kodekloud-c4ac6d9a/_-d-o-x8WFPLT_DD/images/OpenShift-4/Storage-Templates-and-Catalog/ResourceQuota-Overview/quota-total-compute-resources-pending.jpg?fit=max\&auto=format\&n=_-d-o-x8WFPLT_DD\&q=85\&s=4a19c9aee11e7cdf5cb69498a137f33a)

For example, if you set a memory quota for a namespace and the sum of running pods' memory requests already consumes that quota, creating additional pods that request more memory will remain in Pending (or fail to be created). To resolve this situation you can:

* Increase the ResourceQuota for that namespace.
* Terminate or scale down running workloads so resources are freed and available for new pods.

If a new pod remains in Pending due to ResourceQuota, it is not a scheduler issue — check the namespace's ResourceQuota and the sum of current requests/limits before troubleshooting scheduling.

![A slide titled "Quota" that reads "Provide Constraints for Resources" and shows a pink rounded slider with markers labeled "Limit 01" and "New Limit." Red rounded-square buttons numbered "1." and "2." appear along the left side.](https://mintcdn.com/kodekloud-c4ac6d9a/_-d-o-x8WFPLT_DD/images/OpenShift-4/Storage-Templates-and-Catalog/ResourceQuota-Overview/quota-constraints-pink-slider-limit-01.jpg?fit=max\&auto=format\&n=_-d-o-x8WFPLT_DD\&q=85\&s=c9c3eaf261e306b0c16c24d52a645847)

ResourceQuota also provides fine-grained control over how many Kubernetes objects of particular types can exist in a namespace—helpful for controlling storage and cluster-wide resources like load balancers.

![A slide titled "Quota" showing three red icons and three bullet points about resource limits: "Provide Constraints for Resources", "Limit the Quantity of Resources", and "Limit Total Number of Storage and Computer Resources."](https://mintcdn.com/kodekloud-c4ac6d9a/_-d-o-x8WFPLT_DD/images/OpenShift-4/Storage-Templates-and-Catalog/ResourceQuota-Overview/quota-resource-limits-storage-compute.jpg?fit=max\&auto=format\&n=_-d-o-x8WFPLT_DD\&q=85\&s=dad042608c8b67aa2bf43437a5e47a64)

Common ResourceQuota use cases

| Use case | What to limit | Example field(s) |
| - | - | - |
| Aggregate compute | Sum of requests and limits for CPU and memory | `requests.memory`, `limits.memory`, `requests.cpu`, `limits.cpu` |
| Storage objects | Number of PVCs used in a namespace | `persistentvolumeclaims` |
| Object counts | Limit counts of Secrets, ConfigMaps, Services, etc. | `secrets`, `configmaps`, `services`, `services.loadbalancers` |

Below are two common ResourceQuota examples you might apply.

* Example 1: Limit aggregated memory requests and limits in the `test` namespace.

```yaml theme={null}
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

This ResourceQuota ensures the sum of all containers' memory requests in the `test` namespace cannot exceed `512Mi`, and the sum of all memory limits cannot exceed `1000Mi`.

* Example 2: Limit counts of specific core objects in a namespace.

```yaml theme={null}
apiVersion: v1
kind: ResourceQuota
metadata:
  name: core-object-counts
spec:
  hard:
    configmaps: "10"
    persistentvolumeclaims: "4"
    replicationcontrollers: "20"
    secrets: "10"
    services: "10"
    services.loadbalancers: "2"
```

In this example, the namespace may have at most 10 ConfigMaps, 4 PVCs, 20 ReplicationControllers, 10 Secrets, 10 Services total, and up to 2 Service objects of type LoadBalancer.

Best practices

* Define sensible per-pod resource `requests` and `limits` so scheduler decisions are predictable.
* Apply ResourceQuota at the namespace level to control overall consumption across teams or applications.
* Combine ResourceQuota with LimitRange to enforce defaults and minimum/maximum per-pod/container values.

Using ResourceQuota together with LimitRange helps ensure consistent resource allocation and prevents noisy neighbors from consuming all namespace resources.

Further reading and references

* [Kubernetes ResourceQuota documentation](https://kubernetes.io/docs/concepts/policy/resource-quotas/)
* [OpenShift Projects and Resource Quotas](https://docs.openshift.com/)

