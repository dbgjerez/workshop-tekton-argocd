
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

## Tekton

As we're using ArgoCD, we only have to apply the ArgoCD bootstrap application and it's going to install the Tekton Operator. This operation was done in the previous step. 

# Pipelines

