---
title: .alter-merge database sharding policy command - Azure Data Explorer
description: This article describes the .alter-merge database sharding policy command in Azure Data Explorer.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: yonil
ms.service: data-explorer
ms.topic: reference
ms.date: 10/10/2021
---
# .alter-merge database sharding policy

Change the database sharding policy. Use the [sharding policy](../management/shardingpolicy.md) to manage data sharding for databases and tables.  

The sharding policy defines if and how [Extents (data shards)](../management/extents-overview.md) in the Azure Data Explorer cluster should be sealed. When a database is created, it contains the default data sharding policy. This policy is inherited by all tables created in the database (unless the policy is explicitly overridden at the table level).

## Syntax

`.alter-merge` `database` *DatabaseName* `policy` `sharding`

## Arguments

*DatabaseName* - Specify the name of the database. 

## Returns

Returns a JSON representation of the policy.

## Example

The following command changes a single property for the sharding policy for a database:

```kusto
.alter-merge database MyDatabase policy sharding 
@'{ "MaxExtentSizeInMb": 1024}'
```