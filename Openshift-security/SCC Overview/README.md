# SCC Overview

> Overview of Security Context Constraints in Kubernetes/OpenShift explaining host level controls for pod privileges, kernel confinement, best practices and how SCCs complement network policies and policy as code

Security is often under-resourced in many engineering organizations. When you compare the number of application developers to dedicated security engineers, the security team is usually much smaller — and sometimes non-existent. Industry research highlights the risk: the Red Hat State of Kubernetes and Container Security Report attributes a large portion of incidents to misconfiguration (nearly 70%), with the remainder being runtime security incidents.

![A screenshot of a web article titled "How to handle Kubernetes security" showing the overview paragraph about remediating vulnerabilities, misconfigurations, and runtime threats. The page includes an updated date (June 17, 2022) and a source link to Red Hat.](https://mintcdn.com/kodekloud-c4ac6d9a/_-d-o-x8WFPLT_DD/images/OpenShift-4/OpenShift-Security/SCC-Overview/how-to-handle-kubernetes-security-red-hat.jpg?fit=max\&auto=format\&n=_-d-o-x8WFPLT_DD\&q=85\&s=18295c941da9097f03da4dffe20f1aaa)

Security Context Constraints (SCC) are one of the platform-level controls designed to reduce these risks by limiting what pods and containers can do on the host.

What SCCs control

* Pod runtime privileges: whether a container can run as root or be privileged.
* Host access: ability to mount host paths, use host networking, or access host PID namespaces.
* Linux security mechanisms: SELinux labels, AppArmor profiles, and Linux capabilities.

SCCs are a policy mechanism focused on host- and OS-level privileges—complementary to network policy and higher-level policy engines.

![A slide-like graphic titled "Security Context Constraints" that highlights "Control Permissions for Pods." It shows a red "Access Control" button and a simple icon suggesting system-level security controls.](https://mintcdn.com/kodekloud-c4ac6d9a/_-d-o-x8WFPLT_DD/images/OpenShift-4/OpenShift-Security/SCC-Overview/security-context-constraints-pods-access-control.jpg?fit=max\&auto=format\&n=_-d-o-x8WFPLT_DD\&q=85\&s=72a1df1818caef059847e38aa36fcd4b)

How SCCs relate to other controls

* Network policies (Kubernetes NetworkPolicy): control connectivity (ingress/egress) between pods, namespaces, and external endpoints.
* SCCs: control privileges and capabilities at the OS/host level (privileged mode, filesystem access, capabilities, SELinux/AppArmor labels).
* Open Policy Agent (OPA) and Gatekeeper: provide policy-as-code across many Kubernetes resource types and admission-time checks; they are more flexible for custom policies but operate at a different layer than SCCs.

Use these mechanisms together: network policy reduces who can reach a pod; SCCs reduce what that pod can do once running; OPA/Gatekeeper can enforce higher-level organizational rules across resources.

![A stylized search bar reading "what about Open Policy Agent?" with a red magnifying-glass icon and two buttons below labeled "OPA" (red) and "SCC" (blue). A simple outlined box/package icon sits beneath the buttons.](https://mintcdn.com/kodekloud-c4ac6d9a/_-d-o-x8WFPLT_DD/images/OpenShift-4/OpenShift-Security/SCC-Overview/open-policy-agent-search-opa-scc.jpg?fit=max\&auto=format\&n=_-d-o-x8WFPLT_DD\&q=85\&s=1d79cb542669e701f011ff978a0c641d)

Kernel-level enforcement and runtime confinement

SCCs may leverage native Linux kernel security modules such as SELinux and AppArmor to confine containerized processes, restrict filesystem access, and limit capabilities. These kernel primitives are powerful for reducing the attack surface of workloads.

AppArmor and SELinux-based controls are Linux-kernel specific. If you run Windows worker nodes, kernel-level modules like AppArmor will not apply the same way.

![A screenshot of the Kubernetes documentation webpage titled "Restrict a Container's Access to Resources with AppArmor," showing the article text, left navigation menu, and a right-hand table of contents.](https://mintcdn.com/kodekloud-c4ac6d9a/_-d-o-x8WFPLT_DD/images/OpenShift-4/OpenShift-Security/SCC-Overview/kubernetes-apparmor-restrict-container-access.jpg?fit=max\&auto=format\&n=_-d-o-x8WFPLT_DD\&q=85\&s=bad5a2fc04fa0e69947ba7af6a19a256)

Community guidance and common misconfigurations

Community resources—such as the OWASP Top Ten for Kubernetes—catalog common misconfigurations and suggest secure defaults. Typical recommendations include: avoid running as root, enable read-only root filesystems, disallow privileged containers, and minimize Linux capabilities.

Pod and container securityContext examples

Below are concise examples showing where common securityContext fields belong and how they apply.

Example: Pod-level runAsUser — applies to all containers in the Pod.

```yaml theme={null}
apiVersion: v1
kind: Pod
metadata:
  name: run-as-user-example
spec:
  securityContext:
    # Set UID for all containers in this pod (non-root recommended)
    runAsUser: 1000
  containers:
    - name: app
      image: nginx:latest
```

Example: Container-level read-only root filesystem.

```yaml theme={null}
apiVersion: v1
kind: Pod
metadata:
  name: read-only-fs
spec:
  containers:
    - name: app
      image: nginx:latest
      securityContext:
        # Make the container filesystem read-only
        readOnlyRootFilesystem: true
```

Example: Container-level privileged flag (use with extreme caution).

```yaml theme={null}
apiVersion: v1
kind: Pod
metadata:
  name: privileged-pod
spec:
  containers:
    - name: privileged-app
      image: busybox:latest
      securityContext:
        # Allow the container to run in privileged mode (equivalent to Docker's --privileged)
        privileged: true
```

Running containers in privileged mode or granting excessive Linux capabilities is high risk. Avoid privileged containers unless absolutely necessary and audit any capability additions.

Best-practice checklist

| Area | Recommendation | Example / Why |
| - | - | - |
| UID/GID | Avoid root; set `runAsUser` | Reduces impact of a container compromise |
| Filesystem | Use `readOnlyRootFilesystem: true` where possible | Enforces immutability and prevents on-disk tampering |
| Privileges | Disallow `privileged: true`; drop unnecessary capabilities | Minimizes host-level escalation opportunities |
| Kernel confinement | Use SELinux/AppArmor profiles | Strong OS-level confinement for processes |
| Network | Apply NetworkPolicy for ingress/egress control | Limits network-based attack paths |
| Policy-as-code | Use OPA/Gatekeeper for admission policies | Enforces organization-wide rules at admission time |

References and further reading

* [SELinux Project](https://selinuxproject.org/)
* [AppArmor](https://apparmor.net/)
* [Open Policy Agent (OPA)](https://www.openpolicyagent.org/)
* [Gatekeeper (Open Policy Agent)](https://open-policy-agent.github.io/gatekeeper/)
* [OWASP Kubernetes Top Ten](https://owasp.org/www-project-kubernetes-top-ten/)
* [Kubernetes Network Policies](https://kubernetes.io/docs/concepts/services-networking/network-policies/)

Summary

SCCs offer platform-integrated, host-level controls to reduce the attack surface of containerized workloads. Use SCCs alongside network policies and a policy-as-code tool such as OPA/Gatekeeper to create layered defenses: limit network access, minimize runtime privileges, and enforce organizational policies at admission time. These combined controls significantly reduce the likelihood and impact of misconfiguration- and runtime-related incidents.
