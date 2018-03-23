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

A deployment for this reference architecture is available on [GitHub][ref-arch-repo-folder]. It deploys the following:

  * A Windows virtual machine to simulate an on-premises database server. It includes SQL Server 2017 and related tools.
  * An Azure storage account that provides Blob storage to hold data exported from the SQL Server database.
  * An Azure SQL Data Warehouse instance.
  * An Azure Analysis Services instance.

### Prerequisites

Perform these prequisite steps before deploying the reference architecture to your subscription.

1. Clone, fork, or download the zip file for the [Azure reference architectures][ref-arch-repo] GitHub repository.

2. Install the [Azure Building Blocks][azbb-wiki].

3. From a command prompt, bash prompt, or PowerShell prompt, login to your Azure account by using the command below and following the instructions.

  ```bash
  az login  
  ```

### Deploy the simulated on-premises server using azbb

First you'll deploy a virtual machine as a simulated on-premises server, which includes SQL Server 2017 and related tools. To deploy this virtual machine, follow these steps:

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

2. If the resource group for your Azure resources doesn't exist yet, run the Azure CLI command below to create the resource group, replacing the bracketed parameters specified. Note that you can deploy to a different resource group or location than you used for the on-premises server that you deployed previously. 

    ```bash
    az group create --name <resource_group_name> --location <location>  
    ```

3. Run the Azure CLI command below to deploy the Azure resources, replacing the bracketed parameters specified. Note that the `storageAccountName` value must be between 3 and 24 characters long and is limited to numbers and lowercase letters only.

    ```bash
    az group deployment create --resource-group <resource_group_name> --template-file azure-resources-deploy.json --parameters "dwServerName"="<dw_server_name>" "dwServerAdmin"="<dw_server_admin_username>" "dwServerPassword"="<dw_server_admin_password>" "storageAccountName"="<storage_account_name>" "analysisServerName"="<analysis_server_name>" "analysisServerAdmin"="exampleusername@contoso.com"
    ```

4. Verify the deployment in the Azure portal by reviewing the resources in the resource group you specified above. You should see the storage account, Azure SQL Data Warehouse instance, and Analysis Services instance that were created.

### Export the source data to Azure Blob storage 

1. Use Remote Desktop to connect to the simulated on-premises server you created previously.

2. To copy the source data from the SQL Server database into Azure Blob storage via `bcp` and `azcopy`, run the commands below from a PowerShell prompt on the server, replacing the bracketed parameters specified. You can find the storage account key via the "Access keys" setting for the storage account in the Azure portal.  

    ```powershell
    cd 'C:\SampleDataFiles\reference-architectures\data\enterprise_bi_sqldw\onprem'

    .\Load_SourceData_To_Blob.ps1 -File .\sql_scripts\db_objects.txt -Destination 'https://<storage_account_name>.blob.core.windows.net/wwi' -StorageAccountKey '<storage_account_key>'
    ```

3. In the Azure portal, verify that the source data was copied to Blob storage by navigating to the storage account you specified, selecting the Blob service, and opening the `wwi` container. You should see a list of tables prefaced with `WorldWideImporters_Application_*`.

### Execute the data warehouse scripts

1. From your Remote Desktop session, open SQL Server Management Studio (SSMS) via the Windows start menu. Connect to the database engine for the SQL Data Warehouse, using the fully qualified version of the `dwServerName` value you specified above (for example, `examplesqldw1.database.windows.net`) along with the credentials you specified for `dwServerAdmin` and `dwServerPassword`.

2. Navigate to the `C:\Repos\reference-architectures\data\enterprise_bi_sqldw\azure\sqldw_scripts` folder on the on-premises server. You will execute the scripts in this folder in numerical order (`STEP_1` through `STEP_7`) as indicated below.

3. Open the `STEP_1` script in a query window. Select the `master` database in SSMS and execute the script.

4. Open the `STEP_2` script in a query window. Select the `wwi` database in SSMS and execute the script.

5. In SSMS, open a new connection to the database engine for the SQL Data Warehouse, using the `LoaderRC20` user and the password indicated in the `STEP_1` script.  
    a. Open the `STEP_3` script in a query window for this connection.  
    b. In the script, set the SECRET value to your storage account key.  
    c. In the script, set the LOCATION value to specify your storage account container using the format "wasbs://wwi@`your_storage_account_name`.blob.core.windows.net".

6. Execute scripts STEP_4 through STEP_7 sequentially using an SSMS query window for the same connection.

7. In SMSS, you should see a set of `prd.*` tables in the `wwi` database. Verify that the data was generated by running a test query such as `SELECT TOP 10 * FROM prd.CityDimensions`.

### Build the Azure Analysis Services model

1. From the Windows start menu on the on-premises server, open **SQL Server Data Tools 2015**.

2. Select **File -> New -> Project -> Templates -> Business Intelligence -> Analysis Services -> Analysis Services Tabular Project**. Name your project and click **OK**.
3. In the Tabular model designer dialog box, select the **Integrated workspace** option and set the **Compatibility level** to `SQL Server 2017 / Azure Analysis Services (1400)`, then click **OK**.
4. In the Tabular Model Explorer window, right-click the project and select **Import from Data Source**.
5. Select **All -> Azure SQL Data Warehouse**, then click **Connect**.
6. Set the Server value to the fully qualified name of your Azure SQL Data Warehouse, and the Database value to `wwi`, then click **OK**.
7. In the next dialog box, choose **Database** authentication and enter your Azure SQL Data Warehouse admin user name and password, then click **OK**.
8. In the Navigator dialog box, select the checkboxes for **prd.CityDimensions**, **prd.DateDimensions**, and **prd.SalesFact**, then click **Load**. When processing is complete, click **Close**. You should now see a tabular view of the data.
9. In the Tabular Model Explorer window, right-click the project and select **Model View -> Diagram View**.  
    a. In the diagram view, drag the **[prd.SalesFact].[WWI City ID]** field to the **[prd.CityDimensions].[WWI City ID]** field to create a relationship.  
    b. Drag the **[prd.SalesFact].[Invoice Date Key]** field to the **[prd.DateDimensions].[Date]** field.  
    c. From the Visual Studio **File** menu, choose **Save All**.  
10. In the Solution Explorer window, right-click on the project and select **Properties**. Set the Server property to the URL of your Azure Analysis Services instance. This value is the **Server Name** property of your Analysis Services instance displayed in the Azure portal. Click **OK**.
11. In the Solution Explorer window, right-click on the project and select **Deploy**. Sign into Azure if prompted. When processing is complete, click **Close**.
12. In the Azure portal, view the details for your Azure Analysis Services instance. Verify that your model appears in the list of models.

### Analyze the data via Power BI Desktop

1. From the Windows start menu on the on-premises server, open **Power BI Desktop**.
2. In the **Visualizations** pane, select the **Stacked Bar Chart** icon. Resize the workspace to make it larger.
3. In the Home ribbon tab, select **Get Data -> Analysis Services**.
4. Enter the URL of your Analysis Services instance, then click **OK**. Sign into Azure if prompted.
5. In the Navigator window, expand the tabular project that you deployed, select the model that you created, and click **OK**.
6. In the Fields pane, expand **prd.CityDimensions**.
    a. Drag the **WWI City ID** field to below the Axis label.
    b. Drag the **City** field to below to below the Legend label.
7. In the Fields pane, expand **prd.SalesFact**, and drag the **Total Excluding Tax** field to below the Value label.
8. Under Visual Level Filters, select **WWI City ID**, set the **Filter Type** to `Top N`, and set **Show Items** to `Top 10`.
9. From **prd.SalesFact**, drag the **Total Excluding Tax** field to below the **By Value** label, and click **Apply Filter**.

## Next steps

- For more information about this reference architecture, visit our [GitHub repository][ref-arch-repo-folder].
- Learn about the [Azure Building Blocks][azbb-repo].

<!-- links -->

[azure-cli-2]: /azure/install-azure-cli
[azbb-repo]: https://github.com/mspnp/template-building-blocks
[azbb-wiki]: https://github.com/mspnp/template-building-blocks/wiki/Install-Azure-Building-Blocks
[github-folder]: https://github.com/mspnp/reference-architectures/tree/master/data/enterprise_bi_sqldw
[ref-arch-repo]: https://github.com/mspnp/reference-architectures
[ref-arch-repo-folder]: https://github.com/mspnp/reference-architectures/tree/master/data/enterprise_bi_sqldw

