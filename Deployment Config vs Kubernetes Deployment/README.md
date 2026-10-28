# Deployment Config vs Kubernetes Deployment

> Comparison of Kubernetes Deployment and OpenShift DeploymentConfig, their controllers, selectors, triggers, example manifests, and recommendations for when to use each.

In Kubernetes you typically use a Deployment (which manages ReplicaSets) to create and maintain Pods. OpenShift supports standard Kubernetes Deployments, but it also provides its own object type: the DeploymentConfig. This page compares both objects, shows example manifests, and summarizes when to use each.

## Example manifests

Below are two example manifests: a standard Kubernetes Deployment and an OpenShift DeploymentConfig. Note the different `apiVersion` and `kind` for the OpenShift-specific object.

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
---
apiVersion: apps.openshift.io/v1
kind: DeploymentConfig
metadata:
  name: example
  namespace: mikel1992-dev
spec:
  selector:
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

DeploymentConfig is an OpenShift-specific API object (provided by OpenShift's API extensions) and is implemented by OpenShift's controllers and operators.

## What does a Deployment do?

* A Deployment is a Kubernetes controller that ensures a desired number of Pods from a Pod template are running.
* You declare the replica count (for example `replicas: 3`) and the Deployment controller uses a ReplicaSet to maintain that count.
* The Deployment spec contains the Pod template where you define container images, ports, volumes, environment variables, and other container-level settings.
* When you update the Pod template (for example, change the container image), the Deployment controller creates a new ReplicaSet and performs a controlled rollout (rolling update by default).

Common commands:

* `kubectl apply -f deployment.yaml`
* `kubectl rollout status deployment/example`
* `kubectl rollout history deployment/example`

## What does a DeploymentConfig do?

* A DeploymentConfig is conceptually similar: it stores a Pod template and the desired replica count, and its controller creates ReplicationController objects to realize that desired state.
* DeploymentConfig supports OpenShift-specific triggers that can start a new deployment automatically when something changes — for example:
  * `imageChange` triggers hooked to ImageStreams.
  * `configChange` triggers when the DeploymentConfig template or strategy changes.
* DeploymentConfig supports several deployment strategies (Rolling, Recreate, Custom), and can be managed via the `oc` CLI in OpenShift.

Common OpenShift commands:

* `oc apply -f deploymentconfig.yaml`
* `oc rollout status dc/example`
* `oc tag` / `oc import-image` (for ImageStream workflows)

![A side-by-side infographic showing "Deployment" vs "DeploymentConfig" with circular segmented charts and a central rocket icon. Each side lists roles like controller/template for a pod, specifying container specs, replicas, and managing image or configuration changes.](https://mintcdn.com/kodekloud-c4ac6d9a/Xa5GerWBw0vsOVlC/images/OpenShift-4/Concepts-Builds-and-Deployments/Deployment-Config-vs-Kubernetes-Deployment/deployment-vs-deploymentconfig-infographic.jpg?fit=max\&auto=format\&n=Xa5GerWBw0vsOVlC\&q=85\&s=456230ef0d1e2fa185693ef7f57d2f80)

## Key differences

* Controller types
  * Kubernetes Deployment → creates ReplicaSets.
  * OpenShift DeploymentConfig → creates ReplicationControllers.
* Selector capabilities
  * ReplicaSet (used by Deployments) supports set-based selector requirements via `matchExpressions` as well as `matchLabels`.
  * ReplicationController (used by DeploymentConfig) supports only equality-based selectors (label equality via `matchLabels`).
* Update/triggers and strategies
  * Deployments perform rollouts (rolling update by default) when the Pod template changes. Kubernetes provides rollout commands (`kubectl rollout`) and declarative updates (`kubectl apply`) to manage this.
  * DeploymentConfig supports additional OpenShift triggers (notably `imageChange` triggers tied to ImageStreams and `configChange`) that can automatically trigger a new deployment when an ImageStream or config changes. DeploymentConfig also supports multiple deployment strategies (Rolling, Recreate, Custom).

### Quick comparison table

| Aspect | Kubernetes Deployment | OpenShift DeploymentConfig |
| - | - | - |
| Controller created | `ReplicaSet` | `ReplicationController` |
| Selector syntax | `matchLabels`, `matchExpressions` | `matchLabels` only |
| Automatic image triggers | Not built-in (use external pipelines or controllers) | `imageChange` triggers tied to ImageStreams |
| CLI tooling | `kubectl` (`kubectl apply`, `kubectl rollout`) | `oc` (integration with Builds and ImageStreams) |
| Portability | Portable across Kubernetes distributions | Tied to OpenShift APIs and extensions |
| Use-case fit | Standard Kubernetes workloads, portability | OpenShift-specific workflows (ImageStreams, Builds) |

Both controllers ensure the declared replica count. If you declare `replicas: 3` and two Pods fail, the ReplicaSet or ReplicationController will start new Pods to reach the declared count.

Use a standard Kubernetes Deployment unless you need OpenShift-only features such as `imageChange` triggers tied to ImageStreams or tighter integration with OpenShift Builds and the `oc` workflow.

## Recommendation

* Choose Kubernetes Deployments for portability and compatibility with upstream Kubernetes tooling and CI/CD pipelines.
* Choose DeploymentConfig when you require OpenShift-native capabilities: ImageStream-driven workflows, automatic image triggers, or an existing OpenShift Build/Deploy pipeline.

## Summary

* Deployments and DeploymentConfigs both manage Pod templates and scaling.
* They differ in the underlying controller (ReplicaSet vs ReplicationController), selector semantics, and integration with OpenShift features.
* For most use cases, prefer Kubernetes Deployments for portability; use DeploymentConfig when you specifically require OpenShift features.

## Links and references

* [Kubernetes Deployments documentation](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/)
* [OpenShift DeploymentConfig documentation](https://docs.openshift.com/container-platform/latest/nodes/deployment_config.html)
* [OpenShift ImageStreams](https://docs.openshift.com/container-platform/latest/openshift_images/image_streams.html)

