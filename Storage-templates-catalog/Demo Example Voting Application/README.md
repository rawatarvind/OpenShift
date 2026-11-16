# Demo Example Voting Application

> Guide to deploying a multi-tier example voting application on OpenShift using Redis, PostgreSQL, a Python front-end, Node.js results service, and a worker, including config and troubleshooting.

Hello, and welcome to this lesson on OpenShift for Beginners ([OpenShift 4](https://learn.kodekloud.com/user/courses/openshift-4)). This guide walks through deploying a multi-tier example voting application on an OpenShift cluster. The demo uses multiple microservices (front-end, Redis, results service with PostgreSQL, and a worker) to show how services, environment variables, builds, and deployments work together.

The application source is in a GitHub repository (forked from the canonical example voting app). A small change was added to require a Redis password so services authenticate with each other.

## Project setup

Create a new project/namespace for this demo. In the OpenShift web console you can create a new project named `voting-application`, or use the CLI:

```bash theme={null}
oc new-project voting-application
```

We will deploy Redis, the front-end (`vote`), the results service (`result`), and a background worker (`worker`) inside this project.

## Deploy Redis (cache / queue)

The front-end depends on Redis, so deploy Redis first. If Redis is not visible in your cluster service catalog, import a template from the OpenShift Origin examples.

![A screenshot of a GitHub repo page (openshift/origin/examples/db-templates) showing a file listing of JSON database templates (mariadb, mongodb, mysql, postgresql, redis) and a README titled "OpenShift 3 Database Examples."](https://mintcdn.com/kodekloud-c4ac6d9a/_-d-o-x8WFPLT_DD/images/OpenShift-4/Storage-Templates-and-Catalog/Demo-Example-Voting-Application/openshift-db-templates.jpg?fit=max\&auto=format\&n=_-d-o-x8WFPLT_DD\&q=85\&s=028e688c28d50c372fd4842344b0d9dd)

Steps to add Redis via template:

* Browse [https://github.com/openshift/origin/tree/master/examples/db-templates](https://github.com/openshift/origin/tree/master/examples/db-templates) and open the **Redis Ephemeral** template.
* Copy the JSON/YAML, then in the OpenShift web console choose **Add** → **Import YAML/JSON**, paste the template, click **Create**, and when prompted choose **Save Template** so Redis appears in your catalog.

The template exposes several parameters you can set when creating the Redis service; for example:

```json theme={null}
[
  {
    "description": "Password for the Redis connection user.",
    "displayName": "Redis Connection Password",
    "from": "[a-zA-Z0-9]{16}",
    "generate": "expression",
    "name": "REDIS_PASSWORD",
    "required": true
  },
  {
    "description": "Version of Redis image to be used (3.2 or latest).",
    "displayName": "Version of Redis Image",
    "name": "REDIS_VERSION",
    "required": true,
    "value": "3.2"
  }
]
```

After saving the template and creating a Redis instance from the catalog, leave defaults or configure parameters as needed. Provide a password (we will use this same password in the `vote` application). After creation, note the connection details you are shown; for example:

```text theme={null}
The following service(s) have been created in your project: redis.

Password: redis_password

Connection URL: redis://redis:6379/

For more information about using this template, including OpenShift considerations, see https://github.com/sclorg/redis-container/blob/master/3.2
```

Internally this Redis service is reachable by the DNS name `redis` from other pods in the same project.

## Deploy the front-end (vote) application

The front-end is a Python Flask app located in the `vote` folder of the repository. Create a new application using the Git repository and set the context directory to `/vote` (use Advanced Options). Choose a name (for example `vote`) and leave other settings at their defaults. OpenShift will automatically create a BuildConfig and a DeploymentConfig and will begin a build.

You can track build progress in the web console Overview; a build push may show output like:

```text theme={null}
Pushing image 172.30.1.1:5000/voting-application/vote:latest ...
Pushed 0/10 layers, 1% complete
Pushed 1/10 layers, 14% complete
Pushed 2/10 layers, 22% complete
Pushed 3/10 layers, 31% complete
Pushed 4/10 layers, 41% complete
Pushed 5/10 layers, 51% complete
```

If the app route opens but voting fails with an authentication error, the reason is typically that the `REDIS_PASSWORD` environment variable is not set in the `vote` DeploymentConfig. The app expects the password in `REDIS_PASSWORD`.

Here is the relevant excerpt from the Python Flask application showing how the password is read from the environment and used by Redis:

```python theme={null}
from flask import Flask, render_template, request, make_response, g
from redis import Redis
import os
import socket
import random
import json

option_a = os.getenv('OPTION_A', "Cats")
option_b = os.getenv('OPTION_B', "Dogs")
# Default here matches the example Redis password used earlier; in production, do not hardcode secrets.
redis_password = os.getenv('REDIS_PASSWORD', "redis_password")
hostname = socket.gethostname()

app = Flask(__name__)

def get_redis():
    if not hasattr(g, 'redis'):
        g.redis = Redis(host="redis", db=0, password=redis_password, socket_timeout=5)
    return g.redis

@app.route("/", methods=['POST', 'GET'])
def hello():
    voter_id = request.cookies.get('voter_id')
    # ... rest of the handler that uses get_redis() ...
```

Do not hardcode sensitive credentials in application code or environment defaults for production. Use Kubernetes/OpenShift Secrets or a secret manager to store passwords securely.

To fix the missing Redis password, edit the `vote` DeploymentConfig (or use the web console Environment tab) and add an environment variable `REDIS_PASSWORD` with the same password you provided when creating Redis. After saving the environment change, the pod will redeploy.

![A screenshot of the OpenShift Origin web console open to the "Example Voting Application" project, showing the "Environment" tab for the "vote" deployment. The page displays an editable environment variable entry named REDIS\_PASSWORD with options to add values or load from a Config Map/Secret.](https://mintcdn.com/kodekloud-c4ac6d9a/_-d-o-x8WFPLT_DD/images/OpenShift-4/Storage-Templates-and-Catalog/Demo-Example-Voting-Application/openshift-vote-env-redis-password.jpg?fit=max\&auto=format\&n=_-d-o-x8WFPLT_DD\&q=85\&s=e159ab085730d0e388dea779fd5ade3b)

After the redeploy, casting a vote should succeed and you will see confirmation in the UI (a check mark next to the chosen option).

![A browser screenshot of a "Cats vs Dogs!" voting page with two large buttons labeled "CATS" (selected, with a check mark) and "DOGS." The page footer shows "Processed by container ID vote-2-rh5bv."](https://mintcdn.com/kodekloud-c4ac6d9a/_-d-o-x8WFPLT_DD/images/OpenShift-4/Storage-Templates-and-Catalog/Demo-Example-Voting-Application/cats-vs-dogs-vote-screenshot.jpg?fit=max\&auto=format\&n=_-d-o-x8WFPLT_DD\&q=85\&s=f4ce84e26128330c36609907625beb72)

## Deploy PostgreSQL and the results service

The results application needs PostgreSQL to persist aggregated votes. PostgreSQL is available in the catalog by default. When you add the database instance, ensure the service name matches what the application expects — in this example the applications expect the DB service to be named `db`. Configure the database user and password as required.

For production deployments, store credentials in OpenShift Secrets (or a dedicated credentials manager) instead of plain environment variables.

Table: application components and expected service names / ports

| Component | Purpose | Expected service name / port |
| - | - | - |
| vote (Python front-end) | User-facing voting UI | connects to `redis` on port `6379` |
| redis | Cache / queue for votes | service name `redis` |
| result (Node.js results service) | Real-time results display | expects DB service named `db`, listens on `PORT` (set to `8080`) |
| worker | Background worker that moves votes from Redis → PostgreSQL | connects to `redis` and `db` |

### Deploy the results service

The result app is a Node.js service located in the `/result` context directory of the repository. Use Advanced Options and set the context dir to `/result`. Provide a name (for example `result`) and the Git repo URL.

Key details from the Node.js app:

```javascript theme={null}
var express = require('express'),
    async = require('async'),
    pg = require("pg"),
    cookieParser = require('cookie-parser'),
    bodyParser = require('body-parser'),
    methodOverride = require('method-override'),
    app = express(),
    server = require('http').Server(app),
    io = require('socket.io')(server);

io.set('transports', ['polling']);

var port = process.env.PORT || 4000;

io.sockets.on('connection', function (socket) {
    socket.emit('message', { text : 'Welcome!' });

    socket.on('subscribe', function (data) {
        socket.join(data.channel);
    });
});
```

Set the `PORT` environment variable to `8080` so OpenShift exposes the service on the platform-standard HTTP port. After creating this app, verify the route and open the results page.

If the results page shows an immediate 50/50 tie even after you voted, the most likely cause is that the worker responsible for moving votes from Redis into PostgreSQL is not running.

## Deploy the worker (background processor)

The worker code is in the `/worker` directory. Unlike the other components, we will build the worker with a Docker strategy using the Dockerfile in that context directory.

Create a new application for `worker` (Advanced Options, context `/worker`). Open the BuildConfig for the newly created app and edit the YAML: switch the `strategy` from `Source` to `Docker`, and remove the `from` section under the strategy so the Dockerfile in the repo is used directly. Save the BuildConfig and start a build:

```bash theme={null}
oc start-build worker -n voting-application
```

The Docker build will run, build the image, and push it to the internal registry. Typical trimmed build output:

```text theme={null}
---> Using cache
Step 9 : CMD java -jar target/worker-jar-with-dependencies.jar
---> Using cache
Successfully built a818f483a745

Pushing image 172.30.1.1:5000/voting-application/worker:latest ...
Pushed 0/10 layers, 9% complete
Pushed 1/10 layers, 11% complete
Pushed 2/10 layers, 21% complete
Pushed 3/10 layers, 33% complete
Pushed 4/10 layers, 45% complete
Pushed 5/10 layers, 56% complete
Pushed 6/10 layers, 65% complete
Pushed 7/10 layers, 75% complete
Pushed 8/10 layers, 83% complete
Pushed 9/10 layers, 91% complete
```

After the worker is running, inspect its logs to confirm it connects to Redis and PostgreSQL and processes the vote queue. Example worker log lines:

```text theme={null}
1 Connected to redis
2 Connected to db
3 Watching vote queue
4 Processing vote for 'a' by 'a44ff790cd088d2'
5 Processing vote for 'b' by 'a44ff790cd088d2'
6 Processing vote for 'a' by 'a44ff790cd088d2'
7 Processing vote for 'b' by 'a44ff790cd088d2'
```

Once the worker processes queued votes, the results page will reflect new votes immediately.

## Troubleshooting tips

* If you see authentication errors from Redis or PostgreSQL, double-check that the corresponding environment variables (`REDIS_PASSWORD`, DB credentials) are set on the correct DeploymentConfig and that the service names match what the applications expect (`redis`, `db`).
* Use `oc logs <pod>` to view pod logs (including the worker) and `oc get pods` to check pod statuses.
* Prefer Secrets for credentials: create a Secret and reference it from your DeploymentConfig (or use the web console to mount secrets as env vars).

## Summary

This lesson covered deploying a multi-tier voting application on OpenShift:

* Add/import templates (Redis) and register them in the catalog.
* Deploy the `vote` (Python) front-end and ensure `REDIS_PASSWORD` is set.
* Deploy PostgreSQL from the catalog and name the service `db`.
* Deploy the `result` (Node.js) service and set `PORT=8080`.
* Build the `worker` via Docker strategy and verify that it processes votes from Redis into PostgreSQL.

Links and references

* [OpenShift Documentation](https://docs.openshift.com/)
* [OpenShift Origin examples (db-templates)](https://github.com/openshift/origin/tree/master/examples/db-templates)
* [Redis container images / SCL](https://github.com/sclorg/redis-container)

That completes this demo on deploying the example voting application on OpenShift.

