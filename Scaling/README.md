# Scaling

> Explains how to scale applications in OpenShift and Kubernetes using Deployments, ReplicaSets, CLI, web console, and Horizontal Pod Autoscalers with examples and best practices.

This article explains how to scale applications in OpenShift and Kubernetes.

A Deployment is the common higher-level controller used to manage application replicas. A Deployment creates and manages a ReplicaSet (and previously a ReplicationController) which ensures the desired number of Pod replicas exist. To scale an application, you change the desired replica count in the controller’s spec; the ReplicaSet will then create or remove Pods to match that target.

Key concepts:

* Deployment: declarative controller that manages ReplicaSets and rolling updates.
* ReplicaSet: ensures a specific number of Pod replicas run.
* Service / Router: distributes traffic to Pods (internal via Service, external via OpenShift Router).

You can change the replica count in several common ways:

* Edit the Deployment YAML and update `spec.replicas`.
* Use command-line shortcuts such as `oc scale` or `kubectl scale`.
* Use the OpenShift web console for one-click scaling.
* Configure automatic scaling with a Horizontal Pod Autoscaler (HPA) to scale based on CPU, memory, or custom metrics.

![A slide titled "Scale Deployment" showing a diagram of a deployment/replication controller with stacked pod/container icons on the left and, on the right, a webapp dashboard for a "simple-webapp" deployment displaying details and a circle indicating 5 pods.](https://mintcdn.com/kodekloud-c4ac6d9a/Xa5GerWBw0vsOVlC/images/OpenShift-4/Networks-Services-Routes-and-Scaling/Scaling/scale-deployment-simple-webapp-5-pods.jpg?fit=max\&auto=format\&n=Xa5GerWBw0vsOVlC\&q=85\&s=ebb864e989f2b1fc0aeed8b13090148f)

Practical examples

* Example Deployment YAML (relevant parts shown):

```yaml theme={null}
apiVersion: apps/v1
kind: Deployment
metadata:
  name: simple-webapp
spec:
  replicas: 3
  selector:
    matchLabels:
      app: simple-webapp
  template:
    metadata:
      labels:
        app: simple-webapp
    spec:
      containers:
      - name: web
        image: nginx:1.21
        ports:
        - containerPort: 80
```

* Scale from the command line:

```bash theme={null}
# Scale to 5 replicas (OpenShift)
oc scale deployment/simple-webapp --replicas=5

# Or with kubectl
kubectl scale deployment/simple-webapp --replicas=5
```

* Query the current state:

```bash theme={null}
# Check the Deployment and its replica count
oc get deployment simple-webapp

# List Pods created by the ReplicaSet
oc get pods -l app=simple-webapp
```

* Enable automatic scaling with a Horizontal Pod Autoscaler (HPA):

```bash theme={null}
# Create an HPA that scales between 2 and 10 replicas targeting 80% CPU
oc autoscale deployment/simple-webapp --min=2 --max=10 --cpu-percent=80
```

Summary table — scaling methods

| Method | When to use | Example / Command |
| - | - | - |
| Manual edit (YAML) | Declarative change checked into Git / CI | Edit `spec.replicas` in the Deployment manifest |
| CLI scale | Quick, ad-hoc changes or scripts | `oc scale deployment/simple-webapp --replicas=5` |
| Web console | One-off interactive scaling | Use the OpenShift console’s scale control |
| HPA (autoscaling) | Scale based on CPU/memory/custom metrics | `oc autoscale deployment/simple-webapp --min=2 --max=10 --cpu-percent=80` |

Links and references

* [OpenShift Scaling and Autoscaling](https://docs.openshift.com/)
* [Kubernetes Deployments](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/)
* [Kubernetes Horizontal Pod Autoscaler](https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale/)

Using the OpenShift web console is the quickest way for interactive scaling. For reproducible automation and CI/CD, prefer `oc`/`kubectl` commands or defining an HPA in YAML and managing it via GitOps.

Before scaling up, verify your project/cluster has sufficient resource quotas (CPU, memory) and available nodes. When scaling down, be careful with stateful workloads—Pods may be terminated. Use PodDisruptionBudgets and safe shutdown procedures for stateful applications.



