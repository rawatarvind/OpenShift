# Demo Dynamic Provisioning

> Explains Kubernetes and OpenShift dynamic storage provisioning using CSI drivers, StorageClasses, PersistentVolumes, and PersistentVolumeClaims with an Azure Files example.

We'll step back and look at how storage works from a Kubernetes perspective (the same concepts apply in OpenShift). Kubernetes uses a plugin architecture called the Container Storage Interface (CSI) to allow the cluster to talk to external storage providers such as Azure Storage (Disks/Files), AWS (EBS/EFS), NetApp, and others. As long as a vendor provides a CSI driver, Kubernetes can provision and attach storage through that driver.

The CSI spec: [https://kubernetes.io/docs/concepts/storage/container-storage-interface/](https://kubernetes.io/docs/concepts/storage/container-storage-interface/)

![The image shows logos for AWS S3, NetApp and Microsoft Azure on the left. A double-headed arrow connects them to an orange abstract company/service logo on the right.](https://mintcdn.com/kodekloud-c4ac6d9a/_-d-o-x8WFPLT_DD/images/OpenShift-4/Storage-Templates-and-Catalog/Demo-Dynamic-Provisioning/aws-netapp-azure-bidirectional-orange-logo.jpg?fit=max\&auto=format\&n=_-d-o-x8WFPLT_DD\&q=85\&s=aa7a6cce237926bdf26bd98f560b782b)

How it works conceptually

* The CSI driver is the plugin that actually talks to the external storage backend.
* A StorageClass is a Kubernetes object that describes provisioning behavior and references the CSI provisioner.
* PersistentVolumes (PVs) represent actual storage resources. PVs may be created statically by an administrator or dynamically by a StorageClass/provisioner.
* PersistentVolumeClaims (PVCs) are requests for storage from pods. A PVC specifies size, access modes, and optionally a StorageClass. When dynamic provisioning is enabled, the StorageClass/provisioner creates a PV that satisfies the PVC.

Quick provisioning paths

* Dynamic provisioning: PVC → StorageClass → CSI provisioner → provision storage → create PV → bind to PVC.
* Static provisioning: Admin creates PV → PVC binds to existing PV.

The term "persistent storage" in OpenShift maps directly to Kubernetes PersistentVolumes. The StorageClass, PersistentVolume, and PersistentVolumeClaim objects behave the same in both platforms.

Kubernetes storage objects (at-a-glance)

| Resource Type | Purpose | Example / Command |
| - | - | - |
| StorageClass | Describes how to provision storage and references a CSI provisioner | See `storage.k8s.io/v1` StorageClass in the YAML below |
| PersistentVolume (PV) | A piece of storage in the cluster (static or dynamically created) | Admin-created PV or provisioned by StorageClass |
| PersistentVolumeClaim (PVC) | Request for storage by a pod | `kubectl apply -f pvc.yaml` |
| Pod / Deployment | Consumes a PVC and mounts the volume into containers | `volumeMounts` + `volumes` in Pod/Deployment spec |

Example: StorageClass + PVC + Deployment using Azure Files CSI

* The StorageClass references the Azure Files CSI provisioner (`file.csi.azure.com`) and sets mount options and parameters (for example, SKU).
* The PVC requests storage from that StorageClass.
* The Deployment mounts the resulting volume into nginx containers.

```yaml theme={null}
# StorageClass using the Azure Files CSI driver
kind: StorageClass
apiVersion: storage.k8s.io/v1
metadata:
  name: azurefile-csi
provisioner: file.csi.azure.com
allowVolumeExpansion: true
mountOptions:
  - dir_mode=0777
  - file_mode=0777
  - uid=0
  - gid=0
  - mfsymlinks
  - cache=strict
  - actimeo=30
parameters:
  skuName: Premium_LRS
---
# PersistentVolumeClaim that requests 10Gi from the azurefile-csi StorageClass
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: azurefile
spec:
  accessModes:
    - ReadWriteMany
  storageClassName: azurefile-csi
  resources:
    requests:
      storage: 10Gi
---
# Deployment that mounts the PVC into containers
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginxdeployment
spec:
  selector:
    matchLabels:
      app: nginxdeployment
  replicas: 2
  template:
    metadata:
      labels:
        app: nginxdeployment
    spec:
      containers:
      - name: nginx
        image: nginx:latest
        volumeMounts:
        - name: azure
          mountPath: /mnt/azure
      volumes:
      - name: azure
        persistentVolumeClaim:
          claimName: azurefile
```

Manifest explanation (component-by-component)

* StorageClass (`azurefile-csi`)
  * `provisioner: file.csi.azure.com` points to the Azure Files CSI driver.
  * `allowVolumeExpansion: true` enables resizing the volume after provisioning.
  * `mountOptions` and `parameters` such as `skuName: Premium_LRS` control how the Azure Files share is created and mounted.

* PersistentVolumeClaim (`azurefile`)
  * Requests `10Gi` with `ReadWriteMany` access and explicitly references the `azurefile-csi` StorageClass.
  * When the PVC is created, the StorageClass/provisioner dynamically provisions a backing PV and binds it to the PVC.

* Deployment (`nginxdeployment`)
  * Mounts the bound PVC at `/mnt/azure` in each nginx pod using `volumeMounts` and `volumes`.
  * The `claimName` must match the PVC name.

Resources and further reading

* Kubernetes Container Storage Interface (CSI): [https://kubernetes.io/docs/concepts/storage/container-storage-interface/](https://kubernetes.io/docs/concepts/storage/container-storage-interface/)
* StorageClass docs: [https://kubernetes.io/docs/concepts/storage/storage-classes/](https://kubernetes.io/docs/concepts/storage/storage-classes/)
* PersistentVolume and PersistentVolumeClaim: [https://kubernetes.io/docs/concepts/storage/persistent-volumes/](https://kubernetes.io/docs/concepts/storage/persistent-volumes/)

This dynamic provisioning flow is identical in OpenShift: the cluster uses a StorageClass to talk to a CSI provisioner, which provisions storage and returns a PersistentVolume that satisfies the PersistentVolumeClaim — your pods then mount that volume.

