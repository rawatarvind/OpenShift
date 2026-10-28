# Deploying applications CLI

> Guide to deploying the Sock Shop microservices demo on OpenShift and Kubernetes using oc or kubectl, including CLI commands, routes, and troubleshooting tips.

The Sock Shop microservices demo is a popular, realistic example application that showcases a typical microservices architecture: multiple Deployments, Services, databases, and other Kubernetes/OpenShift resources. It’s widely used to validate platform configurations, test observability tools, and exercise CI/CD pipelines.

If you want to run the upstream Kubernetes manifests with kubectl:

```bash theme={null}
kubectl create namespace sock-shop
kubectl apply -f complete-demo.yaml
```

For this guide we use the OpenShift-adapted microservices demo. The OpenShift variant contains manifests adjusted to run without cluster-admin privileges and to leverage OpenShift features such as Routes.

As you explore the repository you’ll find the full UI and many individual services (front-end, catalog, carts, orders, databases, etc.):

![A screenshot of a GitHub README page for a microservices demo displaying a webpage header with a close-up photo of striped socks and three feature boxes, followed by a Weave Scope visualization diagram. The image shows the browser UI and tabs around the README content.](https://mintcdn.com/kodekloud-c4ac6d9a/Xa5GerWBw0vsOVlC/images/OpenShift-4/OpenShift-Concepts-Projects-and-Users/Deploying-applications-CLI/github-readme-microservices-striped-socks-scope.jpg?fit=max\&auto=format\&n=Xa5GerWBw0vsOVlC\&q=85\&s=31162d1ffddc11f282d2a8f7cc96a249)

The OpenShift microservices demo manifests are prepared to run without cluster-admin rights. Use `oc` commands on OpenShift; `kubectl` works for upstream Kubernetes clusters.

Quick walkthrough — deploy the demo from the terminal

1. Clone the repository and change into it:

```bash theme={null}
git clone https://github.com/microservices-demo/microservices-demo.git
cd microservices-demo
```

2. Create an OpenShift project and apply the demo manifest:

```bash theme={null}
oc new-project sock-shop
oc apply -f ./complete-demo.yaml
```

Applying the manifest creates many Deployments and Services required by the app. You should see resource creation messages similar to:

```bash theme={null}
deployment.apps/carts-db created
service/carts-db created
deployment.apps/carts created
service/carts created
deployment.apps/front-end created
service/front-end created
route.route.openshift.io/front-end created
...
```

3. Expose the front-end service (if the manifest didn't already create a Route), then list the Route to get the public host:

```bash theme={null}
oc expose service/front-end
oc get route
```

A sample `oc get route` output showing the HTTP host:

```text theme={null}
NAME        HOST/PORT                                                                SERVICES     PORT    TERMINATION
front-end   front-end-sock-shop.apps.xxxxxxxxxxxxxxxxxxxxx.com                        front-end    web
```

Open the HOST/PORT URL in your browser to view the running application.

4. Verify Deployments and Pods:

```bash theme={null}
oc get deployments
oc get pods
```

You can also watch progress live:

```bash theme={null}
oc get pods -w
```

In the OpenShift web console navigate to:

* Workloads → Deployments to inspect each Deployment
* Workloads → Pods to view pod status and logs
* Networking → Routes to find the front-end URL

Common CLI commands (quick reference)

| Task | Command | Notes / Example |
| - | - | - |
| Create project | `oc new-project sock-shop` | Creates an OpenShift project and sets it as the current context |
| Apply manifest | `oc apply -f ./complete-demo.yaml` | Deploys all resources defined in the manifest |
| Expose a service | `oc expose service/front-end` | Creates an OpenShift Route for HTTP access |
| List routes | `oc get route` | Shows route hostnames for browser access |
| List pods | `oc get pods` | Check pod status and restarts |
| Tail pods | `oc logs -f <pod-name>` | Inspect logs for troubleshooting |
| Watch resources | `oc get pods -w` | Watch pods until they reach Running/Completed |

Why use the CLI?

* Speed and repeatability: Applying manifests via `oc` or `kubectl` is fast and automatable.
* Scriptable: CLI commands integrate readily into CI/CD pipelines and automation scripts.
* OpenShift-specific features: `oc` adds convenience commands such as `oc new-project` and native support for Routes (which often replace Kubernetes Ingress on OpenShift).
* Debugging: CLI access to `oc logs`, `oc exec`, and `oc describe` makes troubleshooting straightforward.

Deployment options on OpenShift

* From the web console (UI)
* Directly from a Git repository (Build/Apps pipelines, GitOps)
* From the terminal using `oc` (covered above)

Try each method to determine which best fits your workflow. For a fast, repeatable demo environment the CLI approach is often the quickest.

Pods may take a few minutes to reach Running status. If pods are in CrashLoopBackOff or Pending, check events and logs: `oc describe pod <pod>` and `oc logs <pod>` to identify image pull errors, liveness/readiness failures, or resource quota issues.

Further reading and references

* Microservices Demo (Sock Shop) repository: [https://github.com/microservices-demo/microservices-demo](https://github.com/microservices-demo/microservices-demo)
* OpenShift `oc` CLI documentation: [https://docs.openshift.com/](https://docs.openshift.com/)
* Kubernetes `kubectl` documentation: [https://kubernetes.io/docs/reference/kubectl/](https://kubernetes.io/docs/reference/kubectl/)
