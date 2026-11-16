# Persistent Storage Static vs Dynamic Provisioning StorageClasses

> Explains OpenShift StorageClasses, static versus dynamic provisioning, selecting provisioners, reclaim and binding policies, creating StorageClasses and PVCs, and troubleshooting persistent volume allocation.

Before jumping into the hands‑on steps, here are the official OpenShift references that explain dynamic provisioning and StorageClass resources. StorageClasses let you describe different classes of storage in your cluster and instruct provisioners (usually CSI drivers) how to create PersistentVolumes (PVs) on demand.

You can review OpenShift's documentation on dynamic provisioning here: [https://docs.openshift.com/container-platform/4.8/storage/dynamic\_provisioning.html](https://docs.openshift.com/container-platform/4.8/storage/dynamic_provisioning.html)

![A screenshot of the Red Hat OpenShift documentation webpage showing the "Dynamic provisioning" article, with a left navigation menu and a table of contents on the right. The page explains StorageClass objects and dynamic storage provisioning in OpenShift.](https://mintcdn.com/kodekloud-c4ac6d9a/_-d-o-x8WFPLT_DD/images/OpenShift-4/Storage-Templates-and-Catalog/Persistent-Storage-Static-vs-Dynamic-Provisioning-StorageClasses/openshift-dynamic-provisioning-storageclass-docs.jpg?fit=max\&auto=format\&n=_-d-o-x8WFPLT_DD\&q=85\&s=668c5dd9bc6829b3d1331299f1ad2ee9)

If you scroll that documentation page, you’ll find a list of supported dynamic provisioning plug-ins. These map to Container Storage Interface (CSI) drivers and in-tree provisioners that OpenShift supports. Common examples include OpenStack Cinder, AWS EBS, Azure Disk/File, Google Persistent Disk, and vSphere for VMware environments.

This means you should choose a provisioner that matches your infrastructure: use the Azure provisioner for Azure clusters, AWS/EBS for EKS/OpenShift on AWS, vSphere for VMware, etc.

Step-by-step: create a StorageClass in the OpenShift web console

* From the console, navigate to Storage → StorageClasses.
* If you want a clean demo view, remove any existing StorageClasses you don’t need.
* Click Create StorageClass and provide a name (for example, `localstorageclass`) and an optional description.

Important fields to choose when creating a StorageClass:

* Reclaim policy
  * `Retain` — the PV and its underlying data are preserved after the PVC is deleted.
  * `Delete` — the underlying storage asset is deleted when the PV is released.

* Volume binding mode
  * `Immediate` — provision the volume as soon as the PVC is created.
  * `WaitForFirstConsumer` — delay provisioning until the PVC is scheduled to a Pod. This lets the scheduler pick a node with matching topology or available local storage.

Use `WaitForFirstConsumer` when your storage has node- or zone-local constraints (for example, local PVs or zonal cloud disks). This ensures volumes are provisioned on nodes where the Pods can actually run.

* Provisioner — select the CSI or in-tree provisioner that corresponds to your platform. The console lists many built-in and CSI options (AWS, GCP, Azure, Cinder, vSphere, etc.).

![Screenshot of the Red Hat OpenShift web console on the Storage > StorageClasses page, showing a "Provisioner" dropdown with various kubernetes.io provisioner options (gce-pd, glusterfs, cinder, azure, vsphere, etc.). The left navigation menu highlights Storage and StorageClasses.](https://mintcdn.com/kodekloud-c4ac6d9a/_-d-o-x8WFPLT_DD/images/OpenShift-4/Storage-Templates-and-Catalog/Persistent-Storage-Static-vs-Dynamic-Provisioning-StorageClasses/openshift-storageclasses-provisioner-dropdown.jpg?fit=max\&auto=format\&n=_-d-o-x8WFPLT_DD\&q=85\&s=a2dfafa196fe4204a1d826aedf717912)

For this demonstration we selected “No Provisioner” (i.e., create a StorageClass without a dynamic provisioner). That configuration requires either:

* creating matching PersistentVolumes (static provisioning), or
* installing a CSI driver that implements dynamic provisioning for that StorageClass.

You can inspect the StorageClasses you created in the console or via the CLI. Useful oc commands:

```bash theme={null}
# List all StorageClasses
oc get storageclass

# Short alias
oc get sc

# Describe a specific StorageClass
oc describe sc localstorageclass
```

Create a PersistentVolumeClaim (PVC) that uses the StorageClass

* In the console go to Storage → PersistentVolumeClaims and click Create Persistent Volume Claim.
* Select your StorageClass (e.g., `localstorageclass`), give the claim a name (for example, `localstorageclass-claim`), choose an access mode (commonly `ReadWriteOnce` for block devices attached to one node), set the requested size (for example, 10 GiB), and select the volume mode (`Filesystem` is typical).

![A screenshot of the Red Hat OpenShift web console on the PersistentVolumeClaims page showing one PVC (crc-image-registry-storage) bound to PV pv0001 with 100 GiB capacity. The left navigation highlights Storage and the top bar shows the kubeadmin user.](https://mintcdn.com/kodekloud-c4ac6d9a/_-d-o-x8WFPLT_DD/images/OpenShift-4/Storage-Templates-and-Catalog/Persistent-Storage-Static-vs-Dynamic-Provisioning-StorageClasses/openshift-pvc-crc-image-registry-pv0001.jpg?fit=max\&auto=format\&n=_-d-o-x8WFPLT_DD\&q=85\&s=a284e57d0f906c7e4ea2cf9789663ab9)

Here’s the Create PersistentVolumeClaim form in the console:

![A screenshot of the Red Hat OpenShift web console displaying the "Create PersistentVolumeClaim" form in the "default" project, with fields for claim name, access mode, size, and volume mode. The left sidebar shows navigation items like Storage → PersistentVolumeClaims and other console menus.](https://mintcdn.com/kodekloud-c4ac6d9a/_-d-o-x8WFPLT_DD/images/OpenShift-4/Storage-Templates-and-Catalog/Persistent-Storage-Static-vs-Dynamic-Provisioning-StorageClasses/openshift-create-pvc-form-default.jpg?fit=max\&auto=format\&n=_-d-o-x8WFPLT_DD\&q=85\&s=7d737afd84d5adec2b32f052671728e5)

After clicking Create:

* If a matching PV is available (or the StorageClass has a working provisioner), the PVC will move to `Bound`.
* If no PV is available and there is no provisioner (or the provisioner cannot satisfy the request), the PVC will remain `Pending`.

![Screenshot of the Red Hat OpenShift web console showing a PersistentVolumeClaim named "localstorageclass-claim" in the default project. The PVC is marked Pending and shows details like requested capacity (10 GiB) and storage class "localstorageclass."](https://mintcdn.com/kodekloud-c4ac6d9a/_-d-o-x8WFPLT_DD/images/OpenShift-4/Storage-Templates-and-Catalog/Persistent-Storage-Static-vs-Dynamic-Provisioning-StorageClasses/openshift-pvc-pending-localstorageclass-claim-default.jpg?fit=max\&auto=format\&n=_-d-o-x8WFPLT_DD\&q=85\&s=045b8360fe4134e062504d079468d8a2)

Quick mapping: environment → recommended provisioner

| Environment | Recommended provisioner examples |
| - | - |
| AWS | `kubernetes.io/aws-ebs` or appropriate CSI driver |
| Azure | `kubernetes.io/azure-disk`, `kubernetes.io/azure-file` or Azure CSI |
| GCP | `kubernetes.io/gce-pd` or GCE CSI |
| VMware | `kubernetes.io/vsphere-volume` or vSphere CSI |
| OpenStack | `kubernetes.io/cinder` |

If you select “No Provisioner” for a StorageClass, the cluster will not dynamically create PVs for PVCs using that class. You must either create matching PVs manually (static provisioning) or deploy a compatible CSI driver that provides dynamic provisioning.

Summary and best practices

* StorageClass defines how volumes are provisioned and what provisioner to use.
* Reclaim policy (`Retain` vs `Delete`) controls whether underlying storage is preserved after PVC deletion.
* Volume binding mode (`Immediate` vs `WaitForFirstConsumer`) determines when and where volumes are created — use `WaitForFirstConsumer` for topology-aware or local storage.
* Always choose a provisioner that matches your underlying infrastructure or install a CSI driver for your storage backend.
* Use `oc get sc`, `oc describe sc <name>`, `oc get pvc`, and `oc get pv` to troubleshoot claims and volumes.

References and further reading

* OpenShift dynamic provisioning: [https://docs.openshift.com/container-platform/4.8/storage/dynamic\_provisioning.html](https://docs.openshift.com/container-platform/4.8/storage/dynamic_provisioning.html)
* Kubernetes Container Storage Interface (CSI): [https://kubernetes.io/docs/concepts/storage/container-storage-interface/](https://kubernetes.io/docs/concepts/storage/container-storage-interface/)
* OpenShift storage concepts: [https://docs.openshift.com/container-platform/latest/storage/understanding-persistent-storage.html](https://docs.openshift.com/container-platform/latest/storage/understanding-persistent-storage.html)
