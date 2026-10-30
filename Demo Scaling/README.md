# Demo Scaling

> A beginner demo showing how to deploy a simple Flask web app to OpenShift, scale replicas, and observe routing, load balancing, and session affinity via Routes and Services.

Welcome to this demo on scaling applications in OpenShift for beginners. In this lesson we deploy a simple Python web application, scale it up and down, and observe how an OpenShift Route and Service distribute traffic across application instances.

This demo uses a small, colorful Flask app that selects a background color on startup (or via an environment variable). The repository for the app is named Simple Web App Color.

![A GitLab project page for "simple-webapp-color" showing project details, repository controls (star, fork, HTTP URL) and an orange banner about adding an SSH key. The left sidebar displays navigation items like Overview, Details, Repository, Issues and CI/CD.](https://mintcdn.com/kodekloud-c4ac6d9a/Xa5GerWBw0vsOVlC/images/OpenShift-4/Networks-Services-Routes-and-Scaling/Demo-Scaling/gitlab-simple-webapp-color-ssh-banner.jpg?fit=max\&auto=format\&n=Xa5GerWBw0vsOVlC\&q=85\&s=818aef06eb51f156bc9084323fdbd20d)

Overview of the demo workflow

* Create a new project (namespace) in OpenShift.
* Add the application by pointing the console to the Git repository.
* Build and deploy the image (watch the push progress).
* Access the app via the Route URL.
* Scale replicas from the web console and observe traffic distribution.
* Optionally change the router balancing annotations to alter session affinity.

Application code (Flask)
The app renders a simple page showing the container hostname and a background color. The color is chosen randomly on first start unless `APP_COLOR` is set.

```python theme={null}
import os
import socket
import random
from flask import Flask, render_template

app = Flask(__name__)

color_codes = {
    "red": "#e74c3c",
    "green": "#16a085",
    "blue": "#2980b9",
    "blue2": "#30336b",
    "pink": "#be2edd",
    "darkblue": "#130f40"
}

# Use environment variable APP_COLOR if set; otherwise choose a random color
color = os.environ.get("APP_COLOR") or random.choice(list(color_codes.keys()))

@app.route("/")
def main():
    # Render the template with the container/host name and the chosen color
    return render_template("hello.html", name=socket.gethostname(), color=color_codes[color])

@app.route("/color/<new_color>")
def new_color(new_color):
    # Allow selecting a color via URL path (must be a key in color_codes)
    return render_template("hello.html", name=socket.gethostname(), color=color_codes[new_color])

if __name__ == "__main__":
    app.run(host="0.0.0.0", port=8080)
```

What happens when you deploy

* The OpenShift Build will create an image and push it to the internal registry. During the push you might see progress lines like:

```text theme={null}
Pushed 6/10 layers, 66% complete
Pushed 7/10 layers, 75% complete
Pushed 7/10 layers, 92% complete
Pushed 8/10 layers, 94% complete
Pushed 9/10 layers, 98% complete
Pushed 10/10 layers, 100% complete
Push successful
```

* OpenShift creates a Deployment (or DeploymentConfig), Pods, a Service that selects those Pods, and a Route that exposes the Service externally.
* The Route provides the external URL you use to access the app. Each request to that Route will be forwarded to one of the Service endpoints (pods) according to the router’s load-balancing behavior.

Scaling from the OpenShift console

* In the web console you can increase or decrease the replica count using the up/down arrows on the application’s Deployment card.
* Scaling updates the Deployment's replica count and the OpenShift controller creates or removes pods to match the desired replicas.
* Because the Service selector targets the Deployment’s pods, newly created pods automatically become endpoints and receive traffic.

Why you see different colors

* Each pod may choose a different color at startup, so multiple replicas can show different background colors.
* When you scale to 2+ replicas and access the Route URL from different browser sessions (or incognito windows), you will likely hit different pods and see different colors.
* If you refresh the same browser tab repeatedly and keep getting the same pod, that indicates session affinity on the router (sticky sessions) or a load-balancing algorithm that consistently maps your client to the same backend.

OpenShift router balancing and session affinity
The router’s behavior (balance algorithm, cookie usage, session affinity) is controlled via Route annotations. To change how traffic is balanced, edit the Route YAML and add annotations such as:

```yaml theme={null}
metadata:
  annotations:
    haproxy.router.openshift.io/balance: "roundrobin"
    haproxy.router.openshift.io/disable_cookies: "true"
```

Common router-related annotations

| Annotation | Purpose |
| - | - |
| `haproxy.router.openshift.io/balance` | Selects the load-balancing algorithm (e.g., `roundrobin`, `leastconn`, etc.). |
| `haproxy.router.openshift.io/disable_cookies` | If set to `"true"`, disables router cookie-based stickiness. |
| `haproxy.router.openshift.io/cookie_name` | Sets the name of the cookie used for session affinity. |
| `haproxy.router.openshift.io/pod-concurrent-connections` | Limits concurrent connections routed to a single pod (useful for throttling). |

After you save the Route with `balance: "roundrobin"` and `disable_cookies: "true"`, subsequent requests from the same browser tab will be more evenly distributed across pods instead of being pinned to one.

Scaling via CLI (optional)
You can also scale from the command line using `oc`. Example:

* Scale to 3 replicas:
  `oc scale deployment/simple-webapp-color --replicas=3 -n your-project`
* Check pods and endpoints:
  `oc get pods -n your-project`
  `oc get endpoints -n your-project`

Best practice for session state

If you need session affinity for application-level sessions, consider using a dedicated session store (for example, Redis, a database, or another central store) instead of relying on router stickiness. Router annotations for sticky sessions can help but are not a replacement for centralized session management.

Quick reference: OpenShift resources involved

| Resource | Role |
| - | - |
| Deployment / DeploymentConfig | Manages pod replicas and rollouts |
| ReplicaSet | Maintains a stable set of pod replicas for Deployments |
| Pod | The running container instance of your app |
| Service | Routes traffic to pods that match its selector |
| Route | Exposes a Service externally and is handled by the OpenShift router |

Links and references

* [OpenShift Documentation](https://docs.openshift.com/)
* [OpenShift Routes (reference)](https://docs.openshift.com/container-platform/latest/networking/routes/route-configuration.html)
* [Flask Documentation](https://flask.palletsprojects.com/)
* [Redis (session store)](https://redis.io/)

