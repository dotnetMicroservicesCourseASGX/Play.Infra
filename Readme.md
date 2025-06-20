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
$appname="playeconomyaxsg1"
az group create --name $appname --location eastus
```
## Create the cosmos Db account
```powershell
## que la ubicacion sea la misma que la del grupo de recursos
az provider register --namespace Microsoft.DocumentDB
az cosmosdb create --name $appname --resource-group $appname --kind MongoDB --enable-free-tier
```

## Create the Service Bus namespace
```powershell
az provider register --namespace Microsoft.DocumentDB

az servicebus namespace create --resource-group $appname --name $appname --sku Standard
```

## Creating the Container Registry
```powershell
az provider register --namespace Microsoft.ContainerRegistry
az acr create --resource-group $appname --name $appname --sku Basic
```

## Creating the AKS cluster
```powershell
az provider register --namespace Microsoft.ContainerService
az aks create -n $appname -g $appname --node-vm-size Standard_B2s --node-count 2 --attach-acr $appname --enable-oidc-issuer --enable-workload-identity --generate-ssh-keys 
az aks get-credentials -n $appname -g $appname

az aks scale --resource-group $appname --name $appname --node-count 4
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

## Configuring Emissary-ingress routing
```powershell
kubectl apply -f .\emissary-ingress\listener.yaml -n $namespace
kubectl apply -f .\emissary-ingress\mappings.yaml -n $namespace
```

## Installing cert-manager
```powershell
helm repo add jetstack https://charts.jetstack.io --force-update
helm install cert-manager jetstack/cert-manager --namespace $namespace --version v1.18.0 --set crds.enabled=true

```

## Creating the cluster issuer
```powershell
kubectl apply -f .\cert-manager\cluster-issuer.yaml -n $namespace
kubectl apply -f .\cert-manager\acme-challenge.yaml -n $namespace
``` 

## Creating the tls certificate
```powershell
kubectl apply -f .\emissary-ingress\tls-certificate.yaml -n $namespace

```

## Enabling TLS and HTTPS
```powershell
kubectl apply -f .\emissary-ingress\host.yaml -n $namespace
``` 