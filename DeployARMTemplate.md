<properties
   pageTitle="Deploying an Azure Resource Manager Template"
   description="Deploying your applications via Azure Resource Manager Templates allows you to quickly and easily provision your applications in Azure via declarative JSON."
   services="na"
   documentationCenter="na"
   authors="rjmax"
   manager="gautamt"
   editor=""/>

<tags
   ms.service="na"
   ms.devlang="na"
   ms.topic="article"
   ms.tgt_pltfrm="ibiza"
   ms.workload="na"
   ms.date="04/13/2015"
   ms.author="Your MSFT alias or your full email address;semicolon separates two or more"/>

# Deploy an Azure Resource Manager Template

Deploying your applications via Azure Resource Manager Templates allows you to quickly and easily provision your applications in Azure via declarative JSON. A single template can contain multiple services, including Virtual Machines, Virtual Networks, Storage, App Services, Databases, etc.

Resource Manager Templates are always deployed into Resource Groups; Resource Groups are used to organize resources into collections of entities which share a common lifecycle.  In most cases, a Resource Group maps to either a single application, or in the case of larger applications, an application tier.

Within a Resource Group, Deployments are used to track the execution of a Resource Manager Template, provide the status of the deployment, and return any output from template execution.

To deploy a template, parameters usually are supplied by the user to customize resources.  These parameters can be supplied either inline or via a parameters file.

## Concepts

- Resource Group: Collection of entities which share a common lifecycle
- Resource Manager Template: A declarative JSON file which defines the goal state of a deployment
- Deployment: operation which tracks execution of a Resource Manager Template
- Parameters: values provided by users to customize deployed resources

## Scenarios

With Resource Manager Templates, you can:

- Deploy complex multitier applications, such as Microsoft SharePoint, using a single deployment template.
- Consistently and repeatedly deploy your applications to support multiple dev, test, and production environments.
- View success or failure of deployments and troubleshoot deployment failures using deployment audit logs.

## How to use the Portal

Guess what?  Every application in the Gallery is backed by an Azure Resource Manager template!  By simply creating a Virtual Machine, Virtual Network, Storage Account, App Service, or Database through the portal, you're already reaping the benefits of Azure Resource Manager without additional effort.

To troubleshoot deployments through the portal, click **Browse** -> **Resource Groups** -> <YourResourceGroupName>.  From here, click on the **Events** tile under the **Monitoring** lens.  Finally, you can select an individual **operation** and **event** to view details.

## How to use PowerShell

>[AZURE.NOTE]To install the latest Azure PowerShell cmdlets, please follow [these](http://bing.com) instructions.

1. Login to Azure using `Add-AzureAccount`
2. If you have multiple subscriptions, select your desired subscription using `Select-AzureSubscription -SubscriptionID <YourSubscriptionId>`
3. Switch the PowerShell mode to work with Azure Resource Manager `Switch-AzureMode AzureResourceManager`
4. If you do not have an existing Resource Group, create a new Resource Group `New-AzureResourceGroup -Name <YourResourceGroupName> -Location <YourLocationName>`
5. Create a new Resource Group Deployment
    - Using inline parameters
    `New-AzureResourceGroupDeployment -Name <YourDeploymentName> -ResourceGroupName <YourResourceGroupName> -TemplateFile <PathOrLinkToTemplate> -myParameterName "parameterValue"`
    - Using a parameter object
    `$parameters = @{"<ParameterName>"="<Parameter Value>"}`
    `New-AzureResourceGroupDeployment -Name <YourDeploymentName> -ResourceGroupName <YourResourceGroupName> -TemplateFile <PathOrLinkToTemplate> -TemplateParameterObject $parameters`
    - Using a parameter file
    `New-AzureResourceGroupDeployment -Name <YourDeploymentName> -ResourceGroupName <YourResourceGroupName> -TemplateFile <PathOrLinkToTemplate> -TemplateParameterFile <PathOrLinkToParameterFile>`
6. Get information about deployment failures `Get-AzureResourceGroupLog -ResourceGroup <YourResourceGroupName> -Status Failed`
7. Get detailed information about deployment failures `Get-AzureResourceGroupLog -ResourceGroup <YourResourceGroupName> -Status Failed -DetailedOutput`

## How to use xplat-cli

>[AZURE.NOTE]To install the latest Azure xplat cli, please follow [these](http://bing.com) instructions.

1. Login to Azure using `azure login`
2. If you have multiple subscriptions, select your desired subscription using `azure account set <YourSubscriptionNameOrId>`
3. Switch the PowerShell mode to work with Azure Resource Manager `azure config mode arm`
4. If you do not have an existing Resource Group, create a new Resource Group `azure group create -n <YourResourceGroupName> -l <YourLocationName>`
5. Create a new Resource Group Deployment
    - Using inline parameters using a local template
    `azure group deployment create -f <PathToTemplate> {"ParameterName":"ParameterValue"} -g <ResourceGroupName> -n <YourDeploymentName>`
    - Using inline parameters using a link to a template
    `azure group deployment create --template-uri <LinkToTemplate> {"ParameterName":"ParameterValue"} -g <ResourceGroupName> -n <YourDeploymentName>`
    - Using a parameter file
    `azure group deployment create -f <PathToTemplate> -e <PathToParameterFile> -g <ResourceGroupName> -n <YourDeploymentName>`
6. Get information about your latest deployment `azure group log show -l <ResourceGroupName>`
7. Get detailed information about deployment failures `azure group log show -l -v <ResourceGroupName>`

## How to use the REST API
1. Set common headers and parameters, including authentication tokens, as outlined [here](https://msdn.microsoft.com/en-us/library/azure/8d088ecc-26eb-42e9-8acc-fe929ed33563#bk_common).
2. Create the Resource Group (additional information can be found [here](https://msdn.microsoft.com/en-us/library/azure/dn790525.aspx)).
```
PUT https://management.azure.com/subscriptions/<YourSubscriptionId>/resourcegroups/<YourResourceGroupName>?api-version=2015-01-01
<common headers per https://msdn.microsoft.com/en-us/library/azure/8d088ecc-26eb-42e9-8acc-fe929ed33563#bk_common>
{
    "location": "West US",
    "tags": {
    "tagname1": "tagvalue1"
    }
}
```
3. Create Resource Group Deployment (additional information can be found [here](https://msdn.microsoft.com/en-us/library/azure/dn790564.aspx)).
```
PUT https://management.azure.com/subscriptions/<YourSubscriptionId>/resourcegroups/<YourResourceGroupName>/providers/Microsoft.Resources/deployments/<YourDeploymentName>?api-version=2015-01-01
 <common headers per https://msdn.microsoft.com/en-us/library/azure/8d088ecc-26eb-42e9-8acc-fe929ed33563#bk_common>
 {
   "properties": {
     "templateLink": {
       "uri": "http://mystorageaccount.blob.core.windows.net/templates/template.json",
       "contentVersion": "1.0.0.0",
     },
     "mode": "Incremental",
     "parametersLink": {
       "uri": "http://mystorageaccount.blob.core.windows.net/templates/parameters.json",
       "contentVersion": "1.0.0.0",
     }
   }
 }
```
4. Get the status of the template deployment
(additional information can be found [here](https://msdn.microsoft.com/en-us/library/azure/dn790565.aspx)).
```
GET https://management.azure.com/subscriptions/<YourSubscriptionId>/resourcegroups/<YourResourceGroupName>/providers/Microsoft.Resources/deployments/<YourDeploymentName>?api-version=2015-01-01
 <common headers per https://msdn.microsoft.com/en-us/library/azure/8d088ecc-26eb-42e9-8acc-fe929ed33563#bk_common>
```
5. If you want to retrieve additional information about template deployments, please see our reference documentation [here](https://msdn.microsoft.com/en-us/library/azure/dn790549.aspx).

<!-- links to other topics -->
## Next steps



<!--Image references-->
[exampleimage]: ./media/markdown-template-for-new-articles/octocats.png
