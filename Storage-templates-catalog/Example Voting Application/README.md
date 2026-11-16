# Example Voting Application

> Explains microservices using a sample voting app, its Redis worker PostgreSQL components, data flow and steps to containerize and deploy the stack.

Hello and welcome to this lesson.

My name is Michael Levan.

In this lesson we will explore microservices architecture using a simple web application, and then deploy that application.

![A presentation slide with a blue left panel labeled "Microservices" and a mostly white right area showing a circular arrangement of small blue dots. A blue footer bar with a name is visible along the bottom.](https://mintcdn.com/kodekloud-c4ac6d9a/_-d-o-x8WFPLT_DD/images/OpenShift-4/Storage-Templates-and-Catalog/Example-Voting-Application/microservices-slide-blue-panel-dotted-circle.jpg?fit=max\&auto=format\&n=_-d-o-x8WFPLT_DD\&q=85\&s=504717ce9be5a911913678cf5fb44d1e)

Let's get started.

We'll use the well-known sample application commonly demonstrated with [Docker Training Course for the Absolute Beginner](https://learn.kodekloud.com/user/courses/docker-training-course-for-the-absolute-beginner). It's a compact but complete example that illustrates how an application stack can be built from multiple cooperating services and deployed in containers.

At a high level, this voting application provides two user-facing interfaces: one to cast votes and another to view aggregated results. Behind those interfaces, several services communicate to record, process, and persist votes. Below is a concise summary of the components and their roles.

This lesson walks through the components of the sample voting app and the data flow between them, then shows how to deploy the complete stack on a container platform.

| Component | Role / Responsibility | Typical Implementation | Purpose |
| - | - | - | - |
| Voting web app | Front-end UI for casting votes | [Python](https://www.python.org/) web app | Presents choices (e.g., cat vs dog) and submits votes to a fast queue/store |
| Redis | Fast in-memory datastore / queue | [Redis](https://redis.io/) | Holds incoming votes temporarily so the UI remains responsive |
| Worker (background processor) | Asynchronous processing | Any background worker process | Consumes votes from Redis and updates the persistent store |
| PostgreSQL | Persistent relational datastore | [PostgreSQL](https://www.postgresql.org/) | Stores aggregated vote counts and serves read requests |
| Results web app | Front-end UI for aggregated results | [Node.js](https://nodejs.org/) app | Reads aggregated counts from PostgreSQL and displays totals |

How the data flows (user action → stored result):

1. A user selects a choice (for example, "cat") in the voting UI.
2. The Voting web app writes the vote to Redis, acting as a fast, ephemeral queue.
3. The Worker process reads new votes from Redis asynchronously.
4. The Worker increments the appropriate counter in PostgreSQL.
5. The Results web app queries PostgreSQL and displays the updated totals to users.

This separation of responsibilities illustrates core microservices principles:

* Separation of concerns: UI, background processing, and persistence are decoupled.
* Asynchronous processing: Redis + worker reduces latency for the user-facing UI and improves throughput.
* Polyglot architecture: different services can use different runtimes (Python, Node.js, etc.), chosen for the task.

Why use this pattern?

* Faster UI response time because write-heavy operations are handled asynchronously.
* Greater resilience: if PostgreSQL is slow or temporarily unavailable, Redis queues continue to accept votes.
* Better scalability: workers can be scaled independently from front-end services.

What you’ll see in this lesson

* A walkthrough of each component in the sample voting app.
* How the components talk to each other (Redis → Worker → PostgreSQL).
* Steps to containerize and deploy the full stack to a container platform.

Further reading and references

* [Docker Training Course for the Absolute Beginner](https://learn.kodekloud.com/user/courses/docker-training-course-for-the-absolute-beginner)
* [Redis documentation](https://redis.io/)
* [PostgreSQL documentation](https://www.postgresql.org/)
* [Python official site](https://www.python.org/)
* [Node.js official site](https://nodejs.org/)
* [Kubernetes Basics](https://kubernetes.io/docs/concepts/overview/what-is-kubernetes/)

