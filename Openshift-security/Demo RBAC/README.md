# Demo RBAC

> Demonstrates creating namespace Roles, ServiceAccounts, and RoleBindings in OpenShift Kubernetes, reusing ClusterRoles, verifying permissions, and RBAC best practices

This guide demonstrates how to assign a namespace-scoped Role to a ServiceAccount using RoleBinding in an OpenShift / Kubernetes cluster. It first highlights built-in ClusterRoles you can reuse, then walks through creating a namespace-scoped Role, a ServiceAccount, and a RoleBinding that ties them together. Finally, it shows how to verify permissions using kubectl impersonation.

Built-in ClusterRoles are documented in the OpenShift docs:

* [Using RBAC to define and apply permissions](https://docs.openshift.com/container-platform/latest/authentication/using-rbac.html)

![A screenshot of the Red Hat OpenShift documentation page showing a table of default cluster roles and their descriptions under "Using RBAC to define and apply permissions." The page also shows the left navigation menu and a table of contents on the right.](https://mintcdn.com/kodekloud-c4ac6d9a/_-d-o-x8WFPLT_DD/images/OpenShift-4/OpenShift-Security/Demo-RBAC/openshift-rbac-default-cluster-roles.jpg?fit=max\&auto=format\&n=_-d-o-x8WFPLT_DD\&q=85\&s=5a52826b321e463ec33b73dd85d0ff08)

Common built-in roles (high-level):

| Role | Scope / Purpose | Short description |
| - | - | - |
| `admin` | Namespace | Project/namespace administrator — can manage most resources in the namespace. |
| `edit` | Namespace | Modify resources in a namespace (create/update/delete). |
| `view` | Namespace | Read-only access to resources in a namespace. |
| `cluster-admin` | Cluster | Full superuser access across the entire cluster. |
| `cluster-reader` / `cluster-status` | Cluster | Read-only visibility into cluster resources or status. |
| `self-provisioner` | Cluster | Allows creating new projects (useful for delegated project creation). |

These built-in ClusterRoles are convenient for common use cases. For finer control, create:

* a namespace-scoped `Role` (for permissions limited to a single namespace), or
* a `ClusterRole` (for cluster-wide permissions or to reuse across namespaces).

In the OpenShift console (User Management → Roles) you can browse Roles and ClusterRoles. The console lists roles and the namespaces they apply to.

![A screenshot of the Red Hat OpenShift web console showing the Roles page with a list of cluster roles and their namespaces on the right and the admin navigation menu (Storage, Builds, User Management, Roles) on the left. The top banner notes this is a local cluster for development and testing.](https://mintcdn.com/kodekloud-c4ac6d9a/_-d-o-x8WFPLT_DD/images/OpenShift-4/OpenShift-Security/Demo-RBAC/openshift-console-roles-local-cluster.jpg?fit=max\&auto=format\&n=_-d-o-x8WFPLT_DD\&q=85\&s=77d375e886c8f08246d38b57cb230285)

Role vs ClusterRole — key distinctions

* Role (`rbac.authorization.k8s.io/v1`, kind: `Role`) is namespace-scoped — it grants permissions only within one namespace.
* ClusterRole is cluster-scoped — it can grant permissions across the cluster, or be referenced inside a namespace using a RoleBinding.

Example: create a namespace-scoped Role that grants read-only (view) permissions on Pods in the `default` namespace:

```yaml theme={null}
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: viewpods
  namespace: default
rules:
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["get", "watch", "list"]
```

This Role allows any subject bound to it to list, watch, and get Pods in the `default` namespace only.

Create a ServiceAccount to represent a non-human identity used by workloads:

```yaml theme={null}
apiVersion: v1
kind: ServiceAccount
metadata:
  name: podviewer
  namespace: default
```

Resource names (ServiceAccounts, Roles, RoleBindings, etc.) must be lowercase and follow Kubernetes naming rules (DNS-1123).

Bind the Role to the ServiceAccount using a RoleBinding in the same namespace. A RoleBinding associates a Role with one or more subjects (Users, Groups, ServiceAccounts) within the binding's namespace:

```yaml theme={null}
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: podviewer-viewpods-binding
  namespace: default
subjects:
- kind: ServiceAccount
  name: podviewer
  namespace: default
roleRef:
  kind: Role
  name: viewpods
  apiGroup: rbac.authorization.k8s.io
```

You can also create this RoleBinding from the OpenShift console using the Create RoleBinding UI — choose the role (e.g., `viewpods`) and the subject (ServiceAccount `podviewer` in `default`).

![Screenshot of the Red Hat OpenShift web console on the "Create RoleBinding" page, with the role name set to "viewpods" and the subject selected as a ServiceAccount in the "default" namespace named "podviewer."](https://mintcdn.com/kodekloud-c4ac6d9a/_-d-o-x8WFPLT_DD/images/OpenShift-4/OpenShift-Security/Demo-RBAC/openshift-create-rolebinding-viewpods-podviewer.jpg?fit=max\&auto=format\&n=_-d-o-x8WFPLT_DD\&q=85\&s=c199641986a2e3fab1294e1b0fc1cd37)

Verify the ServiceAccount's permissions using kubectl impersonation. The `kubectl auth can-i` command is useful to check allowed and denied actions for a given subject.

To test that `podviewer` can list pods in `default`:

```bash theme={null}
kubectl auth can-i list pods --as system:serviceaccount:default:podviewer -n default
# expected output: "yes"
```

To confirm a denied action (for example, listing deployments which the Role does not allow):

```bash theme={null}
kubectl auth can-i list deployments --as system:serviceaccount:default:podviewer -n default
# expected output: "no"
```

Best practices and tips

* Use `Role` when permissions should be limited to a single namespace; use `ClusterRole` for cluster-wide privileges or re-usable rules across namespaces.
* Prefer ServiceAccounts for non-human identities (workloads, controllers).
* Restrict permissions to the minimum verbs/resources required (least privilege).
* Audit RoleBindings and ClusterRoleBindings carefully — a binding can elevate privileges if it references a powerful Role or ClusterRole.

Be careful when using `cluster-admin` or broad ClusterRoles in RoleBindings/ClusterRoleBindings — they grant powerful privileges that can affect the entire cluster.

Summary

* Role = namespace-scoped permissions. ClusterRole = cluster-scoped permissions.
* Create ServiceAccounts for workload identities and bind them to Roles/ClusterRoles using RoleBindings/ClusterRoleBindings.
* Verify access with `kubectl auth can-i --as system:serviceaccount:<ns>:<sa> -n <ns>`.
* Keep resource names lowercase and follow Kubernetes naming rules.

Links and references

* OpenShift RBAC docs: [https://docs.openshift.com/container-platform/latest/authentication/using-rbac.html](https://docs.openshift.com/container-platform/latest/authentication/using-rbac.html)
* Kubernetes RBAC: [https://kubernetes.io/docs/reference/access-authn-authz/rbac/](https://kubernetes.io/docs/reference/access-authn-authz/rbac/)

