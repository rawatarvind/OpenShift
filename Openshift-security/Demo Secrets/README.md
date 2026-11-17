# Demo Secrets

> Guide to creating and consuming Kubernetes/OpenShift Secrets via manifests and the OpenShift console, with deployment examples and best practices for handling sensitive values

This guide shows two common ways to create Kubernetes/OpenShift Secrets (manifest + web console), how to consume a Secret from a Deployment, and a few best practices for handling sensitive values.

Overview:

* Create a Secret from a YAML manifest (preferred for reproducibility and GitOps).
* Create a Secret via the OpenShift web console (quick, GUI-driven).
* Consume a Secret from a Deployment using `env.valueFrom.secretKeyRef`.

Quick comparison

| Method | When to use | Example / CLI |
| - | - | - |
| Manifest | Reproducible, source-controlled (encrypt/seal before committing) | `oc apply -f secret.yaml` |
| CLI (ad-hoc) | Scripted or interactive creation | `oc create secret generic my-secret --from-literal=key=value` |
| Web console | Quick GUI creation and inspection | Workloads → Secrets → Create Secret |
| Source secret (builds) | Connect builds to external repos (use encrypted values) | Use Console option: Source secret |

Resources and references:

* [Kubernetes Secrets](https://kubernetes.io/docs/concepts/configuration/secret/)
* [OpenShift CLI (oc) documentation](https://docs.openshift.com/)
* [Sealed Secrets](https://github.com/bitnami-labs/sealed-secrets) — for encrypting secrets for Git

***

## 1) Create a Secret manifest

Below is a minimal Secret manifest. Use `stringData` to supply plain-text values; Kubernetes will convert these to base64 and store them under `data`. Alternatively you can manually include base64-encoded values under `data`.

```yaml theme={null}
apiVersion: v1
kind: Secret
metadata:
  namespace: default
  name: dbpassword
type: Opaque
stringData:
  MONGODB_PASSWORD: password
```

Apply the manifest with the OpenShift CLI:

```bash theme={null}
oc apply -f secret.yaml
```

To remove it (for testing the console flow, for example):

```bash theme={null}
oc delete -f secret.yaml
```

When creating a Secret manifest you define:

* `apiVersion` (core/v1)
* `kind: Secret`
* `metadata` (for example `namespace` and `name`)
* `type` (for example `Opaque`, `kubernetes.io/dockerconfigjson`, etc.)
* secret values via `data` (base64) or `stringData` (plain-text)

Secrets and ConfigMaps share a similar key/value structure in YAML, but Secrets are intended for sensitive data and stored base64-encoded in `data`.

***

## 2) Create a Secret in the OpenShift web console

In the OpenShift console navigate to Workloads → Secrets and click Create Secret. You will see options such as YAML, Webhook secret, Source secret (from Git), Image pull secret, and Key/value secret.

![A screenshot of the Red Hat OpenShift web console showing the "Secrets" list for the default project, with several Kubernetes secrets displayed. The Create menu is open, showing secret types such as Key/value secret, Image pull secret, Source secret, and Webhook secret.](https://mintcdn.com/kodekloud-c4ac6d9a/_-d-o-x8WFPLT_DD/images/OpenShift-4/OpenShift-Security/Demo-Secrets/openshift-console-secrets-create-menu.jpg?fit=max\&auto=format\&n=_-d-o-x8WFPLT_DD\&q=85\&s=46dbec308c99ad42ec2a0761ca57c7c7)

Source secrets are used to connect builds to external repositories; avoid storing plain secrets in Git unless the values are encrypted or sealed. For a simple key/value secret:

* Choose "Key/value secret"
* Set a name (for example `dbpassword`)
* Add key `MONGODB_PASSWORD`
* Paste the password (or upload a file)
* Click Create

After creation you can inspect details or download the YAML for portability/automation.

![A screenshot of the Red Hat OpenShift web console showing the Secret details page for a secret named "dbpassword" in the default project. The page displays metadata like type (Opaque), namespace, and buttons to add the secret to a workload.](https://mintcdn.com/kodekloud-c4ac6d9a/_-d-o-x8WFPLT_DD/images/OpenShift-4/OpenShift-Security/Demo-Secrets/redhat-openshift-secret-dbpassword-details.jpg?fit=max\&auto=format\&n=_-d-o-x8WFPLT_DD\&q=85\&s=a6b9bd931dbb4787decc440ce1faa46d)

Using `stringData` in a Secret manifest lets you write plain-text values in YAML; Kubernetes writes base64-encoded entries into `data`. If you prefer, place base64-encoded values manually under `data` instead of using `stringData`.

You can inspect a Secret with the CLI, and decode a stored value:

* Show the secret YAML:
  * `oc get secret dbpassword -o yaml`
* Print and decode a single key:
  * `oc get secret dbpassword -o jsonpath="{.data.MONGODB_PASSWORD}" | base64 --decode && echo`

***

## 3) Consume the Secret in a Deployment

Below is a sample Deployment manifest that consumes the `dbpassword` Secret via an environment variable. Note the `valueFrom.secretKeyRef` that references the Secret name and key — the Deployment manifest does not contain the plaintext password.

```yaml theme={null}
apiVersion: apps/v1
kind: Deployment
metadata:
  name: carts-db
  namespace: default
  labels:
    name: carts-db
spec:
  replicas: 1
  selector:
    matchLabels:
      name: carts-db
  template:
    metadata:
      labels:
        name: carts-db
    spec:
      containers:
      - name: carts-db
        image: centos/mongodb-34-centos7
        resources:
          requests:
            memory: "100Mi"
        ports:
        - name: mongo
          containerPort: 27017
        env:
        - name: MONGODB_USER
          value: sock-user
        - name: MONGODB_PASSWORD
          valueFrom:
            secretKeyRef:
              name: dbpassword
              key: MONGODB_PASSWORD
        - name: MONGODB_DATABASE
          value: data
        - name: MONGODB_ADMIN_PASSWORD
          value: admin
        volumeMounts:
        - mountPath: /tmp
          name: tmp-volume
      volumes:
      - name: tmp-volume
        emptyDir: {}
```

Apply the Deployment:

```bash theme={null}
oc apply -f deployment.yaml
```

Useful commands to inspect the Deployment and Pod(s):

```bash theme={null}
oc get pods -l name=carts-db
oc describe pod <pod-name>
oc get deployment carts-db -o yaml
```

When viewing the Pod/Deployment YAML (console or `oc get -o yaml`) you will see the environment variable configured to pull its value from the Secret — the actual secret value is not displayed in plaintext in the Deployment manifest.

Secrets are base64-encoded, not encrypted, by default in etcd. For stronger protection, integrate a secrets management solution (for example Sealed Secrets, HashiCorp Vault, or OpenShift's encryption-at-rest) and avoid committing plain secrets to Git. See linked resources above for options.

