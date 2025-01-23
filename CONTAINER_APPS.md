### Setup infrastructure on Azure

# Create a resource group
az group create --name layoutlm-group --location eastus

# Create Azure Container Registry (ACR)
az acr create --resource-group layoutlm-group --name layoutlmregistry --sku Standard --admin-enabled true

# Get ACR credentials (save these for later)
az acr credential show --name layoutlmregistry
(save it for latter use)

# Register a container apps service to use the required machine
az provider register --namespace Microsoft.App

# Get the log analytics workspace id and key
go to portal and search 'Log Analytics Workspaces' select the one you are using or create one with the same resource group.
in workspace on left hand side select 'Agents', in 'Log Analytics agent instructions' tab there will be an id and two keys you can use either one

# Enable the Environment for Container Apps Create a Container Apps Environment that provides the networking and management for your app
az containerapp env create --name layoutlm-env --resource-group layoutlm-group --location eastus --logs-workspace-id <LOG_ANALYTICS_WORKSPACE_ID> --logs-workspace-key <LOG_ANALYTICS_WORKSPACE_KEY>

# Login to ACR
az acr login --name layoutlmregistry

# Get ACR credentials
az acr credential show --name layoutlmregistry --query "{username:username, password:passwords[0].value}" --output table


### Build and deploy model to Kubernetes (from local)

docker build --platform linux/amd64 -t layoutlmregistry.azurecr.io/layoutlm:<VERSION> .
docker push layoutlmregistry.azurecr.io/layoutlm:<VERSION>


### Create the Container App Use the pushed image to create your Container App:
az containerapp create --name layoutlm-app --resource-group layoutlm-group --environment layoutlm-env --image layoutlmregistry.azurecr.io/layoutlm:<VERSION> --target-port 8000 --ingress external --cpu 4 --memory 8Gi --registry-server layoutlmregistry.azurecr.io --registry-username <ACR_USERNAME> --registry-password <ACR_PASSWORD>



### Post deployment steps / Sanity

# Check Container App Status
az containerapp show --name layoutlm-app --resource-group layoutlm-group --query properties

# Fetch Logs
az containerapp logs show --name layoutlm-app --resource-group layoutlm-group

# Test the Service Retrieve the external URL for the app:
az containerapp show --name layoutlm-app --resource-group layoutlm-group --query properties.configuration.ingress.fqdn --output tsv
