---
title: "ICorDebugCode::GetFunction Method | Microsoft Docs"
ms.custom: ""
ms.date: "03/30/2017"
ms.prod: ".net-framework"
ms.reviewer: ""
ms.suite: ""
ms.technology: 
  - "dotnet-clr"
ms.tgt_pltfrm: ""
ms.topic: "reference"
apiname: 
  - "ICorDebugCode.GetFunction"
apilocation: 
  - "mscordbi.dll"
apitype: "COM"
f1_keywords: 
  - "ICorDebugCode::GetFunction"
dev_langs: 
  - "C++"
helpviewer_keywords: 
  - "GetFunction method, ICorDebugCode interface [.NET Framework debugging]"
  - "ICorDebugCode::GetFunction method [.NET Framework debugging]"
ms.assetid: c568b737-fdb2-4816-accd-051f5ab760f1
caps.latest.revision: 10
author: "rpetrusha"
ms.author: "ronpet"
manager: "wpickett"
---
# ICorDebugCode::GetFunction Method
Gets the "ICorDebugFunction" associated with this "ICorDebugCode".  
  
## Syntax  
  
```  
HRESULT GetFunction (  
    [out] ICorDebugFunction **ppFunction  
);  
```  
  
#### Parameters  
 `ppFunction`  
 [out] A pointer to the address of the function.  
  
## Remarks  
 `ICorDebugCode` and `ICorDebugFunction` maintain a one-to-one relationship.  
  
## Requirements  
 **Platforms:** See [System Requirements](../../../../docs/framework/get-started/system-requirements.md).  
  
 **Header:** CorDebug.idl, CorDebug.h  
  
 **Library:** CorGuids.lib  
  
 **.NET Framework Versions:** [!INCLUDE[net_current_v10plus](../../../../includes/net-current-v10plus-md.md)]  
  
## See Also  
 