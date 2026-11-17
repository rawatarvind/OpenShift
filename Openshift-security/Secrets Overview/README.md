# Secrets Overview

> Explains Kubernetes Secrets, what they store, how to create and consume them, storage and security risks, and best practices like encryption and external secret managers.

At some point you'll need to pass a password, connection string, API token, or other sensitive value into a pod (or multiple pods). In Kubernetes, that object is called a Secret.

Secrets are intended for small pieces of sensitive data such as:

* Passwords
* API keys or tokens
* Database connection strings
* Usernames or short configuration strings (it can even be a simple `hello world` string)

You can store essentially any small piece of sensitive data as a Secret.

![A minimalist UI mockup showing a blue key icon next to a rounded label reading "Store Sensitive info." Below it is a panel with red rounded buttons labeled "User," "Anything," "Hello!?," and "You Got Options!"](https://mintcdn.com/kodekloud-c4ac6d9a/_-d-o-x8WFPLT_DD/images/OpenShift-4/OpenShift-Security/Secrets-Overview/store-sensitive-key-ui-buttons.jpg?fit=max\&auto=format\&n=_-d-o-x8WFPLT_DD\&q=85\&s=7193b30d6b9b691a5de4e954ffab8887)

Key facts about Kubernetes Secrets

* Default type: `Opaque`.
* Storage: Secret data is stored in the API as base64-encoded values. Base64 is an encoding, not encryption.
* Persistence: Secret objects are stored in [etcd](https://etcd.io/).
* Access: Anyone with direct etcd access or API access with sufficient RBAC privileges can read them unless you enable encryption or restrict access.

For production clusters you should enable Kubernetes [encryption at rest](https://kubernetes.io/docs/tasks/administer-cluster/encrypt-data/) (for example, envelope encryption of secrets in [etcd](https://etcd.io/)) or integrate an external secrets manager such as [HashiCorp Vault](https://www.vaultproject.io/), [AWS Secrets Manager](https://aws.amazon.com/secrets-manager/), or [Azure Key Vault](https://azure.microsoft.com/en-us/services/key-vault/). Even when using an external manager, understanding how Kubernetes Secrets work and how to create and consume them is important.

When a Secret is made available to a pod (mounted as files or injected as environment variables), the secret data is presented to the container in plain text at runtime. Because of these trade-offs, many production deployments combine Kubernetes Secrets with encryption at rest, strict RBAC, audit logging, and external secret managers.

![A blue square icon with a white key is shown on the left, and to the right are rounded tiles displaying the HashiCorp Vault, AWS, and Kubernetes logos.](https://mintcdn.com/kodekloud-c4ac6d9a/_-d-o-x8WFPLT_DD/images/OpenShift-4/OpenShift-Security/Secrets-Overview/key-icon-vault-aws-kubernetes-tiles.jpg?fit=max\&auto=format\&n=_-d-o-x8WFPLT_DD\&q=85\&s=0c3fe1e99c74c796646053f3801313cc)

Treat Kubernetes Secrets as sensitive. By default they are base64-encoded only. For production, enable encryption at rest and/or use an external secrets manager (Vault, AWS Secrets Manager, Azure Key Vault, etc.).

Common ways to create and consume Secrets

1. Create a Secret from literals (CLI)

```bash theme={null}
# Create a generic secret with a single key "password"
kubectl create secret generic my-secret --from-literal=password='s3cr3t'
```

2. Create a Secret from files (CLI)

```bash theme={null}
# Create a TLS-style Secret from certificate and key files
kubectl create secret tls tls-secret --cert=path/to/tls.crt --key=path/to/tls.key

# Or a generic secret from files, mapping filenames to keys
kubectl create secret generic app-creds --from-file=username=./username.txt --from-file=password=./password.txt
```

3. Secret manifest (YAML)

```yaml theme={null}
apiVersion: v1
kind: Secret
metadata:
  name: my-secret
type: Opaque
data:
  # values must be base64-encoded
  username: bXktdXNlcg==
  password: c3VwZXItc2VjcmV0
```

4. Inspecting and decoding secrets

```bash theme={null}
# View secret metadata and base64-encoded values
kubectl get secret my-secret -o yaml

# Decode a value locally
echo 'c3VwZXItc2VjcmV0' | base64 --decode
```

How pods consume Secrets

* Environment variables (individual keys)

```yaml theme={null}
apiVersion: v1
kind: Pod
metadata:
  name: secret-env-pod
spec:
  containers:
    - name: app
      image: busybox
      command: ["sh", "-c", "echo SECRET_USERNAME=$SECRET_USERNAME; sleep 3600"]
      env:
        - name: SECRET_USERNAME
          valueFrom:
            secretKeyRef:
              name: my-secret
              key: username
```

* Environment variables (all keys from a secret)

```yaml theme={null}
envFrom:
  - secretRef:
      name: my-secret
```

* Mounted as files (volume)

```yaml theme={null}
volumes:
  - name: secret-vol
    secret:
      secretName: my-secret
containers:
  - name: app
    image: busybox
    volumeMounts:
      - name: secret-vol
        mountPath: /etc/secret
        readOnly: true
```

When mounted, each key becomes a file under `/etc/secret/<key>` containing the decoded value.

Security considerations and best practices

* Do not store secrets in container images, Git repositories, or plain-text manifests checked into source control.
* Restrict access to Secrets via RBAC and audit access to the API server and etcd.
* Prefer `type: kubernetes.io/tls` for TLS artifacts and `type: dockerconfigjson` for image pull secrets—use specific types when applicable.
* Use encryption at rest for secrets in etcd: [https://kubernetes.io/docs/tasks/administer-cluster/encrypt-data/](https://kubernetes.io/docs/tasks/administer-cluster/encrypt-data/)
* Consider external secret management (Vault, AWS Secrets Manager, Azure Key Vault) for centralized lifecycle, rotation, and fine-grained access control.

Quick reference table

| Topic | Purpose | Example/Command |
| - | - | - |
| Create from literal | Quick secret creation | `kubectl create secret generic my-secret --from-literal=password='s3cr3t'` |
| Create from file | Use existing files | `kubectl create secret generic app-creds --from-file=./creds` |
| TLS secret | Store certificate + key | `kubectl create secret tls tls-secret --cert=crt --key=key` |
| Inspect secret | View encoded values | `kubectl get secret my-secret -o yaml` |
| Decode value | Convert base64 to plain | `echo 'cGFzc3dvcmQ=' \| base64 --decode` |

Secrets are only base64-encoded in the API by default—not encrypted. Without encryption at rest or an external secrets manager, Secrets stored in etcd can be read by anyone with sufficient privileges. Do not treat default Secrets as fully secure.

Next steps

The following demo shows how to create Secrets and how they are consumed by pods — including examples for CLI, YAML manifests, and runtime consumption patterns. For more details, see the Kubernetes documentation on Secrets: [https://kubernetes.io/docs/concepts/configuration/secret/](https://kubernetes.io/docs/concepts/configuration/secret/)

