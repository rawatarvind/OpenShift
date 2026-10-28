# Demo DeploymentConfigDeployment

> Compare Kubernetes Deployment and OpenShift DeploymentConfig by deploying identical CartsDB workloads to highlight controller behavior, console differences, and OpenShift specific features like imageChange triggers

In this hands-on lesson we deploy the same CartsDB workload twice in OpenShift so you can compare behavior and console representation:

1. As a standard Kubernetes Deployment (`apps/v1`).
2. As an OpenShift `DeploymentConfig` (`apps.openshift.io/v1`).

Both manifests use the `centos/mongodb-34-centos7` image and identical container config so differences you observe come from the controller types and OpenShift-specific features.

***

## 1) Deploy CartsDB as a Kubernetes Deployment

Create a new Deployment in the OpenShift console (or with `oc apply -f`) using the manifest below.

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
      containers:
      - name: carts-db
        image: centos/mongodb-34-centos7
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
      nodeSelector:
        beta.kubernetes.io/os: linux
```

This manifest:

* Runs one replica of the MongoDB container and exposes port 27017.
* Provides environment variables for credentials and the database name.
* Mounts an in-memory `emptyDir` at `/tmp`.
* Pins the Pod to Linux nodes with `nodeSelector`.

Note: Newer clusters prefer the stable node label `kubernetes.io/os: linux`. The example uses the legacy `beta.kubernetes.io/os: linux`, which may be deprecated—use the stable label for portability.

After creating the Deployment:

* OpenShift will create a ReplicaSet that in turn manages the Pod(s).
* The console shows Deployment metadata (name, namespace, labels, update strategy, selector), and provides convenient inspection of Pods, env vars, volumes, and events.

![A screenshot of the Red Hat OpenShift Container Platform web console showing the Environment tab for the "carts-db" container with environment variables (e.g., MONGODB\_USER = "sock-user", MONGODB\_PASSWORD, MONGODB\_DATABASE, etc.). The left sidebar shows Administrator → Workloads → Deployments and a red banner notes the cluster is for development/testing only.](https://mintcdn.com/kodekloud-c4ac6d9a/Xa5GerWBw0vsOVlC/images/OpenShift-4/Concepts-Builds-and-Deployments/Demo-DeploymentConfigDeployment/openshift-carts-db-env-variables-console.jpg?fit=max\&auto=format\&n=Xa5GerWBw0vsOVlC\&q=85\&s=6298b825165f2c9eacbed4aef5d02d46)

What to check in the console after Deployment creation:

* ReplicaSet presence and Pod count.
* Pod events and container logs for startup validation.
* Container environment variables and mounted volumes.

Practical tip:

* The web console is great for inspection and monitoring, but avoid making transient/manual edits directly in the console unless you intend those changes to be permanent. Manifests from source control, CI/CD, or GitOps will typically overwrite manual edits.

***

## 2) Delete the Deployment and create a DeploymentConfig

Next, delete the previous Deployment and create an OpenShift `DeploymentConfig` with the same container configuration. Paste the manifest below into the console’s Create DeploymentConfig flow or use `oc apply -f`:

```yaml theme={null}
apiVersion: apps.openshift.io/v1
kind: DeploymentConfig
metadata:
  name: carts-db
  labels:
    name: carts-db
spec:
  replicas: 1
  template:
    metadata:
      labels:
        name: carts-db
    spec:
      containers:
      - name: carts-db
        image: centos/mongodb-34-centos7
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
      nodeSelector:
        beta.kubernetes.io/os: linux
```

OpenShift will create a ReplicationController and start the Pod for this DeploymentConfig.

![A screenshot of the Red Hat OpenShift Container Platform web console showing the DeploymentConfig details for "carts-db" with the ReplicationControllers tab selected. It lists one replication controller "carts-db-1" with status "1 of 1 pods" and phase "Complete."](https://mintcdn.com/kodekloud-c4ac6d9a/Xa5GerWBw0vsOVlC/images/OpenShift-4/Concepts-Builds-and-Deployments/Demo-DeploymentConfigDeployment/openshift-deploymentconfig-carts-db-rc.jpg?fit=max\&auto=format\&n=Xa5GerWBw0vsOVlC\&q=85\&s=7ccf99df7f316ebc605f4364c16f4356)

***

## Key differences: Deployment vs DeploymentConfig

Below is a concise comparison to help you decide which resource to use.

| Aspect | Kubernetes Deployment (`apps/v1`) | OpenShift DeploymentConfig (`apps.openshift.io/v1`) |
| - | -: | - |
| API / Kind | `apiVersion: apps/v1`, `kind: Deployment` | `apiVersion: apps.openshift.io/v1`, `kind: DeploymentConfig` |
| Controller created | ReplicaSet | ReplicationController |
| Selectors | Requires explicit `spec.selector.matchLabels` matching `spec.template.metadata.labels` | Historically used template labels for selection; ensure labels/selectors are consistent |
| OpenShift-only features | N/A (portable Kubernetes primitive) | Supports imageChange triggers, lifecycle hooks, and build/deploy integrations |
| Portability | Portable across Kubernetes distributions | OpenShift-specific; not portable to vanilla Kubernetes without translation |

Additional notes:

* Deployments are the Kubernetes standard for managing replicated Pods and rolling updates.
* DeploymentConfigs are useful when you require OpenShift integrations (for example, automatic redeploys from internal image builds via imageChange triggers).

***

## Best practices and recommendations

Red Hat recommends using standard Kubernetes Deployments for most workloads because they are portable and part of the Kubernetes API. Use DeploymentConfig only when you need OpenShift-specific features such as `imageChange` triggers or build/deploy hooks.

Operational guidance:

* Manage manifests in source control and deploy via CI/CD or GitOps for consistency and auditability.
* Use the OpenShift console to monitor, inspect events, and debug—but commit desired manifest changes back to source control rather than editing ad-hoc in the console.
* Prefer stable node labels (`kubernetes.io/os: linux`) for portability across clusters.

***

## Summary

* Deployments (`apps/v1`) are the recommended, portable Kubernetes primitive and create ReplicaSets.
* DeploymentConfigs (`apps.openshift.io/v1`) are OpenShift-specific and create ReplicationControllers; they add features integrated with OpenShift builds and image workflows.
* Use the console for visualization and quick debugging; manage definitive configuration through versioned manifests and CI/CD.

## Links and references

* [Kubernetes Deployments (official docs)](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/)
* [OpenShift DeploymentConfig (official docs)](https://docs.openshift.com/container-platform/latest/nodes/deployment/what-deploymentconfigs-are.html)
* [OpenShift Console — Workloads](https://docs.openshift.com/container-platform/latest/applications/deployments/)

