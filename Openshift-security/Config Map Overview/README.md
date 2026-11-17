# Config Map Overview

> Overview of Kubernetes ConfigMaps in OpenShift 4, storing non-sensitive key value configuration and how to create, manage, and consume them using declarative or imperative workflows

ConfigMaps in OpenShift 4 behave exactly like ConfigMaps in Kubernetes. Use the same API and commands (`oc` or `kubectl`) to create, update, and consume them—no OpenShift-specific changes are required for standard ConfigMap workflows.

ConfigMaps are a Kubernetes namespaced resource for storing non-confidential configuration data as key-value pairs. They let you inject configuration into containers at runtime without baking that configuration into your container images or Deployment manifests. This improves reuse and separation of configuration from code or images.

A few common patterns for ConfigMap usage:

* Provide environment variables to Pods (`env` or `envFrom`).
* Mount configuration files into containers as a volume.
* Store small runtime configuration snippets consumed by applications.

![A presentation slide titled "ConfigMap" with a graphic of red stacked blocks labeled "Environment Variables," "Configuration Data," and "Parameters" next to a gear/circuit icon. The left side shows vertical red buttons with chevrons and the caption "Add Environment Variables and Config Data."](https://mintcdn.com/kodekloud-c4ac6d9a/_-d-o-x8WFPLT_DD/images/OpenShift-4/OpenShift-Security/Config-Map-Overview/configmap-environment-variables-config-data.jpg?fit=max\&auto=format\&n=_-d-o-x8WFPLT_DD\&q=85\&s=7d4ced871c9388a051cb00dfe1602b6c)

What a ConfigMap stores

* Stored under the `data` field as string key-value pairs.
* Not intended for sensitive values—use Kubernetes Secrets for passwords, tokens, and other sensitive items.

By default, ConfigMaps are stored in etcd in plaintext unless encryption-at-rest is enabled. Any user with RBAC read permissions in the namespace can view them. Never store secrets in ConfigMaps; use Kubernetes Secrets and protect them with appropriate RBAC and encryption-at-rest.

## Ways to create and inspect ConfigMaps

Below are the recommended workflows and quick-create options.

1. Declarative YAML manifest (recommended for source control)

Example ConfigMap manifest:

```yaml theme={null}
apiVersion: v1
kind: ConfigMap
metadata:
  name: my-config
data:
  username: "appuser"
  database: "prod_db"
  log_level: "info"
```

Apply it:

```bash theme={null}
oc apply -f my-configmap.yaml
# or
kubectl apply -f my-configmap.yaml
```

2. Imperative (quick/one-off) commands

Create from literals:

```bash theme={null}
oc create configmap my-config \
  --from-literal=username=appuser \
  --from-literal=database=prod_db

# kubectl equivalent
kubectl create configmap my-config \
  --from-literal=username=appuser \
  --from-literal=database=prod_db
```

Create from a file (file contents become the value for the key named after the file):

```bash theme={null}
oc create configmap my-config --from-file=app.conf=/path/to/app.conf
# or from an env file with KEY=VALUE lines
oc create configmap my-config --from-env-file=env.list
```

3. Inspecting ConfigMaps

List ConfigMaps in the current namespace:

```bash theme={null}
oc get configmaps
# or
kubectl get configmaps
```

Describe a specific ConfigMap:

```bash theme={null}
oc describe configmap my-config
# or
kubectl describe configmap my-config
```

Show the full YAML for a ConfigMap:

```bash theme={null}
oc get configmap my-config -o yaml
# or
kubectl get configmap my-config -o yaml
```

Table: Creation Methods at a Glance

| Method | Use case | Example command |
| - | - | - |
| Declarative YAML | Source control, repeatable deployments | `oc apply -f my-configmap.yaml` |
| Imperative literals | Quick one-off values | `oc create configmap my-config --from-literal=key=value` |
| From file/env file | Large file values or many keys | `oc create configmap my-config --from-file=app.conf=/path/to/app.conf` |

## How to consume a ConfigMap in a Pod

* Environment variables using `envFrom` (imports all keys as env vars):

```yaml theme={null}
apiVersion: v1
kind: Pod
metadata:
  name: example-pod
spec:
  containers:
    - name: app
      image: nginx:alpine
      envFrom:
        - configMapRef:
            name: my-config
```

* Single env var from a specific key (explicit mapping):

```yaml theme={null}
env:
  - name: DB_NAME
    valueFrom:
      configMapKeyRef:
        name: my-config
        key: database
```

* Mount as a volume (expose keys as files at mountPath):

```yaml theme={null}
apiVersion: v1
kind: Pod
metadata:
  name: example-pod
spec:
  containers:
    - name: app
      image: nginx:alpine
      volumeMounts:
        - name: config-volume
          mountPath: /etc/config
  volumes:
    - name: config-volume
      configMap:
        name: my-config
```

Notes on consumption

* Environment injection reads values as strings (no types).
* When mounted as files, each key becomes a file with the key name and the file contents equal to the key's value.
* Updates to ConfigMaps will reflect in environment variables only after Pod restart; mounted volumes can reflect updates automatically depending on kubelet sync behavior.

## Summary

* ConfigMaps store non-sensitive configuration as key-value pairs and are namespaced Kubernetes resources.
* Create them declaratively (recommended) or imperatively for quick tasks.
* Consume ConfigMaps as environment variables (`env`, `envFrom`) or mounted files (volumes).
* Use Kubernetes Secrets (not ConfigMaps) for sensitive data and protect secrets using RBAC and encryption-at-rest.

## Links and references

* [Kubernetes Documentation - ConfigMap](https://kubernetes.io/docs/concepts/configuration/configmap/)
* [OpenShift 4 course on KodeKloud](https://learn.kodekloud.com/user/courses/openshift-4)
* [Kubernetes for the Absolute Beginners - Hands-on Tutorial on KodeKloud](https://learn.kodekloud.com/user/courses/kubernetes-for-the-absolute-beginners-hands-on-tutorial)

