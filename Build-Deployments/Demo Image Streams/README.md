# Demo Image Streams

> Guide to creating an OpenShift ImageStream for the Sock Shop carts service and deploying it with oc new-app using an internal tag to decouple from external registries

This lesson shows how to create an OpenShift ImageStream and deploy an application from a specific ImageStream tag. ImageStreams decouple your deployment resources from remote container registries by keeping tag references inside the cluster. This example uses the Sock Shop microservices demo and demonstrates creating an ImageStream for the carts service and deploying it with `oc new-app`.

* Relevant links and references:
  * Sock Shop demo repository: [https://github.com/microservices-demo/microservices-demo](https://github.com/microservices-demo/microservices-demo)
  * OpenShift Builds and ImageStreams documentation: [https://docs.openshift.com/](https://docs.openshift.com/)

## View existing ImageStreams

In the OpenShift console go to Builds → ImageStreams. Many applications (including the Sock Shop) already create ImageStreams for their build outputs. For example, the Git-based build for the Sock Shop has an ImageStream that lists the labels associated with the image:

```yaml theme={null}
app: microservices-demo-git
app.kubernetes.io/component: microservices-demo-git
app.kubernetes.io/instance: microservices-demo-git
app.kubernetes.io/name: microservices-demo-git
app.kubernetes.io/part-of: complete-demo-yaml-app
app.openshift.io/runtime: golang
app.openshift.io/runtime-version: 1.16.7-ubi7
```

If you want to create a specific ImageStream for one of the Sock Shop services (for example, the carts service), click "Create Image Stream" in the console.

![A Red Hat OpenShift Container Platform web console screenshot showing the ImageStreams page for the "default" project, listing image streams like "approved-apache", "goweb", and "go-web-api" with labels and creation timestamps. The left navigation (Workloads, Networking, Storage, Builds) and a "Create ImageStream" button are also visible.](https://mintcdn.com/kodekloud-c4ac6d9a/Xa5GerWBw0vsOVlC/images/OpenShift-4/Concepts-Builds-and-Deployments/Demo-Image-Streams/redhat-openshift-imagestreams-default-console.jpg?fit=max\&auto=format\&n=Xa5GerWBw0vsOVlC\&q=85\&s=15316afac8e346916e7684d576fb655e)

## Create an ImageStream for the cart service

In the create dialog, replace the placeholder YAML with the following ImageStream manifest. This ImageStream points to the external Docker image `weaveworksdemos/carts:0.4.8` and creates the tag `0.4.8` in the cluster:

```yaml theme={null}
kind: ImageStream
metadata:
  name: carts
spec:
  lookupPolicy:
    local: false
  tags:
    - name: "0.4.8"
      from:
        kind: DockerImage
        name: weaveworksdemos/carts:0.4.8
      referencePolicy:
        type: Source
```

Quick reference: ImageStream tag fields

| Field | Purpose | Example |
| - | - | - |
| `metadata.name` | Name of the ImageStream in the project/namespace | `carts` |
| `tags[].name` | The tag created inside the ImageStream | `0.4.8` |
| `tags[].from` | Source of the image (Docker image, ImageStreamTag, etc.) | `weaveworksdemos/carts:0.4.8` |
| `lookupPolicy.local` | Whether the cluster will resolve images from its internal registry | `false` |

## Verify the image and Deployment configuration

The carts Deployment used in this example pulls `weaveworksdemos/carts:0.4.8`. Here is the relevant portion of that Deployment for reference:

```yaml theme={null}
targetPort: 27017
selector:
  name: carts-db
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: carts
  labels:
    name: carts
spec:
  replicas: 1
  selector:
    matchLabels:
      name: carts
  template:
    metadata:
      labels:
        name: carts
    spec:
      containers:
        - name: carts
          image: weaveworksdemos/carts:0.4.8
          ports:
            - containerPort: 80
          # startupProbe:
          #   httpGet:
          #     path: /
          #     port: 80
          #   failureThreshold: 30
          #   periodSeconds: 10
```

## Deploy from the ImageStream with oc new-app

After creating the ImageStream, you can create an application from that tag using `oc new-app`. ImageStream references include the project (namespace), for example: `default/carts:0.4.8` refers to the `carts` ImageStream in the `default` project with tag `0.4.8`.

Run:

```bash theme={null}
oc new-app --image-stream=default/carts:0.4.8
```

If a Deployment (or other resource) with the same name already exists, `oc new-app` will fail because it tries to create duplicate resources. Example output when a deployment named `carts` already exists:

```bash theme={null}
$ oc new-app --image-stream=default/carts:0.4.8
--> Found image c004737 (5 years old) in image stream "default/carts" under tag "0.4.8" for "default/carts:0.4.8"

--> Creating resources ...
error: deployments.apps "carts" already exists
--> Failed
```

Inspect current deployments to confirm whether the `carts` deployment already exists:

```bash theme={null}
$ oc get deployments
NAME                   READY   UP-TO-DATE   AVAILABLE   AGE
approved-apache        1/1     1            1           8m29s
carts                  1/1     1            1           3m2s
catalogue              1/1     1            1           3d18h
catalogue-db           1/1     1            1           3d18h
front-end              1/1     1            1           3d18h
goweb                  1/1     1            1           3d18h
mongodb                0/1     1            0           5m36s
nginx-deployment       2/2     2            2           3d18h
orders                 1/1     1            1           3d18h
orders-db              1/1     1            1           3d18h
payment                1/1     1            1           3d18h
queue-master           1/1     1            1           3d18h
rabbitmq               1/1     1            1           3d18h
shipping               1/1     1            1           3d18h
user                   1/1     1            1           3d18h
user-db                1/1     1            1           3d18h
```

If you want to re-create the deployment with `oc new-app`, delete the existing deployment first:

```bash theme={null}
oc delete deployment carts
```

You should see a deletion confirmation:

```bash theme={null}
deployment.apps "carts" deleted
```

Then re-run `oc new-app`:

```bash theme={null}
oc new-app --image-stream=default/carts:0.4.8
```

On success you will see output similar to:

```bash theme={null}
--> Found image c004737 (5 years old) in image stream "default/carts" under tag "0.4.8" for "default/carts:0.4.8"

--> Creating resources ...
    deployment.apps "carts" created
--> Success
Run 'oc status' to view your app.
```

Verify the newly created deployment:

```bash theme={null}
$ oc get deployments
NAME                  READY   UP-TO-DATE   AVAILABLE   AGE
approved-apache       1/1     1            1           8m58s
carts                 1/1     1            1           8s
catalogue             1/1     1            1           3d18h
...
```

This confirms the carts application is running and was created from the ImageStream tag `0.4.8`.

Remember: `oc new-app` creates new resources. If resources with the same names already exist (for example, a Deployment named `carts`), `oc new-app` will fail. Use `oc apply`, `oc replace`, or delete the existing resource before running `oc new-app`, depending on your workflow.

