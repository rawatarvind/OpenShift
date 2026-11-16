# Demo Install Single Node using Sandbox

> Guide to using the Red Hat OpenShift Developer Sandbox for quick cloud-hosted ephemeral OpenShift demos and experimentation as an alternative to local CodeReady Containers

If you prefer not to install CodeReady Containers (CRC) locally — for example, your workstation lacks resources or you want a quick cloud-hosted demo — the OpenShift Developer Sandbox provides a managed, ephemeral OpenShift cluster hosted by Red Hat.

Quick start

1. Sign in to the OpenShift web console (use your Red Hat account).
2. Open the OpenShift Developer Sandbox: [https://developers.redhat.com/develop/get-started/sandbox](https://developers.redhat.com/develop/get-started/sandbox)
3. On the sandbox page click "Learn More About the Sandbox" and follow the prompts to start your environment.
4. If prompted, complete the additional identity verification (phone number + SMS code) to enable the sandbox for your account.

Some Red Hat accounts require an extra security step to enable the Sandbox. If you are prompted, you must provide a phone number and enter the verification code sent via SMS to activate your sandbox.

The Developer Sandbox is ideal for learning and experimentation. It provides a fully managed OpenShift cluster in the cloud that is ephemeral and not suitable for production workloads — it will be deleted after the sandbox period (typically 30 days).

Example environment (typical constraints)

| Resource | Typical value |
| - | - |
| RAM | 7 GB |
| Storage | 15 GB |
| Time limit | 30 days |
| Intended use | Learning, demos, short experiments |

Example output (sample lscpu / system summary from a sandbox session)

```bash theme={null}
[your@sandbox ~]$ lscpu
# Example resources reported for the sandbox environment:
RAM: 7GB
Storage: 15GB
Time limit: 30 days
Awesome: YES
```

Why use the Sandbox vs CRC?

| Option | Best for | Notes |
| - | - | - |
| CodeReady Containers (CRC) | Local development where you need full control and persistent local cluster | Requires a powerful machine and local installation. See: [https://developers.redhat.com/products/codeready-containers/overview](https://developers.redhat.com/products/codeready-containers/overview) |
| OpenShift Developer Sandbox | Fast, cloud-hosted demos and tutorials without local install | Managed by Red Hat, ephemeral (automatic deletion), ideal for quick experimentation. See: [https://developers.redhat.com/develop/get-started/sandbox](https://developers.redhat.com/develop/get-started/sandbox) |

Once your sandbox starts, you'll have a fully functional OpenShift instance hosted by Red Hat, with cluster components managed in the cloud. Because the environment is temporary, avoid deploying any production workloads — create separate, long-lived clusters for production instead. You can start new sandboxes later if you need additional demo time.

![A Red Hat OpenShift Dedicated web console screenshot showing a project dashboard with a left-hand navigation menu. The main area displays an application/topology view with small resource nodes labeled as workspace and terminal.](https://mintcdn.com/kodekloud-c4ac6d9a/Xa5GerWBw0vsOVlC/images/OpenShift-4/Getting-Started-with-OpenShift/Demo-Install-Single-Node-using-Sandbox/openshift-dedicated-project-dashboard-topology.jpg?fit=max\&auto=format\&n=Xa5GerWBw0vsOVlC\&q=85\&s=d8c7b55216db30ebc4f5e6272ee7715f)

Links and references

* OpenShift Developer Sandbox: [https://developers.redhat.com/develop/get-started/sandbox](https://developers.redhat.com/develop/get-started/sandbox)
* CodeReady Containers (CRC): [https://developers.redhat.com/products/codeready-containers/overview](https://developers.redhat.com/products/codeready-containers/overview)
* OpenShift Documentation: [https://docs.openshift.com/](https://docs.openshift.com/)

