
# Demo User Management

> Guide to managing OpenShift users, identities, service accounts, groups, and RoleBindings using oc CLI and web console with examples and best practices

This guide covers basic user, identity, service account, and group management in OpenShift. It shows both CLI (`oc`) and web console workflows, with practical commands and YAML snippets you can reuse. Use this to quickly create users, map identities, manage service accounts, and assign roles via RoleBindings.

Quick reference links

* OpenShift User Management concepts: [https://docs.openshift.com](https://docs.openshift.com)
* Kubernetes RBAC overview: [https://kubernetes.io/docs/reference/access-authn-authz/rbac/](https://kubernetes.io/docs/reference/access-authn-authz/rbac/)

## Common oc commands

| Task | Command / Example |
| - | - |
| Create a user | `oc create user <username>` |
| List users | `oc get user` |
| Create identity mapping | `oc create useridentitymapping <identity> <username>` |
| Create a service account | `oc create sa <name> -n <namespace>` |
| Grant a role to a service account | `oc policy add-role-to-user <role> system:serviceaccount:<namespace>:<sa>` |
| Create a group (YAML) | Use Group YAML in the console or `oc apply -f group.yaml` |
| Create a RoleBinding (console or `oc`) | Use console form or `oc create rolebinding ...` |

## Create a user

Use the CLI to create a user named `miketest` and verify it exists:

```powershell theme={null}
PS C:\Users\mike> oc create user miketest
user.user.openshift.io/miketest created
PS C:\Users\mike> oc get user
NAME        UID                                     FULL NAME    IDENTITIES
developer   577f362a-643f-456b-903d-1ccbfc0eb5f5                developer:dev
kubeadmin   3348c0e2-1e47-41c9-8548-ed8450ffbca2                developer:kubeadmin
miketest    e22c031c-5a37-4e10-90ec-5301379f451e
PS C:\Users\mike>
```

Verifying the `oc get user` output confirms the user exists in the cluster. Users are OpenShift API objects under `user.openshift.io/v1`.

## Create an identity mapping for the user

OpenShift separates users and identities. To attach an external identity to an OpenShift user, create a user identity mapping. For example, map the identity `allow_all:miketest` to the OpenShift user `miketest`:

```powershell theme={null}
PS C:\Users\mike> oc create useridentitymapping allow_all:miketest miketest
useridentitymapping.user.openshift.io/allow_all:miketest created
PS C:\Users\mike> oc get user
NAME        UID                                     FULL NAME    IDENTITIES
developer   577f362a-643f-456b-643f-456b-903d...               developer:dev
kubeadmin   3348c0e2-1e47-41c9-8548-ed84...               developer:kubeadmin
miketest    e22c031c-5a37-4e10-90ec-5301379f451e               allow_all:miketest
PS C:\Users\mike>
```

After mapping, the `IDENTITIES` column shows the attached identity. This is required when integrating external identity providers or when your authentication flow provides identities separate from OpenShift user objects.

## Service accounts (UI and CLI)

Service accounts are Kubernetes core objects (apiVersion `v1`) that represent identities for in-cluster processes (pods). You can create them from the OpenShift web console or via `oc`.

Example ServiceAccount YAML for the `limittester` namespace:

```yaml theme={null}
apiVersion: v1
kind: ServiceAccount
metadata:
  name: example
  namespace: limittester
```

Create from the CLI (without `-n` will place it in the current project, commonly `default`):

```powershell theme={null}
PS C:\Users\mike> oc create sa mikesa
serviceaccount/mikesa created

PS C:\Users\mike> oc get sa -n default
NAME      SECRETS   AGE
mikesa    1         1m
PS C:\Users\mike>
```

To explicitly create it in a target namespace, include `-n`:

```powershell theme={null}
PS C:\Users\mike> oc create sa mikesa -n limittester
serviceaccount/mikesa created
PS C:\Users\mike> oc get sa -n limittester
NAME      SECRETS   AGE
mikesa    1         5s
PS C:\Users\mike>
```

Always specify the target namespace (with `-n <namespace>`) when creating resources from the CLI to avoid unintentionally creating them in the `default` project.

Grant a role to a service account (example: give `mikesa` the `view` role in `default`):

```powershell theme={null}
PS C:\Users\mike> oc policy add-role-to-user view system:serviceaccount:default:mikesa
clusterrole.rbac.authorization.k8s.io/view added: "system:serviceaccount:default:mikesa"
PS C:\Users\mike>
```

Inspect the ServiceAccount object in YAML via the console or CLI. A typical ServiceAccount object contains metadata, secrets, and optional `imagePullSecrets`:

```yaml theme={null}
kind: ServiceAccount
apiVersion: v1
metadata:
  name: mikesa
  namespace: default
  uid: 83d6f8fd-b6db-4528-8210-167dc4b6aa18
  resourceVersion: "60308"
  creationTimestamp: "2022-09-20T10:55:51Z"
secrets:
  - name: mikesa-dockercfg-hwc22
imagePullSecrets:
  - name: mikesa-dockercfg-hwc22
```

ServiceAccount is a core API object (group \`\`, version `v1`) used extensively for pod identity and integrations.

## Groups and RoleBindings (UI)

Groups let you manage collections of users. RoleBindings assign roles to subjects (users, groups, or service accounts) within a namespace. Use the console to create groups and RoleBindings, or apply YAML from the CLI.

Create another user (example `test`) via CLI:

```powershell theme={null}
PS C:\Users\mike> oc create user test
user.user.openshift.io/test created
PS C:\Users\mike>
```

Example Group YAML you can use in the console to create a group and add users:

```yaml theme={null}
apiVersion: user.openshift.io/v1
kind: Group
metadata:
  name: test-roles
users:
  - test
  - another-user
```

After creating a Group, bind a role to it using a RoleBinding. A RoleBinding grants a specific role (for example `admin`) inside a namespace to the subjects listed.

Use the web console to create a RoleBinding (provide a name, target namespace, and role), or create one with `oc`. The console form looks like this:

![A screenshot of the Red Hat OpenShift web console showing the "Create RoleBinding" form with radio options for Namespace or Cluster-wide role bindings and input fields for name and namespace. A dark left navigation menu lists sections like Networking, Storage, Builds, and RoleBindings.](https://mintcdn.com/kodekloud-c4ac6d9a/_-d-o-x8WFPLT_DD/images/OpenShift-4/OpenShift-Security/Demo-User-Management/openshift-console-create-rolebinding-form.jpg?fit=max\&auto=format\&n=_-d-o-x8WFPLT_DD\&q=85\&s=fd059567b7032b3ab0ea248fefed31f0)

For example, create a RoleBinding named `testbinding` that grants the `admin` role in the `default` namespace to the `test-roles` group. After binding, group members (such as `test`) will have admin privileges in that namespace.

Once created, review the RoleBinding details in the console to confirm scope and subjects:

![A screenshot of the Red Hat OpenShift web console showing RoleBinding details for an object named "testbinding" in the default namespace. The left sidebar shows navigation items (Networking, Storage, User Management) and a red banner warns the cluster is for development/testing.](https://mintcdn.com/kodekloud-c4ac6d9a/_-d-o-x8WFPLT_DD/images/OpenShift-4/OpenShift-Security/Demo-User-Management/openshift-webconsole-rolebinding-testbinding-warning.jpg?fit=max\&auto=format\&n=_-d-o-x8WFPLT_DD\&q=85\&s=5054dacb22e676e9156ff21a7db721d7)

RoleBindings and ClusterRoleBindings grant permissions. Always verify the role, namespace (scope), and subjects before applying bindings—especially when assigning `admin` or cluster-level roles.

## Next steps and further reading

* Study RBAC in depth: roles, ClusterRoles, RoleBindings, ClusterRoleBindings.
* Learn authentication integrations for external identity providers and how identities map to OpenShift users.
* Use `oc describe` and `oc get -o yaml` to inspect objects and troubleshoot access issues.

References

* [OpenShift documentation on identity and access management](https://docs.openshift.com)
* [Kubernetes RBAC documentation](https://kubernetes.io/docs/reference/access-authn-authz/rbac/)

