# Deploying applications web console

> Guide to deploying applications on OpenShift web console using pasted manifests or importing from Git to build, push, and deploy apps and inspect Deployments, Pods, and Routes

When deploying applications on OpenShift, the web console provides several UI-driven options that are fast and approachable. Two common methods are:

* Paste a Kubernetes manifest to create a single Deployment.
* Import an application from a Git repository so the console can build, push, and deploy the app automatically (using a Dockerfile or BuildConfig).

This guide walks through both approaches using the OpenShift web console, shows where to inspect runtime objects (Deployments, Pods, Routes), and includes example manifests and application source to help you get started.

## Quick option: Paste a manifest to create a Deployment

For a one-off or simple deployment, use Administrator → Workloads → Deployments → Create Deployment and paste your manifest YAML.

Minimal example (replicas and container details):

```yaml theme={null}
replicas: 3
template:
  metadata:
    labels:
      app: httpd
  spec:
    containers:
      - name: httpd
        image: >-
          image-registry.openshift-image-registry.svc:5000/opensh
        ports:
          - containerPort: 8080
```

A more complete, typical NGINX Deployment manifest:

```yaml theme={null}
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  selector:
    matchLabels:
      app: nginx
  replicas: 2 # tells deployment to run 2 pods matching the template
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
        - name: nginx
          image: nginx:1.14.2
          ports:
            - containerPort: 80
```

Paste or edit the manifest in the console and click Create. The console will create the Deployment and display detailed configuration: update strategy, maxUnavailable/maxSurge, node selectors, and the pod selector.

![A screenshot of the Red Hat OpenShift Container Platform web console showing "Deployment details" for an nginx-deployment in the default project with 2 pods. The left sidebar shows the Administrator menu with Workloads → Deployments selected.](https://mintcdn.com/kodekloud-c4ac6d9a/Xa5GerWBw0vsOVlC/images/OpenShift-4/OpenShift-Concepts-Projects-and-Users/Deploying-applications-web-console/openshift-deployment-nginx-default-2-pods.jpg?fit=max\&auto=format\&n=Xa5GerWBw0vsOVlC\&q=85\&s=b6564e0ba2f591cd9c6b5532cc2cb683)

From the Deployment details you can navigate to matching pods to inspect their status, view metrics, see the YAML, view environment variables, access logs, and open an interactive terminal session (exec). The Pods view lists running pods that belong to the Deployment:

![A screenshot of the Red Hat OpenShift Container Platform web console showing the "Project: default" pods/deployments view. The page lists nginx pods/deployments with status "Running" and a left-hand navigation menu (Workloads, Pods, Deployments, etc.).](https://mintcdn.com/kodekloud-c4ac6d9a/Xa5GerWBw0vsOVlC/images/OpenShift-4/OpenShift-Concepts-Projects-and-Users/Deploying-applications-web-console/openshift-console-default-nginx-pods.jpg?fit=max\&auto=format\&n=Xa5GerWBw0vsOVlC\&q=85\&s=27756d9f92a5b143dad74ea8e8fb9f1a)

Click a pod to view details and access logs, metrics, and an interactive terminal:

![A screenshot of the Red Hat OpenShift Container Platform web console showing the Pod details page for "nginx-deployment-8d545c96d-rm72z" with its status marked Running. The left navigation lists workloads (Pods, Deployments, etc.) and the top tabs show Details, Metrics, YAML, Environment, Logs, Events, and Terminal.](https://mintcdn.com/kodekloud-c4ac6d9a/Xa5GerWBw0vsOVlC/images/OpenShift-4/OpenShift-Concepts-Projects-and-Users/Deploying-applications-web-console/openshift-pod-details-nginx-deployment-running.jpg?fit=max\&auto=format\&n=Xa5GerWBw0vsOVlC\&q=85\&s=21b8783837da3b976b51390067ed14ff)

Opening the Terminal tab runs an exec into the container (similar to `kubectl exec`/`oc rsh`). For example, listing files inside a typical nginx container:

```bash theme={null}
# ls
bin  boot  dev  docker-entrypoint.d  docker-entrypoint.sh  etc  home  lib  lib64  media  mnt  opt  proc  root  run  sbin  srv  sys  tmp  usr  var
#
```

For simple, single-object deployments the pasted-manifest approach is fast and transparent. If your application requires multiple objects (Services, ConfigMaps, Secrets, Builds), consider importing from Git so the console can create all required resources automatically.

## Import from Git: build, push, and deploy an entire app

For multi-object applications you can import the source repository (Developer → Add → From Git). The web console detects a Dockerfile or BuildConfig and will create Builds, ImageStreams, and DeploymentConfigs/Deployments for you.

Example repository (a simple Go web API): [https://github.com/AdminTurnedDevOps/GoWebAPI](https://github.com/AdminTurnedDevOps/GoWebAPI)

![A dark-mode GitHub repository page for "AdminTurnedDevOps/GoWebAPI" showing the file list, recent commit, README preview, and repo details (branches, commits, languages). The screenshot includes the browser tabs and Windows taskbar at the bottom.](https://mintcdn.com/kodekloud-c4ac6d9a/Xa5GerWBw0vsOVlC/images/OpenShift-4/OpenShift-Concepts-Projects-and-Users/Deploying-applications-web-console/github-gowebapi-repo-dark-screenshot.jpg?fit=max\&auto=format\&n=Xa5GerWBw0vsOVlC\&q=85\&s=a293260d868087be182fa38492d42ace)

Copy the Git HTTPS URL and paste it into Developer → Add → From Git. You can set an application name, choose Deployment vs. DeploymentConfig, and specify a target port. The import form optionally creates a Route (a public URL) to expose the application outside the cluster.

![A screenshot of the Red Hat OpenShift Container Platform web console showing the "Import from Git / Dockerfile" form, with the application name set to "goweb" and options to create or cancel. The left sidebar shows Developer navigation like Topology, Builds, and ConfigMaps.](https://mintcdn.com/kodekloud-c4ac6d9a/_-d-o-x8WFPLT_DD/images/OpenShift-4/OpenShift-Concepts-Projects-and-Users/Deploying-applications-web-console/openshift-web-console-import-goweb.jpg?fit=max\&auto=format\&n=_-d-o-x8WFPLT_DD\&q=85\&s=888ae4cbe5a5161be144012bd4bb30f8)

Choose the target port (this Go app listens on 8080) and enable “Create a route to the Application” to expose it via an externally reachable hostname. A Route maps a service to an externally accessible hostname handled by the cluster’s router.

![A screenshot of the Red Hat OpenShift web console showing the Developer view with the "Advanced options" panel for creating a DeploymentConfig, including a Target port set to 8080 and a checked "Create a route to the Application" option. The left sidebar lists menu items like Topology, Builds, Helm, and ConfigMaps.](https://mintcdn.com/kodekloud-c4ac6d9a/Xa5GerWBw0vsOVlC/images/OpenShift-4/OpenShift-Concepts-Projects-and-Users/Deploying-applications-web-console/openshift-console-developer-deploymentconfig-8080-route.jpg?fit=max\&auto=format\&n=Xa5GerWBw0vsOVlC\&q=85\&s=ef0e8bad3bd83cc7cbd62a22fb039ff0)

Click Create — the console starts a Build, pushes an image into OpenShift’s internal registry, and creates the Deployment/DeploymentConfig and Route. After the build finishes you’ll see a running Pod and an active Route you can open in your browser. The root path should return the API response implemented by the source code.

Repository main.go (simple Go web API returning JSON):

```go theme={null}
package main

import (
	"encoding/json"
	"fmt"
	"log"
	"net/http"
)

type whoami struct {
	Name  string `json:"name"`
	Title string `json:"title"`
	State string `json:"state"`
}

func main() {
	http.HandleFunc("/", whoAmI)
	log.Fatal(http.ListenAndServe(":8080", nil))
}

func whoAmI(w http.ResponseWriter, r *http.Request) {
	who := []whoami{
		{
			Name:  "Michael Levan",
			Title: "Kubernetes Engineer",
			State: "NJ",
		},
	}

	js, err := json.Marshal(who)
	if err != nil {
		http.Error(w, err.Error(), http.StatusInternalServerError)
		return
	}

	w.Header().Set("Content-Type", "application/json")
	fmt.Fprintf(w, "%s", js)
}
```

After import and a successful build you can verify the running Pod in Administrator → Workloads → Pods and find the Route in Networking → Routes. Opening the Route URL will display the JSON output from the running container.

Creating a Route exposes your service externally. Do not expose sensitive internal services or data without proper authentication, TLS, and access controls.

## Quick comparison

| Method | Best for | What the console creates |
| - | - | - |
| Paste manifest (Workloads → Deployments → Create) | Single Deployment or quick tests | Deployment and Pods; you manage additional objects manually |
| Import from Git (Developer → Add → From Git) | Full applications with builds | Build, ImageStream, Deployment/DeploymentConfig, Service, optional Route |

## Next steps and references

* To use the terminal instead of the console, deploy and manage resources with `oc` or `kubectl`. See:
  * OpenShift docs: [https://docs.openshift.com/](https://docs.openshift.com/)
  * Kubernetes basics: [https://kubernetes.io/docs/concepts/overview/what-is-kubernetes/](https://kubernetes.io/docs/concepts/overview/what-is-kubernetes/)
* For build and image management learn more about BuildConfigs and ImageStreams in OpenShift documentation.

Summary

* Use pasted manifests for quick, single-object Deployments.
* Use the Import (From Git) flow to build, push, and deploy code stored in Git, with automatic creation of builds, image streams, and routes.
* You can also manage and deploy using `oc`/`kubectl` from the terminal for automation and scripting.
