# Image Streams Overview

> Explains OpenShift ImageStreams which track image tags and digests as metadata pointers, enabling automated build and deployment triggers and decoupling image publishing from deployment manifests.

An ImageStream in OpenShift provides an indirection layer between your deployment manifests and the container images stored in registries. By tracking tags and tag-to-digest mappings, an ImageStream enables automated updates to builds and deployments when an image changes.

Summary

* ImageStreams store metadata (tags and references) that point to images in registries — they do not store image layer data.
* They let you decouple image publishing from deployment manifests so CI/CD pipelines can push images and trigger rollouts automatically.
* OpenShift's build and deployment systems integrate with ImageStreams to support automated image-change triggers.

How ImageStreams work (practical flow)

1. A new image is pushed to a registry (for example, `myapp:3.4` is pushed or a tag is moved).
2. The ImageStream tracks the registry location and records the tag-to-digest mapping (it updates the ImageStreamTag).
3. Builds and deployments configured to watch that ImageStream receive image-change notifications and can roll out new images automatically.

Where the images live

| Registry type | Typical examples |
| -: | - |
| Integrated OpenShift registry | OpenShift internal registry (`image-registry.openshift-image-registry.svc:5000`) |
| External registries | [Docker Hub](https://hub.docker.com/), [Quay](https://quay.io/), other Docker/OCI registries |

Key behaviors and benefits

* ImageStreams maintain tag metadata and references; the actual image blobs remain in the source registry.
* Using ImageStreams avoids editing multiple deployment manifests to change image tags; the ImageStreamTag provides a stable reference.
* Native OpenShift triggers (e.g., ImageChange) can perform controlled, automated rollouts when a tag changes.
* This is functionally similar to a rolling update: a new image version becomes available and the platform updates running pods in a controlled way.

ImageStream behavior overview

* ImageStreams act as watchers or pointers to registries. When a registry image is updated (a new digest under a given tag), the ImageStream can be updated to point to the new digest.
* You can map tags to digests and keep a stable ImageStreamTag name that deployments reference.
* Deployments or DeploymentConfigs configured with an ImageChange trigger or that reference `ImageStreamTag` will consume new images automatically without manual manifest edits.

Example: ImageStream and DeploymentConfig trigger (OpenShift)

Here is an ImageStream that references an external image, followed by a DeploymentConfig that uses an ImageChange trigger. When the ImageStreamTag `myapp:3.4` is updated, the DeploymentConfig can automatically roll out the new image.

```yaml theme={null}
# ImageStream that points at an external Docker image
apiVersion: image.openshift.io/v1
kind: ImageStream
metadata:
  name: myapp
spec:
  tags:
    - name: "3.4"
      from:
        kind: DockerImage
        name: "quay.io/myorg/myapp:3.4"
```

```yaml theme={null}
# DeploymentConfig that will roll out automatically when the ImageStreamTag myapp:3.4 is updated
apiVersion: apps.openshift.io/v1
kind: DeploymentConfig
metadata:
  name: myapp
spec:
  template:
    spec:
      containers:
        - name: myapp
          image: "myapp:3.4"
  triggers:
    - type: ImageChange
      imageChangeParams:
        automatic: true
        containerNames:
          - myapp
        from:
          kind: ImageStreamTag
          name: "myapp:3.4"
```

Practical notes and best practices

* Use ImageStreams to decouple image publishing from deployment manifests — this streamlines CI/CD: image builds update ImageStreams and deployments respond automatically.
* Remember: ImageStreams store only metadata (tags and digests). Actual pulls still happen from the referenced registry during deployment.
* For fully automated flows on OpenShift, integrate builds → ImageStream tag updates → DeploymentConfig (or Deployment) triggers.
* When referencing external registries, ensure the cluster has appropriate pull secrets and network access to those registries.

ImageStreams store tag mappings and metadata, not the image layers themselves. They act as a pointer to the registry and enable triggers for builds and deployments.

Links and references

* [OpenShift ImageStreams documentation](https://docs.openshift.com/container-platform/latest/openshift_images/images_understanding.html)
* [Kubernetes Deployments concept](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/)
* [Docker Hub](https://hub.docker.com/)
* [Quay](https://quay.io/)

