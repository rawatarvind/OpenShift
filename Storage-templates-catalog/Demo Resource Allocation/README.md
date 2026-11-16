# Demo Resource Allocation

> Adding CPU and memory requests and limits to a carts-db Kubernetes Deployment with example manifest and OpenShift application steps

In this lesson you'll add resource requests and limits to the `carts-db` Deployment (part of the Sock Shop demo) so the container has guaranteed CPU/memory and a capped memory usage. Below is a complete Deployment manifest that shows where to add `resources.requests` and `resources.limits`, along with the container image, ports, environment variables, and an in-memory `emptyDir` volume.

```yaml theme={null}
apiVersion: apps/v1
kind: Deployment
metadata:
  name: carts-db
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
      nodeSelector:
        beta.kubernetes.io/os: linux
      containers:
        - name: carts-db
          image: centos/mongodb-34-centos7
          resources:
            requests:
              memory: "64Mi"
              cpu: "250m"
            limits:
              memory: "128Mi"
          ports:
            - name: mongo
              containerPort: 27017
          env:
            - name: MONGODB_USER
              value: sock-user
            - name: MONGODB_PASSWORD
              value: password
            - name: MONGODB_DATABASE
              value: data
            - name: MONGODB_ADMIN_PASSWORD
              value: admin
          volumeMounts:
            - mountPath: /tmp
              name: tmp-volume
      volumes:
        - name: tmp-volume
          emptyDir:
            medium: Memory
```

What this manifest configures:

* Requests: `cpu: "250m"`, `memory: "64Mi"` — guaranteed resources used by the scheduler when placing the Pod.
* Limits: `memory: "128Mi"` — maximum memory the container may use. (No CPU limit specified here, so the container can burst CPU beyond `250m` if node capacity allows.)

Summary table

| Setting | Purpose | Example |
| -: | - | - |
| Request (cpu) | Scheduling guarantee for CPU | `250m` |
| Request (memory) | Scheduling guarantee for memory | `64Mi` |
| Limit (memory) | Maximum memory the container can use | `128Mi` |

How to apply this in the OpenShift web console

1. Open the OpenShift console in your course environment: [https://learn.kodekloud.com/user/courses/openshift-4](https://learn.kodekloud.com/user/courses/openshift-4).
2. In the Administrator view navigate to Workloads → Deployments.
3. Click Create Deployment and switch to the YAML editor.
4. Paste the manifest above and create the Deployment.

![A Red Hat OpenShift web console screenshot showing the "Deployments" page for the "default" project with the message "No Deployments found" and a blue "Create Deployment" button. The left-hand navigation menu and administrator header are also visible.](https://mintcdn.com/kodekloud-c4ac6d9a/_-d-o-x8WFPLT_DD/images/OpenShift-4/Storage-Templates-and-Catalog/Demo-Resource-Allocation/openshift-console-deployments-default-empty.jpg?fit=max\&auto=format\&n=_-d-o-x8WFPLT_DD\&q=85\&s=2c964fd3c8ba6febc5a2f277c5d073a2)

After creating the Deployment, OpenShift will create the Pod for it. Wait a moment — the Deployments page should show the Deployment with one running pod:

```text theme={null}
Status: 1 pod running
```

![A screenshot of the Red Hat OpenShift web console showing the "carts-db" deployment details in the default project. A circular status chart shows one running pod for that deployment.](https://mintcdn.com/kodekloud-c4ac6d9a/_-d-o-x8WFPLT_DD/images/OpenShift-4/Storage-Templates-and-Catalog/Demo-Resource-Allocation/openshift-carts-db-1-running-pod.jpg?fit=max\&auto=format\&n=_-d-o-x8WFPLT_DD\&q=85\&s=89596b9d14edda74fefb13d88c850e28)

Inspect the Pod to confirm it's Running and to view the applied resource settings. From the Pod details page you can open the Pod YAML and confirm the `resources` block is present.

![A screenshot of the Red Hat OpenShift web console showing the Pod details page for "carts-db-dff84c9cd-cnhsv" in the default project with status "Running." The left sidebar shows the Administrator menu with Workloads → Pods selected.](https://mintcdn.com/kodekloud-c4ac6d9a/_-d-o-x8WFPLT_DD/images/OpenShift-4/Storage-Templates-and-Catalog/Demo-Resource-Allocation/openshift-pod-cartsdb-dff84c9cdcnhsv-default-running.jpg?fit=max\&auto=format\&n=_-d-o-x8WFPLT_DD\&q=85\&s=ea46c5ea7ab96136901e976d43eae273)

Example excerpt from the PodSpec (what you should see in the Pod YAML):

```yaml theme={null}
containers:
  - name: carts-db
    resources:
      limits:
        memory: 128Mi
      requests:
        cpu: 250m
        memory: 64Mi
```

Why use requests and limits?

* Requests ensure the scheduler reserves node capacity for your Pod.
* Limits prevent containers from consuming excessive resources and impacting other workloads.
* Omitting CPU limits allows CPU bursting, but scheduling still uses the request value.

For cluster-wide enforcement, use ResourceQuotas (Kubernetes) or project quotas (OpenShift) to enforce aggregate limits per namespace so you don't have to set requests/limits on every single Deployment manually. See:

* Kubernetes ResourceQuotas: [https://kubernetes.io/docs/concepts/policy/resource-quotas/](https://kubernetes.io/docs/concepts/policy/resource-quotas/)
* OpenShift documentation: [https://docs.openshift.com/](https://docs.openshift.com/)

Resource requests reserve capacity on a node and influence scheduling; limits cap the maximum usage. Setting sensible requests and limits helps cluster stability and prevents noisy neighbors. If you omit a CPU limit, the container may burst CPU usage (subject to the node's available CPU) but the scheduler will use the request value when placing the Pod.
