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