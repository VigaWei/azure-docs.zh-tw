---
title: Azure Marketplace 中的受控服務供應項目
description: 受控服務供應專案可讓您將資源管理供應專案銷售給 Azure Marketplace 中的客戶。
ms.date: 07/28/2020
ms.topic: conceptual
ms.openlocfilehash: 6c3047cd95128f689e75d9c1f5fba5a39f86291c
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/09/2020
ms.locfileid: "88163317"
---
# <a name="managed-service-offers-in-azure-marketplace"></a>Azure Marketplace 中的受控服務供應項目

本文說明[Azure Marketplace](https://azuremarketplace.microsoft.com)中**受管理的服務**供應專案類型。 受管理的服務供應專案可讓您透過 [Azure Lighthouse](../overview.md)為客戶提供資源管理服務。 您可以將這些供應專案提供給所有潛在客戶，或只提供給一或多個特定客戶。 由於您會直接向客戶收取與這些受控服務相關的費用，因此 Microsoft 不會向您收取任何費用。

## <a name="understand-managed-service-offers"></a>瞭解受管理的服務供應專案

受控服務可簡化將客戶上架到 Azure Lighthouse 的流程。 當客戶在 Azure Marketplace 中購買供應項目時，他們將能夠指定應上架的訂用帳戶和 (或) 資源群組。

之後，您組織中的使用者將能夠根據您在建立供應專案時所定義的存取權，透過 [Azure 委派的資源管理](azure-delegated-resource-management.md)，在您的管理租使用者內處理這些資源。 這是透過指定 Azure Active Directory (Azure AD) 使用者、群組和服務主體的資訊清單來完成，這些使用者、群組和服務主體會有客戶資源的存取權，以及定義其存取層級的角色。 透過將權限指派給 Azure AD 群組，而非一系列個別使用者或應用程式帳戶，您便可以在需要變更存取權時個別地加入或移除使用者。

## <a name="public-and-private-offers"></a>公用和私人供應項目

每個受管理的服務供應專案都包含一或多個方案。 方案可以是私用或公用方案。

如果您想要將供應項目限制為僅提供特定客戶使用，則可以發佈私人方案。 當您這樣做時，就只能針對您所提供的特定訂用帳戶識別碼購買方案。 如需詳細資訊，請參閱[私人供應項目](../../marketplace/private-offers.md) \(部分機器翻譯\)。

> [!NOTE]
> 透過雲端解決方案提供者 (CSP) 計畫的轉銷商所建立的訂用帳戶不支援私人優惠。

公用方案能讓您將服務推廣給新的客戶。 當您僅需要客戶租用戶的有限存取權時，這通常是較為適合的選擇。 當您與客戶建立關聯性之後，如果他們決定將額外存取權授與您的組織，您可以僅為該客戶發佈新的私人方案，或是[使用 Azure Resource Manager 範本將他們上線以取得進一步存取](../how-to/onboard-customer.md)。

如有需要，您可以在相同的供應項目中同時包含公用與私人方案。

> [!IMPORTANT]
> 當您將某個方案發佈為公用之後，便無法再將其變更為私人。 若要控制哪些客戶可以接受您的供應項目和委派資源，請使用私人方案。 有了公用方案，您無法將可用性限制為僅供某個客戶甚至是特定數目的客戶使用，不過您可以選擇完全停止銷售該方案。 客戶接受供應項目後，您可以[移除委託的存取權](../how-to/remove-delegation.md) (僅在您發布該供應項目時包含 [角色定義] 設定為[受控服務註冊分配刪除角色](../../role-based-access-control/built-in-roles.md#managed-services-registration-assignment-delete-role)的 [授權])。 您也可以與客戶聯繫，並要求其[移除您的存取權](../how-to/view-manage-service-providers.md#add-or-remove-service-provider-offers)。

## <a name="publish-managed-service-offers"></a>發佈受控服務供應專案

若要瞭解如何發佈受控服務供應專案，請參閱將 [受控服務供應專案發佈至 Azure Marketplace](../how-to/publish-managed-services-offers.md)。

## <a name="next-steps"></a>後續步驟

- 了解 [Azure 委派的資源管理](azure-delegated-resource-management.md)與[跨租用戶管理體驗](cross-tenant-management-experience.md)。
- 將[受控服務](../how-to/publish-managed-services-offers.md)供應專案發佈至 Azure Marketplace。
