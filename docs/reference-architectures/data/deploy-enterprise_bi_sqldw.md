---
title: Gain business insights from relational data
description: >-
  How to gain business insights from relational data, paying attention to scalability, resiliency,
  manageability, and security.

author: alexbuckgit

ms.date: 03/21/2018

 pnp.series.title: Data reference architectures
 pnp.series.next: 
 pnp.series.prev: ./index
---

## Deploy the solution

A deployment for this reference architecture is available on [GitHub][github-folder].

### Prerequisites

Perform these prequisite steps before deploying the reference architecture to your subscription.

1. Clone, fork, or download the zip file for the [Azure reference architectures][ref-arch-repo] GitHub repository.

2. Install the [Azure Building Blocks][azbb-wiki].

3. From a command prompt, bash prompt, or PowerShell prompt, login to your Azure account by using the command below and following the instructions.

  ```bash
  az login  
  ```

### Deploy the simulated on-premises server using azbb

First you will deploy a virtual machine as a simulated on-premises server, which includes SQL Server 2017 and related tools. To deploy this virtual machine, follow these steps:

1. Navigate to the `data\enterprise-bi-sqldw\onprem\templates` folder of the repository you downloaded in the prerequisites above.

2. Edit the `onprem.parameters.json` file to add a user name and password between the quotes, as shown below.
    
    ```bash
    "adminUsername": "",
    "adminPassword": "",
    ```

      Also change the values in the `SqlUserCredentials` section to match the user name and password you chose above. Note the `.\\` prefix in the userName property.
    
    ```bash
    "SqlUserCredentials": {
      "userName": ".\\username",
      "password": "password"
    }
    ```

3. Run `azbb` as shown below to deploy the on-premises server.

    ```bash
    azbb -s <subscription_id> -g <resource_group_name> -l <location> -p onprem.parameters.json --deploy
    ```

4. Verify the deployment in the Azure portal by reviewing the resources in the resource group you specified above. You should see the `sql-vm1` virtual machine and its associated resources.

### Deploy the Azure resources

1. Navigate to the `data\enterprise-bi-sqldw\azure\templates` folder of the repository you downloaded in the prerequisites above.

2. If the resource group for your Azure resources doesn't exist yet, run the Azure CLI command below to create the resource group, replacing the bracketed parameters specified. Note that you can deploy these resources to different a resource group or location than the on-premises server you deployed previously. 

    ```bash
    az group create --name <resource_group_name> --location <location>  
    ```

3. Run the Azure CLI command below to deploy the Azure resources, replacing the bracketed parameters specified. Note that the `storageAccountName` value must be between 3 and 24 characters long and is limited to numbers and lowercase letters only.

    ```bash
    az group deployment create --resource-group <resource_group_name> --template-file azure-resources-deploy.json --parameters "dwServerName"="<dw_server_name>" "dwServerAdmin"="<dw_server_admin_username>" "dwServerPassword"="<dw_server_admin_password>" "storageAccountName"="<storage_account_name>" "analysisServerName"="<analysis_server_name>" "analysisServerAdmin"="exampleusername@contoso.com"
    ```

4. Verify the deployment in the Azure portal by reviewing the resources in the resource group you specified above. You should see the storage account, destination SQL Server, SQL data warehouse, and Analysis Services instance that were created.

### Copy the source data into blob storage 

1. Use Remote Desktop to connect to the simulated on-premises server you created previously.

2. To copy the source data from the SQL Server database into Azure Blob storage via `bcp` and `azcopy`, run the commands below from a PowerShell prompt on the server, replacing the bracketed parameters specified. You can find the storage account key via the "Access keys" setting for the storage account in the Azure portal.  

    ```powershell
    cd 'C:\SampleDataFiles\reference-architectures\data\enterprise_bi_sqldw\onprem'

    .\Load_SourceData_To_Blob.ps1 -File .\sql_scripts\db_objects.txt -Destination 'https://<storage_account_name>.blob.core.windows.net/' -StorageAccountKey '<storage_account_key>'
    ```

3. Verify that the source data was copied to Blob storage via the Azure portal. Navigate to the storage account you specified, select the Blob service, and open the `wwi` container. You should see a list of `WorldWideImporters_Application_*` tables.

### Execute the data warehouse scripts

TBD

### Build the Azure Analysis Services model

TBD

### Analyze the data via Power BI Desktop

TBD

## Next steps

- For more information about this reference architecture, visit our [GitHub repository][ref-arch-repo-folder].
- Learn about the [Azure Building Blocks][azbb-repo].

<!-- links -->

[azure-cli-2]: /azure/install-azure-cli
[azbb-repo]: https://github.com/mspnp/template-building-blocks
[azbb-wiki]: https://github.com/mspnp/template-building-blocks/wiki/Install-Azure-Building-Blocks
[ref-arch-repo]: https://github.com/mspnp/reference-architectures
[ref-arch-repo-folder]: https://github.com/mspnp/reference-architectures/tree/master/data/enterprise_bi_sqldw

