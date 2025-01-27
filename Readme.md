### Setup infrastructure on Azure

# Create a resource group
az group create --name layoutlm-group --location eastus

# Create Azure Container Registry (ACR)
az acr create --resource-group layoutlm-group --name layoutlmregistry --sku Standard --admin-enabled true

# Get ACR credentials (save these for later)
az acr credential show --name layoutlmregistry
(save it for latter use)

# Register a container service to use the required machine
az provider register --namespace Microsoft.ContainerService

# Create AKS cluster with powerful CPU nodes
az aks create --resource-group layoutlm-group --name layoutlm-cluster --node-count 1 --node-vm-size Standard_D4s_v3 --generate-ssh-keys --attach-acr layoutlmregistry

# Get ACR credentials
az acr credential show --name layoutlmregistry --query "{username:username, password:passwords[0].value}" --output table


# Put ACR credentails as kubernetes secret, so your kubernetes resources can use these secrets to pull image from ACR
kubectl create secret docker-registry acr-auth-2 --docker-server=layoutlmregistry.azurecr.io --docker-username=<USERNAME> --docker-password=<PASSWORD> --docker-email=<EMAIL>

---
Standard_D4s_v3 - 4 CPU and 16G Memory
---
2 c
8 m

### Machine setup (one time steps)

# Az cli login (if not done already)
az login --tenant <TENENT_ID>

# Get credentials for kubectl
az aks get-credentials --resource-group layoutlm-group --name layoutlm-cluster

# Login to ACR
az acr login --name layoutlmregistry



### Build and deploy model to Kubernetes (from local)

1. Update the version in `deployment.yaml` file at `image: layoutlmregistry.azurecr.io/layoutlm:<VERSION>`
2. Update the same version in below commands

```
docker build --platform linux/amd64 -t layoutlmregistry.azurecr.io/layoutlm:<VERSION> .
docker push layoutlmregistry.azurecr.io/layoutlm:<VERSION>
kubectl apply -f deployment.yaml

```


### Post deployment steps / Sanity

# See the runnig pods and their status
kubectl get pods

# Check kubernetes Service/model logs
kubectl logs svc/layoutlm-service

# To get service IP, run following command and use `ExternalIP`
kubectl get service layoutlm-service