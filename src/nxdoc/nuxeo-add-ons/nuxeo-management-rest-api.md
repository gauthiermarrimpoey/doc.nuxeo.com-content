---
title: 'Nuxeo Management Rest API'
review:
    comment: ''
    date: '2019-07-26 16:26'
    status: ok
labels:
    - lts2019-ok    
toc: true
confluence:
    ajs-parent-page-id: '16089349'
    ajs-parent-page-title: Nuxeo Add-Ons
    ajs-space-key: NXDOC
    ajs-space-name: Nuxeo Platform Developer Documentation
    canonical: Nuxeo+Management+Rest+API
    canonical_source: 'https://doc.nuxeo.com/display/NXDOC/Nuxeo+Management+Rest+API'
    page_id: ''
    shortlink: ''
    shortlink_source: ''
    source_link: /display/NXDOC/Nuxeo+Management+Rest+API
tree_item_index: 300
history:
---

The Nuxeo Management Rest API is a set of endpoints allowing the management of the Nuxeo Platform.

This page will explain these endpoints and there purpose.

## Installation

{{{multiexcerpt 'MP-installation-easy' page='Generic Multi-Excerpts'}}}

## Management Rest API Endpoints

In this section we will present the different endpoints and we will show the way to call each one of them using `curl`.

### Elasticsearch

Nuxeo plateform can be configured to use and communicate with [Elasticsearch]({{page page='elasticsearch-setup'}}).

#### Indexing Endpoints

The indexing endpoint is defined as below:

**Url**
```
POST http://NUXEO_SERVER/nuxeo/site/management/elasticsearch/{repositoryName}/reindex
```

`{repositoryName}` should be replaced by the repository name that we want to index.

**Response**
If successful, returns a json reprsentation of [Bulk Action Status]({{page page='bulk-action-framework'}}#bulk-rest-api).

**Status Codes**
- 200 *OK* - Success.

**Indexing a full Nuxeo [repository]({{page page='elasticsearch-setup'}}#configuration-for-multi-repositories)**

The example below will index the `default` repository:

```bash
curl -X POST -u Administrator:Administrator \
-H 'Content-Type: application/json' \
http://NUXEO_SERVER/nuxeo/site/management/elasticsearch/default/reindex
```

```bash
{
   "entity-type": "bulkStatus",
   "commandId": "d3bc0a81-b061-447f-b307-c3db78b5f457",
   "state": "SCHEDULED",
   "processed": 0,
   "error": false,
   "errorCount": 0,
   "total": 0,
   "action": "index",
   "username": "system",
   "submitted": "2019-07-26T14:12:29.224Z",
   "scrollStart": null,
   "scrollEnd": null,
   "processingStart": null,
   "processingEnd": null,
   "completed": null,
   "processingMillis": 0
}
```

**Indexing a set of documents on a Nuxeo repository**

If we want to restrict the indexing on a set of desired documents, then we can add a [NXQL]({{page page='nxql'}}) query paramter as below

**Url**
```
POST http://NUXEO_SERVER/nuxeo/site/management/elasticsearch/{repositoryName}/reindex?query=nxql
```

`nxql` will be the NXQL query

**Response**
If successful, returns a json reprsentation of [Bulk Action Status]({{page page='bulk-action-framework'}}#bulk-rest-api).

**Status Codes**
- 200 *OK* - Success.

```bash
curl -X POST -u Administrator:Administrator \
-H 'Content-Type: application/json' \
http://NUXEO_SERVER/nuxeo/site/management/elasticsearch/default/reindex?query=select+%2A+from+document+where+dc%3Atitle+like+%27My+Title%25%27%27
```

```bash
{  
   "entity-type":"bulkStatus",
   "commandId":"90037d73-ed19-48cc-a4ad-09f3999cf014",
   "state":"SCHEDULED",
   "processed":0,
   "error":false,
   "errorCount":0,
   "total":0,
   "action":"index",
   "username":"system",
   "submitted":"2019-07-26T09:26:08.761Z",
   "scrollStart":null,
   "scrollEnd":null,
   "processingStart":null,
   "processingEnd":null,
   "completed":null,
   "processingMillis":0
  }
```

In this example the `NXQL` query is `select * from document where dc:title like 'My Title%'`.

**Indexing a specific document and his children**

We can index a given document and his children recursively for a given repository, in this case the URL endpoint will follow:

**Url**
```
POST http://NUXEO_SERVER/nuxeo/site/management/elasticsearch/{repositoryName}/{documentId}/reindex
```

`documentId` will be replaced by the identifier of the document

**Response**
If successful, returns a json reprsentation of [Bulk Action Status]({{page page='bulk-action-framework'}}#bulk-rest-api).

**Status Codes**
- 200 *OK* - Success.


for our example let's assume that on our Nuxeo plateform we had defined a root document (for example a `Workspace` document type that contains many `Files`) as below:

```bash
<document repository="default" id="1fa9d3fa-04cc-4956-bc6c-8317b803e131">
<system>
    <type>WorkspaceRoot</type>
    <path>default-domain/workspaces</path>
    <lifecycle-state>project</lifecycle-state>
    <lifecycle-policy>default</lifecycle-policy>
    ...
```

The endpoint call will be

```bash
curl -X POST -u Administrator:Administrator \
-H 'Content-Type: application/json' \
http://NUXEO_SERVER/nuxeo/site/management/elasticsearch/default/1fa9d3fa-04cc-4956-bc6c-8317b803e131/reindex
```

```bash
{  
   "entity-type":"bulkStatus",
   "commandId":"f37503d3-51b2-462e-8f1e-fa240fafa861",
   "state":"SCHEDULED",
   "processed":0,
   "error":false,
   "errorCount":0,
   "total":0,
   "action":"index",
   "username":"system",
   "submitted":"2019-07-26T09:45:09.874Z",
   "scrollStart":null,
   "scrollEnd":null,
   "processingStart":null,
   "processingEnd":null,
   "completed":null,
   "processingMillis":0
 }
```

The `1fa9d3fa-04cc-4956-bc6c-8317b803e131` represent the identifier of our document root id.

All indexing endpoints that was defined above will use the [Bulk Action Framework]({{page page='bulk-action-framework'}}).

#### Flush Endpoint

We can flush a given repository as below:

**Url**
```
POST http://NUXEO_SERVER/nuxeo/site/management/elasticsearch/{repositoryName}/flush
```

**Status Codes**
- 204 *No Content* - Success.

The URL endpoint will follow: .

As example we flush the `default` repository

```bash
curl -X POST -u Administrator:Administrator \
-H 'Content-Type: application/json' \
http://NUXEO_SERVER/nuxeo/site/management/elasticsearch/default/flush
```

#### Optimize Endpoint

We can optimize a given repository as below:

**Url**
```
POST http://NUXEO_SERVER/nuxeo/site/management/elasticsearch/{repositoryName}/optimize
```

**Status Codes**
- 204 *No Content* - Success.

The URL endpoint will follow: .

As example we optimize the `default` repository

```bash
curl -X POST -u Administrator:Administrator \
-H 'Content-Type: application/json' \
http://NUXEO_SERVER/nuxeo/site/management/elasticsearch/default/optimize
```

### Renditions

In the Nuxeo Platform, [Renditions]({{page page='renditions'}}) are used for exports. this section will show the different endpoints to recompute Pictures / Thumbnails of a set of documents.

#### Recompute a rendition on Pictures

To recompute picture views for the documents matching a given query, you call the endpoint as below:

**Url**
```
POST http://NUXEO_SERVER/nuxeo/site/management/renditions/pictures/recompute
```

**Response**
If successful, returns a json reprsentation of [Bulk Action Status]({{page page='bulk-action-framework'}}#bulk-rest-api).

**Status Codes**
- 200 *OK* - Success.

```bash
curl -u Administrator:Administrator \
-H 'Content-Type: application/json' \
-X POST http://NUXEO_SERVER/nuxeo/site/management/renditions/pictures/recompute \
-d 'query=SELECT * FROM Document WHERE ecm:mixinType = "Picture"'
```

```bash
{
   "entity-type": "bulkStatus",
   "commandId": "93baf453-5d34-4bb9-9370-54ec4ffe78b2",
   "state": "SCHEDULED",
   "processed": 0,
   "error": false,
   "errorCount": 0,
   "total": 0,
   "action": "recomputeViews",
   "username": "system",
   "submitted": "2019-07-26T12:10:07.567Z",
   "scrollStart": null,
   "scrollEnd": null,
   "processingStart": null,
   "processingEnd": null,
   "completed": null,
   "processingMillis": 0
}
```

As showed is the snipet above, we lauch the rendtion recomputation on all documents that have a `Picture` `MixinType` (no matter if the document contains a renditon picture or not), the query is provided as Http body request of the `POST` call, in the case where this query is not provided, the recomputation will be launched on all documents without a picture rendition (`SELECT * FROM Document WHERE ecm:mixinType = 'Picture' AND picture:views/*/title IS NULL`)

```bash
curl -u Administrator:Administrator \
-H 'Content-Type: application/json' \
-X POST http://NUXEO_SERVER/nuxeo/site/management/renditions/pictures/recompute
```

```bash
{
   "entity-type": "bulkStatus",
   "commandId": "93baf453-5d34-4bb9-9370-54ec4ffe78b2",
   "state": "SCHEDULED",
   "processed": 0,
   "error": false,
   "errorCount": 0,
   "total": 0,
   "action": "recomputeViews",
   "username": "system",
   "submitted": "2019-07-26T12:10:07.567Z",
   "scrollStart": null,
   "scrollEnd": null,
   "processingStart": null,
   "processingEnd": null,
   "completed": null,
   "processingMillis": 0
}
```

#### Recompute a rendition on Thumbnails

To recompute thumbnails of the documents matching a given query, you call the endpoint as below:

**Url**
```
POST http://NUXEO_SERVER/nuxeo/site/management/renditions/thumbnails/recompute
```

**Response**
If successful, returns a json reprsentation of [Bulk Action Status]({{page page='bulk-action-framework'}}#bulk-rest-api).

**Status Codes**
- 200 *OK* - Success.

```bash
curl -u Administrator:Administrator \
-H 'Content-Type: application/json' \
-X POST http://NUXEO_SERVER/nuxeo/site/management/renditions/thumbnails/recompute \
-d 'query=SELECT * FROM Document WHERE ecm:mixinType = "Thumbnail"'
```

The response will be as below
```bash
{
   "entity-type": "bulkStatus",
   "commandId": "0e1e6800-631a-4e04-a47c-241ea7b3596a",
   "state": "SCHEDULED",
   "processed": 0,
   "error": false,
   "errorCount": 0,
   "total": 0,
   "action": "recomputeThumbnails",
   "username": "system",
   "submitted": "2019-07-26T12:13:31.361Z",
   "scrollStart": null,
   "scrollEnd": null,
   "processingStart": null,
   "processingEnd": null,
   "completed": null,
   "processingMillis": 0
}
```

As showed is the snipet above, we lauch the rendtion recomputation on all documents that have a `Thumbnail` `MixinType` (no matter if the document contains a renditon thumbnail or not), the query is provided as Http body request of the `POST` call, in the case where this query is not provided, the recomputation will be launched on all documents that match the defaut query `SELECT * FROM Document WHERE ecm:mixinType = 'Thumbnail' AND thumb:thumbnail/data IS NULL AND ecm:isVersion = 0 AND ecm:isProxy = 0 AND ecm:isTrashed = 0`

```bash
curl -u Administrator:Administrator \
-H 'Content-Type: application/json' \
-X POST http://NUXEO_SERVER/nuxeo/site/management/renditions/thumbnails/recompute
```

```bash
{
   "entity-type": "bulkStatus",
   "commandId": "0e1e6800-631a-4e04-a47c-241ea7b3596a",
   "state": "SCHEDULED",
   "processed": 0,
   "error": false,
   "errorCount": 0,
   "total": 0,
   "action": "recomputeThumbnails",
   "username": "system",
   "submitted": "2019-07-26T12:13:31.361Z",
   "scrollStart": null,
   "scrollEnd": null,
   "processingStart": null,
   "processingEnd": null,
   "completed": null,
   "processingMillis": 0
}
```

All renditions endpoints that was defined above will use the [Bulk Action Framework]({{page page='bulk-action-framework'}}).

### Repair Works in failure

[Nuxeo Stream]({{page page='nuxeo-stream'}}) provides a Stream processing pattern.

When a stream work fails it will be stored in a Dead Letter Queue (DLQ), When the cause of the failure requires manual intervention (fix a misconfiguration, restart a service, fix a disk full, re-deployment ...) a retry policy is not enough. [WorkManager DLQ](https://jira.nuxeo.com/browse/NXP-27148), this work can be relaunched after this manual intervention, this is can done using the endpoint as below:

**Url**
```
POST http://localhost:8080/nuxeo/site/management/workmanager/runworksinfailure
```

**Response**
If successful, returns a json representation of the total of the processed documents and of those that were successful.

**Status Codes**
- 200 *OK* - Success.


```bash
curl -X POST -u Administrator:Administrator \
-H 'Content-Type: application/json' \
-X POST http://localhost:8080/nuxeo/site/management/workmanager/runworksinfailure
```

```bash
{
 "total": 0,
 "success": 0
}
```

If needed we can define an execution timeout as below

```bash
curl -X POST -u Administrator:Administrator \
-H 'Content-Type: application/json' \
-X POST http://localhost:8080/nuxeo/site/management/workmanager/runworksinfailure
-d timeoutSeconds=10
```

```bash
{
 "total": 0,
 "success": 0
}
```
