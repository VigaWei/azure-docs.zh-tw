---
author: ggailey777
ms.service: azure-functions
ms.topic: include
ms.date: 05/27/2019
ms.author: glenga
ms.openlocfilehash: d697334fe56fb9133a06cee79067c60bc3a37281
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 11/25/2020
ms.locfileid: "96010441"
---
安裝繫結擴充功能最簡單的方式是啟用[擴充功能套件組合](../articles/azure-functions/functions-bindings-register.md#extension-bundles)。 當您啟用套件組合時，系統會自動安裝一組預先定義的擴充功能套件。

若要啟用擴充功能套件組合，請開啟 host.json 檔案，並更新其內容以符合下列程式碼：

```json
{
    "version": "2.0",
    "extensionBundle": {
        "id": "Microsoft.Azure.Functions.ExtensionBundle",
        "version": "[1.*, 2.0.0)"
    }
}
```