# Build and Build Config Overview

> Explains OpenShift BuildConfig and how builds convert source code into container images, including source, build strategies, outputs, triggers, and post build steps.

At a high level, a build takes source code and produces a container image that can run across your application stack — frontend, middleware, backend, etc. This is the same concept as running [`docker build`](https://docs.docker.com/engine/reference/commandline/build/) with a Dockerfile to produce a runnable image.

![A simplified architecture diagram showing a dark "Code Base" panel with sections labeled Frontend, Middleware, and Backend. A red user icon on the left connects to the code base, which links to three red package/service icons on the right.](https://mintcdn.com/kodekloud-c4ac6d9a/Xa5GerWBw0vsOVlC/images/OpenShift-4/Concepts-Builds-and-Deployments/Build-and-Build-Config-Overview/codebase-frontend-middleware-backend-user-services.jpg?fit=max\&auto=format\&n=Xa5GerWBw0vsOVlC\&q=85\&s=f706a160b2b82083ca2fa46bf70beb4a)

In OpenShift, a BuildConfig is the declarative resource that describes how to transform source code into a container image. A BuildConfig typically defines:

* where the source lives (git repo, binary, or other),
* which build strategy to use (source-to-image, Dockerfile, or pipeline),
* where the resulting image should be pushed (ImageStream or external registry),
* triggers that automatically start builds (GitHub, generic webhooks, image changes),
* and any post-build steps (tests or scripts).

Example BuildConfig (typical fields and structure):

```yaml theme={null}
apiVersion: build.openshift.io/v1
kind: BuildConfig
metadata:
  name: ruby-sample-build
spec:
  runPolicy: Serial
  triggers:
    - type: GitHub
      github:
        secret: secret101
    - type: Generic
      generic:
        secret: secret101
    - type: ImageChange
  source:
    git:
      uri: https://github.com/openshift/ruby-hello-world
  strategy:
    sourceStrategy:
      from:
        kind: ImageStreamTag
        name: ruby-20-centos7:latest
  output:
    to:
      kind: ImageStreamTag
      name: origin-ruby-sample:latest
  postCommit:
    script: bundle exec rake test
```

BuildConfigs are often generated automatically (for example, when importing an application image, using the web console, or certain CLI workflows). You do not always need to author them by hand, but understanding the fields helps you customize and automate builds.

Key concepts to remember:

* A build is the process that turns source code into a runnable container image.
* A BuildConfig is the declarative definition of that process: it specifies the source, strategy, output, triggers, and post-build steps that orchestrate the build lifecycle.
* Build strategies determine how the image is created (S2I, Dockerfile, or pipelines). The sample above uses a `sourceStrategy` referencing an `ImageStreamTag`.

BuildConfig field reference (summary)

| Field | Purpose | Example |
| - | - | - |
| `spec.source` | Where build input comes from (git, binary, image). | `git: https://github.com/openshift/ruby-hello-world` |
| `spec.strategy` | Build method: `sourceStrategy` (S2I), `dockerStrategy`, or `pipelineStrategy`. | `sourceStrategy` using `ImageStreamTag: ruby-20-centos7:latest` |
| `spec.output.to` | Destination for the resulting image, often an `ImageStreamTag`. | `ImageStreamTag: origin-ruby-sample:latest` |
| `spec.triggers` | Events that start new builds automatically (GitHub, Generic webhook, ImageChange). | `type: GitHub` with a webhook secret |
| `spec.postCommit` | Commands or scripts run after build completes (tests, migrations). | `script: bundle exec rake test` |

Be careful with webhook secrets embedded in BuildConfig YAML (the sample uses `secret101`). Treat webhook and registry secrets as sensitive: store them securely and rotate them regularly. Prefer using OpenShift Secrets and secure automation to avoid exposing credentials in plain YAML.

See also and further reading:

* [OpenShift Builds — Understanding Builds and BuildConfigs](https://docs.openshift.com/container-platform/latest/cicd/builds/understanding-builds.html)
* [Source-to-Image (S2I) Overview](https://github.com/openshift/source-to-image)
* [`docker build` reference](https://docs.docker.com/engine/reference/commandline/build/)
