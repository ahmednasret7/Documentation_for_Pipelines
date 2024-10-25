# To Create a CI/CD pipeline in Azure DevOps for C# Backend App and Push it to a Container Registry.

**Step 1: Connect to the repo in the Azure DevOps pipelines.

1. Go to your Azure DevOps project.
....Select Pipelines from the left-hand menu.

2. Select New pipeline.

3. Connect to your code hosting platform.
....For this case we are using Github and existing repo.


4. Select the repo in which the code resides.


# The first section is the triggers.

trigger:
  branches:
    include:
    - main
  paths:
    include:
    - azure-devops-pipelines.yml
    exclude:
    - Readme.md

# Second Section is the Variables.
**Here are the variables, 
Image Name in my docker hub.
The Container-App Name for the backend app.
The Container-App env needed for the Container-App backend.
The Resource Group Name in which all these components resides.
The Tag variable which changes automatically with every build with a builtin function from azure.


variables:
  IMAGE_NAME: ahmednasr7/azureci-image1
  CONTAINERAPPS_APP: album-backend-api
  CONTAINERAPPS_ENVIRONMENT: aca-env1
  RESOURCE_GROUP: rg-containerapps-workshop1
  TAG: '$(Build.BuildId)'

# Build and Push Docker Image to Docker Hub using Service Connection

stages:
- stage: Build
  displayName: 'Build and push image'
  jobs:
  - job: BuildAndDeploy
    displayName: 'Build and Deploy Container to Azure'
    pool:
      vmImage: ubuntu-latest

    steps:      
    
      - task: Docker@2
        displayName: 'Build and Push Docker Image to Docker Hub'
        inputs:
          containerRegistry: 'ahmednasr7-dockerhub' # Name of your Docker Hub service connection
          repository: '$(IMAGE_NAME)'
          command: 'buildAndPush'
          dockerfile: './backend_api/backend_api_csharp/Dockerfile'
          tags: $(TAG) # Or any other tag you prefer
          buildContext: './backend_api/backend_api_csharp'

# Deploy the container to Azure Container Apps


- stage: Deploy
  displayName: 'Deploy to Azure Container Apps'
  jobs:
  - job: Deploy
    displayName: 'Deploy to Azure Container Apps Job'
    pool:
      vmImage: ubuntu-latest

    steps:

    - task: AzureCLI@2
      displayName: 'Deploy to Azure Container App Task'
      inputs:
        azureSubscription: 'Azure-connection-ahmednasret7' # Use the Azure service connection for deployment
        scriptType: 'bash'
        scriptLocation: 'inlineScript'
        inlineScript: |
          # Create the Container Apps environment if it doesn't exist
          az containerapp env create --name $(CONTAINERAPPS_ENVIRONMENT) --resource-group $(RESOURCE_GROUP)
          
          # Create the container app and deploy the container image
          az containerapp create --name $(CONTAINERAPPS_APP) \
            --resource-group $(RESOURCE_GROUP) \
            --environment $(CONTAINERAPPS_ENVIRONMENT) \
            --image docker.io/$(IMAGE_NAME):$(Build.BuildId) \
            --target-port 3500 \
            --ingress external