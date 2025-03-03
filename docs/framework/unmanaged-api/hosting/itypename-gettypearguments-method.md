---
title: "ITypeName::GetTypeArguments Method | Microsoft Docs"
ms.custom: ""
ms.date: "03/30/2017"
ms.prod: ".net-framework-4.6"
ms.reviewer: ""
ms.suite: ""
ms.technology: 
  - "dotnet-clr"
ms.tgt_pltfrm: ""
ms.topic: "reference"
apiname: 
  - "ITypeName.GetTypeArguments"
apilocation: 
  - "mscoree.dll"
apitype: "COM"
f1_keywords: 
  - "GetTypeArguments"
dev_langs: 
  - "C++"
helpviewer_keywords: 
  - "ITypeName::GetTypeArguments method [.NET Framework hosting]"
  - "GetTypeArguments method [.NET Framework hosting]"
ms.assetid: 638d77df-ff9c-40d9-88ee-930f5f87ada1
caps.latest.revision: 7
author: "rpetrusha"
ms.author: "ronpet"
manager: "wpickett"
---
# ITypeName::GetTypeArguments Method
This method supports the .NET Framework infrastructure and is not intended to be used directly from your code.  
  
## Syntax  
  
```  
HRESULT GetTypeArguments (  
    [in] DWORD           count,  
    [out] ITypeName**    rgpArguments,  
    [out, retval] DWORD* pCount  
);  
```  
  
## Requirements  
 **Platforms:** See [System Requirements](../../../../docs/framework/get-started/system-requirements.md).  
  
 **Header:** MSCorEE.h  
  
 **Library:** Included as a resource in MSCorEE.dll  
  
 **.NET Framework Versions:** [!INCLUDE[net_current_v20plus](../../../../includes/net-current-v20plus-md.md)]  
  
## See Also  
 [Hosting Interfaces](../../../../docs/framework/unmanaged-api/hosting/hosting-interfaces.md)