# Demo OpenShift UI Overview

> High-level tour of the OpenShift web console and oc CLI explaining Developer and Administrator perspectives, topology, observability, deployments, resources, and basic commands for deploying and inspecting apps

Now that you’ve explored several OpenShift installation options — including the CodeReady Containers (CRC) developer environment, the OpenShift sandbox, and production-style clusters — this article gives a focused, high-level tour of the OpenShift web console (UI) and the `oc` CLI. You’ll see how the Developer and Administrator perspectives differ, where to inspect topology and workload details, how observability is presented, and how to authenticate and run basic commands with `oc`. Deployments and other resource types are introduced with examples you can reuse.

For further reading:

* CRC: [https://developers.redhat.com/products/codeready-containers/overview](https://developers.redhat.com/products/codeready-containers/overview)
* OpenShift docs: [https://docs.openshift.com/](https://docs.openshift.com/)
* Kubernetes Basics: [https://kubernetes.io/docs/concepts/overview/what-is-kubernetes/](https://kubernetes.io/docs/concepts/overview/what-is-kubernetes/)

Developer vs Administrator views

OpenShift’s console provides two main perspectives (or "views") that emphasize different workflows. The following table summarizes the core focus of each:

| Perspective | Focus | Typical tasks |
| - | - | - |
| Developer | Application-centric workflows and topology | Create/deploy apps, view service connections, inspect builds and application configuration |
| Administrator | Cluster-level operations and configuration | Manage operators, cluster settings, storage, networking, RBAC and quotas |

Both views overlap in many areas — for example, Deployments, Services, Routes and Secrets are visible in either perspective — but the UI surfaces different sets of tools depending on the chosen perspective.

Topology, workloads and resource details

* In Developer → Topology you get an architectural view of applications and how components connect (services, routes, builds).
* Clicking a workload node or selecting a Workloads item reveals resource details such as Deployments, ReplicaSets, ConfigMaps, labels, namespace, annotations, and creation time.

![A screenshot of the Red Hat OpenShift Dedicated web console showing the Topology view with a selected operator/workspace node. The right-hand panel displays operator details such as name, namespace, labels and creation time.](https://mintcdn.com/kodekloud-c4ac6d9a/Xa5GerWBw0vsOVlC/images/OpenShift-4/Getting-Started-with-OpenShift/Demo-OpenShift-UI-Overview/openshift-dedicated-topology-operator-details.jpg?fit=max\&auto=format\&n=Xa5GerWBw0vsOVlC\&q=85\&s=0ae97e3a65b5efa22e15f6feb3c2d0de)

Observability: events, metrics, and alerts

* The Observe section contains cluster events, metrics, and alerting information.
* Events provide a chronological timeline of cluster activity (pod/container start/stop, ReplicaSet scaling, etc.), useful for debugging transient issues.
* Metrics queries let you visualize CPU, memory, network and custom metrics for chosen resources. If nothing is deployed you’ll see little data, but the UI and queries are identical across development and production clusters.

![A screenshot of the Red Hat OpenShift Dedicated web console showing the Developer → Observe → Events tab for the project "mikel1992-dev." It displays a timeline of streaming cluster events (pod/container start/stop and replica set activity) with timestamps.](https://mintcdn.com/kodekloud-c4ac6d9a/Xa5GerWBw0vsOVlC/images/OpenShift-4/Getting-Started-with-OpenShift/Demo-OpenShift-UI-Overview/openshift-events-timeline-mikel1992-dev.jpg?fit=max\&auto=format\&n=Xa5GerWBw0vsOVlC\&q=85\&s=5cd9b40dfe4dd355691140bd2b4d84e9)

* Selecting a metric (for example, CPU usage) plots it in the Metrics explorer, letting you correlate spikes with events or pod restarts.

![A screenshot of the Red Hat OpenShift Dedicated web console showing the Observe → Metrics page. A dropdown menu for selecting queries is open, listing metrics like CPU usage, memory usage, bandwidth and packet rates.](https://mintcdn.com/kodekloud-c4ac6d9a/Xa5GerWBw0vsOVlC/images/OpenShift-4/Getting-Started-with-OpenShift/Demo-OpenShift-UI-Overview/openshift-dedicated-observe-metrics-dropdown.jpg?fit=max\&auto=format\&n=Xa5GerWBw0vsOVlC\&q=85\&s=a5dfaa4fb69d600f7801b5f4e7255443)

Search and Resources

* The Search page lets you find resources in a project by type and filter (label selectors such as `app=frontend` are commonly used).
* Use Search to quickly locate resource kinds across namespaces or inspect metadata without navigating multiple menus.

![A screenshot of the Red Hat OpenShift Dedicated developer console showing the Search page for project "mikel1992-dev" with the Resources dropdown expanded, listing resource types like APIRequestCount, APIScheme, APIServer, and more. The sidebar navigation is visible and a label filter "app=frontend" is applied while the main pane reads "No resources selected."](https://mintcdn.com/kodekloud-c4ac6d9a/Xa5GerWBw0vsOVlC/images/OpenShift-4/Getting-Started-with-OpenShift/Demo-OpenShift-UI-Overview/openshift-resources-app-frontend-mikel1992-dev.jpg?fit=max\&auto=format\&n=Xa5GerWBw0vsOVlC\&q=85\&s=336d0e6ba93258a8920a286c65113942)

Builds, Helm charts, ConfigMaps and Secrets

* Developer view includes BuildConfigs (when using OpenShift builds), integrated support for deploying Helm charts, and easy access to ConfigMaps and Secrets for application configuration and credentials.
* The console displays Secrets and ConfigMaps in a table with type, size and creation timestamp, and you can view or edit their YAML directly from the UI.

In OpenShift a “project” is essentially a Kubernetes namespace with OpenShift-specific metadata and RBAC defaults. For most day-to-day tasks you can treat them the same way.

![A screenshot of the Red Hat OpenShift Dedicated web console showing the "Secrets" page for project "mikel1992-dev." The table lists secret names (e.g., builder-token, default-token, deployer-token), their Kubernetes types, sizes and creation timestamps.](https://mintcdn.com/kodekloud-c4ac6d9a/Xa5GerWBw0vsOVlC/images/OpenShift-4/Getting-Started-with-OpenShift/Demo-OpenShift-UI-Overview/openshift-dedicated-secrets-mikel1992-dev.jpg?fit=max\&auto=format\&n=Xa5GerWBw0vsOVlC\&q=85\&s=745894fadc0c10708df159d0bf102dda)

Administrator portal and cluster-level resources

* Administrator view surfaces operators, cluster workloads, networking, storage, builds, user management, cluster settings and resource quotas.
* It also shows developer-oriented resources (services, routes, secrets), so administrators can manage both platform and application-level concerns from the same console.

Workloads → Deployments

* Administrator → Workloads → Deployments lets you create and manage Deployments directly in the UI.
* The console exposes the raw Kubernetes manifest (YAML) for a Deployment so you can learn, edit and apply manifests in the browser.

![Screenshot of the Red Hat OpenShift Dedicated web console displaying a project's "Deployments" page. It shows one deployment entry with 0 pods, label and pod-selector info, a "Create Deployment" button, and the left-hand navigation menu.](https://mintcdn.com/kodekloud-c4ac6d9a/Xa5GerWBw0vsOVlC/images/OpenShift-4/Getting-Started-with-OpenShift/Demo-OpenShift-UI-Overview/openshift-dedicated-deployments-zero-pods.jpg?fit=max\&auto=format\&n=Xa5GerWBw0vsOVlC\&q=85\&s=466e3684257dd65e576a07ea9544e203)

Example Deployment manifest

* The YAML below is a canonical Kubernetes Deployment manifest (apiVersion: `apps/v1`). You can paste it into the console editor or apply it with `oc apply -f <file>`.

```yaml theme={null}
apiVersion: apps/v1
kind: Deployment
metadata:
  name: example
  namespace: mikel1992-dev
spec:
  selector:
    matchLabels:
      app: httpd
  replicas: 3
  template:
    metadata:
      labels:
        app: httpd
    spec:
      containers:
      - name: httpd
        image: >-
          image-registry.openshift-image-registry.svc:5000/openshift/httpd:latest
        ports:
        - containerPort: 8080
```

oc CLI overview

* The OpenShift CLI is `oc`. It mirrors `kubectl` for many generic Kubernetes commands (`oc get pods` behaves like `kubectl get pods`) and also includes OpenShift-specific subcommands for builds, routes, and other platform resources.
* You can access a web-embedded terminal from the console (click the terminal icon) or install `oc` locally and authenticate from your workstation.

![A screenshot of the Red Hat Hybrid Cloud Console showing the "Command-line interface (CLI) tools" downloads page for OpenShift, with rows for different CLI tools and Download buttons. The page includes OS and architecture selectors and a left-side navigation menu.](https://mintcdn.com/kodekloud-c4ac6d9a/Xa5GerWBw0vsOVlC/images/OpenShift-4/Getting-Started-with-OpenShift/Demo-OpenShift-UI-Overview/red-hat-openshift-cli-downloads.jpg?fit=max\&auto=format\&n=Xa5GerWBw0vsOVlC\&q=85\&s=12f0ee69ac8f3a365459037f49cc1b9d)

Installing and logging in with oc

* Download the `oc` CLI from console.redhat.com/openshift (OpenShift → Downloads → Command-line interface (oc)) or from the official OpenShift downloads page.
* After installing, authenticate using the token the console provides (Console → Copy Login Commands → Display Token). The console gives you an `oc login` command and the server URL for your environment.

Never share your login token. Treat it like a password — tokens grant API access to your account and cluster resources.

Example login and token commands (replace the token and server with values from your Console):

```bash theme={null}
sha256~mC19qEdARbsLxD-agVHo9eZMYAFYo2Tx7GCulza-r9E

oc login --token=sha256~mC19qEdARbsLxD-agVHo9eZMYAFYo2Tx7GCulza-r9E --server=https://api.sandbox.x8i5.p1.openshiftapps.com:6443

curl -H "Authorization: Bearer sha256~mC19qEdARbsLxD-agVHo9eZMYAFYo2Tx7GCulza-r9E" "https://api.sandbox.x8i5.p1.openshiftapps.com:6443/apis/user.openshift.io/v1/users/~"
```

Example session showing a successful login and listing pods:

```bash theme={null}
# Example session output after logging in and selecting a project
./oc login --token=sha256~mCl9qEdARbsLxD-agVHo9eZMYAFYo2Tx7GCuIza-r9 --server=https://api.sandbox.x8i5.p1.openshiftapps.com:6443
Logged into "https://api.sandbox.x8i5.p1.openshiftapps.com:6443" as "mikel1992" using the token provided.

You have one project on this server: "mikel1992-dev"

Using project "mikel1992-dev".
./oc get pods
NAME                                           READY   STATUS    RESTARTS   AGE
workspaced0af5646651143db-56cfddf4b7-jwc2s    2/2     Running   0          6m12s
```

Common oc commands

| Task | Command |
| - | - |
| Login | `oc login --server=<API_SERVER> --token=<TOKEN>` |
| List projects | `oc get projects` |
| Use project | `oc project <project-name>` |
| List pods | `oc get pods` |
| List deployments | `oc get deployments` |
| Apply YAML | `oc apply -f <file.yaml>` |
| View resource YAML | `oc get deployment <name> -o yaml` |

Wrapping up

* This article provided a practical tour of the OpenShift console (Developer and Administrator views), topology and observability screens, how the console displays canonical Kubernetes manifests, and how to use `oc` for authentication and inspecting cluster resources.
* Key topics introduced: Deployments, Services, Routes, Builds, ConfigMaps, Secrets, metrics and events, and `oc` basics.

Additional links and references

* OpenShift Documentation: [https://docs.openshift.com/](https://docs.openshift.com/)
* OpenShift CLI downloads: [https://console.redhat.com/openshift](https://console.redhat.com/openshift) (OpenShift → Downloads → Command-line interface (oc))
* Kubernetes Concepts: [https://kubernetes.io/docs/concepts/](https://kubernetes.io/docs/concepts/)

