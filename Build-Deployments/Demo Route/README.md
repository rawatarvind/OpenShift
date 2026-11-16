# Demo Route

> Guide to viewing, deleting, and recreating OpenShift Routes to expose the Sock Shop front-end and restore external access by mapping routes to services and target ports

Assuming the Sock Shop application is already deployed, open the OpenShift console and navigate to Workloads → Deployments to confirm all Sock Shop components are running (carts, carts-db, catalogue, front-end, orders, etc.).

![A screenshot of the Red Hat OpenShift Container Platform web console showing the "Deployments" view for the default project, listing deployments (carts, carts-db, catalogue, front-end, orders, etc.) with status, labels and pod selectors. The left sidebar shows Workloads navigation and there's a "Create Deployment" button in the header.](https://mintcdn.com/kodekloud-c4ac6d9a/Xa5GerWBw0vsOVlC/images/OpenShift-4/Concepts-Builds-and-Deployments/Demo-Route/openshift-console-deployments-list.jpg?fit=max\&auto=format\&n=Xa5GerWBw0vsOVlC\&q=85\&s=1bda470102d7c16a53bc2ab7c8b7c2f5)

Next, go to Networking → Routes to view how the front-end is exposed externally. Clicking the front-end route opens the external URL so you can verify the application is reachable.

If you want to practice route management, delete the route and recreate it. When you delete a route, confirm the deletion in the prompt.

![A screenshot of the Red Hat OpenShift web console showing a "Delete Route?" confirmation dialog asking to delete the "front-end" route in the "default" namespace, with Cancel and Delete buttons. The blurred background shows the Routes page and the left-hand navigation menu.](https://mintcdn.com/kodekloud-c4ac6d9a/Xa5GerWBw0vsOVlC/images/OpenShift-4/Concepts-Builds-and-Deployments/Demo-Route/openshift-delete-route-confirmation-front-end.jpg?fit=max\&auto=format\&n=Xa5GerWBw0vsOVlC\&q=85\&s=a4e7f83b6cbba35260bb8d899e8e9614)

Deleting a Route removes external access (hostname/path) but does not delete the underlying `Service`, `Deployment`, or Pods. To restore external access, create a new Route that targets the existing Service and port.

To recreate the route:

* Click Create Route.
* Give it a name (for example, `frontend-app`).
* Optionally specify a hostname (if you manage DNS) or leave it for OpenShift to auto-generate using the cluster domain.
* Optionally set a path (for example, `/test`) or leave it at the root `/`.
* Select the Service (the front-end service) and the target port.
* If needed, enable TLS and supply certificates for a secure route.

Use the form below to configure the route values and then save.

![A screenshot of the Red Hat OpenShift Container Platform web console showing the "Create Route" form with the Name field filled as "frontend-app" and fields for Hostname, Path, Service, and Target port. The left navigation highlights Networking → Routes.](https://mintcdn.com/kodekloud-c4ac6d9a/Xa5GerWBw0vsOVlC/images/OpenShift-4/Concepts-Builds-and-Deployments/Demo-Route/openshift-create-route-frontend-app.jpg?fit=max\&auto=format\&n=Xa5GerWBw0vsOVlC\&q=85\&s=17d99cf529afcd65d2ccca57e82efda9)

Quick reference — example route settings:

| Field | Example |
| - | - |
| Name | `frontend-app` |
| Hostname | `optional-custom-host.example.com` (or leave blank for auto-generated) |
| Path | `/test` (optional) |
| Service | `front-end` |
| Target port | `http` or the port exposed by the service |
| TLS | `Edge`, `Passthrough`, or `Reencrypt` (if required) |

After creating the route, return to the Routes list and open the external URL to verify the application is reachable again.

![A screenshot of the Red Hat OpenShift Container Platform web console showing the "frontend-app" Route details (status Accepted) with the left-hand navigation highlighting Networking → Routes. The page shows route metadata like namespace, labels, and the external URL.](https://mintcdn.com/kodekloud-c4ac6d9a/Xa5GerWBw0vsOVlC/images/OpenShift-4/Concepts-Builds-and-Deployments/Demo-Route/openshift-console-frontend-app-route-accepted.jpg?fit=max\&auto=format\&n=Xa5GerWBw0vsOVlC\&q=85\&s=0c0014fa9490d31237e945d67d941e15)

You should now see the application available at the new route. Note that the route name can change (for example, from `front-end` to `frontend-app`), but the route maps the external hostname/path to the same Service and target port to restore external access.

Links and references:

* OpenShift Routes documentation: [https://docs.openshift.com/container-platform/latest/networking/routes/route-configuration.html](https://docs.openshift.com/container-platform/latest/networking/routes/route-configuration.html)

