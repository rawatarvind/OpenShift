# Route overview

> Overview of OpenShift Routes that expose cluster Services via hostnames, explain request flow, TLS termination options, and example Route manifest for external access

When a user types a URL on a laptop, tablet, or phone, that request must be routed to the correct application endpoint — or to an error page when something goes wrong. In OpenShift, a Route is the object that exposes an in-cluster Service under a publicly reachable hostname (for example, `www.example.com`).

![A simple diagram showing a desktop computer with a user icon connected through a red "URL" block to two endpoints: a warning/alert symbol and a terminal/command icon. It appears to illustrate a URL routing user requests to error handling and command/execution paths.](https://mintcdn.com/kodekloud-c4ac6d9a/Xa5GerWBw0vsOVlC/images/OpenShift-4/Concepts-Builds-and-Deployments/Route-overview/user-url-routing-error-command.jpg?fit=max\&auto=format\&n=Xa5GerWBw0vsOVlC\&q=85\&s=c5132702aec28f8a22f80ff0de808157)

Typical request flow for an OpenShift Route:

* A client requests a hostname (the URL).
* The request reaches the cluster router (OpenShift's HAProxy-based router / load balancer).
* The router matches the hostname (and optional path or TLS settings) to a Route object.
* The Route forwards the request to a Service.
* The Service load-balances traffic to one or more Pods (the containers running your application).

Here is a minimal example Route manifest that maps `www.example.com` to a service named `example-service` and terminates TLS at the router (edge termination):

```yaml theme={null}
apiVersion: route.openshift.io/v1
kind: Route
metadata:
  name: example-route
spec:
  host: www.example.com
  to:
    kind: Service
    name: example-service
  port:
    targetPort: 8080
  tls:
    termination: edge
```

OpenShift Routes are the platform-native way to expose Services externally. They provide hostname-based routing, multiple TLS termination modes, and custom annotations for advanced behavior. Use Routes when you need tight integration with OpenShift's router features and when you want simple, declarative exposure of Services.

TLS termination options for Routes

| Termination Type | Description | Typical use case |
| - | -: | - |
| `edge` | TLS is terminated at the router; traffic between router and backend is plain HTTP. | When you trust the internal network or want the router to handle TLS. |
| `re-encrypt` | Router terminates TLS and then re-encrypts to the backend (requires a backend certificate). | When you need end-to-end encrypted traffic but want the router to present a public cert. |
| `passthrough` | Router passes TLS through to the backend without terminating it. | When the backend must handle TLS (for example, mutual TLS or SNI-based routing). |

Why Routes vs Kubernetes Ingress / LoadBalancer

* From a Kubernetes viewpoint, Routes serve the same purpose as placing an Ingress or external LoadBalancer in front of a Service: they expose in-cluster Services to external clients.
* OpenShift Routes are an OpenShift-native construct with built-in router integration and additional features (TLS modes, annotations, host-based behavior) that simplify many typical exposure patterns.

Further reading and references:

* OpenShift Routes docs: [https://docs.openshift.com/container-platform/latest/networking/routes/understanding-ingress.html](https://docs.openshift.com/container-platform/latest/networking/routes/understanding-ingress.html)
* Kubernetes Ingress overview: [https://kubernetes.io/docs/concepts/services-networking/ingress/](https://kubernetes.io/docs/concepts/services-networking/ingress/)
