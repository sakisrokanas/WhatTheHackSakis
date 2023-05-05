//Set variables
RG=wthrpsrg
az account set --<your subscription here>

//Create a firewall rule that allows access from Azure services
az sql server firewall-rule create -g $RG -s <your sql db server here> -n azureaccess --start-ip-address 0.0.0.0 --end-ip-address 0.0.0.0


//1.Create a user-assigned managed identity
//Create a managed identity in the resource group.

az identity create --name wthacrid --resource-group $RG

//2.Create a container registry
//Create a container registry 
az acr create --name wthrpsacr --resource-group $RG --sku Basic --admin-enabled true

//Retrieve the administrative credentials of acr
az acr credential show --resource-group $RG --name wthrpsacr

//3.Push the sample image to Azure Container Registry
//login to the container registry:
docker login wthrpsacr.azurecr.io --username wthrpsacr
//tag your local Docker image to the registry
docker tag code_rockpaperscissors-server wthrpsacr.azurecr.io/code_rockpaperscissors-server:latest
//push the image to the registry:
docker push wthrpsacr.azurecr.io/code_rockpaperscissors-server:latest


//4. Authorize the managed identity for your registry
//Retrieve the principal ID for the managed identity:
principalId=$(az identity show --resource-group $RG --name wthacrid --query principalId --output tsv)
//Retrieve the resource ID for the container registry:
registryId=$(az acr show --resource-group $RG --name wthrpsacr --query id --output tsv)
//Grant the managed identity permission to access the container registry:
az role assignment create --assignee $principalId --scope $registryId --role "AcrPull"

//5. Create the web app
//Create an App Service plan using
az appservice plan create --name wthrpsappplan --resource-group $RG --is-linux
//Create the web app
az webapp create --resource-group $RG --plan wthrpsappplan --name wthrpswebappsakis1 --deployment-container-image-name wthrpsacr.azurecr.io/code_rockpaperscissors-server:latest

//6. Configure the web app
//set the environment variable for Azure SQL server
az webapp config appsettings set -g $RG -n wthrpswebappsakis1 --settings 'ConnectionStrings:DefaultConnection'='<your DB server here>,1433;Database=RockPaperScissorsBoom;Persist Security Info=False;User ID=<username here>;Password=<password here>;'




//Enable the system-assignedmanaged identity in the web app
id=$(az identity show --resource-group $RG --name wthacrid --query id --output tsv)
az webapp identity assign --resource-group $RG --name wthrpswebappsakis1 --identities $id
//Configure your app to pull from Azure Container Registry by using managed identities
appConfig=$(az webapp config show --resource-group $RG --name wthrpswebappsakis1 --query id --output tsv)
az resource update --ids $appConfig --set properties.acrUseManagedIdentityCreds=True
