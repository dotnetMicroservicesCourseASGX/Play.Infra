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
## Installing Emissary-ingress
```powershell
helm repo add datawire https://app.getambassador.io
helm repo update

kubectl apply -f https://app.getambassador.io/yaml/emissary/3.9.1/emissary-crds.yaml
 
kubectl wait --timeout=90s --for=condition=available deployment emissary-apiext -n emissary-system
 
$namespace="emissary" 
helm install emissary-ingress datawire/emissary-ingress --set service.annotations."service\.beta\.kubernetes\.io/azure-dns-label-name"=$appname --namespace $namespace --create-namespace 

kubectl rollout status deployment/emissary-ingress -n $namespace -w

kubectl -n emissary wait --for condition=available --timeout=90s deploy -lapp.kubernetes.io/instance=emissary-ingress

```