# Demo Projects and Users

> Demo walkthrough of OpenShift projects and users on Minishift, showing oc login, listing users, creating projects from the web console, and granting cluster admin to a web console user

Welcome. In this demo we'll explore how projects and users work in OpenShift, using a Minishift local cluster for illustration. You'll learn how to:

* Log in with the CLI
* Inspect cluster users
* Create a project from the web console
* Add a web-console user and grant cluster-admin privileges (for administrative inspection)

Prerequisites:

* A running Minishift/OpenShift single-node environment
* `oc` CLI configured to point at your cluster

## 1) Log in to the CLI

Log in to the cluster using the `oc login` command. In this example we use the built-in system administrator account on Minishift, which typically does not prompt for a password:

```bash theme={null}
C:\minishift-1.16.1-windows-amd64> oc login -u system:admin
Logged into "https://192.168.99.102:8443" as "system:admin" using existing credentials.

You have access to the following projects and can switch between them with 'oc project <projectname>':

  default
  kube-public
  kube-system
* myproject
  openshift
  openshift-infra
  openshift-node
  openshift-web-console

Using project "myproject".

C:\minishift-1.16.1-windows-amd64>
```

Because this is Minishift, `system:admin` login completes immediately without a password prompt. The output shows the projects available to the current user and the currently selected project (`myproject`).

## 2) Inspect cluster users

To list users known to the cluster, use `oc get users`. In Minishift the default identity provider is often the AnyPassword provider, which accepts any password and auto-creates users when they sign in via the web console:

```bash theme={null}
C:\minishift-1.16.1-windows-amd64> oc get users
NAME        UID                                     FULL NAME    IDENTITIES
developer   758ed92d-4c3b-11e8-8098-e2e0de05d311               anypassword:developer

C:\minishift-1.16.1-windows-amd64>
```

If you sign in to the web console with a new username (for example, `developer2`), that identity will be created automatically. After logging in as `developer2` from the console, the `oc get users` output will include the new user:

```bash theme={null}
C:\minishift-1.16.1-windows-amd64> oc get users
NAME        UID                                     FULL NAME    IDENTITIES
developer   758ed92d-4c3b-11e8-8098-e2e0de05d311               anypassword:developer
developer2  ba585475-4c43-11e8-8098-e2e0de05d311               anypassword:developer2

C:\minishift-1.16.1-windows-amd64>
```

## 3) Create a project from the web console

Switch back to (or sign in as) a regular developer user and create a new project from the web console:

1. Click Create Project.
2. Enter a project name and (optionally) a display name.
3. Click Create.
4. Open the project to view its console and any deployed applications.

![A screenshot of the OpenShift Origin web console showing the "my-new-project" overview page with a "Get started with your project" message and a prominent "Browse Catalog" button. The left sidebar shows menu items like Overview, Applications, Builds, Resources, Storage, Monitoring, and Catalog.](https://mintcdn.com/kodekloud-c4ac6d9a/Xa5GerWBw0vsOVlC/images/OpenShift-4/OpenShift-Concepts-Projects-and-Users/Demo-Projects-and-Users/openshift-origin-my-new-project-overview.jpg?fit=max\&auto=format\&n=Xa5GerWBw0vsOVlC\&q=85\&s=4dec1c00f518d7eeeded22cb11c77e07)

The project Overview displays resource counts, application status, and quick actions. When viewing the list of projects in the web console, you will only see those projects your current console user has permission to view.

Note: Default system projects (for example, `default`, `kube-system`, `openshift`) are visible to cluster administrators but may not be visible to regular users.

## 4) Create an administrative web-console user (to inspect default projects)

Because `system:admin` is a cluster-internal account and typically cannot be used to sign in to the web console, create a regular web-console user and then give that user cluster-admin privileges for administrative inspection.

First, sign in to the web console with the username `administrator`. With Minishift's AnyPassword identity provider, this will create the user automatically. Confirm with:

```bash theme={null}
C:\minishift-1.16.1-windows-amd64> oc get users
NAME            UID                                     FULL NAME    IDENTITIES
administrator   4343778b-4c44-11e8-8098-e2e0de05d311               anypassword:administrator
developer       758ed92d-4c3b-11e8-8098-e2e0de05d311               anypassword:developer
developer2      ba585475-4c43-11e8-8098-e2e0de05d311               anypassword:developer2

C:\minishift-1.16.1-windows-amd64>
```

Grant the `cluster-admin` cluster role to the `administrator` user:

```bash theme={null}
C:\minishift-1.16.1-windows-amd64> oc adm policy add-cluster-role-to-user cluster-admin administrator
cluster role "cluster-admin" added: "administrator"

C:\minishift-1.16.1-windows-amd64>
```

Do not grant cluster-admin privileges to accounts you do not trust. Cluster-admin provides full control over the cluster.

After assigning the role, sign in to the web console as `administrator`. You will now be able to see the default system projects and inspect their configurations, permissions, and resources.

If you only need to inspect or manage specific projects, prefer granting namespace-level roles rather than `cluster-admin` to adhere to the principle of least privilege.

## Quick reference — useful `oc` commands

| Command | Purpose |
| - | - |
| `oc login -u <user>` | Log in to the cluster with the given user |
| `oc project <name>` | Switch the active project/namespace |
| `oc get users` | List users known to the cluster |
| `oc adm policy add-cluster-role-to-user cluster-admin <user>` | Grant cluster-admin role to a user |

## Links and references

* OpenShift documentation: [https://docs.openshift.com/](https://docs.openshift.com/)
* Kubernetes Concepts: [https://kubernetes.io/docs/concepts/overview/what-is-kubernetes/](https://kubernetes.io/docs/concepts/overview/what-is-kubernetes/)
* Minishift (local OpenShift) project: [https://github.com/minishift/minishift](https://github.com/minishift/minishift)

This walkthrough covered creating projects from the console, how web-console user identities are created (with the AnyPassword provider in Minishift), and how to add cluster-admin rights to a console user for cluster inspection.

