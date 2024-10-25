# CI/CD Pipeline in Azure DevOps for C# Backend App

## Overview

This pipeline automates the process of building, pushing a Docker image to Docker Hub, and deploying the container to Azure Container Apps.

---

## Step 1: Connect the Repository to Azure DevOps Pipelines

1. Go to your Azure DevOps project.
2. Select **Pipelines** from the left-hand menu.
3. Select **New Pipeline**.
4. Connect to your code hosting platform (e.g., GitHub) and select the repository with your C# code.

---

## Pipeline Configuration

### 1. Trigger Section

Defines the conditions under which the pipeline is triggered.

```yaml
trigger:
  branches:
    include:
      - main
  paths:
    include:
      - azure-devops-pipelines.yml
    exclude:
      - Readme.md
	  
```

---

### 2. Variables Section
Defines pipeline variables, which include Docker image name, app environment, resource group, and a tag that changes with each build.
```yaml
variables:
  IMAGE_NAME: ahmednasr7/azureci-image1              # Docker image name in Docker Hub
  CONTAINERAPPS_APP: album-backend-api               # Container App name for the backend
  CONTAINERAPPS_ENVIRONMENT: aca-env1                # Environment for the Container App backend
  RESOURCE_GROUP: rg-containerapps-workshop1         # Resource group containing all resources
  TAG: '$(Build.BuildId)'                            # Automatically increments with each build
```

---

## Pipeline Stages

### Stage 1: Build and Push Docker Image to Docker Hub using Service Connection
```yaml
`stages:`

- `stage: Build`  
    `displayName: 'Build and push image'`  
    `jobs:`
    - `job: BuildAndDeploy`  
        `displayName: 'Build and Deploy Container to Azure'`  
        `pool:`  
        `vmImage: ubuntu-latest`
        
        `steps:`
        
        - `task: Docker@2`  
            `displayName: 'Build and Push Docker Image to Docker Hub'`  
            `inputs:`  
            `containerRegistry: 'ahmednasr7-dockerhub' # Name of your Docker Hub service connection`  
            `repository: '$(IMAGE_NAME)'`  
            `command: 'buildAndPush'`  
            `dockerfile: './backend_api/backend_api_csharp/Dockerfile'`  
            `tags: $(TAG) # Or any other tag you prefer`  
            `buildContext: './backend_api/backend_api_csharp'`
```

---

### Stage 2: Deploy the container to Azure Container Apps

```yaml
- stage: Deploy
  displayName: 'Deploy to Azure Container Apps'
  jobs:
    - job: Deploy
      displayName: 'Deploy to Azure Container Apps Job'
      pool:
        vmImage: ubuntu-latest
      steps:
        - task: AzureCLI@2
          displayName: 'Deploy to Azure Container App'
          inputs:
            azureSubscription: 'Azure-connection-ahmednasret7'  # Azure service connection
            scriptType: 'bash'
            scriptLocation: 'inlineScript'
            inlineScript: |
              # Create the Container Apps environment if it doesn't exist
              az containerapp env create --name $(CONTAINERAPPS_ENVIRONMENT) --resource-group $(RESOURCE_GROUP)

              # Deploy the container image to Azure Container Apps
              az containerapp create --name $(CONTAINERAPPS_APP) \
                --resource-group $(RESOURCE_GROUP) \
                --environment $(CONTAINERAPPS_ENVIRONMENT) \
                --image docker.io/$(IMAGE_NAME):$(Build.BuildId) \
                --target-port 3500 \
                --ingress external

```           