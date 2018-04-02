# Create a SQL Server AlwaysOn Availability Group (SQL AG) solution in an existing Azure VNET and existing Active Directory (AD) domain across Azure Availability Zones (AZ) using an Azure Internal Load Balancer (ILB)

This Azure ARM template will create a SQL Server AlwaysOn Availability Group using the PowerShell DSC Extension in an existing Azure Virtual Network (VNet) and existing Active Directory (AD) environment. Both SQL Server 2016 and  2017 are supported by this ARM template. The SQL Server VMs will be provisioned across multiple Azure Availability Zones (AZ) and requests will be directed to the SQL AG Listener via the zone redundant Internal Load Balancer (ILB) Standard.

<a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Frolftesmer%2F301-sql-alwayson-md-ilb-zones%2Fmaster%2Fazuredeploy.json" target="_blank"><img src="http://azuredeploy.net/deploybutton.png"/>
</a>


## ARM Template Parameters



## Deploying ARM Templates

You can deploy these samples directly through the Azure Portal or by using the scripts supplied in the root of the repo.

To deploy using the Azure Portal, click the **Deploy to Azure** button above.

To deploy using the command line (using [Azure PowerShell or the Azure CLI](https://azure.microsoft.com/en-us/downloads/)) you can use the scripts.  Simple execute the script and pass in the folder name of the sample you want to deploy.  For example:

```PowerShell
.\Deploy-AzureResourceGroup.ps1 -ResourceGroupLocation 'eastus2' -ArtifactsStagingDirectory '[foldername]'
```
```bash
azure-group-deploy.sh -a [foldername] -l eastus2 -u
```

This ARM template has artifacts that need to be "staged" for deployment (Configuration Scripts, Nested Templates, DSC Packages) so ensure to set the "UploadArtifacts" switch on the command.  You can optionally specify a storage account to use for hosting the ARM template artefacts, and if so the storage account must already exist within the target subscription.  If you don't want to specify a storage account one will be created by the script or reused if it already exists.

```PowerShell
.\Deploy-AzureResourceGroup.ps1 -ResourceGroupLocation 'eastus2' -ArtifactsStagingDirectory '301-sql-alwayson-md-ilb-zones' -UploadArtifacts 
```
```bash
azure-group-deploy.sh -a '301-sql-alwayson-md-ilb-zones' -l eastus2 -u
```
