---
title: "&lt;participants&gt; of WCF | Microsoft Docs"
ms.custom: ""
ms.date: "03/30/2017"
ms.prod: ".net-framework-4.6"
ms.reviewer: ""
ms.suite: ""
ms.technology: 
  - "dotnet-clr"
ms.tgt_pltfrm: ""
ms.topic: "article"
ms.assetid: d99dbddc-0057-4e18-8e42-f91411d39970
caps.latest.revision: 3
author: "Erikre"
ms.author: "erikre"
manager: "erikre"
---
# &lt;participants&gt; of WCF
Configure a list of tracking participants that listen to the tracking records being emitted from the runtime directly and process them in whatever way they are configured. This includes writing to a specific output (e.g., file, Console, ETW), processing/aggregating the records, or any other combination that might be required.  
  
 For more information in workflow tracking and tracking participants, see [Workflow Tracking and Tracing](../../../../../docs/framework/windows-workflow-foundation/workflow-tracking-and-tracing.md) and [Tracking Participants](../../../../../docs/framework/windows-workflow-foundation/tracking-participants.md).  
  
 \<system.serviceModel>  
\<tracking>  
\<participants>  
  
## Syntax  
  
```vb  
   <tracking>    <participants>       <add name="String"            profileName="String"           type="String" />    </participants> </tracking>   
```  
  
## Attributes and Elements  
 The following sections describe attributes, child elements, and parent elements.  
  
### Attributes  
 None.  
  
### Child Elements  
  
|Element|Description|  
|-------------|-----------------|  
|[\<add>](../../../../../docs/framework/configure-apps/file-schema/windows-workflow-foundation/add-of-participants.md)|Contains settings for a tracking participant.|  
  
### Parent Elements  
  
|Element|Description|  
|-------------|-----------------|  
|[\<tracking>](../../../../../docs/framework/configure-apps/file-schema/windows-workflow-foundation/tracking.md)|Represents a configuration section for defining tracking settings for a workflow service.|  
  
## Remarks  
 Tracking participants are used to get the tracking data emitted from the workflow and store it into different mediums. Likewise, any post processing on the tracking Records can also be done within the tracking participant.  
  
 Multiple tracking participants can consume the tracking events simultaneously. Each tracking participant can be associated with a different tracking profile.  
  
 A standard tracking participant is provided which writes the tracking records to an ETW session. The participant is configured on a workflow service by adding a tracking-specific behavior in a configuration file. Enabling an ETW tracking participant allows tracking records to be viewed in the event viewer. If that does not meet your requirements, you can also write a custom tracking participant.  
  
## Example  
 The following configuration example shows the standard ETW tracking participant being configured in the Web.config file.  
  
 The Provider Id that the ETW Tracking Participant uses for writing the Tracking Records to ETW is defined in the `<diagnostics>` section. The tracking participant has a profile associated with it to specify the tracking records it has subscribed to. This is defined by the `profileName` attribute of the `<add>` element. Once these are defined, the Tracking Participant is added to the `<etwTracking>` service behavior. This will add the selected Tracking Participants to the Workflow instance’s extensions, so that they begin to receive the Tracking Records.  
  
```  
  
<configuration>   
  <system.web>   
    <compilation targetFrameworkMoniker=".NETFramework,Version=v4.0"/>   
  </system.web>   
  <system.serviceModel>   
    <diagnostics etwProviderId="52A3165D-4AD9-405C-B1E8-7D9A257EAC9F" />                
    <tracking>   
      <participants>   
        <add name="EtwTrackingParticipant"   
             type="System.Activities.Tracking.EtwTrackingParticipant, System.Activities, Version=4.0.0.0, Culture=neutral, PublicKeyToken=31bf3856ad364e35"   
             profileName="HealthMonitoring_Tracking_Profile"/>   
      </participants>   
    </tracking>   
    <behaviors>   
      <serviceBehaviors>   
        <behavior>   
          <etwTracking profileName="Sample Tracking Profile"/>  
        </behavior>   
      </serviceBehaviors>   
    </behaviors>   
  </system.serviceModel>   
</configuration>  
```  
  
## See Also  
 <xref:System.ServiceModel.Activities.Tracking.Configuration.TrackingSection>   
 <xref:System.ServiceModel.Activities.Description.EtwTrackingBehavior>   
 [Workflow Tracking and Tracing](../../../../../docs/framework/windows-workflow-foundation/workflow-tracking-and-tracing.md)   
 [Tracking Participants](../../../../../docs/framework/windows-workflow-foundation/tracking-participants.md)