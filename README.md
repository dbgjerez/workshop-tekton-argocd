Tekton is a Cloud Native solution to build Continous Integration pipelines. 

The power of Tekton can be combined with ArgoCD as a Continous Deployment tool, getting a CI/CD complete cycle. 

# Tools
* OpenShift Pipelines 1.10.3 
* OpenShift GitOps 1.8.3
* Openshift 4.13

# Installation

## Prerequisites

To deploy this workshop, you need the following operators installed on your OpenShift cluster. You can easily install them from the **OperatorHub**:

1.  **OpenShift GitOps** (based on ArgoCD).
2.  **OpenShift Pipelines** (based on Tekton).

## Initial Setup with GitOps

Once the operators are installed and running, we will use a GitOps approach to configure the rest of the environment.

**1. Get ArgoCD Credentials**

First, you need the route and the admin password to access the ArgoCD UI.

  * Get the ArgoCD route:

    ```bash
    oc get route -n openshift-gitops openshift-gitops-server -o jsonpath='{.spec.host}'
    ```

  * Get the admin password:

    ```bash
    oc -n openshift-gitops get secret openshift-gitops-cluster -o jsonpath='{.data.admin\.password}' | base64 -d
    ```

**2. Grant Permissions to ArgoCD**

ArgoCD needs elevated privileges to manage cluster resources (like `Roles`, `Pipelines`, etc.). To keep the demo simple, we'll apply a `ClusterRole`.

```bash
oc apply -f gitops/cluster-role.yaml
```

**3. Deploy the Bootstrap Application**

Finally, we apply the "root" or bootstrap application. This application tells ArgoCD to read the configuration from this repository and automatically deploy all other applications, dependencies, and pipelines.

```bash
oc apply -f gitops/argocd-project.yaml
oc apply -f gitops/argocd-app-bootstrap.yaml
```

After applying these manifests, you can navigate to the ArgoCD UI and watch as all the workshop components are synchronized.

## Repository Structure

This repository is structured by functionality to maintain a clear layout:

  * **apps**: Contains the configuration of the applications running on the cluster.
  * **dependencies**: System dependencies that will be installed by ArgoCD.
  * **gitops**: Defines ArgoCD objects and cluster configuration (e.g., roles).
  * **pipelines**: Definitions of the Tekton `Pipelines`.
  * **tasks**: Custom Tekton `Tasks` definitions.
  * **triggers**: `Triggers` to automatically execute the pipelines.
  * **workspaces**: PVC creation for pipeline `Workspaces`.

## Tekton

As we're using ArgoCD, we only have to apply the ArgoCD bootstrap application and it's going to install the Tekton Operator. This operation was done in the previous step. 

# Cloud Native Lifecycle

## Continous Integration with Tekton (CI)

### Tasks

A task is a series of ```steps``` that make one or some actions like clone, build, test or something necessary. A task receives input parameters and produces outputs that can be used by others tasks.

By default, Tekton provides a series of tasks for common operations but we can build new tasks. In addition, a task is deployed in OpenShift as a Cloud Native object that can be reused. 

A task can be launched with specific parameters, this instance is called ```TaskRun```. In our case, we only are going to launch its with Pipelines.

For this example, we're going to use these tasks:

* **git-clone:** to clone the repository
* **maven:** to test and build the artifact
* **buildah:** to build the image

### Pipelines

Define a series of ```tasks``` to build a specific application or functional unit. A pipeline can be launched manually using ```PipelineRun``` or using an event.

In our case, we're using the previous task definitions to define the following steps in our pipeline:

![app pipeline](images/quarkus-pipeline.png)

### Triggers

Using triggers, we can manage easily the pipeline execution automatically. For example, in this workshop we configure a webhook in our git repository that initialize the pipeline, to do that, we use the following objects:

* **Route**: the route defines the endpoint where the webhook has to call. 
* **EventListener**: this object defines how it must actuate when a webhook is received. It's essential to complete ```bindings``` and ```template```sections. 
* **TriggerBinding**: indicate the values that substitute the different pipeline variables, for example, you can use values received in the webhook.
* **TriggerTemplate**: the definition of the ```PipilineRun``` which is the object that represents a pipeline execution.

## Continous Deployment with ArgoCD (CD)

Once, our application image is in the containers' registry, we can deploy it. 

ArgoCD ensures that the definition of our deployment is exactly what we have in a Git repository. 

In this example, our pipeline finishes changing the image version in a git repository that ArgoCD is watching. When ArgoCD detects the change, it updates the OpenShift deployment. 

# Run the demo

If you've completed the installation section, you can start this step. 

At this moment, we've deployed all our dependencies, security and ArgoCD applications but we need our code. I've implemented two repositories, one for a Quarkus application and another with a Helm chart that contains the deployment definition. 

* [Quarkus application](https://github.com/dbgjerez/workshop-tekton-argocd-app-quarkus)
* [Helm chart](https://github.com/dbgjerez/workshop-tekton-argocd-app-quarkus-config)

Now, it's the moment to play with them.

## Gogs

Although our repositories are in GitHub we fork them in an internal Git server to avoid a lot of unnecessary commits. 

This internal server is Gogs. You can enter in ArgoCD and visualize the state of Gogs, if it is correctly synchronized we can enter with user and password ```gogs```.

```bash
oc get route -n gogs gogs -o template --template='{{"http://"}}{{.spec.host}}'
```

We can see that we don't have repositories, so we'll initialize it with a specific pipeline. To run it: 

```bash
oc apply -f gitops/init-pipeline-run.yaml
```

We can enter in OpenShift console and see the pipeline execution in the `Pipeline```` tab:

![app pipeline](images/init-pipeline.png)

After some seconds, we can see the projects in gogs.

![Gogs repositories](images/gogs-apps.png)

## Change the application code

At this point, we can clone and modify some code in the application. When we push the changes to the repository our application pipeline will start. We can see it in the same place as the gogs configuration:

![app pipeline](images/quarkus-pipeline.png)

## Check it in ArgoCD

In ArgoCD we have three applications, one per environment (dev, pre and prod), so we can inspect each and test directly the application.

![app pipeline](images/quarkus-apps.png)