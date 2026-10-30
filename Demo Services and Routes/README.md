# Demo Services and Routes

> Lesson on exposing OpenShift applications using Services for internal access and Routes for external HTTP and HTTPS exposure, with YAML examples and verification commands.

Hello and welcome to this lesson on [OpenShift for beginners](https://learn.kodekloud.com/user/courses/openshift-4).

In this lesson you'll learn how to expose an application running on OpenShift so that other workloads inside the cluster — or external users — can reach it. The typical end-to-end flow is:

1. Create a Service to expose your pods inside the cluster (ClusterIP).
2. Create a Route to expose that Service externally (HTTP/HTTPS).
3. Verify the application via the ClusterIP (internal) and the Route (external).

## Create a Service (internal exposure)

If your builds and deployments are already configured, the next step is to create a Service to make your application reachable from other pods and services inside the cluster.

Start by getting a Service YAML template (for example, via the OpenShift console “Learn More” / “Import YAML” option). Create a file named `service-config.yaml` and paste the template into it. An initial template might look like this:

```yaml theme={null}
apiVersion: v1
kind: Service
metadata:
  name: simple-webapp-docker
spec:
  selector:
    docker-registry: default
  clusterIP: 172.30.136.123
  ports:
  - nodePort: 0
    port: 5000
    protocol: TCP
    targetPort: 5000
```

This template contains a few key fields:

| Field | Purpose | Example / Notes |
| - | - | - |
| `metadata.name` | Service name | `simple-webapp-docker` |
| `spec.selector` | Labels used to match Service -> pods | `deploymentconfig: simple-webapp-docker` |
| `clusterIP` | Cluster-only IP assigned to the Service | Typically omitted so OpenShift auto-assigns it |
| `ports` | Port mapping the Service listens on and the container port | `port: 8080`, `targetPort: 8080` |

Update the selector and ports to match your application (in this example the app listens on port `8080`). Remove the `clusterIP` field so OpenShift assigns it automatically:

```yaml theme={null}
apiVersion: v1
kind: Service
metadata:
  name: simple-webapp-docker
spec:
  selector:
    deploymentconfig: simple-webapp-docker
  ports:
  - name: 8080-tcp
    port: 8080
    protocol: TCP
    targetPort: 8080
```

Apply the Service:

* Via the OpenShift web console: choose Import YAML or JSON, paste `service-config.yaml`, and click Create.
* Or via CLI: `oc apply -f service-config.yaml`

After creation, open the Service details in the console to view the assigned Cluster IP and ports.

![A screenshot of the OpenShift Origin web console showing the "Services" page for a project named My WebApplication. It lists two services (simple-webapp-docker and simple-webapp) with Cluster IPs, port 8080/TCP, selectors, and ages.](https://mintcdn.com/kodekloud-c4ac6d9a/Xa5GerWBw0vsOVlC/images/OpenShift-4/Networks-Services-Routes-and-Scaling/Demo-Services-and-Routes/openshift-services-my-webapplication.jpg?fit=max\&auto=format\&n=Xa5GerWBw0vsOVlC\&q=85\&s=4826714815c6e24c56575a8daedbb59f)

The Service will have an assigned Cluster IP (for example `172.30.163.74`). This IP is reachable only from inside the cluster network — not from the public internet.

ClusterIP Services are internal to the cluster. To be reachable from outside the cluster you need to expose the Service using a Route (on OpenShift) or use a `NodePort`/`LoadBalancer` Service type depending on your environment.

You can test the Service from a machine or container that has network access to the cluster. For example, from the OpenShift host VM (Minishift) or from another pod:

```bash theme={null}
# From an environment with cluster network access:
curl http://172.30.163.74:8080
```

Example output:

```text theme={null}
Welcome! Update 4
```

## Create a Route (external exposure)

To make the Service accessible to external HTTP(S) clients, create an OpenShift Route. You can create Routes via YAML (same approach as Services) or from the web console. Here we'll use the console: click Create Route.

The Create Route form allows you to set the route name, hostname, path, service, and target port. If you leave the hostname blank, OpenShift will generate a hostname for you.

![A screenshot of the OpenShift Origin web console displaying the "Create Route" form for a project called "My WebApplication," with fields for Name, Hostname, Path, Service, and Target Port. The left sidebar shows navigation items like Applications, Builds, Resources, Storage, Monitoring, and Catalog.](https://mintcdn.com/kodekloud-c4ac6d9a/Xa5GerWBw0vsOVlC/images/OpenShift-4/Networks-Services-Routes-and-Scaling/Demo-Services-and-Routes/openshift-create-route-form.jpg?fit=max\&auto=format\&n=Xa5GerWBw0vsOVlC\&q=85\&s=00e9ed92248a41180b4c59ba7b078906)

Important Route form fields:

| Field | Purpose |
| - | - |
| Name | Logical name of the route (e.g., `simple-webapp-route`) |
| Hostname | The external host (leave blank for auto-generated host) |
| Path | Optional URL path to match |
| Service | The internal Service to route to (e.g., `simple-webapp-docker`) |
| Target Port | The port on the Service to forward traffic to (e.g., `8080`) |

If you specify a custom hostname (for example `www.example.com`), ensure your external DNS points that hostname to the OpenShift router. Once traffic reaches the cluster, Routes direct it to the correct Service based on hostname/path.

Leave other fields at their defaults and create the Route. After creation you will see the generated hostname for the route in the console.

![Screenshot of the OpenShift Origin web console showing the "simple-webapp-docker" service details, route and traffic info, and a running pod. A green notification at the top-right indicates the route was successfully created.](https://mintcdn.com/kodekloud-c4ac6d9a/Xa5GerWBw0vsOVlC/images/OpenShift-4/Networks-Services-Routes-and-Scaling/Demo-Services-and-Routes/openshift-route-created-simple-webapp-docker.jpg?fit=max\&auto=format\&n=Xa5GerWBw0vsOVlC\&q=85\&s=3c1fdd3d82cab127dd295d35c4934ef0)

Clicking the route URL opens the application in a browser. If you deploy an update (for example change the message from "Update 4" to "Update 5") and push your code, the BuildConfig will trigger a new build and OpenShift will deploy the new image. The updated UI will be visible at the same route URL after the deployment completes.

## Quick reference and commands

* Create a Service:
  * Console: Import YAML / Create → paste YAML → Create
  * CLI: `oc apply -f service-config.yaml`
* Verify Service:
  * `oc get svc`
  * `oc describe svc simple-webapp-docker`
* Create a Route:
  * Console: Applications → Routes → Create Route
  * CLI example: `oc expose svc/simple-webapp-docker --hostname=your.host.example.com`

## Links and references

* [OpenShift Routes documentation](https://docs.openshift.com/container-platform/latest/networking/routes/route-basics.html)
* [OpenShift Services documentation](https://docs.openshift.com/container-platform/latest/networking/service_networking/about-services.html)
* [Kubernetes Services overview](https://kubernetes.io/docs/concepts/services-networking/service/)

