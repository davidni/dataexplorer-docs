---
title: Use wizard for one-time ingestion of historical data with LightIngest (preview)
description: Learn about how to auto-generate an ingest command for LightIngest, a command-line utility for ad-hoc data ingestion into Azure Data Explorer
author: orspod
ms.author: orspodek
ms.reviewer: tzgitlin
ms.service: data-explorer
ms.topic: how-to
ms.date: 07/27/2021
---
# Use wizard for one-time ingestion of historical data with LightIngest (preview)

LightIngest is a command-line utility for ad-hoc data ingestion into Azure Data Explorer. 
To learn more about LightIngest, see [Use LightIngest to ingest data into Azure Data Explorer](lightingest.md).

LightIngest can be particularly useful to load historical data from an existing storage system to Azure Data Explorer. While you can build your own command using the list of [Command-line arguments](lightingest.md#command-line-arguments), this article shows you how to auto-generate this command through an ingestion wizard. In addition to creating the command, you can use this process to create a new table, and create schema mapping. This tool infers schema mapping from your data set.

This article shows you how to create a new table, create schema mapping, and generate a LightIngest command for one-time ingestion using the LightIngest tool.

## Prerequisites

* An Azure subscription. Create a [free Azure account](https://azure.microsoft.com/free/).
* [A cluster and database](create-cluster-database-portal.md).
* [A storage account](/azure/storage/common/storage-quickstart-create-account?tabs=azure-portal).
* LightIngest - download it as part of the [Microsoft.Azure.Kusto.Tools NuGet package](https://www.nuget.org/packages/Microsoft.Azure.Kusto.Tools/). For installation instructions, see [Install LightIngest](lightingest.md#install-lightingest).

## Access the wizard

The wizard can be accessed either from the **Data** tab, or from the **Query** tab of the [Azure Data Explorer WebUI](https://dataexplorer.azure.com/).

1. In the **Data** tab, select **Ingest new data**>**Ingest**

    :::image type="content" source="media/generate-lightingest-command/data-tab.png" alt-text="Screenshot of the Data tab of the Azure Data Explorer WebUI with red boxes around the fields to follow.":::

1. In the **Query tab**, right-click a *database* and select **Ingest new data**.

   :::image type="content" source="media/one-click-ingestion-new-table/one-click-ingestion-in-web-ui.png" alt-text="Ingest new data.":::

In the **Ingest new data** window, the **Destination** tab is selected. The **Cluster** and **Database** fields are automatically populated.

## Destination tab

1. In **Table**, check either **Existing table** or **Create new table**. When creating a new table, enter a name for the new table. You can use alphanumeric, hyphens, and underscores. Special characters aren't supported.

    > [!NOTE]
    > Table names must be between 1 and 1024 characters.

    :::image type="content" source="media/generate-lightingest-command/ingest-new-data.png" alt-text="Screenshot of Azure Data Explorer WebUI with Data tab selected on the lefthand menu, and Ingest new data dialog open to the right.":::

1. Select **Next: Source**

## Source tab

:::image type="content" source="media/generate-lightingest-command/source-tab-lightingest.png" alt-text="Screenshot of Source tab in Ingest new table window. ":::

  1. Under **Source type**, select **From blob container** (blob container, ADLS Gen2 container). 
  1. Select **Ingestion type**>**Historical data**.
  1. You can either **Add URL** manually by copying the Account Key/SAS URL to source, or **Select container** from your storage account.
      > [!NOTE]
      > The SAS URL can be created [manually](/azure/vs-azure-tools-storage-explorer-blobs#get-the-sas-for-a-blob-container) or [automatically](kusto/api/connection-strings/storage.md).
  1. When selecting from your storage account, select your **Storage subscription**, **Storage account**, and **Container** from the dropdown menus.

:::image type="content" source="media/generate-lightingest-command/source-tab-container-from-subscription.png" alt-text="Screenshot of dialog box for selecting container from storage subscription and account.":::

## Filter data

If you want to, filter the data to ingest only files in a specific folder path or with a particular file extension.

:::image type="content" source="media/generate-lightingest-command/filter-data-lightingest.png" alt-text="Screenshot of filtering data in the source tab of the Ingest new data screen.":::

The system will select one of the files at random and the schema will be generated based on that  **Schema defining file**. You can select a different file.

Select **Next: Schema** to view and edit your table column configuration. 

## Edit the schema

In the **Schema** tab:

1. By looking at the name of the source, the service automatically identifies if it is compressed or not. Confirm that the **Compression type** is correct.
1. Confirm the format selected in **Data format**:

     In this case, the data format is **CSV**

1. On tabular data, you can select the check box **Ignore the first record** to ignore the heading row of the file.
1. In the **Mapping name** field, enter a mapping name. You can use alphanumeric characters and underscores. Spaces, special characters, and hyphens aren't supported.

    When using an existing table, you can **Keep current table schema** if the table schema matches the selected format.

:::image type="content" source="media/generate-lightingest-command/schema-tab-lightingest.png" alt-text="Screenshot of the schema tab in the Ingest new data dialog in Azure Data Explorer WebUI.":::

### Edit the table

When ingesting to a new table, alter various aspects of the table when creating the table.

[!INCLUDE [data-explorer-one-click-column-table](includes/data-explorer-one-click-column-table.md)]

> [!NOTE]
> For tabular formats, you can’t map a column twice. To map to an existing column, first delete the new column.

Select **Next: Summary** to generate the LightIngest command.

## Generate the LightIngest command

In the **Data ingestion completed** window, all three steps will be marked with green check marks.

Copy the generated LightIngest command by clicking on the **copy** icon to the top right of the command box.

In the tiles below the ingestion progress, you can download the LightIngest tool.

:::image type="content" source="media/generate-lightingest-command/summary-tab-copy-command.png" alt-text="Screenshot of Summary tab with command generated. You can copy the command using the copy icon above the generated command box." lightbox="media/generate-lightingest-command/summary-tab-copy-command.png":::

To complete the ingestion process, you must [run LightIngest](lightingest.md#run-lightingest) using this copied command.

## Next steps

* [Run LightIngest](lightingest.md#run-lightingest)
* [Query data in Azure Data Explorer](web-query-data.md)
