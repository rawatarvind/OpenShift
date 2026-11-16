# S2I

> Describes Source-to-Image builds that convert source code into OCI container images using pluggable scripts, enabling reproducible, secure, incremental builds without requiring Dockerfiles

This lesson covers Source-to-Image (S2I) builds — a developer-focused workflow that converts application source code into a Docker/OCI-compliant container image without requiring you to author or maintain a Dockerfile.

At a high level: a developer writes code and pushes it to a repository. S2I takes that source, assembles a container image from it, and pushes the resulting image to a container registry. That lets developers focus on source code while S2I handles the image creation and image-layer assembly.

S2I is conceptually similar to [Cloud Native Buildpacks](https://learn.kodekloud.com/user/courses/cloud-native-buildpacks): both automate building runnable images from source, but S2I uses small, pluggable scripts (provided by the builder image or the repository) to control how source is assembled into an image.

Why use S2I? Key benefits:

* No Dockerfile to maintain: S2I creates a Docker/OCI image directly from source.
* Consistent, platform-managed builds: the build runs inside a container environment managed by the platform so you don't need to manage the build runtime.
* Build-time security and access control: S2I builds can be restricted by permissions and authentication to authorized users or CI systems.

![A presentation slide titled "What's the point of S21?" showing step 1: "Create Docker Image from Source Code" with a red numbered box and an icon of a terminal converting into an image.](https://mintcdn.com/kodekloud-c4ac6d9a/Xa5GerWBw0vsOVlC/images/OpenShift-4/Concepts-Builds-and-Deployments/S2I/s21-step1-create-docker-image.jpg?fit=max\&auto=format\&n=Xa5GerWBw0vsOVlC\&q=85\&s=ae7977b7ce20ad38212884d5610419ae)

Because S2I manages image creation, you can focus on source and build logic instead of build assets like Dockerfiles or custom build containers. The platform applies the same build rules reliably and repeatably across environments.

S2I is built with security in mind: access to build actions can be gated by permissions, so only authorized people or CI systems can run builds and push images.

![A presentation slide titled "What's the point of S21?" with four red numbered boxes down the left side. The slide notes "Built for security utilizing permissions" and shows an icon of a card with a padlock and checkmark.](https://mintcdn.com/kodekloud-c4ac6d9a/Xa5GerWBw0vsOVlC/images/OpenShift-4/Concepts-Builds-and-Deployments/S2I/s21-security-slide-permissions-card-lock.jpg?fit=max\&auto=format\&n=Xa5GerWBw0vsOVlC\&q=85\&s=71a6c101395ebd771fe80c76a14ec0bc)

Primary advantages in practice:

* Injects source into a Docker-formatted image: S2I handles assembling image layers and placing compiled artifacts or runtime files where they belong.
* Faster incremental builds: S2I can reuse and incrementally update previous artifacts when configured (for example, via `save-artifacts` or by relying on prebuilt components in the builder image), reducing rebuild work.
* Easier patching and rebuilds: code changes drive consistent rebuilds; build scripts can be written to reuse artifacts and speed up subsequent builds.

![A slide-style infographic titled "Primary Advantages" showing a circular diagram labeled "S21" with three callouts reading "Inject code into a docker-formatted container image," "Faster build times," and "Easily patch the build."](https://mintcdn.com/kodekloud-c4ac6d9a/Xa5GerWBw0vsOVlC/images/OpenShift-4/Concepts-Builds-and-Deployments/S2I/s21-primary-advantages-inject-docker-image.jpg?fit=max\&auto=format\&n=Xa5GerWBw0vsOVlC\&q=85\&s=593d8e208078600b40ac42bd82da35bd)

How S2I organizes build behavior

* S2I relies on small helper scripts that define how source is assembled into the image and how the resulting image should behave at runtime.
* These scripts are flexible: they can be written in any language and either live in the source repository or be provided by the builder image or a remote URL.
* The required script entry points and common script lookup locations are summarized below.

| Script entry points | Purpose |
| - | - |
| `assemble` | Required. Builds the application artifacts and places them into the image filesystem. |
| `run` | Optional. Defines container runtime/start behavior for the produced image. |
| `save-artifacts` | Optional. Persists build outputs between builds to speed incremental rebuilds. |

| Script lookup / provision methods | Example or location |
| - | - |
| `.s2i/bin` | Bundled inside the source repository at `.s2i/bin` |
| `io.openshift.s2i.scripts-url` (annotation) | Provide a URL via repository annotations |
| `image:///path_to_scripts_dir` | Scripts baked into the builder image |
| `file:///path_to_scripts_dir` | Local file URL pointing to scripts |
| `http(s)://path_to_scripts_dir` | Scripts retrieved from an HTTP(S) endpoint |

The `assemble` script is required; it compiles or prepares the application artifacts into the image. `run` and `save-artifacts` are optional but commonly used: `run` describes container start commands and `save-artifacts` preserves build outputs for faster subsequent builds.

Example: a simple Bash `assemble` script

* Below is a minimal `assemble` script pattern. It restores any saved artifacts, moves the application source into place, builds, and installs the artifacts into the image filesystem. This script typically lives at `.s2i/bin/assemble` in the source repo, or can be provided via one of the script URLs above.

```bash theme={null}
#!/bin/bash
set -euo pipefail

# restore build artifacts if any exist
if [ -d /tmp/s2i/artifacts ] && [ "$(ls -A /tmp/s2i/artifacts)" ]; then
    mv /tmp/s2i/artifacts/* "$HOME"/.
fi

# move the application source into the home directory
mv /tmp/s2i/src "$HOME"/src

# build application artifacts
pushd "$HOME" >/dev/null
make all

# install the artifacts into the image filesystem
make install
popd >/dev/null
```

Note: conventions such as `/tmp/s2i/artifacts` and `/tmp/s2i/src` are common in S2I implementations, but exact paths can vary depending on your S2I platform or builder image.

Make sure your builder image and registry credentials are configured correctly: S2I will need permission to push the resulting image to the target registry, and builder images may need access to network resources used during builds (package registries, artifact endpoints).

When to choose S2I

* Use S2I if you want a simple way to convert source code into reproducible, OCI-compliant images without managing Dockerfiles.
* Consider alternatives (like Buildpacks or custom Dockerfiles) when you need a different lifecycle model, language-specific optimizations, or a non-scriptable build pipeline.

Links and references

* [Cloud Native Buildpacks](https://learn.kodekloud.com/user/courses/cloud-native-buildpacks) — comparable build automation approach
* OpenShift S2I docs — see your platform’s documentation for platform-specific S2I features and annotations
* OCI Image Specification — background on OCI-compliant images

Further reading

* If you’re using S2I on OpenShift, consult the OpenShift build and image documentation for examples of annotations, buildconfigs, and registry integration.
* For CI integration, ensure your CI service has the appropriate credentials and permissions to trigger S2I builds and push images to the target registry.

