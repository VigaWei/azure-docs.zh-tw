---
title: 了解 Azure IoT 中心應用程式的裝置範本版本設定 | Microsoft Docs
description: 藉由建立新版本來反覆使用裝置範本，而不會影響您的即時連線裝置
author: dominicbetts
ms.author: dobett
ms.date: 11/06/2020
ms.topic: how-to
ms.service: iot-central
services: iot-central
ms.custom: device-developer
ms.openlocfilehash: 93545c63013c95e3db498b079061da3d9b189efd
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 11/25/2020
ms.locfileid: "95995755"
---
# <a name="create-a-new-device-template-version"></a>建立新的裝置範本版本

*本文適用於解決方案建置人員和裝置開發人員。*

裝置範本包含描述裝置如何與 IoT Central 互動的架構。 這些互動包括遙測、屬性和命令。 裝置和 IoT Central 應用程式都依賴此架構的共用理解來交換資訊。 您只能對架構進行有限的變更，而不會中斷合約，因此大部分的架構變更都需要新版本的裝置範本。 版本設定裝置範本可讓較舊的裝置繼續使用其所瞭解的架構版本，而較新或更新的裝置則會使用更新的架構版本。

裝置範本中的架構是在裝置模型及其介面中定義。 裝置範本包含其他資訊，例如雲端屬性、顯示自訂和 views。 如果您對裝置範本的這些部分進行變更，但未定義裝置與 IoT Central 交換資料的方式，您就不需要為範本版本。

您必須一律先發佈更新的裝置範本，操作員才能使用它。 IoT Central 會讓您無法在不先設定範本版本的情況下發布裝置範本的重大變更。

> [!NOTE]
> 若要深入瞭解如何建立裝置範本，請參閱 [設定和管理裝置範本](howto-set-up-template.md)

## <a name="versioning-rules"></a>版本控制規則

本節摘要說明適用于裝置範本的版本控制規則。 裝置型號和介面都有版本號碼。 下列程式碼片段顯示控溫器裝置的裝置型號。 裝置型號具有單一介面。 您可以在欄位結尾看到版本號碼 `@id` 。

```json
{
  "@context": "dtmi:dtdl:context;2",
  "@id": "dtmi:com:example:Thermostat;1",
  "@type": "Interface",
  "displayName": "Thermostat",
  "description": "Reports current temperature and provides desired temperature control.",
  "contents": [
    // ...
  ]
}
```

若要在 IoT Central UI 中查看這項資訊，請在裝置範本編輯器中選取 [ **view identity** ]：

:::image type="content" source="media/howto-version-device-template/view-identity.png" alt-text="查看介面的身分識別以查看版本號碼":::

下列清單顯示的規則會決定何時必須建立新的版本：

* 在裝置模型發佈之後，您就無法移除任何介面，即使是在新版本的裝置範本中也一樣。
* 發佈裝置模型之後，如果您建立新版本的裝置範本，您可以新增介面。
* 在裝置模型發佈之後，如果您建立新版本的裝置範本，則可以使用較新的版本來取代介面。 例如，如果感應器 v1 裝置範本使用控溫器 v1 介面，您可以建立使用控溫器 v2 介面的感應器 v2 裝置範本。
* 發佈介面之後，您就無法移除任何介面內容，即使是在新版本的裝置範本中也一樣。
* 發佈介面之後，如果您建立新版本的介面和裝置範本，您可以將專案新增至介面內容。 您可以新增至介面的專案包括遙測、屬性和命令。
* 發佈介面之後，如果您建立新版本的介面和裝置範本，您可以對介面中的現有專案進行非架構變更。 介面專案的非架構部分包含顯示名稱和語義型別。 您無法變更之介面專案的架構部分為 [名稱]、[功能類型] 和 [架構]。

下列各節將逐步引導您瞭解在 IoT Central 中修改裝置範本的一些範例。

## <a name="customize-the-device-template-without-versioning"></a>自訂無版本控制的裝置範本

您可以編輯裝置功能的某些元素，而不需要為您的裝置範本和介面進行版本。 例如，其中一些欄位包括顯示名稱、語義類型、最小值、最大值、小數位數、色彩、單位、顯示單位、批註和描述。 若要加入其中一種自訂：

1. 移至 [ **裝置範本** ] 頁面。
1. 選取您要自訂的裝置範本。
1. 選擇 [ **自訂** ] 索引標籤。
1. 此處會列出您裝置模型中定義的所有功能。 您可以編輯、儲存和使用所有欄位，而不需要為您的裝置範本版本。 如果有您想要編輯的欄位是唯讀的，您必須將裝置範本的版本設為已變更。 選取您要編輯的欄位，然後輸入任何新值。
1. 選取 [儲存]。 現在這些值會覆寫一開始儲存在您裝置範本中的任何資料，而且會在應用程式中使用。

## <a name="version-a-device-template"></a>版本裝置範本

建立新版本的裝置範本會建立範本的草稿版本，您可以在其中編輯裝置模型。 任何已發佈的介面在個別設定版本之前都保持發佈。 若要修改已發佈的介面，請先建立新的裝置範本版本。

當您嘗試編輯無法在 [自訂] 區段中編輯的部分裝置型號時，請只將裝置範本設為版本。

若要為裝置範本建立版本：

1. 移至 [ **裝置範本** ] 頁面。
1. 選取您要嘗試版本的裝置範本。
1. 選取頁面頂端的 [ **版本** ] 按鈕，並為範本提供新的名稱。 IoT Central 建議您可以編輯的新名稱。
1. 選取 [建立]。
1. 現在您的裝置範本處於草稿模式。 您可以看到您的介面仍處於鎖定狀態。 您要修改的任何介面版本。

## <a name="version-an-interface"></a>版本介面

將介面版本化可讓您新增、更新和移除您已建立之介面內的功能。

若要將介面設為版本：

1. 移至 [ **裝置範本** ] 頁面。
1. 選取您在草稿模式中擁有的裝置範本。
1. 選取您要進行版本和編輯的已發行模式中的介面。
1. 選取 [介面] 頁面頂端的 [ **版本** ] 按鈕。
1. 選取 [建立]。
1. 現在您的介面處於草稿模式。 您可以新增或編輯介面的功能，而不會中斷現有的自訂和查看。

## <a name="migrate-a-device-across-versions"></a>跨版本遷移裝置

您可以建立多個版本的裝置範本。 經過一段時間後，您就會有多個已連線的裝置使用這些裝置範本。 您可以將裝置從一個版本的裝置範本移轉至另一個版本。 下列步驟說明如何移轉裝置：

1. 移至 **[裝置]** 頁面。
1. 選取您需要移轉到另一個版本的裝置。
1. 選擇 **遷移**：  :::image type="content" source="media/howto-version-device-template/migrate-device.png" alt-text="選擇開始遷移裝置的選項":::
1. 選取具有您想要將裝置遷移至其版本號碼的裝置範本，然後選取 [ **遷移**]。

## <a name="next-steps"></a>後續步驟

如果您是操作員或解決方案產生器，建議的下一個步驟是瞭解 [如何管理您的裝置](./howto-manage-devices.md)。

如果您是裝置開發人員，建議的下一個步驟是閱讀 [Azure IoT Edge 裝置和 Azure IoT Central](./concepts-iot-edge.md)的相關資訊。
