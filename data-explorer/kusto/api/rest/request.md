---
title: Query/management HTTP request - Azure Data Explorer | Microsoft Docs
description: This article describes Query/management HTTP request in Azure Data Explorer.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 01/27/2020
---
# Query/management HTTP request

## Request verb and resource

|Action    |HTTP verb|HTTP resource   |
|----------|---------|----------------|
|Query     |GET      |`/v1/rest/query`|
|Query     |POST     |`/v1/rest/query`|
|Query v2  |GET      |`/v2/rest/query`|
|Query v2  |POST     |`/v2/rest/query`|
|Management|POST     |`/v1/rest/mgmt` |

For example, to send a control command ("management") to a service endpoint,
use the following request line:

```
POST https://help.kusto.windows.net/v1/rest/mgmt HTTP/1.1
```

(See below for the request headers and body to include.)

## Request headers

The following table contains the common headers used to perform query and management
operations.

|Standard header  |Description                                                                                                                    |
|-----------------|-------------------------------------------------------------------------------------------------------------------------------|
|`Accept`         |**Required**. Set this to `application/json`.                                                                                  |
|`Accept-Encoding`|**Optional**. Supported encodings are `gzip` and `deflate`.                                                                    |
|`Authorization`  |**Required**. See [authentication](./authentication.md).                                                                       |
|`Connection`     |**Optional**. It is recommended that `Keep-Alive` be enabled.                                                                  |
|`Content-Length` |**Optional**. It is recommended that the request body length be specified when known.                                          |
|`Content-Type`   |**Required**. Set this to `application/json` with `charset=utf-8`.                                                             |
|`Expect`         |**Optional**. Can be set to `100-Continue`.                                                                                    |
|`Host`           |**Required**. Set this to the fully-qualified domain name that the request was sent to (for example, `help.kusto.windows.net`).|

The following table contains the common custom headers used when performing query
and management operations. Unless indicated otherwise, these headers are used
for telemetry purposes only, and have no functionality impact.

All headers are **optional**. It is **strongly-recommended** that you specify the
`x-ms-client-request-id` custom header. In some scenarios (for example,
cancelling a running query) this header is **mandatory** because it is used to identify
the request.


|Custom header           |Description                                                                                               |
|------------------------|----------------------------------------------------------------------------------------------------------|
|`x-ms-app`              |The (friendly) name of the application making the request.                                                |
|`x-ms-user`             |The (friendly) name of the user making the request.                                                       |
|`x-ms-user-id`          |Same as `x-ms-user`.                                                                                      |
|`x-ms-client-request-id`|A unique identifier for the request.                                                                      |
|`x-ms-client-version`   |The (friendly) version identifier for the client making the request.                                      |
|`x-ms-readonly`         |If specified, forces the request to run in read-only mode (preventing it from making long-lasting changes).|

## Request parameters

The following parameters can be passed in the request. They are encoded in the
request as query parameters or as part of the body, depending on whether GET or
POST is used.

* `csl`: This is a **mandatory** parameter. It holds the text of the query
  or control command to execute.

* `db`: This is an **optional** parameter for some control commands, and **mandatory**
  parameter for other control commands and all queries. It provides the name
  of the "database in scope" - the database that is the target of the
  query or control command.

* `properties`: This is an **optional** parameter. It provides various
  client request properties that modify how the request is processed and its
  results. For a complete description, see [client request properties](../netfx/request-properties.md).

## GET query parameters

When GET is used, the query parameters of the request specify the
request parameters noted above.

## Body

When POST is used, the body of the request is a single JSON document
encoded in UTF-8 that provides the values of the request parameters noted
above.

## Examples

The following example shows the HTTP POST request for a query:

```txt
POST https://help.kusto.windows.net/v2/rest/query HTTP/1.1
```

Request headers:

```txt
Accept: application/json
Authorization: Bearer ...AzureActiveDirectoryAccessToken...
Accept-Encoding: deflate
Content-Type: application/json; charset=utf-8
Host: help.kusto.windows.net
x-ms-client-request-id: MyApp.Query;e9f884e4-90f0-404a-8e8b-01d883023bf1
x-ms-user-id: EARTH\davidbg
x-ms-app: MyApp
```

Request body (newlines introduced for clarity; they are not needed):

```json
{
  "db":"Samples",
  "csl":"print Test=\"Hello, World!\"",
  "properties":"{\"Options\":{\"queryconsistency\":\"strongconsistency\"},\"Parameters\":{},\"ClientRequestId\":\"MyApp.Query;e9f884e4-90f0-404a-8e8b-01d883023bf1\"}"
}
```

The following example shows how to create a request that sends the query above using [curl](https://curl.haxx.se/):

1. Obtain a token for authentication.

* replace `AAD_TENANT_NAME_OR_ID`, `AAD_APPLICATION_ID` and `AAD_APPLICATION_KEY` with the relevant values, after having set up [AAD application authentication](../../management/access-control/how-to-provision-aad-app.md)

```
curl "https://login.microsoftonline.com/AAD_TENANT_NAME_OR_ID/oauth2/token" \
  -F "grant_type=client_credentials" \
  -F "resource=https://help.kusto.windows.net" \
  -F "client_id=AAD_APPLICATION_ID" \
  -F "client_secret=AAD_APPLICATION_KEY"
```

This will provide you with the bearer token:

```
{
  "token_type": "Bearer",
  "expires_in": "3599",
  "ext_expires_in":"3599", 
  "expires_on":"1578439805",
  "not_before":"1578435905",
  "resource":"https://help.kusto.windows.net",
  "access_token":"eyJ0...uXOQ"
}
```

2. Use the bearer token in your request to the query endpoint:

```
curl -d '{"db":"Samples","csl":"print Test=\"Hello, World!\"","properties":"{\"Options\":{\"queryconsistency\":\"strongconsistency\"}}"}"' \
-H "Accept: application/json" \
-H "Authorization: Bearer eyJ0...uXOQ" \
-H "Content-Type: application/json; charset=utf-8" \
-H "Host: help.kusto.windows.net" \
-H "x-ms-client-request-id: MyApp.Query;e9f884e4-90f0-404a-8e8b-01d883023bf1" \
-H "x-ms-user-id: EARTH\davidbg" \
-H "x-ms-app: MyApp" \
-X POST https://help.kusto.windows.net/v2/rest/query
```

3. Read the response according to [this specification](response.md).