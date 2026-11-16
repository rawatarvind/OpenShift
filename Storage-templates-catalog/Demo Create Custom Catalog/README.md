# Demo Create Custom Catalog

> Guide to creating an OpenShift Template that packages the Example Voting Application resources into a catalog item, with steps to export, clean, parameterize, create, and publish.

Welcome. In this lesson you'll create a reusable OpenShift Template that packages the Example Voting Application (Secrets, BuildConfigs, ImageStreams, DeploymentConfigs, Services, and Routes) into a single catalog item. Once added to a project catalog, other developers can deploy the full voting stack via the catalog wizard.

What you'll accomplish

* Identify the resources that make up the voting app (Secrets, BuildConfigs, ImageStreams, DeploymentConfigs, Services, Routes).
* Export and clean each resource's YAML, then assemble them under the Template's `objects:` array.
* Create the Template in a project and add it to the catalog.
* Optionally parameterize values so the catalog wizard prompts users for inputs.

Quick steps (at-a-glance)

| Step | Description | Example / Notes |
| - | - | - |
| 1 | Export resources from your project | Secrets, BuildConfigs, ImageStreams, DeploymentConfigs, Services, Routes |
| 2 | Clean resource YAML | Remove runtime metadata fields (`creationTimestamp`, `resourceVersion`, `uid`, `selfLink`, `status`) |
| 3 | Assemble into a Template file | Add each cleaned manifest under `objects:` |
| 4 | Create the Template in a project | `oc create -f example-voting-app-template.yml -n <project>` |
| 5 | Add parameters (optional) | Move Git repo, branch, credentials, image tags to parameters for the wizard |

Recommended SEO keywords: OpenShift Template, custom catalog, example voting app, BuildConfig, ImageStream, DeploymentConfig, Route, Secrets.

***

## 1. Create the Template file

Create a file named `example-voting-app-template.yml`. Start with the Template API version, kind, and metadata. Use `openshift.io/display-name` as an annotation if you want a friendly label to appear in the catalog.

```yaml theme={null}
apiVersion: template.openshift.io/v1
kind: Template
metadata:
  name: example-voting-app-template
objects: []
```

The `objects` array will contain a list of resource manifests (each item prefixed with `-`). Populate this array with cleaned YAML documents for Secrets, BuildConfigs, ImageStreams, DeploymentConfigs, Services, and Routes.

***

## 2. Secrets

Start with Secrets because multiple components reference DB and Redis credentials. In the OpenShift Console, open each Secret → Actions → Edit YAML and copy it into the template. Remove runtime metadata fields such as `creationTimestamp`, `resourceVersion`, `uid`, `selfLink`, and `status`.

Use `stringData` in the template for clear-text values so users (or parameters) can provide plaintext values which the API converts to base64.

Example cleaned DB Secret entry to include under `objects:`:

```yaml theme={null}
- apiVersion: v1
  kind: Secret
  metadata:
    name: db
  type: Opaque
  stringData:
    database-name: db
    database-password: postgres_password
    database-user: postgres_user
```

![A screenshot of the OpenShift Origin web console showing the "Example Voting Application" Secrets page for a "db" entry, with labels/tags for a postgresql-persistent template and masked database-name, database-password, and database-user values. The left sidebar navigation and action buttons (Add to Application / Actions) are also visible.](https://mintcdn.com/kodekloud-c4ac6d9a/_-d-o-x8WFPLT_DD/images/OpenShift-4/Storage-Templates-and-Catalog/Demo-Create-Custom-Catalog/openshift-secrets-example-voting-db-masked.jpg?fit=max\&auto=format\&n=_-d-o-x8WFPLT_DD\&q=85\&s=524e3b2733078297ec668c4c47e7418a)

Repeat for the Redis secret and keep only necessary fields:

```yaml theme={null}
- apiVersion: v1
  kind: Secret
  metadata:
    name: redis
  type: Opaque
  stringData:
    redis_password: redis_password
```

Tip: Consider converting these values into Template parameters so the catalog wizard prompts users for credentials instead of hard-coding them in the template.

***

## 3. BuildConfigs

Export each BuildConfig from the project, then remove unnecessary metadata and `status`. Keep the `spec` (source Git repo, `contextDir`, build `strategy`, `triggers`, environment variables, etc.). Consider parameterizing the Git URL and branch.

Example cleaned BuildConfig for the `result` app:

```yaml theme={null}
- apiVersion: build.openshift.io/v1
  kind: BuildConfig
  metadata:
    name: result
    labels:
      app: result
  spec:
    source:
      type: Git
      git:
        uri: 'https://github.com/mmumshad/example-voting-app.git'
        ref: master
      contextDir: /result
    strategy:
      type: Source
      sourceStrategy:
        from:
          kind: ImageStreamTag
          name: 'nodejs:8'
          namespace: openshift
        env:
          - name: PORT
            value: '8080'
    runPolicy: Serial
    postCommit: {}
    resources: {}
    triggers:
      - type: Generic
        generic:
          secret: 5a7ae0b1e2c90cbc
      - type: GitHub
        github:
          secret: 7832f312c9c2e7f1
      - type: ImageChange
```

Repeat for `vote` and `worker` BuildConfigs, adjusting `contextDir` and labels.

***

## 4. ImageStreams

Include ImageStream resources so OpenShift can resolve image references and trigger rebuilds/rollouts. Keep the name and the minimal spec required.

```yaml theme={null}
- apiVersion: image.openshift.io/v1
  kind: ImageStream
  metadata:
    name: result
  spec:
    lookupPolicy:
      local: false

- apiVersion: image.openshift.io/v1
  kind: ImageStream
  metadata:
    name: vote
  spec:
    lookupPolicy:
      local: false

- apiVersion: image.openshift.io/v1
  kind: ImageStream
  metadata:
    name: worker
  spec:
    lookupPolicy:
      local: false
```

Local image build/push hint:
If you build images locally and push to the integrated OpenShift registry, tag and push using the registry host and project namespace:

```bash theme={null}
# Tag using your OpenShift integrated registry or another registry host:port
$ sudo docker tag myimage <registry-host>:<port>/voting-application/worker:tag
$ sudo docker push <registry-host>:<port>/voting-application/worker:tag
```

***

## 5. DeploymentConfigs

Export and clean DeploymentConfig resources, removing `status` and runtime metadata while preserving `spec`. Keep container images, environment variables (use `valueFrom.secretKeyRef` for secrets), probes, resource limits, and ports.

Example `db` DeploymentConfig entry:

```yaml theme={null}
- apiVersion: apps.openshift.io/v1
  kind: DeploymentConfig
  metadata:
    name: db
    labels:
      app: db
    annotations:
      template.alpha.openshift.io/wait-for-ready: 'true'
  spec:
    replicas: 1
    selector:
      deploymentconfig: db
    template:
      metadata:
        labels:
          deploymentconfig: db
      spec:
        containers:
          - name: postgresql
            image: 'docker.io/centos/postgresql-96-centos7:latest'
            imagePullPolicy: IfNotPresent
            ports:
              - containerPort: 5432
            env:
              - name: POSTGRESQL_USER
                valueFrom:
                  secretKeyRef:
                    name: db
                    key: database-user
              - name: POSTGRESQL_PASSWORD
                valueFrom:
                  secretKeyRef:
                    name: db
                    key: database-password
              - name: POSTGRESQL_DATABASE
                valueFrom:
                  secretKeyRef:
                    name: db
                    key: database-name
            livenessProbe:
              exec:
                command:
                  - /bin/sh
                  - -i
                  - -c
                  - pg_isready -h 127.0.0.1 -p 5432
              initialDelaySeconds: 30
              periodSeconds: 10
              timeoutSeconds: 1
              failureThreshold: 3
```

Repeat similar cleanup for `vote`, `result`, `worker`, and `redis` DeploymentConfigs.

***

## 6. Services and Routes

Add Service objects for in-cluster networking. The `worker` service is optional if nothing targets it, but including it maintains consistency.

Example Service for `worker`:

```yaml theme={null}
- apiVersion: v1
  kind: Service
  metadata:
    name: worker
    labels:
      app: worker
  spec:
    ports:
      - name: 8080-tcp
        port: 8080
        protocol: TCP
        targetPort: 8080
      - name: 8443-tcp
        port: 8443
        protocol: TCP
        targetPort: 8443
    selector:
      deploymentconfig: worker
```

![A screenshot of the OpenShift Origin web console showing the "Services" page for an "Example Voting Application," listing services (worker, result, db, vote, redis) with their Cluster IPs, ports, selectors, and ages. The left sidebar shows navigation items like Applications, Builds, Resources, Storage, Monitoring, and Catalog.](https://mintcdn.com/kodekloud-c4ac6d9a/_-d-o-x8WFPLT_DD/images/OpenShift-4/Storage-Templates-and-Catalog/Demo-Create-Custom-Catalog/openshift-origin-console-services-voting-app.jpg?fit=max\&auto=format\&n=_-d-o-x8WFPLT_DD\&q=85\&s=87b4ab8305db5eae0e64aa0a6d15c973)

Add Services for `result`, `vote`, and `db`. Then add Route objects for `vote` and `result` so they are reachable externally.

***

## 7. Create the Template in OpenShift (CLI)

When your `example-voting-app-template.yml` is complete, create it in a project where you have permission (for example `voting-application-3`). If you attempt to create it in the `default` namespace as a normal user, you'll get a Forbidden error.

If you run `oc create -f` without specifying a project and you are not a cluster admin, OpenShift will likely reject the request. Always target a project/namespace where you have permission, e.g. `-n voting-application-3`.

Example Minishift environment setup (Windows) — see Minishift docs for details: [Minishift documentation](https://github.com/minishift/minishift).

```bash theme={null}
# Show command environment setup for Minishift (Windows example)
C:\minishift-1.16.1-windows-amd64>minishift.exe oc-env
SET PATH=C:\Users\mmums\.minishift\cache\oc\v3.9.0\windows;%PATH%
REM Run this command to configure your shell:
REM     @FOR /f "tokens=*" %i IN ('minishift oc-env') DO @call %i
```

Set the PATH (Windows example):

```bat theme={null}
C:\minishift-1.16.1-windows-amd64>SET PATH=C:\Users\mmums\.minishift\cache\oc\v3.9.0\windows;%PATH%
```

A typical forbidden error looks like this:

```text theme={null}
d:\...>oc create -f example-voting-app-template.yml
Error from server (Forbidden): error when creating "example-voting-app-template.yml": templates is forbidden: User "app-developer-2" cannot create templates in the namespace "default": User "app-developer-2" cannot create templates in project "default"
```

Create the Template in a project you control:

```bash theme={null}
d:\...>oc create -f example-voting-app-template.yml -n voting-application-3
```

After creating the Template, refresh the OpenShift web console catalog; the new item `example-voting-app-template` should appear. Click it and run the wizard to deploy the voting stack.

***

## 8. Monitoring the deployment

Open the Overview / Monitoring page to watch builds, pods, and events as the application starts. Allow builds and pods time to finish, then open the `vote` and `result` routes to verify front-ends.

![A screenshot of the OpenShift Origin web console showing the "Example Voting Application" Monitoring page, listing Pods (result, vote, db, redis, etc.) and their statuses (Running, Container Creating). The left sidebar shows navigation options like Overview, Applications, Builds, Resources, Storage, Monitoring, and Catalog.](https://mintcdn.com/kodekloud-c4ac6d9a/_-d-o-x8WFPLT_DD/images/OpenShift-4/Storage-Templates-and-Catalog/Demo-Create-Custom-Catalog/openshift-origin-voting-app-monitoring-pods.jpg?fit=max\&auto=format\&n=_-d-o-x8WFPLT_DD\&q=85\&s=87f43377207c63e09d2109db6c5ec68a)

***

## 9. Improve the Template (best practices)

To make the Template friendly and reusable:

* Parameterize values that are likely to change: Git repository URL, branch/ref, image tag, project namespace, database credentials.
* Add `parameters:` entries at the top-level of the Template with `name`, `displayName`, `description`, `required`, and `value` so the catalog wizard asks for them.
* Avoid embedding plaintext credentials in the template. Use `stringData` with parameters or let users supply secrets during the wizard.
* Provide helpful `annotations` and `labels` (for display name, description, and categorization in the catalog).

See OpenShift templates documentation for parameter examples and best practices: [https://docs.openshift.com/container-platform/latest/openshift\_images/using-templates.html](https://docs.openshift.com/container-platform/latest/openshift_images/using-templates.html)

Tip: Use Template parameters for values that change often (git repo, branch, image tag, database credentials). Parameters allow the catalog wizard to prompt users instead of requiring manual file edits.

***

That completes this lesson on creating a custom catalog template for the Example Voting Application. You now have the steps to export, clean, assemble, and publish an OpenShift Template so others can deploy the full application stack from the catalog. Thank you.

