

# Create a SQL Server AlwaysOn Availability Group (SQL AG) solution in an existing Azure VNET and existing Active Directory (AD) domain across Azure Availability Zones (AZ) using an Azure Internal Load Balancer (ILB)

This Azure ARM template will create a SQL Server AlwaysOn Availability Group using the PowerShell DSC Extension in an existing Azure Virtual Network (VNet) and existing Active Directory (AD) environment. Both SQL Server 2016 and  2017 are supported by this ARM template. The SQL Server VMs will be provisioned across multiple Azure Availability Zones (AZ) and requests will be directed to the SQL AG Listener via the zone redundant Internal Load Balancer (ILB) Standard.

This ARM template has several requirements and assumptions in regards to the target deployment environment that must be in place *before* deploying this solution.  Please see below.

For information on Azure AZ, including services and regions that support AZ, see this link - https://docs.microsoft.com/en-us/azure/availability-zones/az-overview

For information on SQL AlwaysOn Availability Groups see this link - https://docs.microsoft.com/en-us/sql/database-engine/availability-groups/windows/always-on-availability-groups-sql-server

<a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Frolftesmer%2F301-sql-alwayson-md-ilb-zones%2Fmaster%2Fazuredeploy.json" target="_blank"><img src="http://azuredeploy.net/deploybutton.png"/>
</a>

### NOTE
The script is intended for demonstration purposes and is not considered "Production Ready".  For simplicity a number of best practice configurations have not been implemented in this solution and should be considered when deploying this template.


## ARM Template Parameters & Requirements & Assumptions

### Location
The name of the Azure region that you want to deploy this solution.  The target region MUST have AZ capabilities enabled, and your subscription must allow for the deployment of services into Azure AZ in that region (ie for AZ Preview you must be signed up and enabled in your subscription)

Ensure to use the Azure region short names "centralus", "westeurope", "eastus2" etc

### Name Prefix
A short alphanumeric text to be added to the front of any resources deployed.  This ensures that for any services that are deployed into an existing Azure Resource Group that the name is unique, is eaisly identifiable and can also be eaisly deleted if required.

3-8 alphanumeric characters.

### VM Size
The name of the type of VM to be deployed to host the SQL AG cluster nodes.

This value must match an exact SQL Azure VM size.  The Azure VM size names can be found here - https://docs.microsoft.com/en-us/azure/virtual-machines/windows/sizes-general

Only VM that allows Premium disk are supported in this solution (ie Ensure the VM is marked with an "S" - "Standard_DS3_v2" and *NOT* "Standard_D3_v2")

### SQL VM Image
Select from the dropdown box for the SQL deployment image that you want to deploy.

SQL AlwaysOn Availability Groups is supported in the major SQL Editions (Enterprise and Developer).  Standard Edition has some limitations and will only deploy Basic Availability Groups, and is not a supported deployment within this template.

This ARM template will deploy SQL Developer Edition (SQLDEV).  If you want to deploy SQL Enterprise Edition, then update the "imageSKU" value in both the "nestedtemplates/deploy-sql-cluster.json" and "nestedtemplates/deploy-sql-cluster-full.json" files to "Enterprise".

See here for SQL 2016 - https://docs.microsoft.com/en-us/sql/sql-server/editions-and-components-of-sql-server-2016#RDBMSHA

See here for SQL 2017 - https://docs.microsoft.com/en-us/sql/sql-server/editions-and-components-of-sql-server-2017#RDBMSHA

### VM Count
The number of Azure VM to deploy.  

Each of these VM will host a SQL deployment and will act as a node in the SQL AlwaysOn cluster.

Must be a numeric value between 2-9.

SQL AlwaysOn supports up to 8 additional replicas (hence the limit of 9 Azure VM)

### VM Disk Size
The size of the SQL data disks to attach to each of the deployed VM.

Must be a numeric value of 128-1023.

This is only related to the SQL data disks and does not impact the Windows OS drive

Each disk attached is a Premium Azure Managed Disk.

### VM Disk Count
The number of SQL data disks to attach to each of the deployed VM.  

Must be a numeric value of 2-32.  

A single Windows Storage Space will be created to span all the attached disks into a single allocated space.

Ensure that the "VM Size" type above that is being deployed supports the number of disks.

This *does not* include the Windows OS disk, which is allocated and attached seperately by default.  

This value *does not* include the D:\ Azure temporary drive which is always attached to the VM by default

Each disk attached is a Premium Azure Managed Disk.

### Existing AD Domain Name
The name of an existing AD Domain into which the SQL VM nodes will be deployed and joined.

The domain name format must be fully qualified (ie "contoso.com" and not "contoso")

This *must* be an existing Active Directory domain.  This template does not create AD servers nor create a domain.

### Existing AD Domain Admin Username & Password
The name/password of an existing account within the existing AD Domain that is a Domain Admin of that domain.

This *must* be an existing AD Account.  This template does not create the AD Account.

This domain account will be used to join the new SQL VM and setup the cluster names into the domain.

### Existing SQL Server Service Account AD Username & Password
The name/password of an existing account within the existing AD Domain that will be used to run the SQL Server Database Engine services on each of the deployed SQL VM.

This account must be different that the one used as the existing AD Domain Admin account.

This *must* be an existing AD Account.  This template does not create the AD Account.

### Existing Azure VNet Resource Group Name
The name of an existing Azure Resource Group (that contains an existing Azure VNet and Subnet (see below)) into which the SQL VM will be deployed.

It is recommended that the Resource Group exist in the same region as the hosted Azure services/resources will be deployed.

This *must* be an existing AD Resource Group.  This template does not create the Azure Resource Group.

### Existing Azure VNet Name
The name of an existing Azure VNet (that contains an existing Azure Subnet (see below)) into which the SQL VM will be deployed.

This VNet must exist in the same region as the target location of this script deployment.

This *must* be an existing Azure VNet.  This template does not create the Azure VNet.

### Existing Azure VNet Subnet Name
### Enable Outbound Internet on SQL VM
### SQL Workload Type
### ARM Template Artefacts


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
