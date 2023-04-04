
Tekton is...

# Tools
* OpenShift Pipelines 1.9 (Tekton Pipelines: v0.41.1)
* OpenShift GitOps 

# Installation

## ArgoCD 

Operator installation:

```bash
oc apply -f operator/openshift-gitops.yaml
```

Retrieve ArgoCD route: 

```bash
oc get route -A | grep openshift-gitops-server | awk '{print $3}'
```

Get the ArgoCD admin password: 

```bash
oc -n openshift-gitops get secret openshift-gitops-cluster -o json | jq -r '.data["admin.password"]' | base64 -d
```

ArgoCD needs some privileges to create specific resources. In this demo, we'll apply cluster-role to ArgoCD to avoid the fine-grain RBAC.

```bash
oc apply -f gitops/cluster-role.yaml
```

Now, we apply the bootstrap application:

```bash
oc apply -f gitops/argocd-app-bootstrap.yaml
```

## Tekton

As we're using ArgoCD, we only have to apply the ArgoCD bootstrap application and it's going to install the Tekton Operator. This operation was done in the previous step. 

# Cloud Native Lifecycle

## Continous Integration with Tekton (CI)

### Tasks

A task is a series of ```steps``` that make one or some actions like clone, build, test or something necessary. A task receives input parameters and produces outputs that can be used by others tasks.

By default, Tekton provides a series of tasks for common operations but we can build new tasks. In addition, a task is deployed in OpenShift as a Cloud Native object that can be reused. 

For this example, we're going to use these tasks:

* **git-clone:** to clone the repository
* **maven:** to test and build the artifact
* **buildah:** to build the image

### Pipelines

Define a series of ```tasks``` to build a specific application or functional unit. A pipeline can be launched manually using ```PipelineRun``` or using an event.

In our case, we're using the previous tasks definition to define the following steps in our pipeline:

// todo include the image

### Triggers

## Continous Deployment with ArgoCD (CD)

