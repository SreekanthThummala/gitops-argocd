# gitops-argocd
Building a Pull-Based DevOps Pipeline with GitHub Actions and Argo CD

# Overview

In the [CI/CD pipeline for container-based workloads](https://learn.microsoft.com/en-us/azure/architecture/example-scenario/apps/devops-with-aks/) document we explore the options of push and pull based deployment methods along with the pros and cons of each. In this section we are going deploy a scenario that explains these two options further. To explore the architecture in more detail, please check out the article above.

### Option \#1 Push-based CI/CD Architecture and Dataflow

![Figure 1 - Option 1 Push based Architecture with GitHub Actions for CI and CD](./media/5ef464b58b9ce8ab4499ed1c2aec882f2.png)

This scenario covers a push-based DevOps pipeline for a web application with a front-end component. This pipeline uses GitHub Actions for build push and deployment. The data flows through the scenario as follows:

1.  The App code is developed.
1.  The App code is committed to the GitHub git repository.
1.  GitHub Actions Builds a container image from the App code and pushes the container image to Azure Container Registry.
1.  A GitHub Actions job deploys (pushes) the App to the AKS cluster via kubectl deployment of the App Kubernetes manifest files.

### Option \#2 Pull-based CI/CD Architecture and Dataflow

![Figure 2 - Option 2 Pull based Architecture with GitHub Actions for CI and Argo CD for CD](./media/72be57feef5bb9b47658cfc16f3d779f3.png)

This scenario covers a pull-based DevOps pipeline for a web application with a front-end component. This pipeline uses GitHub Actions for build and push it uses Argo CD a GitOps operator pull/sync for deployment. The data flows through the scenario as follows:

1.  The App code is developed.
1.  The App code is committed to the GitHub git repository.
1.  GitHub Actions Builds a container image from the App code and pushes the container image to Azure Container Registry.
1.  GitHub Actions Updates a Kubernetes Manifest Deployment file with the current image version based on the version number of the container image in the Azure Container Registry.
1.  The GitOps Operator Argo CD syncs / pulls with the Git repository.
1.  The GitOps Operator Argo CD deploys the app to the AKS cluster.

## Deploy this scenario

Before deploying the push or pull based end to end scenario you need to ensure you have met the prerequisites for this scenario. These prerequisites are listed in this section:

### Prerequisites for these scenarios

1. An existing Azure account. If you don't have an Azure subscription, create a [free account](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) before you begin.
2. A GitHub account. To setup a GitHub account, follow the steps in [Getting started with your GitHub account](https://docs.github.com/en/get-started/onboarding/getting-started-with-your-github-account).
3. A fork of this repo [AKS Baseline Automation](https://github.com/azure/aks-baseline-automation). Note: be sure to uncheck "Copy the main branch only".
4. An AKS cluster. It is highly recommended that you follow one of the two options below to deploy the cluster:
    - **Quick option:** use the [AKS Construction helper](https://azure.github.io/AKS-Construction/) to deploy your Azure Kubernetes Service (AKS) cluster and the supporting services. You can use this pre-configured link: [AKS Construction helper (pre-configured)](https://azure.github.io/AKS-Construction/?ops=managed&cluster.apisecurity=none&addons.ingress=appgw&addons.monitor=aci&addons.azurepolicy=none&addons.networkPolicy=none&addons.csisecret=akvNew&deploy.location=EastUS2&addons.appgwKVIntegration=false) to create an AKS cluster that is Azure AD integrated to use with this CI/CD scenario. This will also create the supporting services such as Azure Container Registry (ACR), Azure Key vault, Application Gateway, and other required Azure resources.
    - **[IaC workflows](../../IaC/README.md) option:** the IaC workflows in this repo will walk you through the process of creating your infrastructure through automation using bicep or terraform. This option will deploy the [Baseline architecture for an Azure Kubernetes Service (AKS) cluster](https://learn.microsoft.com/en-us/azure/architecture/reference-architectures/containers/aks/baseline-aks).

### Next
Pick one of the following options to deploy a workload using automation

:arrow_forward: [Pull option](./app-flask-pull-gitops.md)

## Pull-based CI/CD(GitOps)

This article outlines how to deploy your workload using the pull option as described in the [CI/CD pipeline for container-based workloads](https://learn.microsoft.com/azure/architecture/example-scenario/apps/devops-with-aks) article. To deploy this scenario, follow the prerequisites steps outlined [here](README.md) (if you haven't already), then perform the following steps:

1. Follow steps 1 to 5 in [Option #1 Push-based CI/CD](./app-flask-push-dockerbuild.md) to setup your GitHub environment.
2. Install Argo CD on your AKS cluster by following the steps in [Get Started with Argo CD](https://argo-cd.readthedocs.io/en/stable/getting_started/).
   
3. Run the [.github/workflows/App-Flask-GitOps.yml](../../.github/workflows/App-Flask-GitOps.yml) workflow by clicking on **Actions** and selecting the display name for this workflow, which is **App Deploy Flask - GitOps**. This workflow will build and push the container image to the Azure Container Registry (ACR), then it will update the application *deployment.yaml* file with the names of the pushed image. 
   
    Enter the parameters requested by this workflow as shown below:
       ![](media/b4bf25dc9497c669d54a205648cb864c.png)
4. Create a new app for the App in Argo CD by following [these steps](https://argo-cd.readthedocs.io/en/stable/getting_started/#creating-apps-via-ui). Make sure you enter the following parameters:
   - Application Name: flask
   - Project Name: default
   - Sync Policy: Automatic
   - Source
     - Repository URL: https://github.com/YOURREPO/aks-baseline-automation.git
     - Revision: HEAD
     - Path: workloads/flask
   - Destination:
     - Cluster URL: https://kubernetes.default.svc
     - Namespace: default

    After the app is created, click on the **SYNC** button in the Argo CD portal to deploy it.

    This is an example of the successful App in Argo CD:

![](media/58af037d65b2303dbb1c2d4196ac300f.png)
![](media/66908c97c321303ba2bcd58ba6431bdd.png)

5. Check the deployment from the Azure portal as in [Option #1 Push-based CI/CD](./app-flask-push-dockerbuild.md) to make sure that the application was successfully deployed. Also test that you can access it. 
