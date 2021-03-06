---
title: 快速入門：建立資料科學 VM - Resource Manager 範本
titleSuffix: Azure Data Science Virtual Machine
description: 在此快速入門中，您將使用 Azure Resource Manager 範本，快速部署資料科學虛擬機器
services: machine-learning
author: lobrien
ms.author: laobri
ms.custom: subject-armqs
ms.date: 06/10/2020
ms.service: machine-learning
ms.subservice: data-science-vm
ms.topic: quickstart
ms.openlocfilehash: 8f83cdb09b94fb7eda1f8f700390be8b2c2147e0
ms.sourcegitcommit: eb6bef1274b9e6390c7a77ff69bf6a3b94e827fc
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/05/2020
ms.locfileid: "90883154"
---
# <a name="quickstart-create-an-ubuntu-data-science-virtual-machine-using-an-arm-template"></a>快速入門：使用 ARM 範本，建立 Ubuntu 資料科學虛擬機器

此快速入門將說明如何使用 Azure Resource Manager 範本 (ARM 範本)，建立 Ubuntu 18.04 資料科學虛擬機器。 資料科學虛擬機器是雲端虛擬機器，預先載入了一套資料科學，以及機器學習架構與工具。 當部署了支援 GPU 的計算資源之後，所有工具與程式庫將會設定為使用該 GPU。

[!INCLUDE [About Azure Resource Manager](../../../includes/resource-manager-quickstart-introduction.md)]

如果您的環境符合必要條件，而且您很熟悉 ARM 範本，請選取 [部署至 Azure] 按鈕。 範本會在 Azure 入口網站中開啟。

[![部署至 Azure](../../media/template-deployments/deploy-to-azure.svg)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2F101-vm-ubuntu-DSVM-GPU-or-CPU%2Fazuredeploy.json)

## <a name="prerequisites"></a>必要條件

* Azure 訂用帳戶。 如果您沒有 Azure 訂用帳戶，請在開始前建立[免費帳戶](https://azure.microsoft.com/free/services/machine-learning/)。

* 若要從您的**本機環境**使用本文件中的 CLI 命令，您需要 [Azure CLI](/cli/azure/install-azure-cli)。

## <a name="review-the-template"></a>檢閱範本

本快速入門中使用的範本是來自 [Azure 快速入門範本](https://azure.microsoft.com/resources/templates/101-vm-ubuntu-DSVM-GPU-or-CPU/)。

:::code language="json" source="~/quickstart-templates/101-vm-ubuntu-DSVM-GPU-or-CPU/azuredeploy.json":::

範本中會定義下列資源：

* [Microsoft.Network/networkInterfaces](/azure/templates/microsoft.network/networkinterfaces)
* [Microsoft.Network/networkSecurityGroups](/azure/templates/microsoft.network/networksecuritygroups)
* [Microsoft.Network/virtualNetworks](/azure/templates/microsoft.network/virtualnetworks)
* [Microsoft.Network/publicIPAddresses](/azure/templates/microsoft.network/publicipaddresses)
* [Microsoft.Storage/storageAccounts](/azure/templates/microsoft.storage/storageaccounts)
* [Microsoft.Compute/virtualMachines](/azure/templates/microsoft.compute/virtualmachines)：建立雲端虛擬機器。 此範本已將虛擬機器設定為執行 Ubuntu 18.04 的資料科學虛擬機器。

## <a name="deploy-the-template"></a>部署範本

若要從 Azure CLI 使用範本，請登入並選擇您的訂用帳戶 (請參閱[使用 Azure CLI 登入](/cli/azure/authenticate-azure-cli))。 然後執行：

```azurecli-interactive
read -p "Enter the name of the resource group to create:" resourceGroupName &&
read -p "Enter the Azure location (e.g., centralus):" location &&
read -p "Enter the authentication type (must be 'password' or 'sshPublicKey') :" authenticationType &&
read -p "Enter the login name for the administrator account (may not be 'admin'):" adminUsername &&
read -p "Enter administrator account secure string (value of password or ssh public key):" adminPasswordOrKey &&
templateUri="https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/101-vm-ubuntu-DSVM-GPU-or-CPU/azuredeploy.json" &&
az group create --name $resourceGroupName --location "$location" &&
az deployment group create --resource-group $resourceGroupName --template-uri $templateUri --parameters adminUsername=$adminUsername authenticationType=$authenticationType adminPasswordOrKey=$adminPasswordOrKey &&
echo "Press [ENTER] to continue ..." &&
read
```

當您執行上述命令時，請輸入：

1. 您要建立之資源群組的名稱，該群組中將會包含 DSVM 與相關聯的資源。
1. 您想要在其中進行部署的 Azure 位置。
1. 您要使用的驗證類型 (輸入字串 `password` 或 `sshPublicKey`)。
1. 系統管理員帳戶的登入名稱 (此值不得為 `admin`)。
1. 帳戶密碼或 SSH 公開金鑰的值。

## <a name="review-deployed-resources"></a>檢閱已部署的資源

若要查看您的資料科學虛擬機器：

1. 移至 [Azure 入口網站](https://portal.azure.com)
1. 登入。
1. 選擇您剛才所建立的資源群組。

您會看到資源群組的資訊：

:::image type="content" source="media/dsvm-tutorial-resource-manager/resource-group-home.png" alt-text="內含 DSVM 之基本資源群組的螢幕擷取畫面":::

按一下虛擬機器資源，移至其資訊頁面。 您可以在此找到 VM 的相關資訊，包括連線詳細資料。

## <a name="clean-up-resources"></a>清除資源

若不想使用此虛擬機器，請將其刪除。 因為 DSVM 會與其他資源 (例如儲存體帳戶) 相關聯，建議您刪除先前建立的整個資源群組。 您可以使用入口網站來刪除資源群組，只要按一下 [刪除] 按鈕並確認即可。 或者，您可以使用下列方式，從 CLI 刪除資源群組：

```azurecli-interactive
echo "Enter the Resource Group name:" &&
read resourceGroupName &&
az group delete --name $resourceGroupName &&
echo "Press [ENTER] to continue ..."
```

## <a name="next-steps"></a>後續步驟

在此快速入門中，您使用了 ARM 範本，建立資料科學虛擬機器。

> [!div class="nextstepaction"]
> [範例程式 & ML 逐步解說](dsvm-samples-and-walkthroughs.md)
