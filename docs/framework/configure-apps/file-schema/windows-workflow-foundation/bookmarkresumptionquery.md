---
title: "&lt;bookmarkResumptionQuery&gt; | Microsoft Docs"
ms.custom: ""
ms.date: "03/30/2017"
ms.prod: ".net-framework-4.6"
ms.reviewer: ""
ms.suite: ""
ms.tgt_pltfrm: ""
ms.topic: "reference"
ms.assetid: 226de75d-d25c-48d5-b069-4b7d80a6852b
caps.latest.revision: 4
author: "Erikre"
ms.author: "erikre"
manager: "erikre"
---
# &lt;bookmarkResumptionQuery&gt;
Represents a query that is used to track resumption of a bookmark within a workflow instance. The query is necessary for a tracking participant to subscribe to bookmark resumption records.  
  
 For more information on tracking profile queries, see [Tracking Profiles](../../../../../docs/framework/windows-workflow-foundation/tracking-profiles.md)  
  
 \<system.serviceModel>  
\<tracking>  
\<trackingProfile>  
\<workflow>  
\<bookmarkResumptionQueries>  
\<bookmarkResumptionQuery>  
  
## Syntax  
  
```vb  
<tracking>   <trackingProfile name="Name">       <workflow>          <bookmarkResumptionQueries>             <bookmarkResumptionQuery name="String" />          </bookmarkResumptionQueries>       </workflow>   </trackingProfile></tracking>  
```  
  
## Attributes and Elements  
 The following sections describe attributes, child elements, and parent elements.  
  
### Attributes  
  
|Attribute|Description|  
|---------------|-----------------|  
|name|A string that specifies the name of the bookmark record to subscribe to.|  
  
### Child Elements  
 None.  
  
### Parent Elements  
  
|Element|Description|  
|-------------|-----------------|  
|[\<bookmarkResumptionQueries>](../../../../../docs/framework/configure-apps/file-schema/windows-workflow-foundation/bookmarkresumptionqueries.md)|Represents a collection of queries that are used to track resumption of a bookmark within a workflow instance.|  
  
## See Also  
 [System.ServiceModel.Activities.Tracking.Configuration.BookmarkResumptionQueryElementCollection](assetId:///System.ServiceModel.Activities.Tracking.Configuration.BookmarkResumptionQueryElementCollection?qualifyHint=False&amp;autoUpgrade=True)   
 [System.Activities.Tracking.BookmarkResumptionQuery](assetId:///System.Activities.Tracking.BookmarkResumptionQuery?qualifyHint=False&amp;autoUpgrade=True)   
 [Workflow Tracking and Tracing](../../../../../docs/framework/windows-workflow-foundation/workflow-tracking-and-tracing.md)   
 [Tracking Profiles](../../../../../docs/framework/windows-workflow-foundation/tracking-profiles.md)