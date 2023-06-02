---
title: Create Azure App Service Web Apps
keywords: Microsoft, Az-204
permalink: Create_Azure_App_Service_Web_Apps
folder: microsoft
sidebar: az-204_sidebar
---


# Create Azure App Service Web Apps    
          * create an Azure App Service Web App    
          * enable diagnostics logging    
          * deploy code to a web app    
          * configure web app settings including SSL, API settings, and connection strings    
          * implement autoscaling rules including scheduled autoscaling and autoscaling by    
          * operational or system metrics   

## Web app

```shell
To deploy the web app
az appservice plan create --name $appPlan --resource-group $resourceGroup --location $appLocation --sku FREE
az webapp create --name $appName --resource-group $resourceGroup --plan $appPlan --deployment-source-url $gitRepo

#To create a storage account
az storage account create -n $storageAccount -g $resourceGroup -l $appLocation --sku Standard_LRS
```
## Enable logging using the Azure CLI

```shell
#To enable app logging to the file system
az webapp log config --application-logging true --level verbose --name <app-name> --resource-group <resource-group-name>

#To view the current logging status for an app
az webapp log show --name <app-name> --resource-group <resource-group-name>
```


**Windows app log files**   
For Windows apps, file system log files are stored in a virtual drive that is associated with your Web App. This drive is addressable as D:\Home, and includes a LogFiles folder; within this folder are one or more subfolders:    

Application - Contains application-generated messages, if File System application logging has been enabled.   
DetailedErrors - Contains detailed Web server error logs, if Detailed error messages have been enabled.   
http - Contains IIS-level logs, if Web server logging has been enabled.   
W3SVC<number> - Contains details of all failed http requests, if Failed request tracing has been enabled.   


**Linux app log files**
For Linux Web Apps, the Azure tools currently support fewer logging options than for Windows apps. Redirections to STDERR and STDOUT are managed through the underlying Docker container that runs the app, and these messages are stored in Docker log files. To see messages logged by underlying processes, such as Apache, you will need to open an SSH connection to the Docker container.

## configure web app settings including SSL, API, and connection strings
  Custom configuration and application settings in Azure Web Sites  
  Configure an App Service app in the Azure portal  
  Buy a custom domain name for Azure App Service  
  Add a TLS/SSL certificate in Azure App Service  

### Automate app settings with the Azure CLI
``` shell
#Assign a value to a setting with az webapp config app settings set:
az webapp config appsettings set --name <app-name> --resource-group <resource-group-name> --settings <setting-name>="<value>"

#Show all settings and their values with az webapp config appsettings list:
az webapp config appsettings list --name <app-name> --resource-group <resource-group-name>

#Remove one or more settings with az webapp config app settings delete:
az webapp config appsettings delete --name <app-name> --resource-group <resource-group-name> --setting-names {<names>}
```

