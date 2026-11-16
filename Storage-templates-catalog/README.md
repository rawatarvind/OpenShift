# Storage

> Guide to provisioning and using persistent storage in OpenShift, explaining PersistentVolumes, PersistentVolumeClaims, StorageClasses, access modes, and mounting storage into pods.

Hello and welcome to this article on OpenShift.

This guide explains how persistent storage works in OpenShift, how it is provisioned, and how applications consume it.

Containers are designed to be ephemeral: they are created to process data and are removed when that processing completes. By default, data written inside a container is lost when the container is destroyed.

To persist data beyond a container's lifetime, attach persistent volumes to containers at creation time. Data written to those volumes is retained independently of the container lifecycle.

OpenShift leverages the Kubernetes Persistent Volume (PV) framework to provide cluster storage. Storage can be provisioned by many plugins and providers, including Local, iSCSI, Fibre Channel, NFS, GlusterFS, Ceph RBD, OpenStack Cinder, AWS Elastic Block Store, Azure Disk/File, and VMware vSphere.

![A presentation slide titled "Plugins" showing a bullet list of storage plugins (Local, iSCSI, Fibre Channel, NFS, GlusterFS, Ceph RBD, OpenStack Cinder, AWS EBS, GCE Persistent Disk, Azure Disk/File, VMWare vSphere). To the right are red 3D block and cylinder icons representing infrastructure and storage components.](https://mintcdn.com/kodekloud-c4ac6d9a/_-d-o-x8WFPLT_DD/images/OpenShift-4/Storage-Templates-and-Catalog/Storage/storage-plugins-slide-infrastructure-icons.jpg?fit=max\&auto=format\&n=_-d-o-x8WFPLT_DD\&q=85\&s=613caa815cdd2835c8a1054af69d1dd1)

Storage exposed to the OpenShift cluster appears as a pool of PersistentVolume (PV) resources. Projects (namespaces) request storage from that pool by creating PersistentVolumeClaims (PVCs). When a PVC is bound to a PV, the claim can be mounted into pods so containers can read and write persistent data.

![A stylized red 3D diagram showing Kubernetes persistent volumes: large rectangular PV blocks with floating red cube pointers labeled PVC — 10GB, PVC — 100GB, and PVC — 20GB. Stacked cylinder storage icons and the label "PV Resources — 1TB" appear below.](https://mintcdn.com/kodekloud-c4ac6d9a/_-d-o-x8WFPLT_DD/images/OpenShift-4/Storage-Templates-and-Catalog/Storage/kubernetes-pv-pvc-1tb-3d-red.jpg?fit=max\&auto=format\&n=_-d-o-x8WFPLT_DD\&q=85\&s=69df9b3254af2bd6c7c668544585e9ac)

From the OpenShift web console you can create storage using the Create Storage button, provide a name for the claim, choose an access mode, and specify the size of the claim. Alternatively, create PVCs via YAML and apply them with the CLI (`oc apply -f pvc.yaml`).

Access modes define how a PV can be mounted into nodes and pods:

| Access Mode | What it means | Typical storage types |
| - | - | - |
| ReadWriteOnce (RWO) | Volume can be mounted read-write by a single node at a time. | Block-backed volumes such as AWS EBS, Azure Disk |
| ReadWriteMany (RWX) | Volume can be mounted read-write by multiple nodes (shared). | NFS, distributed file systems (GlusterFS, CephFS) |
| ReadOnlyMany (ROX) | Volume can be mounted read-only by multiple nodes. | Shared file systems for read-only access |

Select the appropriate access mode and size, then create the PersistentVolumeClaim. The web UI exposes these options in a simple form:

![A screenshot of a "Create Storage" web form with fields for Name, Access Mode (Single User, Shared Access, Read Only) and Size. To the right is a red icon showing three users connecting to a shared storage/database cylinder.](https://mintcdn.com/kodekloud-c4ac6d9a/_-d-o-x8WFPLT_DD/images/OpenShift-4/Storage-Templates-and-Catalog/Storage/create-storage-form-shared-access-icon.jpg?fit=max\&auto=format\&n=_-d-o-x8WFPLT_DD\&q=85\&s=9df9acdf374a208e9c6e73f6784e5904)

Once a PVC exists and is bound, attach it to a pod by adding a volume that references the claim in your DeploymentConfig (OpenShift) or Deployment/Pod manifest and mounting it into the container. The minimal example below shows:

* A PVC requesting 5 Gi of storage.
* A DeploymentConfig that mounts the PVC at `/data` inside the container.

```yaml theme={null}
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: my-pvc
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 5Gi
  storageClassName: standard
```

```yaml theme={null}
apiVersion: apps.openshift.io/v1
kind: DeploymentConfig
metadata:
  name: my-app
spec:
  replicas: 1
  selector:
    app: my-app
  template:
    metadata:
      labels:
        app: my-app
    spec:
      containers:
        - name: my-app
          image: nginx:1.21
          volumeMounts:
            - name: app-storage
              mountPath: /data
      volumes:
        - name: app-storage
          persistentVolumeClaim:
            claimName: my-pvc
  strategy:
    type: Rolling
```

Note that the `storageClassName` in your PVC controls which StorageClass (and thus which provisioner) satisfies the claim. Many OpenShift clusters support dynamic provisioning: PVCs are automatically bound to dynamically created PVs using the selected StorageClass.

Access modes are an important constraint when designing storage for your application. Use RWO for single-node writable volumes (common with block storage), RWX when multiple nodes must write concurrently (NFS or distributed filesystems), and ROX when read-only sharing is sufficient.

Be careful with deletion semantics: deleting a PVC can also delete the underlying PV depending on the StorageClass reclaim policy (e.g., `Delete` vs `Retain`). Verify reclaim policies before removing claims to avoid accidental data loss.

Common CLI and troubleshooting commands:

* Create or update resources: `oc apply -f <file.yaml>`
* List claims and volumes: `oc get pvc`, `oc get pv`
* Inspect a claim or volume: `oc describe pvc <name>`, `oc describe pv <name>`
* Check StorageClasses: `oc get storageclass`, `oc describe storageclass <name>`

Further reading and references:

* [OpenShift Storage Concepts](https://docs.openshift.com/)
* [Kubernetes Persistent Volumes](https://kubernetes.io/docs/concepts/storage/persistent-volumes/)
* [StorageClass documentation](https://kubernetes.io/docs/concepts/storage/storage-classes/)

This demonstrates how persistent storage is provisioned and consumed in OpenShift. You can use the web console or the CLI to create PVCs and then reference those claims in DeploymentConfig, Deployment, or Pod manifests to mount persistent storage into your containers.

