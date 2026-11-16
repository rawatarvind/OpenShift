# Demo Build and Build Config

> Guide to creating and managing OpenShift BuildConfigs and builds from Git repositories, using console and oc commands

In the OpenShift console, navigate to Builds → Build Configs and click **Create BuildConfig** to open the example form. This walkthrough shows how to configure a BuildConfig to build the Sock Shop application from source.

1. In your Git repository browser click the **Code** button and copy the HTTPS URL for the repo you want to build.
2. In the BuildConfig form, replace the example Git URI with your Sock Shop repo URL and set the BuildConfig name (for example `sock-shop`).
3. Verify the Git branch (for example `master` or `main`) and choose the build strategy (commonly `Source` for S2I/source builds).

Example BuildConfig (trimmed to show the key fields to change):

```yaml theme={null}
apiVersion: build.openshift.io/v1
kind: BuildConfig
metadata:
  name: sock-shop
  namespace: default
spec:
  source:
    git:
      ref: master
      uri: 'https://github.com/microservices-demo/microservices-demo.git'
    type: Git
  strategy:
    type: Source
```

Key BuildConfig fields

| Field | Purpose | Example / Notes |
| - | - | - |
| `apiVersion` / `kind` | Resource identification | `build.openshift.io/v1`, `BuildConfig` |
| `metadata.name` | BuildConfig resource name | `sock-shop` |
| `spec.source.git.uri` | Git repository URL to build | `https://github.com/microservices-demo/microservices-demo.git` |
| `spec.source.git.ref` | Branch or ref to build | `master` or `main` |
| `spec.strategy.type` | Build strategy (S2I, Docker, Source, etc.) | `Source` |

The important piece that actually provides the build environment is the builder image referenced in `spec.strategy.from` (or other `from` locations). In OpenShift this commonly references an ImageStreamTag (for example `nodejs:14` or `s2i-ruby:2.7`) — the builder image runs the build steps and produces the application image.

The BuildConfig ties your source repository to a builder image (ImageStreamTag). That’s why a BuildConfig takes your code and creates a container image by using another container image as the build environment.

After you save the BuildConfig, OpenShift will schedule the build automatically. Open the BuildConfig details in the console and check the Events or Builds tab to confirm the build started and to inspect any errors.

Automating BuildConfig creation with oc new-app
If you prefer the CLI, `oc new-app` can create an ImageStream, a BuildConfig, a Deployment (or DeploymentConfig), and a Service for you when you point it at a Git repository:

```bash theme={null}
oc new-app https://github.com/AdminTurnedDevOps/GoWebAPI
```

Typical `oc new-app` output:

```text theme={null}
--> Found image 1 for "golang:1.16"
    Go Toolset available as a container is a base platform for building and running various Go applications

* A Docker build using source code from https://github.com/AdminTurnedDevOps/GoWebAPI will be created
    * The resulting image will be pushed to image stream tag "gowebapi:latest"
    * Use 'oc start-build' to trigger a new build

--> Creating resources ...
    imagestream.image.openshift.io "gowebapi" created
    buildconfig.build.openshift.io "gowebapi" created
    deployment.apps "gowebapi" created
    service "gowebapi" created
--> Success
Build scheduled, use 'oc logs -f buildconfig/gowebapi' to track its progress.
Application is not exposed. You can expose services to the outside world by executing one or more of the commands below:
  'oc expose service/gowebapi'
Run 'oc status' to view your app.
```

If you refresh the console Build Configs page you will see the new BuildConfig (for example `gowebapi`) — demonstrating that OpenShift can create BuildConfigs automatically when creating an application from source.

If your Git repo is private, `oc new-app` or the BuildConfig will need credentials or a secret to access the repository. Make sure to create and reference the appropriate secret in `spec.source` before starting the build.

Manually starting or re-triggering a build
Use `oc start-build` to manually start a build from a BuildConfig:

```bash theme={null}
oc start-build <buildconfig-name>
# Example:
oc start-build sock-shop
```

Stream build logs to monitor progress and diagnose problems:

```bash theme={null}
oc logs -f buildconfig/<buildconfig-name>
# Example:
oc logs -f buildconfig/sock-shop
```

Command summary

| Command | Use case | Example |
| - | - | - |
| `oc new-app <git-url>` | Create ImageStream, BuildConfig, Deployment, Service from source | `oc new-app https://github.com/AdminTurnedDevOps/GoWebAPI` |
| `oc start-build <name>` | Trigger a build from an existing BuildConfig | `oc start-build sock-shop` |
| `oc logs -f buildconfig/<name>` | Follow build logs | `oc logs -f buildconfig/sock-shop` |
| Console → Builds → Build Configs | Inspect BuildConfig events, builds, and status | N/A |

Summary / Best practices

* Create or edit a BuildConfig to point at your Git repo and choose an appropriate builder ImageStreamTag (the `from` image).
* Use the console or `oc` commands (`oc new-app`, `oc start-build`, `oc logs -f`) to create, trigger, and monitor builds.
* Check BuildConfig Events in the console to confirm success or view errors.
* For private repos, add the necessary credentials/Secrets and reference them in the BuildConfig before starting builds.

For more details on BuildConfig and ImageStreams, refer to the OpenShift documentation and your cluster’s ImageStream catalog for available builder images.

