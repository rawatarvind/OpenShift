# Demo SCC Overview

> Guide to hardening a Kubernetes/OpenShift Deployment by configuring container securityContext, enforcing readOnlyRootFilesystem, running nonroot, mounting writable volumes and verifying with oc commands

We're using the CartDB deployment shown below as the starting point for this guide. This minimal Deployment manifest shows the typical default state where no explicit security context is configured for the container:

```yaml theme={null}
apiVersion: apps/v1
kind: Deployment
metadata:
  name: carts-db
  namespace: default
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
```

By default, many container images run without a defined `securityContext`. That can result in containers running with broader kernel capabilities or as the root user. If a container does not require elevated privileges or root access, it's safer to explicitly restrict those capabilities.

A recommended hardening step is to set container-level `securityContext` fields and make the container root filesystem read-only. A `readOnlyRootFilesystem: true` reduces the blast radius of a compromise because an attacker cannot persist files to the root filesystem. When enabling read-only root filesystem, you must also mount any directories your application needs to write to (for example `/tmp`) as writable volumes.

Below is a consolidated Deployment manifest that includes node selection, environment variables, resource requests, a memory-backed `emptyDir` for `/tmp`, and explicit container security settings (non-privileged, read-only root filesystem, no privilege escalation).

```yaml theme={null}
apiVersion: apps/v1
kind: Deployment
metadata:
  name: carts-db
  namespace: default
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
        kubernetes.io/os: linux
      volumes:
        - name: tmp-volume
          emptyDir:
            medium: Memory
      containers:
        - name: carts-db
          image: centos/mongodb-34-centos7
          ports:
            - name: mongo
              containerPort: 27017
          resources:
            requests:
              memory: "100Mi"
          env:
            - name: MONGODB_USER
              value: sock-user
            - name: MONGODB_PASSWORD
              valueFrom:
                secretKeyRef:
                  name: dbpassword
                  key: MONGODB_PASSWORD
            - name: MONGODB_DATABASE
              value: data
            - name: MONGODB_ADMIN_PASSWORD
              value: admin
          volumeMounts:
            - mountPath: /tmp
              name: tmp-volume
          securityContext:
            # Ensure the container does not run privileged
            privileged: false
            # Make the container root filesystem read-only
            readOnlyRootFilesystem: true
            # Additional recommended flags
            runAsNonRoot: true
            allowPrivilegeEscalation: false
```

Save this manifest (for example as `deployment.yaml`) and apply it to your OpenShift cluster:

```bash theme={null}
oc apply -f deployment.yaml
```

Verify the Deployment and the Pods created by it:

```bash theme={null}
# Check the deployment status
oc get deployment carts-db

# Check pods created by the deployment
oc get pods -l name=carts-db

# Describe one of the pods to confirm securityContext settings
oc describe pod -l name=carts-db
```

You can also inspect the CartDB Deployment and its Pod in the OpenShift web console by navigating to Workloads → Deployments and selecting the carts-db Deployment. See the OpenShift web console docs for details: [https://docs.openshift.com/container-platform/latest/web\_console/index.html](https://docs.openshift.com/container-platform/latest/web_console/index.html)

Best practice: When enabling `readOnlyRootFilesystem: true`, explicitly mount any directories the application needs to write to (for example `/tmp`) as writable volumes. Otherwise the container may fail at runtime due to write failures.

Be aware of two common pitfalls with these settings:

* `runAsNonRoot: true` will cause the pod to fail to start if the container image expects to run as root. If the image runs as root by default, either remove this flag or set an explicit non-root UID with `runAsUser` (and ensure the image supports that UID).

* `readOnlyRootFilesystem` plus only an ephemeral memory-backed `emptyDir` at `/tmp` is not sufficient for stateful services like MongoDB. The MongoDB image typically requires a writable data directory (for example `/var/lib/mongodb` or the image-specific data path) which should be backed by a `PersistentVolumeClaim` for durability. Also note that `emptyDir` with `medium: Memory` is ephemeral (tmpfs) and will be lost when the pod restarts.

SecurityContext quick reference

| Field | Purpose | Recommended for carts-db example |
| - | - | - |
| `privileged` | Grants full device and kernel access. | `false` — do not run privileged unless absolutely necessary. |
| `readOnlyRootFilesystem` | Prevents writes to the container root filesystem. | `true` — reduces persistence of compromises; mount writable paths explicitly. |
| `runAsNonRoot` | Ensures container does not run as UID 0. | `true` — use if image supports non-root UID, otherwise set `runAsUser`. |
| `allowPrivilegeEscalation` | Controls if a process can gain more privileges (e.g., via setuid). | `false` — restricts privilege escalation. |

Further reading and references

* [Kubernetes Pods and Containers](https://kubernetes.io/docs/concepts/workloads/pods/)
* [Kubernetes Pod SecurityContext](https://kubernetes.io/docs/tasks/configure-pod-container/security-context/)
* [OpenShift Web Console docs](https://docs.openshift.com/container-platform/latest/web_console/index.html)
* [Best practices for running containers securely](https://kubernetes.io/docs/concepts/security/pod-security-standards/)

This manifest and approach help enforce least privilege and minimize attack surface in an OpenShift/Kubernetes deployment while keeping necessary writable paths available to the application.

