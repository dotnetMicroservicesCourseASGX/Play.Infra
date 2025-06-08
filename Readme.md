# Play.Infra
Play Economy Infrastructure components


## Add the GitHub package source
```powershell
$owner="dotnetMicroservicesCourseASGX"
$gh_pat="[PAT HERE]"

dotnet nuget add source --username USERNAME --password $gh_pat --store-password-in-clear-text --name github "https://nuget.pkg.github.com/$owner/index.json"
```

## Creating the Azure resource group
```powershell
$appname="playeconomyaxsg"
az group create --name $appname --location eastus
```
## Create the cosmos Db account
```powershell
az cosmosdb create --name $appname --resource-group $appname --kind MongoDB --enable-free-tier
```

## Create the Service Bus namespace
```powershell
az servicebus namespace create --resource-group $appname --name $appname --sku Standard
```

## Creating the Container Registry
```powershell
az acr create --resource-group $appname --name $appname --sku Basic
```

## Creating the AKS cluster
```powershell
az provider register --namespace Microsoft.ContainerService
az aks create -n $appname -g $appname --node-vm-size Standard_B2s --node-count 2 --attach-acr $appname --enable-oidc-issuer --enable-workload-identity --generate-ssh-keys 
az aks get-credentials -n $appname -g $appname
```

## Creating the Azure Key Vault
```powershell
az provider register --namespace Microsoft.KeyVault
az keyvault create -n $appname -g $appname
```
