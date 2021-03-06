---
title: 為 Azure API for FHIR 設定客戶自控金鑰
description: 透過 Cosmos DB，讓您自己的金鑰功能在 Azure API for FHIR 中受到支援
services: healthcare-apis
author: matjazl
ms.service: healthcare-apis
ms.subservice: fhir
ms.topic: overview
ms.date: 09/28/2020
ms.author: matjazl
ms.openlocfilehash: 05c208ba3c9005d38b8924037748764f8d112e3a
ms.sourcegitcommit: 0ce1ccdb34ad60321a647c691b0cff3b9d7a39c8
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 11/05/2020
ms.locfileid: "93398176"
---
# <a name="configure-customer-managed-keys"></a>設定客戶管理的金鑰

當您建立新的 Azure API for FHIR 帳戶時，預設會使用 Microsoft 代控金鑰來加密您的資料。 現在，您可以使用自己選擇並自行管理的金鑰，為資料新增第二層加密。

在 Azure 中，這通常是使用客戶的 Azure Key Vault (AKV) 中的加密金鑰來完成。 Azure SQL、Azure 儲存體和 Cosmos DB 都是現今提供這項功能的一些範例。 Azure API for FHIR 會利用來自 Cosmos DB 的這種支援。 當您建立帳戶時，您可選擇指定 AKV 金鑰 URI。 我們會在佈建 DB 帳戶時，將此金鑰傳遞給 Cosmos DB。 提出 FHIR 要求時，Cosmos DB 會提取您的金鑰並將其用於加密/解密資料。 若要開始使用，您可以參考下列連結：

- [為您的 Azure 訂用帳戶註冊 Azure Cosmos DB 資源提供者](../cosmos-db/how-to-setup-cmk.md#register-resource-provider) 
- [設定您的 AKV 執行個體](../cosmos-db/how-to-setup-cmk.md#configure-your-azure-key-vault-instance)
-  [將存取原則新增至您的 AKV 執行個體](../cosmos-db/how-to-setup-cmk.md#add-an-access-policy-to-your-azure-key-vault-instance)
- [在 AKV 中產生金鑰](../cosmos-db/how-to-setup-cmk.md#generate-a-key-in-azure-key-vault)

在 Azure 入口網站上建立 Azure API for FHIR 帳戶之後，您可以在 [其他設定] 索引標籤上的 [資料庫設定] 之下看到 [資料加密] 設定選項。根據預設，將會選擇服務代控金鑰選項。 您可以選取 [客戶自控金鑰] 選項，在此指定您的 AKV 金鑰。 您可以在此輸入所複製的金鑰 URI。

:::image type="content" source="media/bring-your-own-key/bring-your-own-key-create.png" alt-text="建立 Azure API for FHIR":::

或者，您可以從 KeyPicker 選擇您的金鑰：

:::image type="content" source="media/bring-your-own-key/bring-your-own-key-keypicker.png" alt-text="KeyPicker":::

對於現有的 FHIR 帳戶，您可以在 [資料庫] 刀鋒視窗中檢視金鑰加密選擇 (服務代控或客戶自控金鑰)，如下所示。 選擇後，便無法修改設定選項。 不過，您可以修改和更新金鑰。

:::image type="content" source="media/bring-your-own-key/bring-your-own-key-database.png" alt-text="資料庫":::

此外，您可以建立所指定金鑰的新版本，在此之後，您的資料將會以新版本加密，而不會發生任何服務中斷。 您也可移除金鑰的存取權，以移除資料的存取權。