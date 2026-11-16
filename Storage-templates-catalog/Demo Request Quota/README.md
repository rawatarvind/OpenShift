# Demo Request Quota

> Guide to create and validate an OpenShift namespace ResourceQuota that limits total pod memory requests and deploy a sample application to observe quota usage.

This lesson shows how to create and validate a namespace-wide ResourceQuota in OpenShift that limits the total memory requested by pods. You'll:

1. Create a namespace (project).
2. Apply a `ResourceQuota` manifest that restricts `requests.memory`.
3. Deploy a sample application that requests memory and observe quota usage.

## 1) Create a project (namespace)

In the OpenShift web console:

* Click Home → Projects → Create Project
* Name: `limittester` (Display name can be the same)
* Description: `namespace for resource testing`

![Screenshot of the Red Hat OpenShift web console with a "Create Project" modal open showing the Name and Display name set to "limittester" and a description field being edited. The left sidebar shows the Administrator menu with Projects selected.](https://mintcdn.com/kodekloud-c4ac6d9a/_-d-o-x8WFPLT_DD/images/OpenShift-4/Storage-Templates-and-Catalog/Demo-Request-Quota/openshift-create-project-limittester.jpg?fit=max\&auto=format\&n=_-d-o-x8WFPLT_DD\&q=85\&s=ecdb09cc39087c3376755dd0f8c83044)

## 2) Create a ResourceQuota manifest

ResourceQuota is in the core API group (`apiVersion: v1`). The example below enforces a namespace-wide cap on the sum of all pod/container memory *requests* in the `limittester` namespace.

```yaml theme={null}
apiVersion: v1
kind: ResourceQuota
metadata:
  name: memlimit
  namespace: limittester
spec:
  hard:
    requests.memory: 512Mi
```

This manifest ensures the total `requests.memory` across the namespace cannot exceed `512Mi`.

Apply the manifest:

```bash theme={null}
oc apply -f limit.yaml
# example output:
# resourcequota/memlimit created
```

Verify the quota exists:

```bash theme={null}
oc get resourcequota -n limittester
```

Example output:

```text theme={null}
PS C:\Users\mike\Desktop> oc get resourcequota -n limittester
NAME       AGE   REQUEST                 LIMIT
memlimit   35s  requests.memory: 0/512Mi
```

The `requests.memory` column shows current usage (0 of 512Mi in this example).

ResourceQuota tracks resource *requests* and/or *limits* depending on which keys you set. In this example, we restrict `requests.memory`, so only memory requests are counted toward the quota.

## Quick reference: ResourceQuota keys

| Field | Description | Example |
| - | - | - |
| `requests.memory` | Sum of all container memory requests in the namespace | `512Mi` |
| `limits.cpu` | Sum of all container CPU limits in the namespace | `2` |
| `pods` | Maximum number of pods allowed in the namespace | `10` |

For full available keys, see the official Kubernetes `ResourceQuota` documentation (link below).

## 3) Test the quota with a Deployment

Create a Deployment that sets a `resources.requests.memory` value. ResourceQuota counts the `requests` field, not `limits` (unless you also set a quota for `limits.memory`).

```yaml theme={null}
apiVersion: apps/v1
kind: Deployment
metadata:
  name: carts-db
  namespace: limittester
  labels:
    name: carts-db
spec:
  replicas: 1
  selector:
    matchLabels:
      name: carts-db
  template:
    metadata:
      labels:
        name: carts-db
    spec:
      containers:
      - name: carts-db
        image: centos/mongodb-34-centos7
        resources:
          requests:
            memory: "100Mi"
        ports:
        - name: mongo
          containerPort: 27017
```

Apply the Deployment:

```bash theme={null}
oc apply -f deployment.yaml
# you may see a PodSecurity warning; the deployment will still be created
```

If you see PodSecurity warnings when applying the deployment, the resource may still be created. However, a strict PodSecurity admission policy or other admission controllers could block pod creation—verify pods are running with `oc get pods -n limittester`.

Check the Deployment and pods:

```bash theme={null}
oc get deployments -n limittester
oc get pods -n limittester
```

Re-check the ResourceQuota usage after the Deployment is running:

```bash theme={null}
oc get resourcequota -n limittester
```

Example output showing quota consumption:

```text theme={null}
PS C:\Users\mike\Desktop> oc get resourcequota -n limittester
NAME       AGE     REQUEST                       LIMIT
memlimit   3m14s   requests.memory: 100Mi/512Mi
PS C:\Users\mike\Desktop>
```

Now `requests.memory` shows `100Mi/512Mi` because the Deployment requests 100Mi. If you create additional pods or increase requests, the quota usage will update. The API will refuse creation of resources that would cause the namespace to exceed the configured `hard` limits.

## Links and references

* OpenShift Documentation: [https://docs.openshift.com/](https://docs.openshift.com/)
* Kubernetes ResourceQuota: [https://kubernetes.io/docs/concepts/policy/resource-quotas/](https://kubernetes.io/docs/concepts/policy/resource-quotas/)
* oc CLI reference: [https://docs.openshift.com/container-platform/latest/cli\_reference/openshift\_cli/getting-started-cli.html](https://docs.openshift.com/container-platform/latest/cli_reference/openshift_cli/getting-started-cli.html)
