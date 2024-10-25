# CI/CD Pipeline in Azure DevOps for C# Backend and Frontend App with Push to Container Registry

## Step 1: Connect the Repository to Azure DevOps Pipelines

1. Go to your Azure DevOps project.
   - Select **Pipelines** from the left-hand menu.
2. Select **New pipeline**.
3. Connect to your code hosting platform.
   - For this case, we are using GitHub with an existing repo.
4. Select the repo in which the code resides.

---

## Pipeline Configuration

### 1. Trigger Section

Defines the trigger settings for the pipeline.

```yaml
trigger: none 
pr: none
```

---

### 2. Variables Section.

Defines key variables including the Docker image names, resource group, Azure Container Registry (ACR), and managed identity configuration.

```yaml
variables:
  IMAGE_NAME: ahmednasr7/azureui-image1            # Docker Hub image name
  CONTAINERAPPS_APP1: album-api                    # Container App name for the backend
  CONTAINERAPPS_APP2: album-ui                     # Container App name for the frontend
  CONTAINERAPPS_ENVIRONMENT: aca-env1              # Environment for Container Apps
  RESOURCE_GROUP: rg-containerapps-workshop1       # Resource group
  ACR_NAME: acrworkshop0021                        # Azure Container Registry name
  IDENTITY: identity-aca-acr                       # Managed identity for ACR access
  TAG: '$(Build.BuildId)'                          # Build tag for versioning
  LOCATION: 'westeurope'                           # Azure location
  AZURE_CONNECTION: 'azure-cli-2024-10-14-06-24-51'  # Azure CLI service connection
```
---

## Pipeline Stages
### Stage: Deploy
The deploy stage sets up the Container Apps environment, creates an Azure Container Registry (ACR), builds and pushes backend and frontend images, and deploys them to Azure Container Apps.

```yaml
stages:
- stage: Deploy
  displayName: 'Deploy to Azure Container Apps'
  jobs:
    - job: Deploy
      displayName: 'Deploy to Azure Container Apps Job'
      pool:
        vmImage: ubuntu-latest
      steps:
```
---

### 1. Checkout Repository
```yaml
      - checkout: self
        displayName: 'Checkout GitHub Repo'
```
---

### 2. Create the Container Apps environment if it doesn't exist
Creates the environment for deploying the containerized apps.


```yaml
      - task: AzureCLI@2
        displayName: 'Create Container Apps Environment'
        inputs:
          azureSubscription: $(AZURE_CONNECTION)
          scriptType: 'bash'
          scriptLocation: 'inlineScript'
          inlineScript: |
            az containerapp env create --name $(CONTAINERAPPS_ENVIRONMENT) --resource-group $(RESOURCE_GROUP) --location $(LOCATION)

```
---
### 3. Create Azure Container Registry (ACR)
Sets up the ACR to store container images



```yaml
      - task: AzureCLI@2
        displayName: 'Create ACR'
        inputs:
          azureSubscription: $(AZURE_CONNECTION)
          scriptType: 'bash'
          scriptLocation: 'inlineScript'
          inlineScript: |
            az acr create --resource-group $(RESOURCE_GROUP) --name $(ACR_NAME) --sku standard --admin-enabled true

```
---
### 4. Build backend API image using ACR

Builds and pushes the Docker image for the backend API.

 ```yaml
      - task: Docker@2
        displayName: 'Build backend using docker'
        inputs:
          containerRegistry: $(AZURE_CONNECTION)   # Service connection to ACR
          repository: '$(CONTAINERAPPS_APP1)'
          command: 'buildAndPush'
          Dockerfile: '**/backend_api/backend_api_csharp/Dockerfile'
          tags: |
            $(Build.BuildId)

```

---

### 5. Create Managed Identity
Creates a managed identity and assigns it the necessary permissions to access ACR.
```yaml
      - task: AzureCLI@2
        displayName: 'Create Managed Identity'
        inputs:
          azureSubscription: $(AZURE_CONNECTION)
          scriptType: 'bash'
          scriptLocation: 'inlineScript'
          inlineScript: |
            az identity create --resource-group $(RESOURCE_GROUP) --name $(IDENTITY)
            IDENTITY_CLIENT_ID=$(az identity show --resource-group $(RESOURCE_GROUP) --name $(IDENTITY) --query clientId -o tsv)
            ACR_ID=$(az acr show --resource-group $(RESOURCE_GROUP) --name $(ACR_NAME) --query id -o tsv)
            az role assignment create --role AcrPull --assignee $IDENTITY_CLIENT_ID --scope $ACR_ID
            echo "Managed Identity Created with Client ID: $IDENTITY_CLIENT_ID"
```
---
### 6. Deploy Backend API Container App
Deploys the backend API container to the Azure Container App environment.
```yaml
      - task: AzureCLI@2
        displayName: 'Deploy Backend API Container App'
        inputs:
          azureSubscription: $(AZURE_CONNECTION)
          scriptType: 'bash'
          scriptLocation: 'inlineScript'
          inlineScript: |
            IDENTITY_RESOURCE_ID=$(az identity show --resource-group $(RESOURCE_GROUP) --name $(IDENTITY) --query id -o tsv)
            az containerapp create \
              --name $(CONTAINERAPPS_APP1) \
              --resource-group $(RESOURCE_GROUP) \
              --environment $(CONTAINERAPPS_ENVIRONMENT) \
              --image $(ACR_NAME).azurecr.io/$(CONTAINERAPPS_APP1):$(TAG) \
              --target-port 3500 \
              --ingress internal \
              --registry-server $(ACR_NAME).azurecr.io \
              --user-assigned $IDENTITY_RESOURCE_ID \
              --registry-identity $IDENTITY_RESOURCE_ID
```


---

### 7.Build frontend UI image using ACR
Builds and pushes the Docker image for the frontend UI.

```yaml
      - task: Docker@2
        displayName: 'Build Frontend UI Image'
        inputs:
          containerRegistry: $(AZURE_CONNECTION)
          repository: '$(CONTAINERAPPS_APP2)'
          command: 'buildAndPush'
          Dockerfile: '**/frontend_ui/Dockerfile'
          tags: |
            $(Build.BuildId)
```

---
### 8. Deploy Frontend UI Container App
Deploys the frontend UI container to the Azure Container App environment, setting an environment variable for the backend API’s base URL.

```yaml
      - task: AzureCLI@2
        displayName: 'Deploy Frontend UI Container App'
        inputs:
          azureSubscription: $(AZURE_CONNECTION)
          scriptType: 'bash'
          scriptLocation: 'inlineScript'
          inlineScript: |
            API_BASE_URL=$(az containerapp show --resource-group $(RESOURCE_GROUP) --name $(CONTAINERAPPS_APP1) --query properties.configuration.ingress.fqdn -o tsv)
            az containerapp create \
              --name $(CONTAINERAPPS_APP2) \
              --resource-group $(RESOURCE_GROUP) \
              --environment $(CONTAINERAPPS_ENVIRONMENT) \
              --image $(ACR_NAME).azurecr.io/$(CONTAINERAPPS_APP2):$(TAG) \
              --target-port 3000 \
              --env-vars API_BASE_URL=https://$API_BASE_URL \
              --ingress external \
              --registry-server $(ACR_NAME).azurecr.io \
              --user-assigned $IDENTITY_RESOURCE_ID \
              --registry-identity $IDENTITY_RESOURCE_ID
```
* * *

* * *

* * *

* * *

* * *

* * *

* * *

* * *

* * *

* * *