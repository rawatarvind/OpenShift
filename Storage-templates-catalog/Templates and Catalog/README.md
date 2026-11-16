# Templates and Catalog

> Explains OpenShift Templates and Catalog, packaging multiple Kubernetes/OpenShift resources into reusable templates and how to author, parameterize, and install them.

Welcome — this lesson introduces OpenShift Templates and the Catalog, aimed at beginners. You’ll learn what the Catalog presents in the web console, how Templates bundle multiple Kubernetes/OpenShift resources, and how to author and install your own Template.

When you sign in to the OpenShift web console, the Catalog is the interface that presents deployment options and ready-made stacks. Instead of manually creating individual BuildConfigs, DeploymentConfigs, Services, ImageStreams, Secrets, and Routes, the Catalog provides packaged Template items so you can deploy a complete application stack in a single step.

Common catalog items (example: Django + PostgreSQL)

* A BuildConfig that builds the application image.
* An ImageStream to tag and track builds.
* A DeploymentConfig to run the application.
* A DeploymentConfig (or StatefulSet/Deployment where appropriate) to run PostgreSQL.
* A Service exposing the database to the application on port 5432.
* A Secret to store database credentials and inject them into pods.
* A Service exposing the application on port 80.
* A Route to expose the application outside the cluster.
* A set of parameters presented in the creation wizard (application name, Git repo, namespace, etc).

All these resources are packaged into a single Template object. Instantiating the Template creates the full stack with minimal input.

![A diagram template of a Kubernetes/OpenShift deployment for a Django + PostgreSQL application. It shows components like Route, Services, Deployments, Secrets, Build, Image Stream and a parameters list.](https://mintcdn.com/kodekloud-c4ac6d9a/_-d-o-x8WFPLT_DD/images/OpenShift-4/Storage-Templates-and-Catalog/Templates-and-Catalog/kubernetes-openshift-django-postgresql-deployment-diagram.jpg?fit=max\&auto=format\&n=_-d-o-x8WFPLT_DD\&q=85\&s=6bc2c4f265a6a3a0bf7dbc1f03eb7d9f)

Resources included in Templates — quick reference

| Resource Type | Role in the Template |
| - | - |
| BuildConfig | Builds application image from source and produces images for ImageStream |
| ImageStream | Tracks image versions and makes them available to Deployments/Builds |
| DeploymentConfig / Deployment / StatefulSet | Runs application and database workloads |
| Service | Provides stable network endpoints between components (app ↔ db) |
| Route | Exposes the application externally via an HTTP(S) route |
| Secret | Stores sensitive data (DB credentials, TLS certs) securely |
| Parameters | Expose user-provided values in the creation form (repo, app name, credentials) |

Creating and adding a Template to the catalog

1. Author a Template YAML (or JSON) that defines:
   * The Template metadata (`metadata.name`, labels, annotations).
   * A `parameters` array for values to collect from users.
   * An `objects` array containing the resource manifests that will be created when the template is instantiated.
2. Remove cluster-specific fields (status, resourceVersion, uid, creationTimestamp) from exported resource manifests before adding them.
3. Replace hard-coded values with template parameters (e.g., `${APP_NAME}`, `${NAMESPACE}`, `${GIT_URI}`).
4. Create the Template in the cluster with `oc create -f`.

Minimal Template skeleton

```yaml theme={null}
apiVersion: v1
kind: Template
metadata:
  name: custom-app
objects:
  # Add resource manifests here (Secret, Service, Route, BuildConfig, DeploymentConfig, ImageStream, etc.)
parameters:
  - name: APP_NAME
    displayName: "Application name"
    description: "Name used for resources"
    required: true
  - name: GIT_URI
    displayName: "Git repository"
    description: "Source code repository URL"
    required: true
```

Example showing how resource manifests sit inside `objects` (illustrative — sensitive fields omitted):

```yaml theme={null}
apiVersion: v1
kind: Template
metadata:
  name: custom-app

objects:
- apiVersion: v1
  kind: Secret
  <.. code hidden ..>

- apiVersion: v1
  kind: Service
  <.. code hidden ..>

- apiVersion: v1
  kind: Service
  <.. code hidden ..>

- apiVersion: v1
  kind: Route
  <.. code hidden ..>

- apiVersion: v1
  kind: BuildConfig
  <.. code hidden ..>

- apiVersion: v1
  kind: DeploymentConfig
  <.. code hidden ..>

- apiVersion: v1
  kind: DeploymentConfig
  <.. code hidden ..>

- apiVersion: v1
  kind: ImageStream
  <.. code hidden ..>

parameters:
- displayName: "Namespace"
  name: "NAMESPACE"
```

Install the template

```bash theme={null}
oc create -f template-config.yaml
```

Example output

```bash theme={null}
› oc create -f template-config.yml
template "custom-app" created
```

If you need YAML for resources to include in a Template, export an existing resource and parameterize it. For example:

```bash theme={null}
oc get service db -o yaml
```

Then remove cluster-specific fields (`status`, `resourceVersion`, `uid`, `creationTimestamp`) and replace hard-coded values with template parameters before adding the manifest to the `objects` array.

Tips and best practices

* Parameterize as much as possible (names, ports, secrets) so the Template is reusable across projects.
* Use descriptive `displayName` and `description` fields for parameters so the Catalog creation wizard is clear to end users.
* Keep Secrets out of source control. Consider generating Secrets at instantiation time or prompting users to supply credentials.
* Test instantiation in a non-production project before adding to a shared Catalog.

Useful links and references

* OpenShift Templates documentation: [https://docs.openshift.com/](https://docs.openshift.com/)
* Kubernetes Concepts (Objects & Manifests): [https://kubernetes.io/docs/concepts/](https://kubernetes.io/docs/concepts/)
* oc CLI reference: [https://docs.openshift.com/en/cli\_reference/index.html](https://docs.openshift.com/en/cli_reference/index.html)

Summary
Templates let you package multiple OpenShift/Kubernetes resources and user-facing parameters into a single reusable object. Adding Templates to the Catalog provides end users with one-click deployment options for complex application stacks like Django + PostgreSQL. Follow the steps above to author, parameterize, and install Templates into your cluster.

