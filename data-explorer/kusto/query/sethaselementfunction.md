---
title: set_has_element() - Azure Data Explorer | Microsoft Docs
description: This article describes set_has_element() in Azure Data Explorer.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 01/23/2020
---
# set_has_element()

Determines whether the specified set contains the specified element.

**Syntax**

`set_has_element(`*array*,*value*`)`

**Arguments**

* *array*: Input array to search.
* *value*: Value to search for. The value should be of type `long`, `integer`, `double`, `datetime`, `timespan`, `decimal`, `string`, or `guid`.

**Returns**

True or false depending on the whether the value exists in the array.

**Example**

```kusto
print arr=dynamic(["this", "is", "an", "example"]) 
| project Result=set_has_element(arr, "example")
```

|Result|
|---|
|1|

**See also**

If you are also interested in the position at which the value exists in the array,
you can use [array_index_of(arr, value)](arrayindexoffunction.md). Both functions are the same, performance-wise.