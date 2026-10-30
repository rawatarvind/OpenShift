# Docker Build

> Guide on using OpenShift Docker-strategy BuildConfigs with Buildah to build OCI compliant container images from Dockerfiles and manage resulting ImageStreamTags

In this lesson we'll cover how OpenShift builds OCI-compliant container images from source using a Dockerfile-based BuildConfig. When your repository includes a Dockerfile, OpenShift executes the Docker strategy using Buildah under the hood to produce images that run across OCI-compatible runtimes.

Builds in OpenShift support multiple input types (Git, binary uploads, Dockerfile, etc.). Using a Dockerfile in your Git repo triggers a Docker-strategy BuildConfig that directs OpenShift to use Buildah to build the image.

What you’ll learn:

* How Buildah enables Dockerfile builds on OpenShift
* The structure of a Docker-strategy BuildConfig
* The example Dockerfile used to build a Go web app
* How to start and monitor a build in the OpenShift console
* Where to find the resulting ImageStreamTag and image metadata

Key tools and specs:

* Buildah — a tool that creates and manipulates OCI-compliant container images: [https://buildah.io/](https://buildah.io/)
* OCI (Open Container Initiative) — image format and runtime specifications: [https://opencontainers.org/](https://opencontainers.org/)

Use Buildah via OpenShift BuildConfigs to produce OCI-compliant images directly inside the cluster—no local image builds required.

## How Buildah and the Docker strategy work

Buildah builds images from Dockerfiles in an OCI-compliant way. It can:

* Create and manipulate working containers
* Mount/unmount container root filesystems to modify contents
* Build images from Dockerfiles and other sources

Because Buildah produces OCI images, the final artifacts are compatible with runtimes like Docker, Podman, CRI-O and Kubernetes distributions.

## BuildConfig (Docker strategy)

A Docker-strategy BuildConfig is similar to other BuildConfigs. The main differences:

* `strategy.type` is `Docker`
* `dockerStrategy.dockerfilePath` points to the Dockerfile to execute

Below is a sample BuildConfig that points to a Git repository containing a Dockerfile and writes the resulting image into an ImageStreamTag named `gowebapi:latest`:

```yaml theme={null}
apiVersion: build.openshift.io/v1
kind: BuildConfig
metadata:
  name: mywebsite
  namespace: default
  labels:
    app: mywebsite
spec:
  source:
    type: Git
    git:
      uri: https://github.com/AdminTurnedDevOps/GoWebAPI
    contextDir: .
  strategy:
    type: Docker
    dockerStrategy:
      dockerfilePath: Dockerfile
  output:
    to:
      kind: ImageStreamTag
      name: gowebapi:latest
```

BuildConfig field summary:

| Field | Purpose | Example / Notes |
| - | - | - |
| `spec.source.git.uri` | Source repository URL | `https://github.com/AdminTurnedDevOps/GoWebAPI` |
| `spec.source.contextDir` | Subdirectory in repo to use as build context | `.` |
| `spec.strategy.type` | Build strategy to execute | `Docker` (uses Buildah to run Dockerfile) |
| `spec.strategy.dockerStrategy.dockerfilePath` | Path to Dockerfile within build context | `Dockerfile` |
| `spec.output.to.kind` | Where to push the resulting image | `ImageStreamTag` |
| `spec.output.to.name` | ImageStreamTag name | `gowebapi:latest` |

Make sure `dockerfilePath` and `contextDir` point to the correct locations relative to the repository root. A mismatched path is a common cause of build failures.

## Dockerfile (example)

This repository uses a simple multi-step build for a Go web application. The Dockerfile below builds the binary and exposes port 8080:

```dockerfile theme={null}
FROM golang:latest

RUN mkdir /app
ADD . /app
WORKDIR /app

RUN go build -o main .

EXPOSE 8080

CMD ["/app/main"]
```

Notes about the Dockerfile:

* `RUN go build -o main .` compiles the Go app to a binary named `main`.
* `WORKDIR /app` sets the working directory for subsequent commands.
* `EXPOSE 8080` documents the port the application listens on (useful metadata in the image).

## Start the build in the OpenShift console

1. In the OpenShift console, go to Builds → Build Configs.
2. Create a new BuildConfig and paste the YAML above (verify the `namespace` is correct).
3. After creating the BuildConfig, click the kebab menu next to it and choose "Start Build".
4. Monitor the build logs in the console to follow progress and troubleshoot errors.

Sample trimmed build log output (OpenShift runs Buildah to perform the Docker strategy; it pulls the base image, copies layers/blobs, builds the image, and pushes to the ImageStream):

```text theme={null}
Pulling image golang:latest ...
Resolving "golang" using unqualified-search registries (/etc/containers/registries.conf)
Trying to pull registry.redhat.io/golang:latest...
Trying to pull registry.access.redhat.com/golang:latest...
Copying blob sha256:53ad072f9cd16fc8eb93b182b20e758e11acc0ef60babef0bf1043c08de1901a
Copying blob sha256:1671565cc8df8c365c9b661d3fbc164e73d01f1b0430c6179588428f99a9da2e
Copying blob sha256:3e94d13e55e7a4ef17ff21376f57fb95c7e1706931f8704aa99260968d81f6e4
Successfully built image sha256:...
Pushed image to image stream tag: gowebapi:latest
```

After the build completes:

* View the Build record (Build details show logs, steps, and build metadata).
* Inspect the ImageStreamTag (`gowebapi:latest`) in the console to see image metadata such as the `CMD`, `WORKDIR`, `EXPOSED PORTS`, and architecture.

![A screenshot of the Red Hat OpenShift Container Platform web console showing ImageStreamTag details for "gowebapi:latest" with configuration info (command /app/main, exposed port 8080/tcp, architecture amd64). The left sidebar shows navigation items like Workloads, Networking, Storage, Builds, and ImageStreams.](https://mintcdn.com/kodekloud-c4ac6d9a/Xa5GerWBw0vsOVlC/images/OpenShift-4/Concepts-Builds-and-Deployments/Docker-Build/openshift-imagestream-gowebapi-8080-amd64.jpg?fit=max\&auto=format\&n=Xa5GerWBw0vsOVlC\&q=85\&s=7a6fa0939454fd92bc500512b63392f4)

## Troubleshooting tips

* If the build fails to find the Dockerfile, confirm `contextDir` and `dockerfilePath` are correct.
* Check network/pull errors when fetching base images (registry access and credentials).
* Use build logs to identify which step failed—Buildah emits detailed step output.
* If the image is pushed but your deployment doesn't start, verify the container port and readiness/liveness probes.

## Links and references

* Buildah: [https://buildah.io/](https://buildah.io/)
* OCI: [https://opencontainers.org/](https://opencontainers.org/)
* OpenShift Builds documentation: [https://docs.openshift.com/container-platform/latest/builds/what\_builds\_are.html](https://docs.openshift.com/container-platform/latest/builds/what_builds_are.html)

