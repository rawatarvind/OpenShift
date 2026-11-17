# User Management Overview users ServiceAccounts Groups

> Explains OpenShift users, service accounts, and groups with creation, management, and least privilege best practices for authentication and authorization.

As with many learning platforms (for example, [Kubernetes for the Absolute Beginners - Hands-on Tutorial](https://learn.kodekloud.com/user/courses/kubernetes-for-the-absolute-beginners-hands-on-tutorial)), OpenShift requires a clear model for authentication and authorization. This article explains OpenShift users, service accounts, and groups — how they differ, how to create and manage them, and best practices for least-privilege access.

## Users

In OpenShift, a user represents a human identity that can log into the cluster. Regular users authenticate via configured identity providers and receive scoped permissions through RoleBindings or ClusterRoleBindings. Permissions can range from read-only to full admin access depending on the roles assigned.

OpenShift also defines system users — accounts created by the platform or infrastructure (for example, component/service identities). System users can have powerful privileges and are intended for platform components rather than interactive use.

Never use a highly privileged system account for routine day-to-day work. Instead, create user-specific accounts with the least privilege required.

![The image is a simple diagram titled "USERS" showing two user types—"1. Regular User" and "2. System User"—on the left. On the right it lists possible granular permissions in red buttons: "Read Only," "Read and Write," and "Literally Everything."](https://mintcdn.com/kodekloud-c4ac6d9a/_-d-o-x8WFPLT_DD/images/OpenShift-4/OpenShift-Security/User-Management-Overview-users-ServiceAccounts-Groups/users-regular-system-permissions-buttons.jpg?fit=max\&auto=format\&n=_-d-o-x8WFPLT_DD\&q=85\&s=d6c91185d184bea0cb206fd0e1817e9b)

### Creating users and identities

OpenShift separates the concept of a user (the local OpenShift object) from an identity (the external credential from an identity provider). Steps to create and map an identity:

* Create an OpenShift user:

```bash theme={null}
oc create user mike
```

* Create an identity that represents the credential from an identity provider. This example uses a provider named `allow_all`:

```bash theme={null}
oc create identity allow_all:mike
```

* Map that identity to the OpenShift user:

```bash theme={null}
oc create useridentitymapping allow_all:mike mike
```

* Remove a user when needed:

```bash theme={null}
oc delete user mike
```

System users are powerful. Do not share system or administrative accounts for regular tasks — use user accounts scoped with least privilege.

## Service accounts

Service accounts are Kubernetes/OpenShift API objects intended for automated processes (Pods, controllers, CI/CD jobs) to access the API. When you create a Pod, you can explicitly set which service account it uses. If omitted, the Pod uses the namespace's `default` ServiceAccount.

Because service account privileges depend on role bindings, relying on a shared or overly-privileged `default` account increases blast radius if a workload is compromised. Best practice: create a dedicated service account per application or job and bind only the minimal permissions required.

![A presentation slide titled "Service Accounts" with three red rounded buttons and the caption "Great for Overall Resource Management" on the left. On the right is a red user icon linked to a red label reading "Specific Job," illustrating general vs. job-specific accounts.](https://mintcdn.com/kodekloud-c4ac6d9a/_-d-o-x8WFPLT_DD/images/OpenShift-4/OpenShift-Security/User-Management-Overview-users-ServiceAccounts-Groups/service-accounts-general-vs-job.jpg?fit=max\&auto=format\&n=_-d-o-x8WFPLT_DD\&q=85\&s=2baee2fe99953f78d1a1ca3ad3aa61ba)

Create and inspect a service account:

```bash theme={null}
oc create sa mikesa
oc get sa
```

Grant a role to a service account using the `oc policy` command. When referencing a service account, use the format `system:serviceaccount:<namespace>:<serviceaccount>`. Example: give the `view` role to `mikesa` in the `sockshop` namespace:

```bash theme={null}
oc policy add-role-to-user view system:serviceaccount:sockshop:mikesa -n sockshop
```

You can also bind roles using RoleBinding or ClusterRoleBinding objects if you prefer declarative YAML manifests.

## Groups

Groups are collections of users that simplify permission management for teams. Assigning a role to a group lets you manage permissions for many users at once instead of for each user individually. Groups can be scoped to specific namespaces/projects (so a group may have access only to a particular project).

![A UI mockup titled "Groups" showing a card with three red user-avatar icons, an "Access: View all" label, and the caption "And 07 Other Engineers..". On the left are three stacked red square buttons with down chevrons and the text "Tie to a NameSpace."](https://mintcdn.com/kodekloud-c4ac6d9a/_-d-o-x8WFPLT_DD/images/OpenShift-4/OpenShift-Security/User-Management-Overview-users-ServiceAccounts-Groups/groups-card-three-avatars-access-namespace.jpg?fit=max\&auto=format\&n=_-d-o-x8WFPLT_DD\&q=85\&s=80a0a9b516fd8e72c310e44d94fd8d1b)

Create a new group and add users (use `oc adm`):

```bash theme={null}
oc adm groups new mikesgroup mike michelle
```

List groups:

```bash theme={null}
oc get groups
```

Grant a role to a group in a specific namespace (example: give `view` to `mikesgroup` in `sockshop`):

```bash theme={null}
oc policy add-role-to-group view mikesgroup -n sockshop
```

## Quick reference

| Resource Type | Purpose | Example command |
| - | - | - |
| User | Human login identity in OpenShift | `oc create user mike` |
| Identity | External credential (provider:user) mapped to a user | `oc create identity allow_all:mike` |
| ServiceAccount | API identity for Pods/controllers; use for automation | `oc create sa mikesa` |
| Group | Collection of users for team-level permissions | `oc adm groups new mikesgroup mike michelle` |
| Role grant (service account) | Bind a role to a service account | `oc policy add-role-to-user view system:serviceaccount:sockshop:mikesa -n sockshop` |
| Role grant (group) | Bind a role to a group in a namespace | `oc policy add-role-to-group view mikesgroup -n sockshop` |

## Best practices

* Follow least-privilege principles: assign only the roles required to perform tasks.
* Avoid using platform/system accounts for interactive work.
* Create dedicated service accounts per application/job and bind minimal roles.
* Use groups to manage team permissions and reduce administrative overhead.
* Prefer declarative RoleBinding/ClusterRoleBinding manifests in GitOps workflows for auditability.

With these primitives — users, identities, service accounts, and groups — you can implement scalable, least-privilege access control in OpenShift.

