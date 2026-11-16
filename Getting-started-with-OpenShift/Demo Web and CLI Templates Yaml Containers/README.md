# Demo Web and CLI Templates Yaml Containers

> Guide to deploying and configuring Azure Red Hat OpenShift clusters, covering installation options, Azure prerequisites, networking, VM sizing, pull secrets, validation and retrieving credentials

CodeReady Containers is now called OpenShift Local; some older docs or videos may still reference CodeReady Containers or the sandbox. This guide walks through installing a production-ready OpenShift cluster, with a focus on Azure Red Hat OpenShift (ARO). It covers the console flows, install choices, Azure prerequisites, networking, and how to retrieve credentials after deployment.

![Screenshot of the Red Hat Hybrid Cloud Console dashboard. It shows App services and Ansible Automation Platform metrics (Kafka Instances 0, Collections 126, Partners 60) with a left navigation menu for OpenShift, RHEL and other services.](https://mintcdn.com/kodekloud-c4ac6d9a/Xa5GerWBw0vsOVlC/images/OpenShift-4/Getting-Started-with-OpenShift/Demo-Web-and-CLI-Templates-Yaml-Containers/red-hat-hybrid-cloud-console-dashboard.jpg?fit=max\&auto=format\&n=Xa5GerWBw0vsOVlC\&q=85\&s=f822eb35fd404077702e17a578c87fa6)

## Where to start

From console.redhat.com navigate to Manage → Infrastructure → OpenShift. This area exposes multiple installation and management flows:

* Assisted Installer clusters
* Cloud provider integrations (Azure Red Hat OpenShift — ARO, AWS, IBM Cloud)
* Register an existing cluster
* Create cluster flows tailored to different environments

Click “Create cluster” to begin the provisioning wizard.

![A screenshot of the Red Hat Hybrid Cloud Console showing the "Create an OpenShift cluster" page with Active subscriptions and a "Create trial cluster" button. The left sidebar displays navigation items like Clusters, Overview, Releases, and Downloads.](https://mintcdn.com/kodekloud-c4ac6d9a/Xa5GerWBw0vsOVlC/images/OpenShift-4/Getting-Started-with-OpenShift/Demo-Web-and-CLI-Templates-Yaml-Containers/red-hat-hybrid-console-create-openshift.jpg?fit=max\&auto=format\&n=Xa5GerWBw0vsOVlC\&q=85\&s=7f1b02ffa3e7d599f01b20d55979bdf7)

Be careful with trial clusters: OpenShift trials may waive the platform license cost, but cloud providers bill for the infrastructure (VMs, storage, networking). Delete trial clusters promptly to avoid unexpected charges.

## Installation types and managed offerings

Under the Cloud section you can choose between fully managed offerings (recommended if you already use a cloud provider) or self-managed installations on cloud VMs or on-premises. Managed options reduce control plane overhead and simplify integrations.

When selecting a cloud provider you will commonly see two installer types:

* Installer‑provisioned infrastructure (IPI): the installer provisions the required cloud resources (recommended for most cloud-based installs).
* User‑provisioned infrastructure (UPI): you create networking and compute, and the installer uses your resources (more flexible for advanced or constrained environments).

![A screenshot of the Red Hat Hybrid Cloud Console showing the "Install OpenShift Container Platform 4" page where you choose an AWS installation type. Two option cards are visible: "Installer‑provisioned infrastructure" (recommended) and "User‑provisioned infrastructure."](https://mintcdn.com/kodekloud-c4ac6d9a/Xa5GerWBw0vsOVlC/images/OpenShift-4/Getting-Started-with-OpenShift/Demo-Web-and-CLI-Templates-Yaml-Containers/red-hat-openshift-aws-install-options.jpg?fit=max\&auto=format\&n=Xa5GerWBw0vsOVlC\&q=85\&s=9100178789637d92535666d8edaabe8d)

### Quick comparison

| Install type | When to use | Pros | Cons |
| - | - | - | - |
| Installer-provisioned (IPI) | Standard cloud deployments | Minimal manual setup, automated networking and control plane | Less control over low-level cloud resources |
| User-provisioned (UPI) | Advanced/custom networking or on-prem | Full control of infra, suitable for constrained environments | More manual steps, higher ops overhead |

## Choosing a managed service (example: Azure Red Hat OpenShift)

If you prefer a fully managed service (recommended when already using Azure), choose Azure Red Hat OpenShift from the Azure Marketplace and click Create. You’ll select a resource group, region, and cluster name; example values used in this walkthrough:

* Resource group: KodeKloud
* Region: East US
* Cluster name: kodekloudtest

Choose control plane (master) VM sizes and worker VM sizes. The number of worker nodes depends on workload and availability needs — two workers provide basic redundancy for small workloads, while production commonly starts with 3 or more workers. The control plane typically uses three nodes for high availability.

![A screenshot of the Red Hat Hybrid Cloud Console showing the OpenShift "Managed services" page with listings for creating clusters on Azure, IBM Cloud, and AWS. A left navigation menu and buttons to "Try it" or "Create cluster" are also visible.](https://mintcdn.com/kodekloud-c4ac6d9a/Xa5GerWBw0vsOVlC/images/OpenShift-4/Getting-Started-with-OpenShift/Demo-Web-and-CLI-Templates-Yaml-Containers/redhat-hybrid-cloud-openshift-managed-services.jpg?fit=max\&auto=format\&n=Xa5GerWBw0vsOVlC\&q=85\&s=b8d16cc87cf1e0d39c3173e5b2e5e64b)

When picking VM sizes the portal shows families and SKUs. In this walkthrough the DSv3 family was used (example: `D8s_v3` with 8 vCPUs and 32 GiB RAM) for control plane and worker sizing.

![A Microsoft Azure "Select a VM size" screen showing VM options with the D8s\_v3 (General purpose) highlighted and columns for vCPUs (8), RAM (32 GiB), data disks (16), Max IOPS (12800) and temp storage (64 GiB). The dialog also shows the subscription (Mike-Pay-As-You-Go), region (East US) and action buttons like "Select" and "Review + create."](https://mintcdn.com/kodekloud-c4ac6d9a/Xa5GerWBw0vsOVlC/images/OpenShift-4/Getting-Started-with-OpenShift/Demo-Web-and-CLI-Templates-Yaml-Containers/azure-vm-d8s-v3-8vcpu-32gib.jpg?fit=max\&auto=format\&n=Xa5GerWBw0vsOVlC\&q=85\&s=3ac2b469ba3633bb89287992337ba208)

## Azure prerequisites — App Registration and pull secret

ARO requires an Azure AD App Registration (a service principal) with appropriate permissions. Steps:

1. In Azure AD create an App Registration.
2. Create a client secret and copy the Application (client) ID and the secret value.
3. In the ARO create form paste the client ID and secret.

![A screenshot of the Microsoft Azure portal showing the App registrations Overview for an application named "kodekloudopenshift," including the application (client) ID, object ID, tenant ID, and client credential/redirect URI options. The left sidebar displays navigation items like Overview, Quickstart, Authentication, Certificates & secrets, and API permissions.](https://mintcdn.com/kodekloud-c4ac6d9a/Xa5GerWBw0vsOVlC/images/OpenShift-4/Getting-Started-with-OpenShift/Demo-Web-and-CLI-Templates-Yaml-Containers/azure-app-registrations-kodekloudopenshift-overview.jpg?fit=max\&auto=format\&n=Xa5GerWBw0vsOVlC\&q=85\&s=6b2923cdbb3741761afafce716ca7969)

You also need the Red Hat pull secret to allow the cluster to pull Red Hat images and content. Obtain the pull secret from the Red Hat console on the OpenShift install pages (for ARO/Azure choose the ARO-provisioned pull secret path) and paste it into the Azure cluster creation form.

![A browser screenshot of the Azure Red Hat OpenShift console. It shows the "Red Hat content access" page with buttons to download or copy a pull secret and a left-side navigation menu (Clusters, Overview, Releases, etc.).](https://mintcdn.com/kodekloud-c4ac6d9a/Xa5GerWBw0vsOVlC/images/OpenShift-4/Getting-Started-with-OpenShift/Demo-Web-and-CLI-Templates-Yaml-Containers/azure-red-hat-openshift-pull-secret.jpg?fit=max\&auto=format\&n=Xa5GerWBw0vsOVlC\&q=85\&s=0e0c4b7fc61482354d64adb4671c9eef)

## Networking and API/Ingress visibility

The installer creates a virtual network with separate subnets for control plane and worker nodes. You must select:

* API server visibility: Public or Private
* Ingress visibility: Public or Private

Best practice: keep the API server private when possible. Make ingress public only for front-end apps that require external access; choose private ingress for internal applications.

![A screenshot of the Microsoft Azure portal showing the "Create Azure Red Hat OpenShift" Networking tab where virtual network, master and worker subnets are being configured. API server and ingress visibility options are shown with "Public" selected and navigation buttons ("Review + create", "Previous", "Next") at the bottom.](https://mintcdn.com/kodekloud-c4ac6d9a/Xa5GerWBw0vsOVlC/images/OpenShift-4/Getting-Started-with-OpenShift/Demo-Web-and-CLI-Templates-Yaml-Containers/create-azure-red-hat-openshift-networking.jpg?fit=max\&auto=format\&n=Xa5GerWBw0vsOVlC\&q=85\&s=747804d0d3aea15dc103ed14fc103282)

## Common validation issues (quota and permissions)

* If you see an App Registration object ID validation error, add the required API permissions. In Azure AD → App Registration → API permissions, add **Azure Service Management** and grant the `user_impersonation` permission or equivalent for the subscription.
* If Azure reports insufficient vCPU quota (this walkthrough required 36 vCPUs for the selected sizes), submit a support request to increase quota.

To request a quota increase in Azure: Help and Support → New Support Request → Service and subscription limits (quotas) → Compute (VM cores). Provide the region and SKU family (e.g., Standard\_DSv3).

![Screenshot of the Microsoft Azure "New support request" page showing a problem description form for a quota issue. It displays fields like Summary ("quota"), Issue type (service and subscription limits), Subscription, Quota type (Compute-VM cores/vCPUs) and a "Manage quota" button.](https://mintcdn.com/kodekloud-c4ac6d9a/Xa5GerWBw0vsOVlC/images/OpenShift-4/Getting-Started-with-OpenShift/Demo-Web-and-CLI-Templates-Yaml-Containers/azure-support-quota-vm-cores.jpg?fit=max\&auto=format\&n=Xa5GerWBw0vsOVlC\&q=85\&s=0988dbd6f32ff6dafa983f3f0a1fbdb0)

After you fill in tags (optional) and click Review + Create, Azure validates the template. Deployment can take several minutes as VMs, networking, and platform services are provisioned.

![Screenshot of the Microsoft Azure portal on the "Create Azure Red Hat OpenShift" review page showing "Validation Passed" and the Review + create tab with deployment details (subscription, resource group, region, cluster name). A notification in the top-right indicates the deployment template is being submitted.](https://mintcdn.com/kodekloud-c4ac6d9a/Xa5GerWBw0vsOVlC/images/OpenShift-4/Getting-Started-with-OpenShift/Demo-Web-and-CLI-Templates-Yaml-Containers/azure-red-hat-openshift-review-validationpassed.jpg?fit=max\&auto=format\&n=Xa5GerWBw0vsOVlC\&q=85\&s=4910bdf57fb312f3ef5c7f79dbcbc8bf)

## Retrieve credentials and sign in

After the cluster is created, open the OpenShift console and sign in. For ARO you can retrieve kubeadmin credentials using the Azure CLI (requires the `az aro` extension):

```bash theme={null}
az aro list-credentials \
  --name kodekloudtest \
  --resource-group KodeKloud
```

Sample JSON output (the kubeadmin username and password will be returned):

```json theme={null}
{
  "kubeadminPassword": "TGDMM-Q85Fj-4X2wx-kKNaK",
  "kubeadminUsername": "kubeadmin"
}
```

Copy the `kubeadminPassword` value and use the `kubeadmin` username to sign in to the OpenShift web console. After login you can begin:

* Deploying applications (Web or CLI)
* Creating projects and namespaces
* Configuring cluster resources (RBAC, network policies, storage classes)

You should now be logged into your Azure Red Hat OpenShift cluster console and ready to continue with OpenShift administration and workloads.

## Checklist — before you create the cluster

| Item | Notes |
| - | - |
| Azure subscription | Ensure sufficient quota for selected VM family (increase if needed). |
| App Registration | Create App Registration, add client secret, note client ID and secret. |
| API permissions | Grant Azure Service Management permissions (user\_impersonation) for the App Registration. |
| Red Hat pull secret | Obtain from Red Hat console and paste in the ARO form. |
| Networking plan | Decide API and ingress visibility (public/private) and subnet strategy. |
| Cluster sizing | Select control plane/worker sizes and number of nodes (3 control plane recommended). |

## Links and references

* [OpenShift Local (formerly CodeReady Containers)](https://developers.redhat.com/products/codeready-containers/overview)
* [Azure Red Hat OpenShift (ARO) documentation](https://learn.microsoft.com/azure/openshift/)
* [Red Hat OpenShift installation guide](https://docs.openshift.com/)
* [Azure support: Increase quota (VM cores)](https://learn.microsoft.com/azure/azure-portal/supportability/resource-quotas)

Tip: For production workloads prefer managed control planes (like ARO) when possible. Managed offerings simplify upgrades, availability, and integration with cloud provider services.

