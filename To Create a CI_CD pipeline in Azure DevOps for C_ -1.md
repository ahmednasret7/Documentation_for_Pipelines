# To Create a CI/CD pipeline in Azure DevOps for C# Backend and Frontend App and Push it to a Container Registry.

**Step 1: Connect to the repo in the Azure DevOps pipelines.

1. Go to your Azure DevOps project.
....Select Pipelines from the left-hand menu.

2. Select New pipeline.

3. Connect to your code hosting platform.
....For this case we are using Github and existing repo.


4. Select the repo in which the code resides.


# The first section is the triggers.

trigger: none
pr: none


# Second Section is the Variables.

**Here are the variables, 
Image Name in my docker hub.
The Container-App Name for the backend app.
The Container-App Name for the frontend app.
The Container-App env needed for the Container-App.
The Resource Group Name in which all these components resides.
The Azure Container registry(ACR) in which the data will be saved.
Identity for accessing the acr.
The Tag variable which changes automatically with every build with a builtin function from azure.

variables:
  IMAGE_NAME: ahmednasr7/azureui-image1
  CONTAINERAPPS_APP1: album-api
  CONTAINERAPPS_APP2: album-ui
  CONTAINERAPPS_ENVIRONMENT: aca-env1
  RESOURCE_GROUP: rg-containerapps-workshop1
  ACR_NAME: acrworkshop0021
  IDENTITY: identity-aca-acr
  TAG: '$(Build.BuildId)'
  LOCATION: 'westeurope'
  AZURE_CONNECTION: 'azure-cli-2024-10-14-06-24-51'

stages:
- stage: Deploy
  displayName: 'Deploy to Azure Container Apps'
  jobs:
  - job: Deploy
    displayName: 'Deploy to Azure Container Apps Job'
    pool:
      vmImage: ubuntu-latest

    steps:
    # Checkout repo
    - checkout: self
      displayName: 'Checkout GitHub Repo'

    # Step 1: Create the Container Apps environment if it doesn't exist
    - task: AzureCLI@2
      displayName: 'Create Container Apps Environment'
      inputs:
        azureSubscription: $(AZURE_CONNECTION)
        scriptType: 'bash'
        scriptLocation: 'inlineScript'
        inlineScript: |
          az containerapp env create --name $(CONTAINERAPPS_ENVIRONMENT) --resource-group $(RESOURCE_GROUP) --location $(LOCATION)

    # Step 2: Create Azure Container Registry (ACR)
    - task: AzureCLI@2
      displayName: 'Create ACR'
      inputs:
        azureSubscription: $(AZURE_CONNECTION)
        scriptType: 'bash'
        scriptLocation: 'inlineScript'
        inlineScript: |
          az acr create --resource-group $(RESOURCE_GROUP) --name $(ACR_NAME) --sku standard --admin-enabled true

    # Step 3: Build backend API image using ACR
    - task: Docker@2
      displayName: 'Build backend using docker'
      inputs:
        containerRegistry: $(AZURE_CONNECTION)  
		#Service connection to ACR

        repository: '$(CONTAINERAPPS_APP1)'
        command: 'buildAndPush'
        Dockerfile: '**/backend_api/backend_api_csharp/Dockerfile'
        tags: |
         $(Build.BuildId)

    # Step 4: Create Managed Identity
    - task: AzureCLI@2
      displayName: 'Create Managed Identity'
      inputs:
        azureSubscription: $(AZURE_CONNECTION)
        scriptType: 'bash'
        scriptLocation: 'inlineScript'
        inlineScript: |
          az identity create --resource-group $(RESOURCE_GROUP)
	  --name $(IDENTITY)
          IDENTITY_CLIENT_ID= $(az identity show --resource-group $(RESOURCE_GROUP)
	  --name $(IDENTITY) --query clientId -o tsv)
          ACR_ID= $(az acr show
	  --resource-group $(RESOURCE_GROUP)
--name $(ACR_NAME) --query id -o tsv)
          az role assignment create
	  --role AcrPull --assignee $IDENTITY_CLIENT_ID --scope $ACR_ID
          echo "Managed Identity Created with Client ID: $IDENTITY_CLIENT_ID"

    # Step 5: Deploy Backend API Container App
    - task: AzureCLI@2
      displayName: 'Deploy Backend API Container App'
      inputs:
        azureSubscription: $(AZURE_CONNECTION)
        scriptType: 'bash'
        scriptLocation: 'inlineScript'
        inlineScript: |
          IDENTITY_RESOURCE_ID= $(az identity show --resource-group $(RESOURCE_GROUP) --name $(IDENTITY) --query id -o tsv)
          az containerapp create
	  --name $(CONTAINERAPPS_APP1)
	  --resource-group $(RESOURCE_GROUP)
	  --environment $(CONTAINERAPPS_ENVIRONMENT)
	  --image $(ACR_NAME).azurecr.io/ $(CONTAINERAPPS_APP1): $(TAG) --target-port 3500
	  --ingress internal --registry-server $(ACR_NAME).azurecr.io --user-assigned $IDENTITY_RESOURCE_ID --registry-identity $IDENTITY_RESOURCE_ID

    # Step 6: Build frontend UI image using ACR
    - task: Docker@2
      displayName: 'Build Frontend UI Image'
      inputs:
        containerRegistry: $(AZURE_CONNECTION)
        repository: ' $(CONTAINERAPPS_APP2)'
        command: 'buildAndPush'
        Dockerfile: '**/frontend_ui/Dockerfile'
        tags: |
         $(Build.BuildId)
    # Step 7: Deploy Frontend UI Container App
    - task: AzureCLI@2
      displayName: 'Deploy Frontend UI Container App'
      inputs:
        azureSubscription: $(AZURE_CONNECTION)
        scriptType: 'bash'
        scriptLocation: 'inlineScript'
        inlineScript: |
          API_BASE_URL= $(az containerapp show --resource-group $(RESOURCE_GROUP) --name $(CONTAINERAPPS_APP1) --query properties.configuration.ingress.fqdn -o tsv)
          az containerapp create
	  --name $(CONTAINERAPPS_APP2)
	  --resource-group $(RESOURCE_GROUP)
	  --environment $(CONTAINERAPPS_ENVIRONMENT)
	  --image $(ACR_NAME).azurecr.io/ $(CONTAINERAPPS_APP2): $(TAG)
--target-port 3000 --env-vars API_BASE_URL=https:// $API_BASE_URL
--ingress external
--registry-server $(ACR_NAME).azurecr.io
--user-assigned $IDENTITY_RESOURCE_ID
--registry-identity $IDENTITY_RESOURCE_ID
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