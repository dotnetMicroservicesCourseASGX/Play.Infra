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