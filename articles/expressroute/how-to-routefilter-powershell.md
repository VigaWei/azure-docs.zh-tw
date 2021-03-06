---
title: 教學課程：針對 Microsoft 對等互連設定路由篩選 - Azure PowerShell
description: 本教學課程說明如何使用 PowerShell 針對 Microsoft 對等互連設定路由篩選。
services: expressroute
author: duongau
ms.service: expressroute
ms.topic: tutorial
ms.date: 10/08/2020
ms.author: duau
ms.custom: seodec18
ms.openlocfilehash: 90d4def5a1c08e305b9315f299e83e2187b6be2c
ms.sourcegitcommit: d103a93e7ef2dde1298f04e307920378a87e982a
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/13/2020
ms.locfileid: "91969869"
---
# <a name="tutorial-configure-route-filters-for-microsoft-peering-using-powershell"></a>教學課程：針對使用 PowerShell 的 Microsoft 對等互連設定路由篩選

> [!div class="op_single_selector"]
> * [Azure 入口網站](how-to-routefilter-portal.md)
> * [Azure PowerShell](how-to-routefilter-powershell.md)
> * [Azure CLI](how-to-routefilter-cli.md)
> 

路由篩選是透過 Microsoft 對等互連使用支援服務子集的方式。 這篇文章中的步驟可協助您設定和管理 ExpressRoute 線路的路由篩選。

透過 Microsoft 對等互連，既可存取 Microsoft 365 服務 (例如 Exchange Online、SharePoint Online 和商務用 Skype)，也可以存取 Azure 公用服務 (例如儲存體和 SQL DB)。 Azure 公用服務可針對每個區域選取，但無法針對每個公用服務加以定義。

在 ExpressRoute 線路中設定 Microsoft 對等互連後，與這些服務相關的所有前置詞，都會透過建立的 BGP 工作階段進行公告。 BGP 社群值附加至每個前置詞，來識別透過前置詞提供的服務。 如需 BGP 社群值和它們對應之服務的清單，請參閱 [BGP 社群](expressroute-routing.md#bgp)。

與所有 Azure 和 Microsoft 365 服務的連線會導致大量的前置詞透過 BGP 進行公告。 大量前置詞會大幅增加網路內路由器維護的路由資料表大小。 如果您計劃僅使用透過 Microsoft 對等互連提供的服務子集，您可以用兩種方式減少路由資料表大小。 您可以：

* 在 BGP 社群上套用路由篩選，以篩選不想要的前置詞。 路由篩選是標準網路做法，在許多網路經常使用。

* 定義路由篩選，並將其套用到 ExpressRoute 線路。 路由篩選是新的資源，可讓您選取計劃透過 Microsoft 對等互連使用的服務清單。 ExpressRoute 路由器只會傳送屬於路由篩選中所識別之服務的前置詞清單。

在本教學課程中，您會了解如何：
> [!div class="checklist"]
> - 取得 BGP 社群值。
> - 建立路由篩選和篩選規則。
> - 將路由篩選關聯至 ExpressRoute 線路。

### <a name="about-route-filters"></a><a name="about"></a>關於路由篩選

在 ExpressRoute 線路上設定 Microsoft 對等互連後，Microsoft 邊緣路由器會透過您的連線提供者，建立一組 BGP 工作階段與邊緣路由器。 沒有路由會公告至您的網路。 若要讓路由公告至您的網路，您必須建立與路由篩選的關聯。

路由篩選可讓您識別想要透過 ExpressRoute 線路的 Microsoft 對等互連使用的服務。 這基本上是所有 BGP 社群值的允許清單。 一旦定義路由篩選資源，並且連結至 ExpressRoute 線路，對應到 BGP 社群值的所有前置詞都會公告至您的網路。

若要連結路由篩選與 Office 365 服務，您必須具有透過 ExpressRoute 取用 Microsoft 365 服務的授權。 如果您未獲授權透過 ExpressRoute 取用 Microsoft 365 服務，連結路由篩選的作業會失敗。 如需授權程序的詳細資訊，請參閱 [Azure ExpressRoute for Microsoft 365](/microsoft-365/enterprise/azure-expressroute)。

> [!IMPORTANT]
> 在 2017 年 8 月 1 日以前設定之 ExpressRoute 線路的 Microsoft 對等互連，會透過 Microsoft 對等互連公告所有服務首碼，即使未定義路由篩選也一樣。 在 2017 年 8 月 1 日當日或以後設定之 ExpressRoute 線路的 Microsoft 對等互連，不會公告任何首碼，直到路由篩選連結至線路為止。
> 
## <a name="prerequisites"></a>必要條件

- 開始設定之前，請檢閱[必要條件](expressroute-prerequisites.md)和[工作流程](expressroute-workflows.md)。

- 您必須具有已佈建 Microsoft 對等互連的使用中 ExpressRoute 線路。 您可以使用下列指示來完成這些工作：
  - 繼續之前，請[建立 ExpressRoute 線路](expressroute-howto-circuit-arm.md)，並由您的連線提供者來啟用該線路。 ExpressRoute 線路必須處於已佈建及已啟用的狀態。
  - [建立 Microsoft 對等互連](expressroute-circuit-peerings.md)，如果您直接管理 BGP 工作階段。 或者，讓您的連線提供者為您的線路佈建 Microsoft 對等互連。
  
[!INCLUDE [cloud-shell-try-it.md](../../includes/cloud-shell-try-it.md)]

### <a name="sign-in-to-your-azure-account-and-select-your-subscription"></a>登入您的 Azure 帳戶並且選取您的訂用帳戶

[!INCLUDE [sign in](../../includes/expressroute-cloud-shell-connect.md)]

## <a name="get-a-list-of-prefixes-and-bgp-community-values"></a><a name="prefixes"></a> 取得前置詞和 BGP 社群值的清單

1. 使用下列 Cmdlet 來取得與可透過 Microsoft 對等互連存取之服務相關聯的 BGP 社群值和前置詞清單：

    ```azurepowershell-interactive
    Get-AzBgpServiceCommunity
    ```

1. 製作您想要在路由篩選中使用的 BGP 社群值清單。

## <a name="create-a-route-filter-and-a-filter-rule"></a><a name="filter"></a>建立路由篩選和篩選規則

路由篩選只能有一個規則，且規則的類型必須是 'Allow'。 此規則可以具有與其相關聯的 BGP 社群值清單。 `az network route-filter create` 命令只會建立路由篩選資源。 建立資源之後，您必須建立規則，然後將它附加到路由篩選物件。

1. 若要建立路由篩選資源，請執行下列命令：

    ```azurepowershell-interactive
    New-AzRouteFilter -Name "MyRouteFilter" -ResourceGroupName "MyResourceGroup" -Location "West US"
    ```

1. 若要建立路由篩選規則，請執行下列命令：
 
    ```azurepowershell-interactive
    $rule = New-AzRouteFilterRuleConfig -Name "Allow-EXO-D365" -Access Allow -RouteFilterRuleType Community -CommunityList 12076:5010,12076:5040
    ```

1. 執行下列命令以將篩選規則新增至路由篩選：
 
    ```azurepowershell-interactive
    $routefilter = Get-AzRouteFilter -Name "RouteFilterName" -ResourceGroupName "ExpressRouteResourceGroupName"
    $routefilter.Rules.Add($rule)
    Set-AzRouteFilter -RouteFilter $routefilter
    ```

## <a name="attach-the-route-filter-to-an-expressroute-circuit"></a><a name="attach"></a>將路由篩選連結至 ExpressRoute 線路

若您只有 Microsoft 對等互連，請執行下列命令，將路由器篩選附加在 ExpressRoute 線路上：

```azurepowershell-interactive
$ckt = Get-AzExpressRouteCircuit -Name "ExpressRouteARMCircuit" -ResourceGroupName "ExpressRouteResourceGroup"
$ckt.Peerings[0].RouteFilter = $routefilter 
Set-AzExpressRouteCircuit -ExpressRouteCircuit $ckt
```

## <a name="common-tasks"></a><a name="tasks"></a>常見工作

### <a name="to-get-the-properties-of-a-route-filter"></a><a name="getproperties"></a>若要取得路由篩選的屬性

若要取得路由篩選的屬性，請使用下列步驟：

1. 執行下列命令以取得路由篩選資源：

   ```azurepowershell-interactive
   $routefilter = Get-AzRouteFilter -Name "RouteFilterName" -ResourceGroupName "ExpressRouteResourceGroupName"
   ```
2. 藉由執行下列命令以取得路由篩選資源的路由篩選規則：

   ```azurepowershell-interactive
   $routefilter = Get-AzRouteFilter -Name "RouteFilterName" -ResourceGroupName "ExpressRouteResourceGroupName"
   $rule = $routefilter.Rules[0]
   ```

### <a name="to-update-the-properties-of-a-route-filter"></a><a name="updateproperties"></a>若要更新路由篩選的屬性

如果路由篩選已連接到線路，BGP 社群清單更新會自動透過已建立的 BGP 工作階段傳播前置詞公告變更。 您可以使用下列命令來更新路由篩選的 BGP 社群清單：

```azurepowershell-interactive
$routefilter = Get-AzRouteFilter -Name "RouteFilterName" -ResourceGroupName "ExpressRouteResourceGroupName"
$routefilter.rules[0].Communities = "12076:5030", "12076:5040"
Set-AzRouteFilter -RouteFilter $routefilter
```

### <a name="to-detach-a-route-filter-from-an-expressroute-circuit"></a><a name="detach"></a>若要從 ExpressRoute 線路取消連結路由篩選

一旦從 ExpressRoute 線路取消連結路由篩選，就不會透過 BGP 工作階段公告任何前置詞。 您可以使用下列命令以從 ExpressRoute 線路取消連結路由篩選：
  
```azurepowershell-interactive
$ckt.Peerings[0].RouteFilter = $null
Set-AzExpressRouteCircuit -ExpressRouteCircuit $ckt
```

## <a name="clean-up-resources"></a>清除資源

您只能在路由篩選尚未連結至任何線路時刪除路由篩選。 請在嘗試刪除之前，確認路由篩選尚未連結至任何線路。 您可以使用下列命令來刪除路由篩選：

```azurepowershell-interactive
Remove-AzRouteFilter -Name "MyRouteFilter" -ResourceGroupName "MyResourceGroup"
```

## <a name="next-steps"></a>後續步驟

如需路由器設定範例的相關資訊，請參閱：

> [!div class="nextstepaction"]
> [設定和管理路由的路由器組態範例](expressroute-config-samples-routing.md)
