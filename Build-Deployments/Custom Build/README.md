# Custom Build

> Describes custom builder container images that encapsulate build toolchains and workflows to produce reproducible artifacts deployable locally, in CI, or on container platforms.

I remember the days when packaging and deploying an application meant compiling a binary and running a single command like `java -jar app.jar`. Today, container images are the dominant distribution format: developers write a `Dockerfile`, build an image, and deploy that image. Even so, most modern builds still need to produce intermediate artifacts — JARs, WARs, ZIPs, or other binaries — before they are containerized.

A custom builder image bridges that gap. It is a purpose-built container image that encapsulates the build toolchain and the build process so the artifact creation happens inside a controlled, reproducible environment.

![A side-by-side diagram showing "The original way" (user → terminal → JAR) and "The newer way" (user → DockerFile → container/CLI). It contrasts building/running a JAR with packaging the app using a Dockerfile.](https://mintcdn.com/kodekloud-c4ac6d9a/Xa5GerWBw0vsOVlC/images/OpenShift-4/Concepts-Builds-and-Deployments/Custom-Build/jar-vs-dockerfile-packaging-diagram.jpg?fit=max\&auto=format\&n=Xa5GerWBw0vsOVlC\&q=85\&s=eff1f6eb2a570e75fa473d11a6ed7dc1)

What is a custom builder image?

* A container image whose single responsibility is to run a defined build workflow.
* Contains the compiler, build tools, scripts, and any environment configuration needed to produce one or more artifacts.
* Designed for reproducibility: the same image + same inputs produce identical outputs.

![Three red rounded-square buttons with white right-pointing arrows are shown across the image. Each button has a caption underneath: "Plain image with build process logic", "Implements extensions like CICD", and "Sort of like a scratch container image."](https://mintcdn.com/kodekloud-c4ac6d9a/Xa5GerWBw0vsOVlC/images/OpenShift-4/Concepts-Builds-and-Deployments/Custom-Build/container-image-types-red-buttons.jpg?fit=max\&auto=format\&n=Xa5GerWBw0vsOVlC\&q=85\&s=c8a5aa8cd8bab9b9cdea36bb8dfd0243)

Key characteristics

* Minimal, single-purpose image containing the build toolchain and scripts.
* Encodes build steps so builds become reproducible and portable.
* Can be invoked locally, inside CI runners, or by a container platform (e.g., OpenShift, Kubernetes).
* Extensible: integrate pipeline logic, hooks, or credential handling for private repos and registries.

A custom builder image packages the build environment and process together. It produces artifacts (JAR/WAR/ZIP/etc.) or container images and can be run locally, in CI, or by a container platform.

Typical inputs required by a custom builder image

| Input | What it is | Example / Notes |
| - | - | - |
| Source repository | Location of application source | `https://github.com/example/repo.git` |
| Repository path | Subdirectory to build (when repo contains multiple projects) | `./services/api` |
| Source reference | Branch, tag, or commit to checkout | `main`, `release/v1.2`, `a1b2c3d` |
| Build script/config | The command or file the image executes to run the build | `./mvnw clean package` or `build.sh` |
| Builder/runtime version | Platform or toolchain version to use | `openjdk:17`, `maven:3.8.6` |
| Output destination | Where to push or store artifacts or images | Container registry URL or artifact repo |
| Output image name & tag | Identifier for pushed container images | `registry.example.com/myapp:1.0.0` |
| Credentials | Access tokens or secrets for private repos/registries | Use secret management (K8s Secret, Vault) |

Recommended best practices

* Keep builder images minimal and focused on the build steps to reduce attack surface and image size.
* Pin tool and base-image versions to ensure reproducibility across builds.
* Run builds as non-root where possible and remove unnecessary packages after the build to minimize image bloat.
* Store credentials securely using platform secrets or a secrets manager; avoid baking credentials into images.
* Integrate the builder image into CI/CD pipelines so builds are traceable and automated.

Useful links and references

* [Dockerfile reference](https://docs.docker.com/engine/reference/builder/)
* [Kubernetes / OpenShift build integrations](https://kubernetes.io/docs/concepts/overview/what-is-kubernetes/)
* [CI/CD best practices](https://en.wikipedia.org/wiki/Continuous_integration)

