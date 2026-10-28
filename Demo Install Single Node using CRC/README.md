# Demo Install Single Node using CRC

> Guide to install and run Red Hat CodeReady Containers to create a local single-node OpenShift cluster for development, covering setup, startup, console access, CLI commands, and troubleshooting.

This guide walks through installing Red Hat CodeReady Containers (CRC) to create a local single-node OpenShift cluster for development and testing. CRC runs a lightweight VM that exposes an OpenShift API and web console, similar in purpose to Docker Desktop or Minikube but tailored for OpenShift.

## Before you begin

* You need a Red Hat account. Visit the OpenShift cluster creation UI at: [https://console.redhat.com/openshift](https://console.redhat.com/openshift)
* Confirm your machine meets CRC prerequisites: virtualization support enabled, sufficient CPU, RAM, and disk space (CRC downloads \~10–15 GB of images and bundles).

Recommended minimums:

| Requirement | Recommendation |
| - | - |
| OS | Linux, macOS, or Windows (64-bit) |
| CPU | 4 cores (more recommended for performance) |
| RAM | 8 GB minimum (16 GB recommended) |
| Disk | 35 GB free (to allow for images and VM disk) |
| Virtualization | Hyper-V (Windows), HyperKit/Hypervisor.framework (macOS), libvirt (Linux) |

For official CRC product information and supported backends, see the CodeReady Containers overview: [https://developers.redhat.com/products/codeready-containers/overview](https://developers.redhat.com/products/codeready-containers/overview)

## 1. Create a local cluster and download the installer

1. Sign in to [https://console.redhat.com/openshift](https://console.redhat.com/openshift).
2. Click Create Cluster → Local.
3. Choose Red Hat OpenShift Local and download the installer for your OS (Linux, macOS, or Windows).
4. Run the installer and proceed through the installer dialogs.

![A screenshot of the Red Hat Hybrid Cloud Console "Create an OpenShift cluster" page with a Windows File Explorer window open and the Red Hat OpenShift Local installer displaying an End-User License Agreement dialog. The installer shows the Apache License text with the "I accept" checkbox checked and the Next button highlighted.](https://mintcdn.com/kodekloud-c4ac6d9a/Xa5GerWBw0vsOVlC/images/OpenShift-4/Getting-Started-with-OpenShift/Demo-Install-Single-Node-using-CRC/red-hat-openshift-local-eula-accepted.jpg?fit=max\&auto=format\&n=Xa5GerWBw0vsOVlC\&q=85\&s=e8f27f4c8be1ab490c8177b3149216ee)

After the installer finishes you will typically be prompted to restart your system. Reboot when requested to ensure virtualization components are initialized correctly.

On Windows, ensure Hyper-V is enabled and your user is a member of the Hyper-V-related groups. CRC configures Hyper-V components during setup. If you use a different virtualization backend, confirm CRC supports it for your OS: [https://developers.redhat.com/products/codeready-containers/overview](https://developers.redhat.com/products/codeready-containers/overview)

## 2. Run CRC setup

Open a terminal (PowerShell on Windows) and verify the `crc` CLI is available. A typical prompt on Windows looks like:

```powershell theme={null}
PS C:\Users\mike>
```

Run the setup command to install prerequisites and configure the virtualization backend:

```powershell theme={null}
crc setup
```

What to expect during setup:

* You may be asked about anonymous usage statistics — you can choose No.
* The setup will configure virtualization components (Hyper-V on Windows), adjust user group membership if needed, and install the CRC VM bundle.
* The required images and bundles are downloaded (this can be large — plan for 10–15 GB and several minutes).

## 3. Start the cluster

After setup completes, start the CRC cluster:

```powershell theme={null}
crc start
```

The `crc start` command creates and boots the VM, initializes OpenShift components, and streams logs. The process may take 10–20 minutes depending on your machine and network.

When startup finishes, CRC prints the web console URL and login credentials. Example output:

```powershell theme={null}
INFO 2 operators are progressing: kube-apiserver, openshift-controller-manager
INFO Operator authentication is progressing
INFO All operators are available. Ensuring stability...
INFO Operators are stable (3/3)...
INFO Adding crc-admin and crc-developer contexts to kubeconfig...
Started the OpenShift cluster.

The server is accessible via web console at:
    https://console-openshift-console.apps-crc.testing

Log in as administrator:
    Username: kubeadmin
    Password: IxAVf-CKLZp-aqT8d-niTJv

Log in as user:
    Username: developer
    Password: developer

Use the 'oc' command line interface: https://docs.openshift.com/container-platform/latest/cli_reference/openshift_cli/oc/index.html
PS> & crc oc-env | Invoke-Expression
PS> oc login -u developer https://api.crc.testing:6443
PS C:\Users\mike>
```

When running, the CRC VM is visible in your OS hypervisor. On Windows you’ll see a VM named "crc" in Hyper‑V Manager.

![A screenshot of the Hyper-V Manager showing a running virtual machine named "crc" with CPU, assigned memory and uptime displayed. The Hyper-V window overlays a Red Hat OpenShift web interface sidebar (Clusters, Overview, Releases).](https://mintcdn.com/kodekloud-c4ac6d9a/Xa5GerWBw0vsOVlC/images/OpenShift-4/Getting-Started-with-OpenShift/Demo-Install-Single-Node-using-CRC/hyperv-crc-running-vm-openshift-sidebar.jpg?fit=max\&auto=format\&n=Xa5GerWBw0vsOVlC\&q=85\&s=8b910e2fd8852f825035448b232d6cee)

## 4. Access the OpenShift console and CLI

Open the web console by either:

* Right-clicking the CodeReady Containers taskbar icon and choosing "Open Console", or
* Visiting the web console URL printed by `crc start` (for example `https://console-openshift-console.apps-crc.testing`) in your browser.

To retrieve credentials later (without restarting the cluster), run:

```powershell theme={null}
crc console --credentials
```

Configure your shell to use the `oc` CLI and then log in as the developer user:

```powershell theme={null}
& crc oc-env | Invoke-Expression
oc login -u developer https://api.crc.testing:6443
```

To log in as the kubeadmin administrator, use the `kubeadmin` password printed by CRC at startup.

## 5. Useful CRC commands

| Command | Purpose |
| - | - |
| `crc setup` | Install prerequisites and configure virtualization integration |
| `crc start` | Create and start the CRC VM and OpenShift cluster |
| `crc stop` | Stop the CRC VM and cluster |
| `crc delete` | Delete the CRC VM and cluster resources |
| `crc console --credentials` | Print the web console credentials |
| `crc version` | Show CRC client version |

## Troubleshooting tips

If the VM fails to start, check that virtualization is enabled in BIOS/UEFI and that no other hypervisor (e.g., VirtualBox) is conflicting with Hyper-V. On Windows, ensure Hyper-V is enabled and your account has the required permissions. Review `crc start` logs for operator errors and consult the CRC documentation if needed.

* If `oc` commands fail, ensure you ran the `crc oc-env` step or that your kubeconfig context was updated.
* If the console URL is inaccessible, verify DNS resolution for `*.crc.testing` (CRC sets up local DNS for the test domains).
* Check CRC logs for details: `crc logs` (or see the VM/hypervisor logs for low-level errors).

## Next steps

With CRC running and `oc` configured, you have a local single-node OpenShift cluster ready for:

* Deploying and testing containerized applications
* Exploring OpenShift features (Projects, Routes, Operators)
* Learning OpenShift administration and CI/CD workflows

Resources:

* Red Hat CodeReady Containers overview: [https://developers.redhat.com/products/codeready-containers/overview](https://developers.redhat.com/products/codeready-containers/overview)
* OpenShift CLI (oc) documentation: [https://docs.openshift.com/container-platform/latest/cli\_reference/openshift\_cli/oc/index.html](https://docs.openshift.com/container-platform/latest/cli_reference/openshift_cli/oc/index.html)

