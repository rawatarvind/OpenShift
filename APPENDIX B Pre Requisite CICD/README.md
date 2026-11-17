# APPENDIX B Pre Requisite CICD

> Introductory overview of CI/CD and Docker explaining build pipelines, artifact packaging, Docker image creation, testing, registry publishing and deployment automation for reproducible releases

This lesson is a prerequisite introduction to CI/CD and Docker intended for beginners. If you already understand these concepts, you can skip ahead to the next lesson where they are applied in practice.

Hello and welcome to this lesson on CI/CD builds.

Once your application is built and the source code is pushed to a repository, the next step is producing a distributable artifact. Different languages and ecosystems produce different artifact formats:

| Language / Platform | Typical Artifact | Example packaging tools |
| - | - | - |
| Java | JAR, WAR, EAR | `maven`, `gradle` |
| Python | `.tar.gz`, `.whl` | `setuptools`, `pip` |
| Ruby | `.gem` | `gem` |

After packaging, the artifact is what you distribute. Typical runtime/install commands include:

```bash theme={null}
java -jar app.jar
pip install app.tar.gz
gem install app.gem
```

However, shipping only the artifact assumes the target environment (operations) will perform additional setup, such as:

* Installing the runtime/platform (Java, Python, Ruby, Tomcat, etc.)
* Installing and configuring dependencies and system packages
* Updating configuration files and system services
* Starting the application

Leaving these steps to manual operations introduces risk: inconsistent environments, configuration drift, and deployment failures. Containerizing the application removes a lot of that variability. By writing a Dockerfile you can encode the runtime, dependencies, and configuration steps needed to run your app. The result is a reproducible, portable Docker image.

Build the Docker image locally with:

```bash theme={null}
docker build -t myapp:latest .
```

Anyone with Docker can run that image:

```bash theme={null}
docker run --rm -p 8080:8080 myapp:latest
```

After building the image, it should go through automated testing. Typical automated test types:

* Unit tests
* Integration tests
* End-to-end/UI tests (e.g., Robot Framework, Selenium)

Only images that pass the test stage are published to a Docker registry (Docker Hub or a private registry), from which they are deployed to container hosts or orchestration platforms such as Kubernetes.

The full sequence — commit source code → build artifact → build Docker image → test → publish image to registry → deploy to environments — is called the build pipeline. When this pipeline is automated with orchestration tools, the practice is referred to as continuous integration and continuous delivery/deployment (CI/CD).

Tools commonly used to automate pipelines include [Jenkins](https://learn.kodekloud.com/user/courses/jenkins), [GitLab CI](https://learn.kodekloud.com/user/courses/gitlab-ci-cd-architecting-deploying-and-optimizing-pipelines), and Bamboo.

![A DevOps build pipeline diagram showing Source Code -> Build -> Test -> Release -> Deploy with icons for Dockerfile, Android testing, a Docker registry (cloud), and Kubernetes. Jenkins is shown above orchestrating the flow between stages.](https://mintcdn.com/kodekloud-c4ac6d9a/Xa5GerWBw0vsOVlC/images/OpenShift-4/Conclusion-and-APPENDIX/APPENDIX-B-Pre-Requisite-CICD/devops-build-pipeline-jenkins-docker-kubernetes.jpg?fit=max\&auto=format\&n=Xa5GerWBw0vsOVlC\&q=85\&s=c9bae4f9fb1053afd12ef30875495e61)

Using automation to run and orchestrate each pipeline stage (build, test, release, deploy) enables frequent, reliable, and repeatable releases. Below is a concise breakdown of common pipeline stages, what they do, and examples:

| Stage | Purpose | Example tools / commands |
| - | - | - |
| Source | Version control and triggers | `git push` (Git) |
| Build | Compile / package the application | `mvn package`, `python setup.py sdist` |
| Image | Create a container image with runtime + artifact | `docker build -t myapp:latest .` |
| Test | Execute automated tests | `pytest`, `mvn test`, Selenium suites |
| Release | Publish the image to a registry | `docker push myrepo/myapp:latest` |
| Deploy | Deploy to environments (staging/production) | Kubernetes manifests, Helm charts, `kubectl apply -f` |

Packaging your application as a Docker image encapsulates the runtime and dependencies, significantly reducing environment-related deployment issues and improving reproducibility across development, testing, and production.

Let us go through a demo of adding a Dockerfile to our application and configuring a simple CI/CD pipeline with [GitLab CI](https://learn.kodekloud.com/user/courses/gitlab-ci-cd-architecting-deploying-and-optimizing-pipelines) in the next lesson.

That's it for this lesson.
