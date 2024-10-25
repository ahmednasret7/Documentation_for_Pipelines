### Create Managed Identity
        
```yaml
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

```
        