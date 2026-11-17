# APPENDIX C Pre Requisite SCM

> A concise primer on source code management, Git workflows, branching, merging, and integrating repositories with CI/CD and OpenShift builds to enable collaborative development and automated pipelines

Hello, and welcome to this lesson on source code management (SCM) and builds.

This section gives a concise, practical introduction to SCM, builds, and pipelines — foundational concepts you should understand before applying them in OpenShift. If you already know version control and branching workflows, feel free to skip ahead; later lessons apply these ideas specifically to OpenShift builds and CI/CD pipelines.

This primer focuses on core SCM concepts (repositories, branching, merges, and collaboration) and common Git workflows. These are the concepts you’ll rely on when configuring builds and pipelines in OpenShift.

Why SCM matters

* Real-world applications are typically composed of multiple components (front-end, back-end APIs, database integrations) that different developers work on simultaneously.
* Without version control, parallel development, environment promotion (dev → staging → prod), and historical tracking become error-prone and chaotic.
* SCM provides a single source of truth, access controls, a complete history of changes, and tooling for safe collaboration.

A single application may contain multiple pieces — for example:

* A web console (HTML/CSS, JavaScript)
* A back-end API (Python, Java, Node.js)
* Database connectors and integration scripts

Because different teams or developers often edit the same repository, SCM systems let you coordinate changes, review diffs, and resolve conflicts in a controlled way.

![A slide titled "Source Code Management" showing icons for an Application, Web Console, and API Backend at the top. Below are developer icons with arrows pointing up to indicate contributions to those components.](https://mintcdn.com/kodekloud-c4ac6d9a/Xa5GerWBw0vsOVlC/images/OpenShift-4/Conclusion-and-APPENDIX/APPENDIX-C-Pre-Requisite-SCM/source-code-management-developer-contributions.jpg?fit=max\&auto=format\&n=Xa5GerWBw0vsOVlC\&q=85\&s=8115e66119c56e672de7ef4a373eebe5)

What an SCM (version control) provides

* Central repository hosting (on-prem or cloud)
* Revision history and commit metadata (who changed what and when)
* Branching strategies for feature development, releases, and hotfixes
* Merge and conflict resolution workflows
* Access control and auditability
* Basic analytics (commits over time, contributor stats)

Example repository layout

```text theme={null}
|-- README.md
|-- Dockerfile
|-- bower.json
|-- package.json
|-- app
   |-- app.js
   |-- web
       |-- index.html
       |-- mddir.js
       |-- routing.js
       |-- server.js
   |-- api
       |-- api.groups.js
       |-- api.posts.js
       |-- api.users.js
       |-- api.widgets.js
       |-- authentication
           |-- oauth.js
           |-- vendor
   |-- database
       |-- db.js
   |-- integration
       |-- servicenow.js
```

Hosting platforms

* Common hosted Git platforms: [GitHub](https://github.com), [GitLab](https://gitlab.com), and [Bitbucket](https://bitbucket.org).
* These platforms add code review tools (merge/pull requests), CI/CD integrations, and role-based access control that are useful when building pipelines and automating OpenShift builds.

How merging works (brief)

* SCM tools compute line-based diffs and use three-way merges that combine non-conflicting edits automatically.
* If two edits conflict on the same lines, the system marks a merge conflict and requires manual resolution.
* The repository maintains a complete history, so you can revert or inspect past changes.

Example diff/merge view showing added and modified lines:

```diff theme={null}
1  client/app/main/main.html
@@ -18,6 +18,7 @@
 <div class="container">
     
     <div class="row" style="margin-top: 10px;">
+    <div class="col-md-3"></div>
     
     <div class="col-md-2" style="text-align: center" ng-repeat="video in $ctrl.videos">
       <a href="" ng-click="$ctrl.showVideoModal(video)" style="color: grey">
         <div style="color: #d9534f">

2  server/config/environment/shared.js
@@ -37,7 +37,7 @@ exports = module.exports = {
 {
   title: 'Overview',
   type: 'overview',
-  video_id: '6sE0Gqcw_4U'
+  video_id: 'e1o8vR5G6n4'
 },
 {
   title: 'Getting Started',
```

![A presentation slide titled "Source Code Management" showing red icon graphics labeled "Revision History," "File Merging," "Host," and "Access Control" alongside a sidebar screenshot of commit history entries.](https://mintcdn.com/kodekloud-c4ac6d9a/Xa5GerWBw0vsOVlC/images/OpenShift-4/Conclusion-and-APPENDIX/APPENDIX-C-Pre-Requisite-SCM/source-code-management-commit-history.jpg?fit=max\&auto=format\&n=Xa5GerWBw0vsOVlC\&q=85\&s=7eee6ea98466d9c7f303f2f6d13d05f4)

Common branching strategies

* Feature branches: isolate work for a single feature (`feature/xyz`)
* Main (or trunk): stable branch for production-ready code (`main` or `master`)
* Release branches: prepare a release (`release/v1.2.0`)
* Hotfix branches: urgent fixes for production issues (`hotfix/issue-123`)

Quick reference: common Git commands

| Action | Purpose | Example |
| - | - | - |
| Clone a remote repo | Get a local copy of the repository | `git clone git@your-git-server:group/project.git` |
| Create a feature branch | Start work on a new feature | `git checkout -b feature/new-feature` |
| Stage changes | Prepare files for commit | `git add .` |
| Commit staged changes | Record a snapshot with a message | `git commit -m "Add new feature"` |
| Push branch to remote | Upload branch to remote server | `git push -u origin feature/new-feature` |
| Pull latest changes | Update local branch with remote changes (rebase recommended) | `git pull --rebase origin main` |

Avoid force pushing (`git push --force`) to shared branches. Force pushes can overwrite collaborators’ commits and complicate history. Use force push only on private or feature branches after coordinating with your team.

SCM integrations with CI/CD and OpenShift builds

* Repositories are the trigger point for automated builds and pipeline runs.
* When you push code or merge a pull request, CI/CD systems (GitLab CI/CD, Jenkins, Tekton, GitHub Actions) can run unit tests, build container images, and create deployment artifacts.
* In OpenShift, build configurations and pipeline definitions reference your Git repository to produce deployment-ready images and trigger deployments.

Next step
If you want a hands-on walkthrough, the next section demonstrates setting up a local Git workflow using [GitLab](https://gitlab.com) — a useful exercise before configuring builds and pipelines in OpenShift.
