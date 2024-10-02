# Azure Compute Services Workshop

## Pre-requisites

1. Azure access
1. Azure CLI
1. `git-bash` or equivalent bash shell

## Setup

1. Login go Azure and create resource group using:

```bash
RG=az-workshop-1
REGION=eastus2
az login
az group create --name $RG --location $REGION
```

## Azure App Service Plan demo

1. Create App Service Plan:

    ```bash
    APP_SERVICE_PLAN=asp-workshop-1
    az appservice plan create --name $APP_SERVICE_PLAN \
    --sku f1 --resource-group $RG --location $REGION --is-linux
    ```

1. Create Tomcat Web App:

    ```bash
    WEBAPP_NAME=tomcat-workshop-1
    az webapp create --name $WEBAPP_NAME --runtime "TOMCAT:10.0-java17" \
    --plan $APP_SERVICE_PLAN --resource-group $RG
    ```

1. Deploy app:

    ```bash
    az webapp deploy --name $WEBAPP_NAME --resource-group $RG --type war \
    --src-url https://tomcat.apache.org/tomcat-10.0-doc/appdev/sample/sample.war
    ```

## Azure Kubernetes Service (AKS) Automatic

### Cluster creation

Instructions are based from: [https://learn.microsoft.com/en-us/azure/aks/learn/quick-kubernetes-automatic-deploy]()

1. Install AKS preview extension:

    ```bash
    az extension add --name aks-preview
    ```

1. Register the following required features:

    ```bash
    az feature register --namespace Microsoft.ContainerService --name EnableAPIServerVnetIntegrationPreview
    az feature register --namespace Microsoft.ContainerService --name NRGLockdownPreview
    az feature register --namespace Microsoft.ContainerService --name SafeguardsPreview
    az feature register --namespace Microsoft.ContainerService --name NodeAutoProvisioningPreview
    az feature register --namespace Microsoft.ContainerService --name DisableSSHPreview
    az feature register --namespace Microsoft.ContainerService --name AutomaticSKUPreview
    ```

1. Create Automatic AKS cluster (instructions from: [https://learn.microsoft.com/en-us/azure/aks/learn/quick-kubernetes-automatic-deploy?pivots=azure-cli]()):

    ```bash
    ALT_REGION=westus2
    CLUSTER=aks-automatic-demo-1
    az aks create --resource-group $RG --name $CLUSTER \
    --location $ALT_REGION  --sku automatic --disable-local-accounts 
    ```

### RBAC Role assignment

NOTE: Depending on subscription restrictions, the last command may not work,
thus it recommended to run these commands using Azure Cloud Shell.

1. Get your user id:
    ```bash
    USERID=$(az ad signed-in-user show --query "id" -o tsv | tr -d '\r')
    ```

1. Get cluster ID:
    ```bash
    AKS_ID=$(az aks show --resource-group $RG --name $CLUSTER --query id --output tsv)

1. Grand RBAC role:
    ```bash
    az role assignment create --role "Azure Kubernetes Service RBAC Cluster Admin" --assignee $USERID --scope $AKS_ID
    ```

### Log into AKS cluster

1. Log into the AKS cluster:

    ```bash
    az aks get-credentials --resource-group $RG --name $CLUSTER
    ```

1. Next use `kubelogin` to convert `kubeconfig` file to use `azurecli`

    ```bash
    kubelogin convert-kubeconfig -l azurecli
    ```

### Deploy AKS application

1. Create namespace:

    ```bash
    kubectl create ns aks-store-demo
    ```

1. Deploy the application:

    ```bash
    kubectl apply -n aks-store-demo -f https://raw.githubusercontent.com/Azure-Samples/aks-store-demo/main/aks-store-ingress-quickstart.yaml
    ```

### Validate application

1. Run this command until ADDRESS is available:

    ```bash
    kubectl get ingress store-front -n aks-store-demo --watch
    ```

1. Naviate to the IP ADDRESS from command below in a browser