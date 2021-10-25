![kustomize plus Codefresh](kustomize-and-codefresh.png)
# Applied GitOps with ArgoCD and Kustomize

This is a sample web application including a kustomization.yaml file, using Kustomize. We'll explain 2 ways to deploy this application using both Kustomize and ArgoCD.

## Prerequisites
- Access to a Kubernetes cluster
- Install [Kustomize](https://kubectl.docs.kubernetes.io/installation/kustomize/)
- Install and configure [Argo CD's CLI and server component](https://argo-cd.readthedocs.io/en/stable/). Please refer to Argo's [documentation](https://argoproj.github.io/argo-cd/getting_started/) to get started.

## Deploy with Kustomize

We will install and deploy this application using Kustomize. This kustomization.yaml file already exists within this application, so we don't need to create or add this YAML file. Start by cloning the repository to your local environment.

`git clone https://github.com/codefresh-contrib/kustomize-gitops-example`

After you create a cluster and have access to it, this application's structure includes:
