# Projects and Users

> Explains OpenShift projects and user types, showing how namespaces, service accounts, authentication, and RBAC provide isolation, access control, and team-based resource management.

Hello and welcome to this lesson.

In this lesson we explain OpenShift projects and users and show how they provide isolation, access control, and grouping for workloads running in a cluster.

![A presentation slide showing the title "Red Hat OPENSHIFT" with the subtitle "Projects and Users" centered over a red-to-purple cloudy gradient background.](https://mintcdn.com/kodekloud-c4ac6d9a/_-d-o-x8WFPLT_DD/images/OpenShift-4/OpenShift-Concepts-Projects-and-Users/Projects-and-Users/red-hat-openshift-projects-users.jpg?fit=max\&auto=format\&n=_-d-o-x8WFPLT_DD\&q=85\&s=04e6a56611aff8bba742ecf7d4d3fda7)

## What projects are and why they matter

A single Kubernetes or OpenShift cluster can run hundreds of deployments and many thousands of pods. Without logical boundaries, that shared infrastructure can create two common problems:

* Unrestricted access: resources are visible to any account with the right cluster permissions, which can let teams accidentally access or modify each other’s deployments.
* Naming collisions: a flat resource namespace means two teams might try to create a Deployment or Service with the same name.

OpenShift solves this using projects. A project is an OpenShift-level abstraction that maps to a Kubernetes namespace and includes metadata and policy that make it easy for teams to group and isolate resources. Users create a project and deploy their application; OpenShift creates and configures the underlying namespace and related project metadata automatically, simplifying day-to-day operations and governance.

![A slide titled "Projects" showing four rounded panels representing team services (Service: team1.myservice through team4.myservice) with group and container icons, and a long row of server icons along the bottom.](https://mintcdn.com/kodekloud-c4ac6d9a/_-d-o-x8WFPLT_DD/images/OpenShift-4/OpenShift-Concepts-Projects-and-Users/Projects-and-Users/projects-team-services-panels-servers.jpg?fit=max\&auto=format\&n=_-d-o-x8WFPLT_DD\&q=85\&s=3db2964242134249368fe9c520670285)

Benefits at a glance:

* Logical isolation for resources and quotas.
* Easier team collaboration without cross-team interference.
* A natural boundary for role-based access control (RBAC), build configs, pipelines, and service accounts.

## OpenShift users: types and responsibilities

OpenShift contains built-in user and account management to support different kinds of identities. There are three main categories:

| User category | Purpose | Example account(s) |
| - | - | - |
| Regular users | Developers and operators who log in interactively to create and manage apps | `developer` |
| System users | Accounts used by cluster components and control-plane services; often auto-created | `system:admin`, `system:masters` |
| Service accounts | Project-scoped accounts for in-cluster processes (pods, controllers) to authenticate to the API | `system:serviceaccount:proj1:db_user` |

Service accounts are ideal for automated access from Pods or controllers, while system users are reserved for cluster-level components and administrative roles.

![A presentation slide titled "Users" showing three red outline icons for user types: Regular, System, and Service. Example accounts under them read "developer", "system:admin"/"system:master", and "system:serviceaccount:proj1:db\_user".](https://mintcdn.com/kodekloud-c4ac6d9a/_-d-o-x8WFPLT_DD/images/OpenShift-4/OpenShift-Concepts-Projects-and-Users/Projects-and-Users/users-regular-system-service-accounts.jpg?fit=max\&auto=format\&n=_-d-o-x8WFPLT_DD\&q=85\&s=a51a018201e0aa081dede2f729ff09cd)

## Authentication and authorization

OpenShift authentication is handled by the cluster OAuth server, which verifies user identities and issues access tokens. Those tokens are used by the API server and the authorization subsystem (for example, [RBAC](https://kubernetes.io/docs/reference/access-authn-authz/rbac/)) to enforce permissions.

Local or single-node development environments (for example, `Minishift` or `CodeReady Containers (CRC)`) commonly use an AllowAll-style identity provider so anyone can log in easily for demos and development. In production, administrators should configure explicit identity providers (LDAP, OAuth, OpenID Connect, etc.). Historically, older OpenShift releases stored OAuth and identity-provider configuration in the master config file at `/etc/origin/master/master-config.yaml`; newer versions manage OAuth differently—consult the version-specific documentation for details.

![A slide titled "OAuth Server" with a compass logo, showing two red server icons: a single server labeled "Allow All" and a cluster of servers labeled "Deny All."](https://mintcdn.com/kodekloud-c4ac6d9a/_-d-o-x8WFPLT_DD/images/OpenShift-4/OpenShift-Concepts-Projects-and-Users/Projects-and-Users/oauth-server-allow-all-deny-all.jpg?fit=max\&auto=format\&n=_-d-o-x8WFPLT_DD\&q=85\&s=aca23f5ceb0d3e593b07a387164ac2ff)

Local/minified setups (Minishift, CRC) are convenient for development and demos because they commonly permit easy login via an AllowAll identity provider. For production clusters you should configure a secure identity provider (LDAP, OAuth, OpenID Connect, etc.) and ensure access is restricted appropriately.

Do not use an AllowAll or DenyAll identity provider in production. Always configure an explicit identity provider and review RBAC rules and project-level policies before granting access.

## Quick reference and next steps

* Use projects to isolate teams and workloads without provisioning separate clusters.
* Use service accounts for in-cluster automation and system users only for cluster components or administrative tasks.
* Configure a secure identity provider and enforce least-privilege with RBAC.

A follow-up demo explores projects and users in action on a running cluster.

That's it for this lesson.

## Links and references

* [Kubernetes RBAC documentation](https://kubernetes.io/docs/reference/access-authn-authz/rbac/)
* [CodeReady Containers (CRC)](https://developers.redhat.com/products/codeready-containers/overview)
* [Minishift (legacy)](https://github.com/minishift/minishift)
* [OpenShift authentication (version-specific)](https://docs.openshift.com/container-platform/latest/authentication/)

