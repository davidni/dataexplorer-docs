---
title: Data soft-delete - Azure Data Explorer
description: This article describes Data soft-delete in Azure Data Explorer.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: slneimer
ms.service: data-explorer
ms.topic: reference
ms.date: 11/21/2021
---
# Data soft-delete (Public Preview)

As a data platform, Azure Data Explorer supports the ability to delete individual records. This is commonly achieved using one of the following methods:

* To delete records for compliance purposes, use [.purge](./data-purge.md) - This method deletes all relevant data from storage artifacts and, once completed, there is no way to recover the deleted data.
* To delete records for any other purpose, use `.delete` as described in this topic - This marks records as deleted but does not necessarily delete the data from storage artifacts. This deletion method is much faster than purge.

> [!WARNING]
> The `.delete` command is designed for deleting **small amounts** of data and is intended to be used **infrequently**. Frequent use of the command may have a significant impact on the performance of your cluster.
>
> To delete large amounts of data, consider using other methods such as [dropping extents](../management/drop-extents.md).

## Use cases

This deletion method should only be used for the unplanned deletion of individual records. For example, if you discover that an IoT device is reporting corrupt telemetry for some time, you should consider using this method to delete the corrupt data.

However, if you want to do a periodic cleanup by regularly deleting duplicate records, or deleting old records per entity, then you should use [Materialized Views](../management/materialized-views/materialized-view-overview.md) instead.

## Deletion process

The soft delete process is performed using the following steps:

1. **Run predicate query**: The table is scanned to identify data extents that contain records to be deleted. The extents identified are those with one or more records returned by the predicate query.
1. **Extents replacement**: The identified extents are replaced with new extents that point to the original data blobs, and also have a new hidden column of type `bool` that indicates per record whether it was deleted or not. Once completed, if no new data is ingested, the predicate query will not return any records if run again.

## Limitations and considerations

* The deletion process is final and irreversible. It isn't possible to undo this process or recover data that has been deleted, even though the storage artifacts are not necessarily deleted following the operation.

* Soft-delete is only available on clusters running Engine V3.

* Soft-delete is only supported for native tables and is not supported for external tables or materialized views.

* Before running soft-delete, verify the predicate by running a query and checking that the results match the expected outcome. You can also run the command in `whatif` mode, that returns the number of records that are expected to be deleted.

* Do not run multiple parallel soft-delete operations on the same table, as this may result in failures of some or all the commands. However, it's possible to run multiple parallel soft-delete operations on different tables.

* Do not run soft-delete and purge commands on the same table in parallel. First wait for one command to complete and only then run the other command.

* Soft-delete is executed against your engine endpoint: `https://[YourClusterName].[region].kusto.windows.net`. The command requires [database admin](../management/access-control/role-based-authorization.md) permissions on the relevant database.

* Soft-delete can affect materialized views based on a source table in which records are deleted. This can happen because every [materialization cycle](../management/materialized-views/materialized-view-overview.md#how-materialized-views-work) adds newly ingested data to the materialized part from the previous cycle. Therefore, if the command deletes newly ingested records before a new cycle begins, those records will not be added to the materialized view. Otherwise, deleting records won't affect the materialized view.

## Deletion performance

The main considerations that can impact the [deletion process](#deletion-process) performance are:

* **Run predicate query**: The performance of this step is very similar to the performance of the predicate itself. It might be slightly faster or slower depending on the predicate, but the difference is expected to be insignificant.
* **Extents replacement**: The performance of this step depends on the following:
    * Record distribution across the data extents in the cluster
    * The number of nodes in the cluster

Unlike `.purge`, the `.delete` command does not reingest the data. It just marks records that are returned by the predicate query as deleted and is therefore much faster.

## Query performance after deletion


Query performance is not expected to noticeably change following the deletion of records.

Performance degradation is not expected because the filter that is automatically added on all queries that filter out records that were deleted is very efficient.

However, query performance is also not guaranteed to improve. While this may happen for some types of queries, it may not happen for some others. In order to improve query performance, extents in which the majority of the records are deleted are periodically compacted by replacing them with new extents that only contain the records that haven't been deleted.

## Impact on COGS (cost of goods sold)

In most cases, the deletion of records won't result in a change of COGS.

* There will be no decrease, because no records are actually deleted. Records are only marked as deleted using a hidden column of type `bool`, the size of which is negligible.
* In most cases, there will be no increase because the `.delete` operation does not require the provisioning of extra resources.
* In some cases, extents in which the majority of the records are deleted are periodically compacted by replacing them with new extents that only contain the records that haven't been deleted. This causes the deletion of the old storage artifacts that contain a large number of deleted records. The new extents are smaller and therefore consume less space in both the Storage account and in the hot cache. However, in most cases, the effect of this on COGS is negligible.

## Triggering the deletion process

### Syntax

```kusto
`.delete` [`async`] `table` *TableName* `records <|` *Predicate*
```

### Arguments

|Name|Type|Required|Description|
|--|--|--|--|
|*async*|string||If specified, indicates that the command runs in asynchronous mode.|
|*TableName*|string|&check;|The name of the table from which to delete records.|
|*Predicate*|string|&check;|The predicate that returns records to delete. Specified as a query.|

> [!NOTE]
> The following restrictions apply to the *Predicate*:
>
> * The predicate should have at least one `where` operator.
> * The predicate can only use the following operators: `extend`, `where` and `project`.
> * The predicate can't reference other tables, nor use `externaldata`.

#### Example

To delete all the records which contain data of a given user:

```kusto
.delete table MyTable records <| MyTable | where UserId == 'X'
```

> [!NOTE]
>
> To determine the number of records that would be deleted by the operation without actually deleting them, check the value in the RecordsMatchPredicate column when running the command in `whatif` mode:
>
> ```kusto
> .delete table MyTable records with (whatif=true) <| MyTable | where UserId == 'X'
> ```

### Output

The output of the command contains information about which extents were replaced.
