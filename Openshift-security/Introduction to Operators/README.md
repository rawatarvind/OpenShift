# Introduction to Operators

> Explains Kubernetes Operators, showing how CRDs add API types and controllers reconcile desired state, with KubeBuilder examples and guidance on using OperatorHub to install existing operators.

Earlier we introduced Operators. This lesson revisits that idea, clarifies key concepts, and shows how CRDs and controllers work together to extend the Kubernetes API.

At a high level, Kubernetes is an API server that exposes programmatic control over your infrastructure. Whether you apply a YAML manifest with `kubectl` or call a client SDK, you're interacting with the Kubernetes API. Because the API is extensible, you can add new resource types and behavior to that API surface.

Built-in API groups expose core resources such as:

* `apps/v1` (e.g., `Deployment`)
* `networking.k8s.io` (e.g., `Ingress`)

Engineers extend the API in two primary ways:

* Write a controller (reconciler) that implements the desired lifecycle and automation for resources (it can manage built-in or custom resources).
* Define a Custom Resource Definition (CRD) that adds a new API resource type.

These approaches are commonly combined: Operators typically include a CRD (to declare a new API type) and a controller (to reconcile and manage instances of that CRD).

![A minimalist diagram with two dashed rounded squares side by side: the left reads "You Create a Full-Blown Controller" and the right reads "Use a Custom Resource Definition." A red diagonal line cuts between the two boxes.](https://mintcdn.com/kodekloud-c4ac6d9a/_-d-o-x8WFPLT_DD/images/OpenShift-4/OpenShift-Security/Introduction-to-Operators/controller-vs-crd-diagram.jpg?fit=max\&auto=format\&n=_-d-o-x8WFPLT_DD\&q=85\&s=17ef09f45ff67fb4fc22d2d8196a098c)

## Custom Resource Definitions (CRDs)

A CRD is defined using the `apiextensions.k8s.io/v1` API. Conceptually, a CRD is "an API for creating APIs": it tells the API server about a new resource type you want exposed under `/apis/<group>/<version>`.

Below is a minimal CRD example for a `crontabs.stable.example.com` resource:

```yaml theme={null}
apiVersion: apiextensions.k8s.io/v1
kind: CustomResourceDefinition
metadata:
  # name must match the spec fields below, and be in the form: <plural>.<group>
  name: crontabs.stable.example.com
spec:
  # group name to use for REST API: `/apis/<group>/<version>`
  group: stable.example.com
  # list of versions supported by this CustomResourceDefinition
  versions:
    - name: v1
      # Each version can be enabled/disabled by served flag.
      served: true
      # One and only one version must be marked as the storage version.
      storage: true
      schema:
        openAPIV3Schema:
          type: object
          properties:
            spec:
              type: object
              properties:
                cronSpec:
                  type: string
                image:
                  type: string
                replicas:
                  type: integer
```

Key CRD concepts:

* `group` and `versions` determine the API endpoint (e.g., `/apis/stable.example.com/v1`).
* `openAPIV3Schema` is used for server-side validation and `kubectl explain` output.
* A CRD alone only defines the resource shape and API surface — behavior is implemented by a controller.

## Controllers (Reconcilers)

Controllers are programs that watch resources and take actions to reconcile actual cluster state toward a desired state declared in an object (the spec). A reconciler typically:

* Watches relevant object kinds (built-in or custom).
* Reads the object's desired state (`spec`) and current state in the cluster.
* Creates, updates, or deletes Kubernetes objects to reach the desired state.
* Updates the object's `status` subresource to reflect observed state.

KubeBuilder is a popular framework that scaffolds both CRDs and controllers in Go. Typical KubeBuilder project layout:

* `api/v1` — API type definitions
* `controllers` — controller/reconciler logic

Example Go API types (simplified) for a fictional `MikesAPI` custom resource:

```go theme={null}
package v1

import (
	metav1 "k8s.io/apimachinery/pkg/apis/meta/v1"
)

// MikesAPISpec defines the desired state of MikesAPI
type MikesAPISpec struct {
	// INSERT ADDITIONAL SPEC FIELDS - desired state of cluster
	// Important: Run "make" to regenerate code after modifying this file

	MikesPhoneNumber string `json:"mikesPhoneNumber,omitempty"`
	MikesAge         string `json:"mikesAge,omitempty"`
}

// MikesAPIStatus defines the observed state of MikesAPI
type MikesAPIStatus struct {
	// INSERT ADDITIONAL STATUS FIELD - define observed state of cluster
	// Important: Run "make" to regenerate code after modifying this file
}

//+kubebuilder:object:root=true
//+kubebuilder:subresource:status

// MikesAPI is the Schema for the mikesapis API
type MikesAPI struct {
	metav1.TypeMeta   `json:",inline"`
	metav1.ObjectMeta `json:"metadata,omitempty"`

	Spec   MikesAPISpec   `json:"spec,omitempty"`
	Status MikesAPIStatus `json:"status,omitempty"`
}
```

A typical controller skeleton generated by KubeBuilder uses `controller-runtime` to implement the reconciler:

```go theme={null}
package controllers

import (
	"context"

	"k8s.io/apimachinery/pkg/runtime"
	ctrl "sigs.k8s.io/controller-runtime"
	"sigs.k8s.io/controller-runtime/pkg/client"
	"sigs.k8s.io/controller-runtime/pkg/log"

	simplyanengineercomv1 "simplyanengineer.com/MikesAPI/api/v1"
)

// APIReconciler reconciles a MikesAPI object
type APIReconciler struct {
	client.Client
	Scheme *runtime.Scheme
}

//+kubebuilder:rbac:groups=simplyanengineer.com,resources=mikesapis,verbs=get;list;watch;create;update;patch;delete
//+kubebuilder:rbac:groups=simplyanengineer.com,resources=mikesapis/status,verbs=get;update;patch
```

Note: the RBAC markers above are parsed by code generators to create ClusterRole/Role rules for the controller.

CRDs and controllers are an advanced topic. If you only take away the high-level idea — "CRDs add API types; controllers implement behavior" — you will already have a useful mental model. It's OK to revisit this topic multiple times while learning.

## When to install an existing Operator

You don't have to write your own CRD/controller if a maintained Operator already exists. OpenShift (and Kubernetes distributions) provide Operator registries such as OperatorHub where many Operators are packaged and ready to install for monitoring, backup, security, cloud integrations, and more.

Installing an Operator via the OpenShift web console is usually simpler than building one from scratch. For example, to install the Datadog Operator:

* Open OperatorHub in the OpenShift console
* Select the Datadog Operator
* Choose an installed namespace (for example, `openshift-operators`)
* Configure update approval and click Install

![Screenshot of the Red Hat OpenShift web console OperatorHub showing the Datadog operator installation page with "openshift-operators" selected as the installed namespace and Install/Cancel buttons. A right column lists Datadog components (Agent, Metric, Monitor) and a red banner warns the cluster is for development/testing only.](https://mintcdn.com/kodekloud-c4ac6d9a/_-d-o-x8WFPLT_DD/images/OpenShift-4/OpenShift-Security/Introduction-to-Operators/openshift-operators-datadog-install-dev-banner.jpg?fit=max\&auto=format\&n=_-d-o-x8WFPLT_DD\&q=85\&s=86201d786dd2b2d3cc4d337ed67ad197)

OperatorHub makes it easy to discover Operators for common vendors and platforms:

![A screenshot of the Red Hat OpenShift web console showing the OperatorHub page with a search for "azu" and operator cards like Azure Service Operator and Dynatrace Operator. The left sidebar shows the Administrator menu with OperatorHub selected.](https://mintcdn.com/kodekloud-c4ac6d9a/_-d-o-x8WFPLT_DD/images/OpenShift-4/OpenShift-Security/Introduction-to-Operators/openshift-operatorhub-azu-search.jpg?fit=max\&auto=format\&n=_-d-o-x8WFPLT_DD\&q=85\&s=bc130cff7d4a4aa2f1a74603880c7d98)

## Quick comparison

| Concept | Purpose | Typical tooling / examples |
| - | - | - |
| Custom Resource Definition (CRD) | Adds a new API type to Kubernetes (defines schema & validation) | `apiextensions.k8s.io/v1`, `openAPIV3Schema` |
| Controller / Reconciler | Implements behavior to reconcile desired vs actual state | Controller-runtime, KubeBuilder, Go |
| Operator | Packaging of CRDs + controllers + (often) webhooks | Operator SDK, KubeBuilder, OperatorHub |

## Summary

* Kubernetes is an extensible API server; Operators extend that API with new types and behavior.
* CRDs define new resource types and their schema; controllers implement the automation to manage those resources.
* KubeBuilder is a common framework to scaffold CRDs and controllers in Go.
* When a maintained Operator exists (for example in OperatorHub), prefer installing it rather than writing your own, unless you need custom behavior.

## Links and references

* [Kubernetes Custom Resource Definitions (CRD) documentation](https://kubernetes.io/docs/tasks/extend-kubernetes/custom-resources/custom-resource-definitions/)
* [KubeBuilder documentation](https://book.kubebuilder.io/)
* [Controller Runtime (sigs.k8s.io/controller-runtime)](https://pkg.go.dev/sigs.k8s.io/controller-runtime)
* [OperatorHub](https://operatorhub.io/)
