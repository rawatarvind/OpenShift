# Demo Setting up user contexts projectwise

> Guide to creating a read-only OpenShift user, granting pod view permissions via ClusterRole and ClusterRoleBinding, binding the user, and verifying access or limiting to a namespace.

In the previous lesson we created a role and a user mapped to a ServiceAccount. This lesson shows how to create a read-only user that can view pods (get, list, watch) but cannot modify cluster resources.

Use case: you need an account that can inspect resources for reporting or audits without granting any write permissions.

## Overview

Steps we'll follow:

1. Create the user with `oc create user`.
2. Create a ClusterRole that grants read-only access to pods.
3. Bind that ClusterRole to the user with a ClusterRoleBinding.
4. Verify the binding and test the user's permissions.

## 1) Create the user

Create a user named `mikeuser` and confirm it exists:

```powershell theme={null}
PS C:\Users\mike\Desktop> oc create user mikeuser
user.user.openshift.io/mikeuser created

PS C:\Users\mike\Desktop> oc get user mikeuser
NAME       UID
mikeuser   aeafd722-de5b-4c0e-8a25-3b205a24167c

PS C:\Users\mike\Desktop> oc describe user mikeuser
Name:         mikeuser
Created:      13 seconds ago
Labels:       <none>
Annotations:  <none>
Identities:   <none>
PS C:\Users\mike\Desktop>
```

## 2) Create a ClusterRole and ClusterRoleBinding (single YAML)

Place both the ClusterRole and ClusterRoleBinding into one YAML file (for example `role.yaml`). This ClusterRole grants only read verbs (`get`, `list`, `watch`) on pods; the binding attaches that role to the `mikeuser` user.

```yaml theme={null}
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: mikesreaduser
rules:
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["get", "watch", "list"]
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: mike-can-read-pods
subjects:
- kind: User
  name: mikeuser
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: ClusterRole
  name: mikesreaduser
  apiGroup: rbac.authorization.k8s.io
```

Remember: subject kinds are case-sensitive. Use `User`, `Group`, or `ServiceAccount` exactly as shown.

Important: the `kind` field under `subjects` is case-sensitive. Using `user` (lowercase) will cause the ClusterRoleBinding to be rejected.

## 3) Apply the YAML

Apply the combined YAML to create the ClusterRole and ClusterRoleBinding:

```powershell theme={null}
PS C:\Users\mike\Desktop> oc apply -f .\role.yaml
clusterrole.rbac.authorization.k8s.io/mikesreaduser created
clusterrolebinding.rbac.authorization.k8s.io/mike-can-read-pods created
PS C:\Users\mike\Desktop>
```

## 4) Verify the binding and test permissions

* In the OpenShift Console: go to Users → select `mikeuser` → RoleBindings. You should see `mike-can-read-pods` referencing `mikesreaduser`, which allows `get`, `list`, and `watch` on `pods`.

* Example `oc get` / `oc describe` output for the ClusterRoleBinding (this mirrors what you might see in the console or from `oc get clusterrolebinding -o yaml`):

```yaml theme={null}
kind: ClusterRoleBinding
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: mike-can-read-pods
  uid: f6951a98-d8f2-4f4d-80de-74c66cf7206e
  resourceVersion: '72095'
  creationTimestamp: '2022-09-20T11:53:02Z'
subjects:
- kind: User
  name: mikeuser
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: ClusterRole
  name: mikesreaduser
  apiGroup: rbac.authorization.k8s.io
```

* Optional quick permission test (run as a cluster-admin or a user who can impersonate):

```bash theme={null}
oc auth can-i list pods --as=mikeuser
```

Expected output: `yes` (if the binding is effective). Try a write verb to confirm denial:

```bash theme={null}
oc auth can-i create pods --as=mikeuser
```

Expected output: `no`

## 5) Restricting access to a single namespace

If you want the user limited to a single namespace instead of cluster-wide pod access, create a `Role` and a `RoleBinding` in that namespace (namespace-scoped). Replace `kind: ClusterRole` / `ClusterRoleBinding` with `Role` / `RoleBinding` and include the target `metadata.namespace` in both objects.

## Quick reference

| Resource Type | Purpose | Example |
| - | - | - |
| User | Identity created in OpenShift | `oc create user mikeuser` |
| ClusterRole | Cluster-scoped permissions | See `role.yaml` above |
| ClusterRoleBinding | Binds ClusterRole to a user across the cluster | See `role.yaml` above |
| Role (namespace) | Namespace-scoped permissions | Create in the target namespace |
| RoleBinding (namespace) | Binds Role to user within a namespace | Create in the same namespace as the Role |

## Links and references

* OpenShift user management: [https://docs.openshift.com/](https://docs.openshift.com/)
* Kubernetes RBAC: [https://kubernetes.io/docs/reference/access-authn-authz/rbac/](https://kubernetes.io/docs/reference/access-authn-authz/rbac/)
* oc CLI reference: [https://docs.openshift.com/container-platform/latest/cli\_reference/openshift\_cli/getting-started-cli.html](https://docs.openshift.com/container-platform/latest/cli_reference/openshift_cli/getting-started-cli.html)

Now `mikeuser` can view pods cluster-wide (or, if you prefer, only in a namespace) without being able to modify resources.

