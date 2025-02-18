---
title: .alter database ingestion batching policy command - Azure Data Explorer
description: This article describes the .alter database ingestion batching policy command in Azure Data Explorer.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: yonil
ms.service: data-explorer
ms.topic: reference
ms.date: 09/26/2021
---
# .alter database ingestion batching policy

Change the database ingestion batching policy. The [ingestionBatching policy](batchingpolicy.md) is a policy object that determines when data aggregation should stop during data ingestion according to specified settings.

## Syntax

* `.alter` `database` *DatabaseName* `policy` `ingestionbatching`

## Example

The following example changes the database IngestionBatching policy:

```kusto
// Change the IngestionBatching policy on database `MyDatabase` to batch ingress data by 300MB 
.alter database MyDatabase policy ingestionbatching @'{"MaximumRawDataSizeMB": 300}'
```

The following example changes the database IngestionBatching policy, with all the batching policy triggers:

```kusto
// Change the IngestionBatching policy on database `MyDatabase` to batch ingress data by 300MB 
.alter database MyDatabase policy ingestionbatching @'{"MaximumBatchingTimeSpan":"00:01:00", "MaximumNumberOfItems": 20, "MaximumRawDataSizeMB": 300}'
```
