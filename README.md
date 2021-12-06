
# Applied GitOps with ArgoCD and Kustomize
![Argo GitOps with Argocd and Kustomize ](pictures/Applied_GitOps_Header.png)
This is a sample web application that includes both a base and overlays folder with both a Staging and Production environment. We'll explain 2 ways to deploy this application using only Kustomize and Kustomize with ArgoCD.

## Prerequisites

- Access to a Kubernetes cluster
- Install [Kustomize](https://kubectl.docs.kubernetes.io/installation/kustomize/)
- Install and configure [Argo CD's CLI and server component](https://argo-cd.readthedocs.io/en/stable/). Please refer to Argo's [documentation](https://argoproj.github.io/argo-cd/getting_started/) to get started.

## Deploy with Kustomize

We will install and deploy this application using only Kustomize. The `kustomization.yaml` file already exists within this application, so we don't need to create or add this YAML file. Start by cloning the repository to your local environment.

`git clone https://github.com/hseligson1/kustomize-gitops-example.git`

This application's structure includes:

```
kustomize-gitops-example
├── app
├── base
│   ├── configMap.yaml
│   ├── deployment.yaml
│   ├── kustomization.yaml
│   └── service.yaml
└── overlays
    ├── production
    │   ├── config-map.yaml
    │   ├── deployment.yaml
    │   └── kustomization.yaml
    └── staging
        ├── config-map.yaml
        └── kustomization.yaml
```

Next, you can configure the cluster with overlays using this command:

`kustomize build kustomize-gitops-example/overlays/staging`

or

`kustomize build kustomize-gitops-example/overlays/production`

This allows you to review the data for both environments. You should see an output similar to this overlay response we executed for staging:

```
apiVersion: v1
data:
  mysqlDB: staging-mysql.example.com:3306
kind: ConfigMap
metadata:
  labels:
    app: demo
    variant: staging
  name: staging-the-map
---
apiVersion: v1
kind: Service
metadata:
  labels:
    app: demo
    variant: staging
  name: staging-demo
spec:
  ports:
  - port: 8080
  selector:
    app: demo
    variant: staging
  type: ClusterIP
---
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: demo
    deployment: demo
    variant: staging
  name: staging-the-deployment
spec:
  replicas: 1
  selector:
    matchLabels:
      app: demo
      variant: staging
  template:
    metadata:
      labels:
        app: demo
        deployment: demo
        variant: staging
    spec:
      containers:
      - env:
        - name: MY_MYSQL_DB
          valueFrom:
            configMapKeyRef:
              key: mysqlDB
              name: staging-the-map
        image: hseligson/kustomize-sample-app:v1.0.1
        imagePullPolicy: Always
        name: the-container
        ports:
        - containerPort: 8080
```
Whenever you make a change within the overlays directories you can apply those changes to the cluster by executing the same command for the deployment:

`kubectl apply -k kustomize-gitops-example/overlays/staging`

or

`kubectl apply -k kustomize-gitops-example/overlays/production`

This returns a response informing you if either environment contains the changes and is deployed. Here's the staging environment example: 
```
configmap/staging-the-map created
service/staging-demo created
deployment.apps/staging-the-deployment created
```
To inspect the deployment and confirm whether or not it's READY, you can execute the command:

`kubectl get deployment staging-the-deployment`

or

`kubectl get deployment production-the-deployment`

Fore more details, you can also execute:

`kubectl describe pods <pod name>`

Congrats, you've deployed an application only using Kustomize and `kubectl`.

***

## Deploy with Kustomize and ArgoCD

Now, that you've deployed your app with Kustomize, let's review how to do the same with ArgoCD. 
We'll explain how to deploy with both the ArgoCD UI and the argocd CLI.

### Deploy with ArgoCD UI

Assuming you've installed and configured ArgoCD already, now you can log into ArgoCD and access the UI.
Next, navigate to the +NEW APP on the left-hand side of the UI. Then add the following below to create the application.

#### General Section:

- Application Name – This is the application name inside ArgoCD. Enter "kustomize-gitops-example"

- Project – This is the project name inside ArgoCD. Since this is a new setup for ArgoCD, a default project is created for us and we’ll select the same.

- Sync Policy – You can choose to auto synchronize the state of application in the Kubernetes with the GitHub repository. Choose "Automatic".

![Argo App General Section](pictures/argocd-create-ui-staging.png)

#### Source Section:

- Repository URL – Provide the url for the GitHub repository containing the application manifests. This is the HTTPS URL for this project.

- Revision – You can choose to provide the specific branch or tag for github repo and sync the same state with Kubernetes details. We’ll choose, "main" or you can keep the default, "HEAD".

- Path – This helps in further segregating application manifests inside the GitHub repository. Select the overlays folder based on environment, "overlays/staging". 

![Argo App Source Section](pictures/argocd-source-ui-staging.png)

#### Destination Section:

- Cluster URL – ArgoCD can be used to connect and deploy application to multiple Kubernetes clusters. Choose the default in-cluster (where Argo CD itself is deployed).

- Namespace – This can be used to select namespace where manifests will be deployed. You can submit one you've already created or you can use the custom namespace option ArgoCD provides.

![Argo App Destination Section](pictures/argocd-ns-ui.png)

#### Kustomize Section 

ArgoCD will read the `kustomization.yaml` file in the path and will prompt you to override with different values. However, we’ll go with the default configuration committed in the github repo.

![Argo App Kustomize Section](pictures/argocd-kustomize-ui-staging.png)

#### Synchronize: 

Afterwards, it will read the parameters and the Kubernetes manifests and auto-sync, since you enabled this function when creating your ArgoCD app. Once the manifests are applied, you can review the application health and resources you deployed.

#### Health Status:

You can now review the application health and the resources deployed. Below is an example of both environments: Staging and Production applications that are deployed and healhy.

![Argo App Deployment](pictures/argocd-deploy-staging-ui.png)

- Note: within this demo we walked you through how to create an ArgoCD application for your Kustomize project for a staging environment. You can repeat this process for other environments like development, production, testing, etc.

![Argo App Deployment](pictures/argocd-cluster-ns-ui.png)

If you need to rollback or view your history for an application due to any errors or issues, you can do so by click on the HISTORY/ROLLBACK button within the UI to view the deployment history. Included is also a revision ID that will link you directly to the Git repo commit that can be identified as the root cause of an error or issue that needs resolved. 

![Argo App History](pictures/argocd-app-history-ui.png)

Congrats, you've deployed an application with Kustomize and applied GitOps with ArgoCD.

***

### Deploy with argocd CLI

Now, that you've successfully deployed a Kustomize project within the ArgoCD UI. Let's do it again with the argocd CLI.
Assuming you're connected to your Kubernetes cluster you need to now log into the argocd CLI.

To login to the CLI, execute the following in your terminal:

`argocd login localhost:8080 --username admin --password <same password used in argocd ui>`

You should be able to login using the same password you used when accessin the ArgoCD UI. 

Next, execute the following:

`argocd app list`

This will return a response that lists your ArgoCD applications. You should see the app we created within the UI, "kustomize-gitops-example-staging".

![ArgoCD CLI Create and List App](pictures/argocd-cli-app-list.png)

If this is the case, then delete the app you previously created wuith the command:

`argocd app delete <argocd application name>`

![ArgoCD CLI Delete App](pictures/argocd-cli-delete-app.png)

Next, deploy the `kustomization.yaml` file within the CLI and reference the Git repository to create the ArgoCD app:

`argocd app create <application name> --repo https://github.com/hseligson1/kustomize-gitops-example.git --revision main --path overlays/staging  --dest-server https://kubernetes.default.svc --dest-namespace staging`

![ArgoCD CLI Create App](pictures/argocd-cli-create-app.png)

This will return a resonse, "application '<application name>' created". Next, sync the deployment managed by ArgoCD:

`argocd app sync <application name>`

This will return a response that allows you to view the application you created and whether or not you were able to synchronize or not.

![ArgoCD CLI Sync App](pictures/argocd-cli-sync-app.png)

Check status of deployment:

`argocd app get <application name>` 

![ArgoCD CLI Get App](pictures/argocd-cli-get-app.png)

### ArgoCD Application History

In case you need to rollback the ArgoCD application, you can do so by rolling back the application to a previously deployed version by the History ID. First, you need access to the application deployment history:

`argocd app history <application name>`

![ArgoCD CLI App History](pictures/argocd-cli-app-history.png)

The response will return the application history, including an ID, date, and branch that any revision was made on the application. You can then use the ID to rollback the application to a specific deployed version:

`argocd app history <application name> <application history id>` 

Congrats, you've created and deployed an Kustomize project with the argocd CLI. Now, each time a new `kustomize.yaml` file is added or modified, ArgoCD will detect those changes and update the deployments for your project. 
