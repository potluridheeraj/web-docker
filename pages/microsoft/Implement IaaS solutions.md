---
title: Implement IaaS solutions
keywords: Microsoft, Az-204
permalink: Implement_IaaS_solutions
folder: microsoft
sidebar: az-204_sidebar
---
# Implement IaaS solutions

    * provision virtual machines (VMs)  
    * configure, validate, and deploy ARM templates 
    * configure container images for solutions    
    * publish an image to the Azure Container Registry    
    * run containers by using Azure Container Instance
    
## Provision VMs

Quickstart: Create a Windows virtual machine in the Azure portal  
[Tutorial: Create and Manage Windows VMs with Azure PowerShell](https://docs.microsoft.com/en-us/azure/virtual-machines/windows/tutorial-manage-vm?WT.mc_id=thomasmaurer-blog-thmaure)    

How to connect and sign on to an Azure virtual machine running Windows  
[Quick steps: Create and use an SSH public-private key pair for Linux VMs in Azure](https://docs.microsoft.com/en-us/azure/virtual-machines/linux/mac-create-ssh-keys?WT.mc_id=thomasmaurer-blog-thmaure)          


{% include note.html content="A resource group must be created before a virtual machine" %}

### Create resource group
```powershell
windows:
New-AzResourceGroup -ResourceGroupName "myResourceGroupVM" -Location "EastUS"

azure CLI or Linux:
az group create  --name myResourceGroup --location "Central US"
```

### Create the VM.    

(Windows)
```powershell
New-AzVm `
    -ResourceGroupName "myResourceGroupVM" `
    -Name "myVM" `
    -Location "EastUS" `
    -VirtualNetworkName "myVnet" `
    -SubnetName "mySubnet" `
    -SecurityGroupName "myNetworkSecurityGroup" `
    -PublicIpAddressName "myPublicIpAddress" `
    -Credential $cred
```
(Linux)
```shell
az vm create \
  --resource-group myResourceGroup \
  --name myVM \
  --image UbuntuLTS \
  --admin-username azureuser \
  --ssh-key-values mysshkey.pub
```

Return the public IP address of the VM
```powershell
Get-AzPublicIpAddress `
   -ResourceGroupName "myResourceGroupVM"  | Select IpAddress
```
    
**Get-AzVMImagePublisher** command to return a list of image publishers.    
**Get-AzVMImageOffer** to return a list of image offers.   

If you are using PowerShell and have the Azure PowerShell module installed you may also connect using the Get-AzRemoteDesktopFile cmdlet, as shown below.  
```powershell
Get-AzRemoteDesktopFile -ResourceGroupName "RgName" -Name "VmName" -Launch
```

### Resize a VM

Before resizing a VM, check if the size you want is available on the current VM cluster. The Get-AzVMSize command returns a list of sizes.

```powershell
Get-AzVMSize -ResourceGroupName "myResourceGroupVM" -VMName "myVM"
```

If the size is available, the VM can be resized from a powered-on state, however it is rebooted during the operation.
```powershell
$vm = Get-AzVM `
   -ResourceGroupName "myResourceGroupVM"  `
   -VMName "myVM"
$vm.HardwareProfile.VmSize = "Standard_DS3_v2"
Update-AzVM `
   -VM $vm `
   -ResourceGroupName "myResourceGroupVM"
```

{% include callout.html content="If the size you want isn't available on the current cluster, the VM needs to be deallocated before the resize operation can occur. Deallocating a VM will remove any data on the temp disk, and the public IP address will change unless a static IP address is being used. " type="primary" %} 

```powershell
Stop-AzVM `
   -ResourceGroupName "myResourceGroupVM" `
   -Name "myVM" -Force
$vm = Get-AzVM `
   -ResourceGroupName "myResourceGroupVM"  `
   -VMName "myVM"
$vm.HardwareProfile.VmSize = "Standard_E2s_v3"
Update-AzVM -VM $vm `
   -ResourceGroupName "myResourceGroupVM"
Start-AzVM `
   -ResourceGroupName "myResourceGroupVM"  `
   -Name $vm.name
```
## ARM Templates

{% include note.html content="ARM Templates is used to create a template which can be used to create apps, VM's or almost anything in Azure " %}

Template file
Within your template, you can write template expressions that extend the capabilities of JSON. These expressions make use of the functions provided by Resource Manager.  

The template has the following sections:  

[**Parameters**](https://docs.microsoft.com/en-us/azure/azure-resource-manager/templates/parameters) - Provide values during deployment that allow the same template to be used with different environments.  

[**Variables**](https://docs.microsoft.com/en-us/azure/azure-resource-manager/templates/variables) - Define values that are reused in your templates. They can be constructed from parameter values. 

[**User-defined functions**](https://docs.microsoft.com/en-us/azure/azure-resource-manager/templates/user-defined-functions) - Create customized functions that simplify your template. 

[**Resources**](https://docs.microsoft.com/en-us/azure/azure-resource-manager/templates/resource-declaration) - Specify the resources to deploy.  

[**Outputs**](https://docs.microsoft.com/en-us/azure/azure-resource-manager/templates/outputs?tabs=azure-powershell) - Return values from the deployed resources.  
### Deploy template

```shell
$templateFile = "{provide-the-path-to-the-template-file}"
New-AzResourceGroupDeployment `
  -Name blanktemplate `
  -ResourceGroupName myResourceGroup `
  -TemplateFile $templateFile

linux:
templateFile="{provide-the-path-to-the-template-file}"
az deployment group create \
  --name blanktemplate \
  --resource-group myResourceGroup \
  --template-file $templateFile
```

### Template Example
```json
{
  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "storagePrefix": {
      "type": "string",
      "minLength": 3,
      "maxLength": 11
    },
    "storageSKU": {
      "type": "string",
      "defaultValue": "Standard_LRS",
      "allowedValues": [
        "Standard_LRS",
        "Standard_GRS",
        "Standard_RAGRS",
        "Standard_ZRS",
        "Premium_LRS",
        "Premium_ZRS",
        "Standard_GZRS",
        "Standard_RAGZRS"
      ]
    },
    "location": {
      "type": "string",
      "defaultValue": "[resourceGroup().location]"
    }
  },
  "variables": {
    "uniqueStorageName": "[concat(parameters('storagePrefix'), uniqueString(resourceGroup().id))]"
  },
  "resources": [
    {
      "type": "Microsoft.Storage/storageAccounts",
      "apiVersion": "2019-04-01",
      "name": "[variables('uniqueStorageName')]",
      "location": "[parameters('location')]",
      "sku": {
        "name": "[parameters('storageSKU')]"
      },
      "kind": "StorageV2",
      "properties": {
        "supportsHttpsTrafficOnly": true
      }
    }
  ],
  "outputs": {
    "storageEndpoint": {
      "type": "object",
      "value": "[reference(variables('uniqueStorageName')).primaryEndpoints]"
    }
  }
}
```

## ACR Container images

configure container images for solutions

```shell
RES_GROUP=$ACR_NAME # Resource Group name

az group create --resource-group $RES_GROUP --location eastus
az acr create --resource-group $RES_GROUP --name $ACR_NAME --sku Standard --location eastus

az acr build --registry $ACR_NAME --image helloacrtasks:v1 .
```
**az keyvault secret set** 
```shell
# Create service principal, store its password in AKV (the registry *password*)
az keyvault secret set \
  --vault-name $AKV_NAME \
  --name $ACR_NAME-pull-pwd \
  --value $(az ad sp create-for-rbac \
                --name $ACR_NAME-pull \
                --scopes $(az acr show --name $ACR_NAME --query id --output tsv) \
                --role acrpull \
                --query password \
                --output tsv)


# Store service principal ID in AKV (the registry *username*)
az keyvault secret set \
    --vault-name $AKV_NAME \
    --name $ACR_NAME-pull-usr \
    --value $(az ad sp list --display-name $ACR_NAME-pull --query [].appId --output tsv)
```

**$ACR_NAME-pull-usr**: The service principal ID, for use as the container registry username.   
**$ACR_NAME-pull-pwd**: The service principal password, for use as the container registry password.

### Deploy a container with Azure CLI
```shell
az container create \
    --resource-group $RES_GROUP \
    --name acr-tasks \
    --image $ACR_NAME.azurecr.io/helloacrtasks:v1 \
    --registry-login-server $ACR_NAME.azurecr.io \
    --registry-username $(az keyvault secret show --vault-name $AKV_NAME --name $ACR_NAME-pull-usr --query value -o tsv) \
    --registry-password $(az keyvault secret show --vault-name $AKV_NAME --name $ACR_NAME-pull-pwd --query value -o tsv) \
    --dns-name-label acr-tasks-$ACR_NAME \
    --query "{FQDN:ipAddress.fqdn}" \
    --output table


# Get the IP address of the Azure container instance running the following command
az container show --resource-group  learn-deploy-acr-rg --name acr-tasks --query ipAddress.ip --output table
```

```shell
# To check your container's status
az container show \
  --resource-group learn-deploy-aci-rg \
  --name mycontainer-restart-demo \
  --query containers[0].instanceView.currentState.state

# View the container's logs to examine the output
az container logs \
  --resource-group learn-deploy-aci-rg \
  --name mycontainer-restart-demo
```

```shell
#az acr login command to complete the operation
az acr login --name <acrName>

#To get the loginServer name, run the following command:
az acr show --name <acrName> --query loginServer --output table

#Result
#------------------
#<acrName>.azurecr.io

#Now, tag the azure-vote-front image with the loginServer of your container registry. Also, add :v1 to the end of the image name. This tag indicates the image version.
docker tag azure-vote-front <acrName>.azurecr.io/azure-vote-front:v1

#Push the azure-vote-front image to the registry
docker push <acrName>.azurecr.io/azure-vote-front:v1

#To return a list of images that have been pushed to your Azure Container registry
az acr repository list --name <acrName> --output table
```

Replicate a container image to different Azure regions
```shell
az acr replication create --registry $ACR_NAME --location japaneast

#retrieve all container image replicas created by running
az acr replication list --registry $ACR_NAME --output table
```

## Azure Container Instance

start a container instance
```shell
az container create --resource-group myResourceGroup --name mycontainer --image mcr.microsoft.com/azuredocs/aci-helloworld --dns-name-label aci-demo --ports 80

az container show --resource-group myResourceGroup --name mycontainer --query "{FQDN:ipAddress.fqdn,ProvisioningState:provisioningState}" --out table

#Output:
#FQDN                               ProvisioningState
#---------------------------------  -------------------
#aci-demo.eastus.azurecontainer.io  Succeeded

#Pull the container instance logs
az container logs --resource-group myResourceGroup --name mycontainer

#In addition to viewing the logs, you can attach your local standard out and standard error streams to that of the container.
az container attach --resource-group myResourceGroup --name mycontainer 
```
