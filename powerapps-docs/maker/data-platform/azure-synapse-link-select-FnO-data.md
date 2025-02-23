---
title: Choose finance and operations data in Azure Synapse Link for Dataverse
description: Learn how to choose finance and operations data in Microsoft Azure Synapse Link for Dataverse and work with Azure Synapse Link and Power BI.
ms.date: 05/30/2024
ms.reviewer: matp 
ms.topic: "how-to"
applies_to: 
  - "powerapps"
author: Milindav
ms.subservice: dataverse-maker
ms.author: Milindav
search.audienceType: 
  - maker
ms.custom: bap-template
---
# Choose finance and operations data in Azure Synapse Link for Dataverse

Microsoft Azure Synapse Link for Dataverse lets you choose data from finance and operations apps. Use Azure Synapse Link to continuously export data from finance and operations apps into Azure Synapse Analytics and Azure Data Lake Storage Gen2.

Azure Synapse Link for Dataverse is a service that's designed for enterprise big data analytics. It provides scalable high availability together with disaster recovery capabilities. Data is stored in the Common Data Model format, which provides semantic consistency across apps and deployments.

Azure Synapse Link for Dataverse offers the following features that you can use with finance and operations data:

- You can choose both standard and custom finance and operations entities and tables.
- Continuous replication of entity and table data is supported. Create, update, and delete (CUD) transactions are also supported.
- You can link or unlink the environment to Azure Synapse Analytics and/or Data Lake Storage Gen2 in your Azure subscription. You don't have to go to the Azure portal or Microsoft Dynamics Lifecycle Services for system configuration.
- You can choose data and explore by using Azure Synapse. You don't have to run external tools to configure Synapse Analytics workspaces.
- All features of Azure Synapse Link for Dataverse are supported. These features include availability in all regions, saving as Parquet Delta files, and restricted storage accounts.
- The table limits in the Export to Data Lake service aren't applicable in Azure Synapse Link for Dataverse.
- By default, saving in Parquet Delta Lake format is enabled for finance and operations data, so that query response times are faster.

> [!NOTE]
> 
> This feature is generally available with finance and operations application versions shown in the following list. If you have not yet applied these application versions, install the latest cumulative update to use this feature.
> 
> - 10.0.36 (PU60) cumulative update 7.0.7036.133 or later.
> - 10.0.37 (PU61) cumulative update 7.0.7068.109 or later.
> - 10.0.38 (PU62) cumulative update 7.0.7120.59 or later
>
> You might need to apply additional updates for recent fixes. More information: [Known limitations with finance and operations tables]
>
> If you're planning to adopt the export to data lake feature in finance and operations apps, consider adopting Azure Synapse Link with finance and operations data support instead. Go to the software lifecycle announcements related to [export to data lake feature](/dynamics365/fin-ops-core/dev-itpro/data-entities/azure-data-lake-ga-version-overview) for more details. For guidance and tools to upgrade from export to data lake to Azure Synapse Link go to [transition from legacy data generation services](azure-synapse-link-transition-from-FnO.md) as well as [TechTalk Series: Synapse Link for Dataverse: Transitioning from Export to Azure Data Lake to Synapse Link](https://aka.ms/TransitiontoSynapseLinkVideos)

## Prerequisites

- You must have a finance and operations sandbox (Tier-2) or higher environment.
- For validation purposes, you can also use a [Power Platform environment provisioned with ERP based templates](/power-platform/admin/unified-experience/tutorial-deploy-new-environment-with-erp-template?tabs=PPAC)
- You can use a Tier-1 environment, also known as a cloud-hosted environment, for validation. Your environments must be version 10.0.36 (PU 60) cumulative update 7.0.7036.133 or later.

   > [!NOTE]
   > With the availability of [Power Platform environment provisioned with ERP based templates](/power-platform/admin/unified-experience/tutorial-deploy-new-environment-with-erp-template?tabs=PPAC), also known as "unified environments" for validation, we plan to remove support for cloud hosted environments (CHE) for validation beginning June 1, 2024. If you're using cloud hosted environments, consider moving to [Power Platform environment provisioned with ERP based templates](/power-platform/admin/unified-experience/tutorial-deploy-new-environment-with-erp-template?tabs=PPAC)

- The finance and operations apps environment must be linked with Microsoft Power Platform. More information: [Link your finance and operations environment with Microsoft Power Platform](#link-your-finance-and-operations-environment-with-microsoft-power-platform)
- Enable **Sql row version change tracking** configuration key. More information: [Add configurations in a finance and operations apps environment](#add-configurations-in-a-finance-and-operations-apps-environment).
- You can't add finance and operations data to an existing storage account that's configured with Azure Synapse Link. You must have access to an Azure subscription so that you can create a new SynapseL Link profile. 
- Depending on how you plan to consume finance and operations data, there are additional prerequisites as shown here.

| How you plan to consume data  |  Azure Synapse Link feature you'll use | Prerequisites and Azure resources needed |
|-------------------------------|------------------------------------|------------------------------------------|
| **Access finance and operations tables via Synapse query** <br><br> Finance and operations tables are saved in delta parquet format enabling better read performance. You can't choose finance and operations tables to be saved in CSV format. | Go to [Add finance and operations tables in Azure Synapse Link](#add-finance-and-operations-tables-in-azure-synapse-link)  |  Azure Data lake <br> Azure Synapse workspace <br> Azure Synapse Spark pool  | 
| **Load incremental data changes into your own downstream data warehouse** <br><br> The system saves incremental changes into files in CSV format. No need to bring Synapse workspace or Spark pool because your data is saved in CSV format.  | Go to [Access incremental data changes from finance and operations](#access-incremental-data-changes-from-finance-and-operations)  <br> Also go to [Azure Synapse Link - incremental update](/power-apps/maker/data-platform/azure-synapse-incremental-updates)) | Azure data lake  |
| **Access finance and operations tables via Microsoft Fabric** <br><br>  No need to bring your own storage, Synapse workspace, or Spark pool because the system uses Dataverse storage and compute resources| [Link to Fabric](azure-synapse-link-view-in-fabric.md)  | Microsoft Fabric workspace |

### Link your finance and operations environment with Microsoft Power Platform

Verify with your finance and operations systems administrator whether your finance and operations environment is linked to Power Platform.

To confirm that the finance and operations apps environment is linked with Microsoft Power Platform, review the **Environment** page in Lifecycle Services.

You can link with Microsoft Power Platform when you deploy the new environment. You can also link existing environments with Power platform. For more information about Microsoft Power Platform integration, go to [Enable the Microsoft Power Platform integration](/dynamics365/fin-ops-core/dev-itpro/power-platform/enable-power-platform-integration#enable-during-deploy).

> [!NOTE]
> Dual-write setup isn't required to enable finance and operations data in Azure Synapse Link.

### Add configurations in a finance and operations apps environment

You must enable the **Sql row version change tracking** configuration key in your finance and operations environment. In finance and operations versions 10.0.39 (PU63) or later, this configuration key might be enabled by default.

To enable this configuration key, you must turn on maintenance mode. More information: [Turn maintenance mode on and off in DevTest/Demo environments hosted in Customer's subscription](/dynamics365/fin-ops-core/dev-itpro/sysadmin/maintenance-mode#turn-maintenance-mode-on-and-off-in-devtestdemo-environments-hosted-in-customers-subscription).

![Screenshot that shows the Sql row version change tracking configuration key enabled.](media/Synapse-Link-Enable-Fno-Configuration.png)

After row version change tracking is enabled, a system event that's triggered in your environment might cause reinitialization of tables in export to data lake. If you have downstream consumption pipelines, you might have to reinitialize the pipelines. More information: [Some tables have been "initialized" without user action](/dynamics365/fin-ops-core/dev-itpro/data-entities/finance-data-azure-data-lake#some-tables-have-been-initialized-without-user-action).

### Additional steps to configure a cloud hosted environment

With the availability of [Power Platform environment provisioned with ERP based templates](/power-platform/admin/unified-experience/tutorial-deploy-new-environment-with-erp-template?tabs=PPAC), also known as "unified environments" for validation, we plan to remove support for cloud hosted environments (CHE) for validation beginning June 1, 2024. If you're using cloud hosted environments, you must perform the following additional configuration steps:

1. Complete a full database synchronization (DBSync) and use Visual Studio to complete the maintenance mode.

1. You need to enable the flights **DMFEnableSqlRowVersionChangeTrackingIndexing** and **DMFEnableCreateRecIdIndexForDataSynchronization** to create indexes required for data synchronization. When these flights are enabled, SQL indexes are created for the `RecId` and `SysRowVersion` fields if they're missing. You can enable the flights by running these SQL statements in Tier 1 environments. These indexes are created in higher environments when enabling change tracking on a table or an entity.

```sql
INSERT INTO SYSFLIGHTING (FLIGHTNAME, ENABLED) VALUES('DMFEnableSqlRowVersionChangeTrackingIndexing', 1)
INSERT INTO SYSFLIGHTING (FLIGHTNAME, ENABLED) VALUES('DMFEnableCreateRecIdIndexForDataSynchronization', 1)
```

3. You need to run the following script to perform initial indexing operations in your environment. If you don't run the script in the CHE environment, you see error "FnO-812" when adding these tables to Azure Synapse Link. This process is auto enabled with sandbox or other higher environments.

```sql
SET NOCOUNT ON;
print 'Put system in Maintainance mode'
print ''
UPDATE SQLSYSTEMVARIABLES SET VALUE = 1 WHERE PARM = 'CONFIGURATIONMODE'
SET NOCOUNT OFF;

DECLARE @SchemaName NVARCHAR(MAX) = 'dbo';
DECLARE @TableId INT;
DECLARE @TableName NVARCHAR(250);
DECLARE @SQLStmt NVARCHAR(MAX);
DECLARE @SlNo INT = 0;

DECLARE Table_cursor CURSOR LOCAL FOR
SELECT T.ID, T.Name
FROM TABLEIDTABLE T
WHERE T.Name in (
SELECT PHYSICALTABLENAME AS TableName FROM AIFSQLROWVERSIONCHANGETRACKINGENABLEDTABLES
UNION SELECT REFTABLENAME AS TableName FROM BUSINESSEVENTSDEFINITION WHERE CHANNEL LIKE 'AthenaFinanceOperationsTableDa%'
)

-- if the concerned tables are not in the above list, then replace the above cursor query with following cursor query
-- and manually enter the tablenames in the where clause
-- DECLARE Table_cursor CURSOR LOCAL FOR
-- SELECT T.ID, T.Name
-- FROM TABLEIDTABLE T
-- WHERE T.Name in ( 'TableName1', 'TableName2', .....)

OPEN Table_cursor;
FETCH NEXT FROM Table_cursor INTO @TableId, @TableName;
WHILE @@FETCH_STATUS = 0
BEGIN
	BEGIN TRY
		BEGIN TRAN
			BEGIN
				-- Script timeout in milliseconds
				SET LOCK_TIMEOUT 1000;
				SET @SlNo = @SlNo + 1;

				-- Add SYSROWVERSION index
				IF NOT EXISTS (SELECT TOP 1 1
					FROM sys.indexes i
					INNER JOIN sys.index_columns ic ON ic.index_id = i.index_id AND ic.object_id = i.object_id
					INNER JOIN sys.columns c ON c.object_id = ic.object_id AND c.column_id = ic.column_id
					INNER JOIN sys.tables t ON t.object_id = c.object_id
					INNER JOIN sys.schemas s ON s.schema_id = t.schema_id
					WHERE s.name = @SchemaName AND ic.index_column_id = 1 AND ic.is_included_column = 0 AND t.name = @TableName AND c.name = 'SYSROWVERSION'
					)
				BEGIN
					SET @SQLStmt = '
					CREATE NONCLUSTERED INDEX AIF_I_' + CAST(@TableId as nvarchar) + 'SQLROWVERSIONIDX
					ON ' + @SchemaName + '.' + @TableName + ' ([SYSROWVERSION] ASC)
					WITH (ONLINE = ON, OPTIMIZE_FOR_SEQUENTIAL_KEY = ON)
					ON [PRIMARY]
					';
					EXEC sp_executesql @SQLStmt;
				END

				-- Add RECID index
				IF NOT EXISTS (SELECT TOP 1 1
					FROM sys.indexes i
					INNER JOIN sys.index_columns ic ON ic.index_id = i.index_id AND ic.object_id = i.object_id
					INNER JOIN sys.columns c ON c.object_id = ic.object_id AND c.column_id = ic.column_id
					INNER JOIN sys.tables t ON t.object_id = c.object_id
					INNER JOIN sys.schemas s ON s.schema_id = t.schema_id
					WHERE s.name = @SchemaName AND ic.index_column_id = 1 AND ic.is_included_column = 0 AND t.name = @TableName AND c.name = 'RECID'
					)
				BEGIN
					SET @SQLStmt = '
					CREATE NONCLUSTERED INDEX AIF_I_' + CAST(@TableId as nvarchar) + 'RECIDDATASYNCIDX
					ON ' + @SchemaName + '.' + @TableName + ' ([RECID] ASC)
					WITH (ONLINE = ON, OPTIMIZE_FOR_SEQUENTIAL_KEY = ON)
					ON [PRIMARY]
					';
					EXEC sp_executesql @SQLStmt;
				END

				SET LOCK_TIMEOUT 0;
			END
		COMMIT TRAN
		print cast(@SlNo as nvarchar) + '. ' + @SchemaName + '.' + @TableName + '(' + cast(@TableId as nvarchar) + ') => succeeded'
	END TRY
	BEGIN CATCH
		print cast(@SlNo as nvarchar) + '. ' + @SchemaName + '.' + @TableName + '(' + cast(@TableId as nvarchar) + ') => SQL error[' + cast(ERROR_NUMBER() as nvarchar) + '] : ' + ERROR_MESSAGE()
		ROLLBACK TRAN
	END CATCH
	FETCH NEXT FROM Table_cursor INTO @TableId, @TableName;
END

CLOSE Table_cursor
DEALLOCATE Table_cursor

SET NOCOUNT ON;
print ''
print 'Put system out of Maintainance mode'
UPDATE SQLSYSTEMVARIABLES SET VALUE = 0 WHERE PARM = 'CONFIGURATIONMODE'
SET NOCOUNT OFF;

print ''
print 'Finished'
```

4. Perform an IISReset operation from the command line to restart the application server.  

## Add finance and operations tables in Azure Synapse Link

You can enable both finance and operations tables and finance and operations entities in Azure Synapse Link for Dataverse. This section is focused on finance and operations tables.

1. Sign in to Power Apps and select the environment you want.
2. On the left navigation pane, select **Azure Synapse Link**.
3. On the command bar of the **Synapse Link** page, select **+ New link to data lake**.
4. Select **Connect to your Azure Synapse Analytics workspace**, and then select the **Subscription**, **Resource group**, and **Workspace name**.
5. Select **Use Spark pool for processing**, and then select the precreated Spark pool and storage account.
6. Select **Next**.
7. Add the tables you want to export. You can choose finance and operations tables provided the [prerequisites](#prerequisites) are met.
8. Select **Advanced**, select **Show advanced configuration settings** and enter the time interval, in minutes, for how often the incremental updates should be captured.
9. Select **Save**. Tables selected are initialized and ready for reporting.

![Adding finance and operations tables in Azure Synapse Link](media/synapse_link_delta_3_loops.gif)

> [!NOTE]
>
> - Finance and operations apps tables are allowed only in Azure Synapse Link. Makers can't see them in the **Tables** area in Power Apps (make.powerapps.com).
> - You don't have to define finance and operations apps tables as virtual tables, and you don't have to enable change tracking for each table.
>
> To include finance and operations tables in Synapse Link, you must enable the [Delta lake feature](/power-apps/maker/data-platform/azure-synapse-link-delta-lake) in your Synapse Link profile. Finance and operations table selection isn't visible if your Synapse Link profile isn't configured for Delta lake.
>
> Delta lake conversion time interval determines how often table data is updated in delta format. For near real time updates, choose 15 minutes or one hour as the desired updated time internal. Choose daily time interval if near real-time updates aren't required. Delta conversion consumes compute resources from the Spark pool you have provided in the configuration of the Synapse Link profile. The lower the time interval, the more compute resources are consumed and you can incur more cost. Open the Spark pool in Azure portal to see the compute cost.
>
> In the event that the system ran into an error during initial sync or updates, you'll see an error icon and a pointer to trouble-shooting documents that can be used to diagnose and resolve the error.

### Known limitations with finance and operations tables

Currently, there are limitations with finance and operations tables and Azure Synapse Link. We're working to address these limitations. To learn more about the upcoming roadmap and stay in touch with the product team, join the [preview Viva Engage group](https://aka.ms/SynapseLinkforDynamics/).

- You must create a new Azure Synapse Link profile. You can't add finance and operations apps tables to existing Azure Synapse Link profiles.
- Don't see all tables? Up to 2,750 Microsoft provided finance and operations apps tables are already enabled in Azure Synapse Link with application version 10.0.38. If you have a previous version of finance and operations apps, not all required tables can be enabled by default. You can enable more tables yourself by extending table properties and enabling the change tracking feature. For more information about how to enable change tracking, see [Enable row version change tracking for tables](/dynamics365/fin-ops-core/dev-itpro/data-entities/rowversion-change-track#enable-row-version-change-tracking-for-tables).
- Don't see your custom tables? You must enable change tracking for them. More information: [Enable row version change tracking for tables](/dynamics365/fin-ops-core/dev-itpro/data-entities/rowversion-change-track#enable-row-version-change-tracking-for-tables). If you're using a cloud hosted environment (CHE), you must perform a database sync operation to reflect the changes.
- You can select a maximum of 1,000 tables in an Azure Synapse Link profile. To enable more tables, create another Azure Synapse Link profile.
- If the table selected contains data columns that are secured via **AOS Authorization**, those columns are ignored and the exported data doesn't contain the column. For example in a custom table named *CustTable*, the column *TaxLicenseNum* has the metadata property **AOS Authorization** set to **Yes**. This column is ignored when *CustTable* data is exported with Azure Synapse Link.
   > [!NOTE]
   > Update your finance and operations environment to these versions or later to enable AOS authorized fields:
   > - PU 63:7.0.7198.105
   > - PU 62:7.0.7120.159
   > 
   > With this update, AOS authorization fields are added to tables:
   >   - Incremental updates include this column.
   >   - Modified records show these columns and value.
   >    - Full refresh includes these fields and all values.
   >      
- [Table inheritance and derived tables](/dynamicsax-2012/developer/table-inheritance-overview) are concepts in finance and operations apps. When choosing a derived table from finance and operations apps, fields from the corresponding base table currently aren't included. You need to select the base table in addition to the derived table if you need access to these fields.
- If the table selected contains data columns that are of **Array** type, those columns are ignored and the exported data doesn't contain the column. For example, in a custom table named *WHSInventTable*, columns **FilterCode** and **FilterGroup** are of type array. These columns aren't exported with Azure Synapse Link.
- In case of finance and operations app tables that exhibit [valid time stamp behavior](/dynamicsax-2012/developer/valid-time-state-tables-and-date-effective-data), only the data rows that are currently valid are exported with Azure Synapse Link. For example, the **ExchangeRate** table contains both current and previous exchange rates. Only currently valid exchange rates are exported in Azure Synapse Link. As a workaround, until this issue is fixed, you can use a table such as ExchangeRateBIEntity.
- When a finance and operations table added to Azure Synapse Link is secured via [extensible data security policies](/dynamics365/fin-ops-core/dev-itpro/sysadmin/extensible-data-security-policies), the system might not export data. This issue is fixed in the latest application update.
  > [!NOTE]
  > Available updates to finance and operations tables with Azure Synapse Link for Dataverse:
  > - Version 10.0.37 (PU61) cumulative update 10.0.1725.175
  > - Version 10.0.38 (PU62) cumulative update 10.0.1777.135
  > - Version 10.0.39 (PU63) cumulative update update 10.0.1860.50
  >
  > You'll need to apply a quality build where the system applies a bypass for extensible data security policies for the Azure Synapse Link service.
  
- Finance and operations apps tables added to an Azure Synapse Link profile might be removed when a back-up is restored in Dataverse. You must add finance and operations tables into the profile after a database restore operation.
- When a finance and operations apps database is restored, tables added to an Azure Synapse Link profile need to be reinitialized. Before reinitializing finance and operations tables, you must also restore the Dataverse database. After restoring the database, you must add finance and operations tables into the profile.
- Finance and operations apps tables included in an Azure Synapse Link profile can't be migrated to a different environment using the import and export profile feature in Azure Synapse Link.
- Special fields such as `TimeZoneID` (TZID), binary fields in finance and operations tables aren't enabled in Azure SynapseL Link.
- Staging and temporary table types in finance and operations apps aren't allowed in Azure Synapse Link.
- **Access finance and operations tables via Synapse query** and  **Access finance and operations tables via Microsoft Fabric** features aren't available in the China region.

## Access incremental data changes from finance and operations

To load incremental data changes from finance and operations into your own downstream data warehouse, create an Azure Synapse Link profile that provides only incremental data. Azure Synapse Link provides an initial export of all data rows and then provides you with access to data that changed periodically. The data is provided in CSV files stored in time stamped folders and you can easily consume the data using Azure Data factory or other data tools. More information:  [Azure Synapse Link - incremental update](azure-synapse-link-incremental.md)

To create an Azure Synapse Link profile with incremental data:

1. Sign in to Power Apps and select the environment you want.
2. On the left navigation pane, select **Azure Synapse Link**.
3. On the **Azure Synapse Link for Dataverse** page, select **+ New link** on the command bar.
4. Select **Subscription**, **Resource group** and a **Storage account**. You don't need to provide a Synapse workspace or a Spark pool.
4. Select **Next**. The option to choose tables appears.
5. Select **Advanced**, select **Show advanced configuration settings**, and then enable the option **Enable incremental update folder structure**
6. In the **Time interval** field, choose the desired frequency for reading incremental data. Using this frequency, the system partitions data into time stamped folders such that you can read the data without being impacted by ongoing write operations.  
7. Select the Dataverse tables you want. You can also select finance and operations tables. The options **Append only** and **Partition** available at a table level are ignored. Data files are always appended and data is partitioned yearly.
8. Select **Save**. Tables selected are initialized and you see incremental data in the storage account.

![Adding incremental data changes from finance and operations tables.](media/Synapse_link_Incremental_3_loops.gif)

> [!NOTE]
>
> If you are upgrading from the export to data lake feature, enabling the incremental data changes option provides similar change data as the [Change feeds feature](/dynamics365/fin-ops-core/dev-itpro/data-entities/azure-data-lake-change-feeds)
>
> We recommend that you create separate Azure Synapse Link profiles for incremental data and tables for ease of management.
>
> When you choose tables and enable incremental data changes, the row count shown in the Azure Synapse Link details page for each table reflects the total number of changes, not the number of records in the table.
>
> The finance and operations table limitations are also applicable to incremental data from tables. More information: [Known limitations with finance and operations tables](#known-limitations-with-finance-and-operations-tables)

## Working with data and metadata  

Enumerated fields are coded data fields in finance and operations apps. For example, the **AssetTrans** table contains a field called **TransType**, which is an **Enumerated** field. Table fields contain numeric codes like 110, 120, or 131, which represent detailed descriptions like "Depreciation", "Lease" or "Major repairs". You can access these detailed descriptions by using the **GlobalOptionsMetadata** table that's automatically exported when you choose a table that contains enumerated fields. Enumerated fields are also called choice labels or, formerly, option sets. More information: [Choice labels](/power-apps/maker/data-platform/azure-synapse-link-choice-labels)

If there are metadata changes to finance and operations tables, for example, a new field is added to a table, and the data exported in Azure Synapse Link reflects the latest metadata inclusive of the change. More information: [Azure Synapse Link FAQ](/power-apps/maker/data-platform/export-data-lake-faq#what-happens-when-i-add-a-column). If you're using Azure Synapse Link to query the data, you see the updated metadata reflected in Azure Synapse Link. If you consume incremental data changes, you can locate updated metadata within the incremental data folder with the latest date stamp. More information: [Incremental folder structure](/power-apps/maker/data-platform/azure-synapse-incremental-updates#view-incremental-folder-at-microsoft-azure-storage)

## Enable finance and operations data entities in Azure Synapse Link

You can enable both finance and operations entities and finance and operations apps tables in Azure Synapse Link for Dataverse. This section is focused on finance and operations data entities.

The process of enabling finance and operations entities has the following steps. Each step is explained in the following subsections.

1. [Enable finance and operations virtual entities in the Power Apps maker portal](#enable-finance-and-operations-virtual-entities-in-power-apps). This step lets you use finance and operations entities in Power Apps (make.powerapps.com) to build apps. You can also use them with Azure Synapse Link.
2. [Enable row version change tracking for Entities](/dynamics365/fin-ops-core/dev-itpro/data-entities/rowversion-change-track#enable-row-version-change-tracking-for-data-entities). You must complete this step to enable Azure Synapse Link to use finance and operations entities.

After you complete both steps, you can select finance and operations entities in Azure Synapse Link under **Dataverse tables**. To create Azure Synapse Link for Dataverse in Delta Lake format, follow the steps in [Export Dataverse data in Delta Lake format](/power-apps/maker/data-platform/azure-synapse-link-delta-lake).

> [!NOTE]
> Finance and operations entities start with the prefix **mserp\_**.

### Enable finance and operations virtual entities in Power Apps

You must enable finance and operations entities as [virtual tables](create-edit-virtual-entities.md) in Dataverse. Makers can then use the chosen finance and operations entities to build apps, and the entities can also be used with Azure Synapse Link.

To enable finance and operations entities, follow the steps in [Enable Microsoft Dataverse virtual entities](/dynamics365/fin-ops-core/dev-itpro/power-platform/enable-virtual-entities).

> [!TIP]
> To validate Azure Synapse Link features, use a few of the sample entities from the following list. They appear under the **Dataverse tables** section in Azure Synapse Link.
>
> - **MainAccountBiEntity** – This entity contains a list of ledger accounts.
> - **ExchangeRateBiEntity** – This entity contains exchange rates in the system.
> - **InventTableBiEntity** – This entity contains a list of inventory items.

### Enable change tracking for finance and operations entities

When you enable change tracking for finance and operations entities, they appear under Dataverse tables in Azure Synapse Link. Finance and operations entities start with the prefix **mserp\_**.

To enable change tracking, follow these steps.

1. In Power Apps, select **Tables** on the left navigation pane, and then select the table you want.
1. Select **Properties** >  **Advanced options**.
1. Select the **Track changes** option, and then select **Save**. If the option is unavailable, see known limitations below. 

### Known limitations with finance and operations entities

Currently, there are several limitations with finance and operations entities and Azure Synapse Link. To learn more about the upcoming roadmap and stay in touch with product team, join the [preview Viva Engage group aka.ms/SynapseLinkforDynamics](https://aka.ms/SynapseLinkforDynamics/).

- Enabling change tracking might fail with the error message "chosen entity doesn't pass the validation rules..." or the **Track changes** checkbox might be  disabled for some tables that are virtual tables. Currently, change tracking can't be enabled for all finance and operations entities. The **Track changes** checkbox is unavailable for entities created in finance and operations in the past for data migration.

   > [!NOTE]
   > For a list of finance and operations entities that pass validation rules, run the **Data entity row version change tracking validation report** available in finance and operations apps at path **System administration/Setup/Row version change tracking/Data entity row version change tracking validation report.** This reports shows entities that pass and fail validation rules.
   >
   > For more information about entity validation rules and how you can fix them, go to [Enable row version change tracking for data entities](/dynamics365/fin-ops-core/dev-itpro/data-entities/rowversion-change-track#enable-row-version-change-tracking-for-data-entities). You might need developer assistance to complete the steps.
   > 
   > If the chosen entity is unavailable because of the change tracking limitation, you might be able to choose the tables that comprise the data from that entity. You can use [EntityUtil solution provided by the FastTrack team](https://github.com/microsoft/Dynamics-365-FastTrack-Implementation-Assets/tree/master/Analytics/DataverseLink/DataIntegration/EntityUtil) to create entity shapes using tables.
   > 

- In case of a database restore operation in Dataverse, finance and operations entities enabled in Azure Synapse Link are removed. To re-enable entities, you need to re-enable corresponding virtual tables for all selected entities, re-enable change tracking, and reselect the tables in Azure Synapse Link. 
