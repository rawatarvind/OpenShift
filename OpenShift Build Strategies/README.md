# OpenShift Build Strategies

> Summary of OpenShift BuildConfig strategies S2I Pipeline Docker and Custom plus build inputs examples security and best practices

This article explains how OpenShift creates container images from application source and how BuildConfigs support different build workflows. OpenShift provides four BuildConfig strategies: Source-to-Image (S2I), Pipeline (Jenkins), Docker, and Custom. Each strategy targets specific workflows and requirements — choose the one that best matches your CI/CD model, build tooling, and maintenance preferences.

## Build strategy overview

* Source-to-Image (S2I)
  * S2I produces runnable, Docker-formatted images by injecting application source into a builder image and running assembly scripts inside that image.
  * The builder image contains runtime and assembly logic; S2I adds your source, executes the assemble step, and outputs a final container image.
  * Best when you want simple builds without managing Dockerfiles and when an S2I builder exists for your language/runtime.

* Pipeline (Jenkins)
  * Pipeline strategy delegates the build process to a Jenkins pipeline defined in a `Jenkinsfile`.
  * OpenShift integrates with Jenkins so OpenShift starts, monitors, and manages the build lifecycle while Jenkins runs the pipeline stages defined in the repository.
  * Use this for complex CI/CD pipelines, multi-stage workflows, or when you need advanced pipeline constructs.

* Docker
  * The Docker strategy runs a `docker build`-equivalent using a Dockerfile from the repository (or a specified context directory).
  * If you already express your image with a Dockerfile and want identical build semantics inside OpenShift, use this strategy.

* Custom
  * Custom strategy uses a provided builder image that contains the custom build tooling and scripts. The image performs the build inside OpenShift.
  * Useful for specialized processes (e.g., building RPMs, creating base images, or other non-standard workflows not covered by S2I or Docker).

![A slide titled "Build Configuration Strategy" featuring four red square buttons with arrow icons arranged in a 2x2 grid, labeled around the sides as Custom (top), Source to Image (right), Docker (left), and Pipeline (bottom).](https://mintcdn.com/kodekloud-c4ac6d9a/Xa5GerWBw0vsOVlC/images/OpenShift-4/Concepts-Builds-and-Deployments/OpenShift-Build-Strategies/build-config-strategy-2x2-grid.jpg?fit=max\&auto=format\&n=Xa5GerWBw0vsOVlC\&q=85\&s=8c9ed4fee950c5147dba643fef2c68a0)

### Quick comparison

| Strategy | Typical use case | Key benefit |
| -: | - | - |
| S2I | Language/runtime-specific builds (Node, Java, Python, etc.) | Minimal Dockerfile maintenance; ready-to-run images |
| Pipeline | Complex CI/CD (Jenkins pipelines) | Full pipeline control, external tooling and staging |
| Docker | Dockerfile-based image builds | Exact parity with local `docker build` workflows |
| Custom | Bespoke build environments | Run any custom build logic inside a controlled image |

## Example BuildConfig snippets

Below are minimal examples showing how a BuildConfig declares the strategy. These are illustration templates — adapt metadata, names, and references to your cluster.

* S2I BuildConfig (strategy: `Source`)

```yaml theme={null}
apiVersion: build.openshift.io/v1
kind: BuildConfig
metadata:
  name: example-s2i
spec:
  source:
    type: Git
    git:
      uri: "https://github.com/example/repo.git"
  strategy:
    type: Source
    sourceStrategy:
      from:
        kind: ImageStreamTag
        name: "nodejs:14"
```

* Docker BuildConfig (strategy: `Docker`)

```yaml theme={null}
apiVersion: build.openshift.io/v1
kind: BuildConfig
metadata:
  name: example-docker
spec:
  source:
    type: Git
    git:
      uri: "https://github.com/example/repo-with-dockerfile.git"
  strategy:
    type: Docker
    dockerStrategy: {}
```

* Pipeline BuildConfig (strategy: `JenkinsPipeline`)

```yaml theme={null}
apiVersion: build.openshift.io/v1
kind: BuildConfig
metadata:
  name: example-pipeline
spec:
  source:
    type: Git
    git:
      uri: "https://github.com/example/repo-with-jenkinsfile.git"
  strategy:
    type: JenkinsPipeline
    jenkinsPipelineStrategy:
      jenkinsfilePath: Jenkinsfile
```

* Custom BuildConfig (strategy: `Custom`)

```yaml theme={null}
apiVersion: build.openshift.io/v1
kind: BuildConfig
metadata:
  name: example-custom
spec:
  source:
    type: Git
    git:
      uri: "https://github.com/example/custom-builder-repo.git"
  strategy:
    type: Custom
    customStrategy:
      from:
        kind: DockerImage
        name: "registry.example.com/custom-builder:latest"
```

## Build inputs

BuildConfigs also define inputs used during the build. These inputs tell OpenShift where to obtain source, Dockerfiles, binaries, images, and any sensitive data required during build-time.

* Git (Source)
  * The most common input; OpenShift clones the specified Git repository and builds from that content.

* Dockerfile
  * A Dockerfile in the repo is used by the Docker strategy (equivalent to running `docker build`).

* Binary
  * Use binary inputs when you want to provide pre-built artifacts from a local filesystem:
  * Examples:
    * `oc start-build myapp --from-file=artifact.jar`
    * `oc start-build myapp --from-dir=dist/`
    * `oc start-build myapp --from-archive=artifact.tar.gz`

* Image (Image source)
  * Use existing images or image references as inputs for multi-stage builds or assembly steps.

* Input Secrets (and ConfigMaps)
  * Mount secrets and ConfigMaps for build-time access to private Git repos, artifact registries, or other protected resources. This avoids embedding credentials in source.

* External Artifacts
  * Fetch artifacts (large binaries, JARs) from external storage during the build. Note the guidance below regarding reproducibility and best practices.

![A slide titled "Build Input" showing six red square icons with white downward chevrons. Each icon is labeled underneath: Git, Dockerfile, Binary, Image, Input Secrets, and External Artifacts.](https://mintcdn.com/kodekloud-c4ac6d9a/Xa5GerWBw0vsOVlC/images/OpenShift-4/Concepts-Builds-and-Deployments/OpenShift-Build-Strategies/build-input-icons-git-dockerfile.jpg?fit=max\&auto=format\&n=Xa5GerWBw0vsOVlC\&q=85\&s=cf822930462c2f2f721f4f7210e30ccd)

External artifacts (large binaries checked into artifact servers) are an older pattern. Today it’s usually better to build from source or use a Dockerfile so builds are reproducible and image layers are managed explicitly.

### Build input examples and commands

| Input type | How to supply | Example command |
| - | - | - |
| Git | Remote repository URL in BuildConfig | See BuildConfig `source.git.uri` example above |
| Binary | Local files or directories | `oc start-build myapp --from-file=app.jar` |
| From image | Use an existing image as a build source | `oc start-build myapp --from-image=registry.example.com/base:latest` |
| Secrets/ConfigMaps | Attach via BuildConfig `source.secret` or `spec.strategy.*.env` | Configure via `oc create secret` and reference in BuildConfig |

## Security and best practices

* Do not embed credentials directly in your source. Use secrets or image pull credentials and reference them in BuildConfig.
* Prefer reproducible builds: building from source or a Dockerfile produces deterministic images and makes caching, layering, and debugging easier.
* For CI/CD at scale, use the Pipeline strategy to encapsulate complex workflows, but ensure Jenkins controllers are hardened and monitored.

Be careful exposing secrets at build time. Always mount secrets via BuildConfig rather than placing credentials in repository files or Dockerfiles.

## Summary guidance

* Use S2I for language/runtime-specific builds where a supported builder exists and you want minimal Dockerfile maintenance.
* Use Pipeline (Jenkinsfile) for complex CI/CD and multi-stage job orchestration.
* Use Docker strategy when you already maintain a Dockerfile and require identical behavior inside OpenShift.
* Use Custom when you need a bespoke build environment or specialized build tooling not provided by other strategies.

## Links and references

* OpenShift Builds documentation: [https://docs.openshift.com/container-platform/latest/builds/using-builds/build-strategies.html](https://docs.openshift.com/container-platform/latest/builds/using-builds/build-strategies.html)
* Source-to-Image (S2I): [https://github.com/openshift/source-to-image](https://github.com/openshift/source-to-image)
* OpenShift Pipelines and Jenkins integration: [https://docs.openshift.com/container-platform/latest/cicd/index.html](https://docs.openshift.com/container-platform/latest/cicd/index.html)
