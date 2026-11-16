# Services and Routes

> Explains OpenShift services and routes, covering stable service endpoints, load balancing to pods, external exposure via routes, TLS termination, and traffic splitting for rollouts.

Hello and welcome. In this lesson we'll cover services and routes in OpenShift: what they are, how they connect and load-balance traffic to pods, how to expose services externally with routes, how to secure routes with TLS, and how to split traffic for rolling or A/B deployments.

Services provide stable network endpoints for groups of pods. Instead of depending on the ephemeral IP address of an individual pod, clients communicate with a service (stable IP and DNS) that load balances to the matching pods. Routes expose services outside the cluster using friendly hostnames, TLS termination, and traffic-splitting features.

Use services to decouple clients from the lifecycle of pods. Configure a route when you need an application reachable at a DNS name (for example `www.somewebapp.com`) or when you need TLS termination and traffic control.

## Why use Services?

* Provide a stable ClusterIP and DNS name for intra-cluster communication.
* Act as a simple load balancer to forward traffic to matching pods.
* Let you change or redeploy pods without updating client configuration.
* Enable exposing an application to external users via a route.

Typical use cases:

* Frontend → Backend via a service
* Backend → Data-processing component via a service
* Exposing an application to end users via a service + route
* Accessing external databases via services

A service includes:

* A stable cluster-internal IP and DNS (ClusterIP)
* Selectors that choose which pods receive traffic (e.g. `deploymentconfig=simple-webapp-docker`)
* One or more service ports and target ports on the pods

![An architecture diagram and dashboard screenshot labeled "Service" for simple-webapp-docker. It shows a ClusterIP (172.30.85.124) exposing port 8080 and routing traffic to backend containers (172.17.0.7) using selector deploymentconfig=simple-webapp-docker.](https://mintlify.s3.us-west-1.amazonaws.com/kodekloud-c4ac6d9a/images/OpenShift-4/Networks-Services-and-Scaling/Services-and-Routes/simple-webapp-docker-service-clusterip-8080.jpg)

## Example: Service selector and port mapping

Here is a minimal Service YAML that maps service port 8080 to the pod target port 8080 and selects pods with `deploymentconfig=simple-webapp-docker`:

```yaml theme={null}
apiVersion: v1
kind: Service
metadata:
  name: simple-webapp
spec:
  selector:
    deploymentconfig: simple-webapp-docker
  ports:
    - name: http
      port: 8080
      targetPort: 8080
  type: ClusterIP
```

To view services and ClusterIP on your cluster:

* List services: `oc get svc`
* Describe a service: `oc describe svc simple-webapp`

## Service types (when you need them)

| Service Type | Use Case |
| - | - |
| `ClusterIP` | Default. Internal access only — stable IP and DNS for intra-cluster clients. |
| `NodePort` | Exposes a port on every node. Useful for debugging or simple access without routes. |
| `LoadBalancer` | Request a cloud provider’s load balancer (if supported). For external access in cloud environments. |

## Routes — expose services externally

A route provides a hostname (for example `www.somewebapp.com`) and proxies external requests to a service. Routes are similar to having an external load balancer or proxy (e.g., HAProxy) in front of the service; OpenShift’s router implements this.

Routes support:

* Hostname-based exposure (DNS → router → service)
* Load balancing between pods behind the service
* TLS termination (edge, passthrough, re-encrypt)
* Behavior for insecure HTTP traffic (allow, redirect, or disable)
* Traffic splitting across multiple services for staged rollouts

Load balancing strategies supported by the router:

| Strategy | Description |
| - | - |
| `source` | Sticky sessions based on client IP (OpenShift default). Keeps a client routed to the same backend for a session. |
| `round-robin` | Evenly cycles requests across backends. |
| `leastconn` | Directs traffic to the backend with the fewest active connections. |

![A diagram titled "Route – Load Balancing" showing user requests going into a Route load balancer. The balancer (roundrobin / leastconn) distributes traffic to backend Docker webapp containers (simple-webapp-docker, 172.17.0.7, port 8080).](https://mintlify.s3.us-west-1.amazonaws.com/kodekloud-c4ac6d9a/images/OpenShift-4/Networks-Services-and-Scaling/Services-and-Routes/route-load-balancer-roundrobin-docker-webapp.jpg)

### Example: route basic spec

A simple Route to expose the `simple-webapp` service at `www.somewebapp.com`:

```yaml theme={null}
apiVersion: route.openshift.io/v1
kind: Route
metadata:
  name: simple-webapp-route
spec:
  host: www.somewebapp.com
  to:
    kind: Service
    name: simple-webapp
  tls:
    termination: edge
```

To list routes: `oc get route`\
To describe a route: `oc describe route simple-webapp-route`

## Route security and TLS termination

Routes let you terminate TLS at the router (edge), pass TLS through (passthrough), or re-encrypt (router decrypts and re-encrypts to the backend). You can also set how to handle insecure traffic via the `insecureEdgeTerminationPolicy` (`Allow`, `Redirect`, `None`).

Common TLS options:

* edge — TLS terminates at the router (provide cert/key on the route)
* passthrough — TLS passes directly to the pod; backend must handle TLS
* re-encrypt — TLS terminates on the router, then is re-encrypted to the backend

![A presentation slide titled "Route - Security" showing a diagram of users hitting a web app route that forwards to a service/pod on port 8080. On the right is a screenshot of a security/TLS configuration panel with TLS termination and insecure traffic options.](https://mintlify.s3.us-west-1.amazonaws.com/kodekloud-c4ac6d9a/images/OpenShift-4/Networks-Services-and-Scaling/Services-and-Routes/route-security-tls-termination-insecure-8080.jpg)

When terminating TLS at the router, ensure certificates are valid and private keys are protected. If you use `passthrough`, backend pods must be configured to serve TLS and handle certificates securely.

Example snippet of a route with TLS termination and insecure redirect:

```yaml theme={null}
apiVersion: route.openshift.io/v1
kind: Route
metadata:
  name: secure-webapp
spec:
  host: secure.example.com
  to:
    kind: Service
    name: simple-webapp
  tls:
    termination: edge
    insecureEdgeTerminationPolicy: Redirect
    certificate: |-
      -----BEGIN CERTIFICATE-----
      ...
      -----END CERTIFICATE-----
    key: |-
      -----BEGIN PRIVATE KEY-----
      ...
      -----END PRIVATE KEY-----
```

## Traffic splitting (A/B testing, canary releases)

Routes support splitting traffic across multiple services using `to` and `alternateBackends` with `weight` values. This is useful for gradual rollouts and A/B testing.

![A diagram titled "Route — Split Traffic" showing users hitting a web route that splits traffic between two backend services (simple-webapp-docker and simple-webapp-docker-new). On the right is a screenshot of an "Alternate Services" panel with service weights set to 50%/50%.](https://mintlify.s3.us-west-1.amazonaws.com/kodekloud-c4ac6d9a/images/OpenShift-4/Networks-Services-and-Scaling/Services-and-Routes/route-split-traffic-50-50.jpg)

Example route that splits traffic 50/50 between `simple-webapp` and `simple-webapp-new`:

```yaml theme={null}
apiVersion: route.openshift.io/v1
kind: Route
metadata:
  name: split-webapp
spec:
  host: split.example.com
  to:
    kind: Service
    name: simple-webapp
    weight: 50
  alternateBackends:
    - kind: Service
      name: simple-webapp-new
      weight: 50
```

Adjust weights to shift traffic gradually from one service to another as you validate the new version.

## Quick commands reference

| Action | Command |
| - | - |
| List services | `oc get svc` |
| Describe service | `oc describe svc <service-name>` |
| List routes | `oc get route` |
| Describe route | `oc describe route <route-name>` |

## Links and references

* [OpenShift Routes documentation](https://docs.openshift.com/)
* [Kubernetes Services](https://kubernetes.io/docs/concepts/services-networking/service/)
* [OpenShift networking overview](https://docs.openshift.com/container-platform/latest/networking/)

Next, we'll demonstrate how services and routes work in a live example and walk through applying the service/route manifests.

