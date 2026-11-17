# RBAC roles rolebinding cluster role clusterrole binding

> Explains Kubernetes RBAC concepts, Roles and ClusterRoles, RoleBindings and ClusterRoleBindings, usage examples and best practices for least privilege access control.

RBAC (Role-Based Access Control) is the single most important mechanism for securing a Kubernetes cluster.

If your organization does not have RBAC configured correctly, you will not have reliable access controls for service accounts, users, and the Kubernetes resources they create and manage — whether those are Deployments, Services, Ingresses, ConfigMaps, Secrets, or any other Kubernetes API object.

![A simple diagram showing security/identity icons — a hardhat/gear symbol, a key, and the Kubernetes logo — with red rounded labels reading "Service Accounts", "Users", and "Resources."](https://mintcdn.com/kodekloud-c4ac6d9a/_-d-o-x8WFPLT_DD/images/OpenShift-4/OpenShift-Security/RBAC-roles-rolebinding-cluster-role-clusterrole-binding/kubernetes-service-accounts-users-resources-icons.jpg?fit=max\&auto=format\&n=_-d-o-x8WFPLT_DD\&q=85\&s=f70619bbdacf9e28f81db764bb008def)

Without RBAC, cluster security will always be a mess.

What is RBAC (role-based access control)?

At its core, RBAC is authorization: specifying which principals (users, groups, or service accounts) may perform which actions on which resources. RBAC rules can be scoped to a single namespace or to the entire cluster.

This guide walks through the core concepts and shows concrete YAML examples so you can apply RBAC to real workloads. Conceptually, RBAC states: “Principal X is allowed to perform verb Y on resource Z (optionally limited to namespace N).”

RBAC controls two things: the actions (verbs) that are allowed (for example, `get`, `list`, `create`, `update`, `delete`) and the resources those actions apply to (for example, pods, deployments, configmaps). RBAC rules can also scope by API group and restrict to specific resource names.

Namespace vs cluster scope

You can have roles that are namespace-scoped or cluster-scoped:

* Namespace-scoped Roles apply only in the namespace where they are defined.
* Cluster-scoped ClusterRoles apply across all namespaces and to cluster-scoped resources.

![A slide titled "What is RBAC?" illustrating authorization and resources, with red blocks labeled "NameSpace Scope" and "Cluster Scope" inside a dashed box. On the left are two red numbered squares (1. and 2.) and the text "NameSpace Scope or Cluster Scope."](https://mintcdn.com/kodekloud-c4ac6d9a/_-d-o-x8WFPLT_DD/images/OpenShift-4/OpenShift-Security/RBAC-roles-rolebinding-cluster-role-clusterrole-binding/what-is-rbac-namespace-cluster-scope.jpg?fit=max\&auto=format\&n=_-d-o-x8WFPLT_DD\&q=85\&s=27a1b1048caeb27e57b584e90a167689)

Terminology and the four building blocks

There are four main RBAC resource types to understand:

| Resource Type | Scope | Purpose | Example |
| - | - | - | - |
| Role | Namespace | Defines allowed verbs on resources in a specific namespace | `Role` example below |
| ClusterRole | Cluster | Defines allowed verbs cluster-wide or for cluster-scoped resources | use via `ClusterRole` or bind with a `RoleBinding` |
| RoleBinding | Namespace | Grants permissions defined in a `Role` or `ClusterRole` to subjects within a namespace | `RoleBinding` example below |
| ClusterRoleBinding | Cluster | Grants permissions defined in a `ClusterRole` to subjects cluster-wide | bind admin-level permissions to a user or group |

A simple mnemonic: whenever you see the `cluster` prefix (ClusterRole or ClusterRoleBinding), it means cluster-scoped. Roles are lists of allowed operations; RoleBindings and ClusterRoleBindings attach those lists to principals.

Role and rule composition

Roles (and ClusterRoles) are composed of rules that specify:

* apiGroups (e.g., `""` for core, `"apps"`, `"batch"`)
* resources (e.g., `pods`, `deployments`, `jobs`)
* verbs (e.g., `get`, `list`, `watch`, `create`, `update`, `patch`, `delete`)
* optional `resourceNames` to restrict access to specific named objects

Minimal example: read-only access to pods in a namespace

```yaml theme={null}
# example: read-pods-role.yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: read-pods
rules:
  - apiGroups: [""]
    resources: ["pods"]
    verbs: ["get", "list", "watch"]
```

RoleBindings: grant a Role to a subject within a namespace

RoleBindings attach a Role to a principal (user, group, or service account) within a namespace. The RoleBinding references the Role and lists the subjects that receive the permissions.

![A presentation slide titled "What are Roles?" showing a role-binding diagram: stacked red user icons on the left, a central red user/role icon, and hat icons paired with red "NameSpace" labels on the right.](https://mintcdn.com/kodekloud-c4ac6d9a/_-d-o-x8WFPLT_DD/images/OpenShift-4/OpenShift-Security/RBAC-roles-rolebinding-cluster-role-clusterrole-binding/role-binding-diagram-users-namespaces.jpg?fit=max\&auto=format\&n=_-d-o-x8WFPLT_DD\&q=85\&s=4184869d54810a179f811622fe3a4af0)

Example RoleBinding that grants the `read-pods` Role to a service account in the same namespace:

```yaml theme={null}
# example: bind-read-pods.yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: bind-read-pods
subjects:
  - kind: ServiceAccount
    name: my-app-sa
    namespace: sock-shop
roleRef:
  kind: Role
  name: read-pods
  apiGroup: rbac.authorization.k8s.io
```

ClusterRoles and ClusterRoleBindings

When permissions need to apply across all namespaces or to cluster-scoped resources (like nodes or persistent volumes), use a ClusterRole. A ClusterRole can be bound cluster-wide with a ClusterRoleBinding, or it can be referenced from a namespace-scoped RoleBinding to grant namespace-level access to cluster-scoped rules.

Key distinctions:

* `ClusterRole` = cluster-scoped permission definition
* `ClusterRoleBinding` = bind that cluster-scoped permission to subjects cluster-wide
* A `RoleBinding` can reference a `ClusterRole` to grant cluster-role permissions within a specific namespace

![A slide titled "What are Roles?" showing red user icons on the left labeled "Cluster Role Binding," a single red user icon in the center, and network-style icons with a cursor pointing to two red rounded boxes labeled "Cluster" on the right.](https://mintcdn.com/kodekloud-c4ac6d9a/_-d-o-x8WFPLT_DD/images/OpenShift-4/OpenShift-Security/RBAC-roles-rolebinding-cluster-role-clusterrole-binding/cluster-role-binding-user-cluster-network.jpg?fit=max\&auto=format\&n=_-d-o-x8WFPLT_DD\&q=85\&s=0af8c7ac786cc14fe7d76c1172205ece)

Best practices and cautions

Avoid granting broad cluster-level permissions (for example, `cluster-admin`) unless absolutely necessary. Follow least-privilege principles: define minimal Roles/ClusterRoles and bind them only to the subjects that need them.

* Prefer namespace-scoped Roles and RoleBindings to limit blast radius.
* Use ClusterRoles sparingly and only when resources or operations truly require cluster scope.
* Use `resourceNames` to restrict access to specific objects when feasible.
* Audit and review Role/RoleBinding/ClusterRole/ClusterRoleBinding objects regularly.

Quick reference: common verbs

* read-related: `get`, `list`, `watch`
* write-related: `create`, `update`, `patch`
* delete: `delete`
* other: `exec`, `impersonate` (special cases)

Summary

* Roles and ClusterRoles define allowed actions on Kubernetes resources.
* RoleBindings and ClusterRoleBindings attach those permissions to principals (users, groups, service accounts).
* Use namespace-scoped Roles/RoleBindings to limit permissions to a single namespace.
* Use ClusterRoles/ClusterRoleBindings for cluster-wide permissions or for cluster-scoped resources.
* Apply the principle of least privilege when designing RBAC for your cluster.

Next steps / Resources

* Review your cluster’s existing Roles and Bindings with `kubectl get roles,rolebindings,clusterroles,clusterrolebindings -A`.
* Test new Roles and Bindings in a non-production namespace before applying cluster-wide.
* Further reading:
  * [Kubernetes RBAC documentation](https://kubernetes.io/docs/reference/access-authn-authz/rbac/)
  * [Kubernetes Authorization Overview](https://kubernetes.io/docs/reference/access-authn-authz/authorization/)

Now that the concepts are clear, try the demo examples above in a test namespace to see RBAC behavior in action.
