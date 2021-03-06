---
title: Azure ExpressRoute：驗證連線能力-疑難排解指南
description: 此頁面提供 ExpressRoute 路線的端對端連線確認和疑難排解的指示。
services: expressroute
author: duongau
ms.service: expressroute
ms.topic: troubleshooting
ms.date: 10/31/2019
ms.author: duau
ms.custom: seodec18
ms.openlocfilehash: af4ef156cccded6afe2db09628446a6ffe1ad53a
ms.sourcegitcommit: 9826fb9575dcc1d49f16dd8c7794c7b471bd3109
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 11/14/2020
ms.locfileid: "92204634"
---
# <a name="verifying-expressroute-connectivity"></a>確認 ExpressRoute 連線
本文將協助您確認 ExpressRoute 連線及針對連線問題進行疑難排解。 ExpressRoute 透過私人連線將內部部署網路延伸至 Microsoft 雲端，這通常是由連線提供者所提供。 ExpressRoute 連線通常牽涉到三個不同的網路區域，如下所示：

-   客戶網路
-   提供者網路
-   Microsoft 資料中心

> [!NOTE]
> 在 ExpressRoute direct 連線模型 (在 10/100 Gbps 頻寬) 提供，客戶可以直接連接到 Microsoft Enterprise Edge (MSEE) 路由器的埠。 因此，在直接連線模式中，只有客戶和 Microsoft 網路區域。
>


本檔的目的是要協助使用者找出連線問題是否存在以及是否存在。 因此，協助向適當的小組尋求支援來解決問題。 如果需要 Microsoft 支援服務解決問題，請利用 [Microsoft 支援服務][Support]開啟支援票證。

> [!IMPORTANT]
> 本文件旨在協助診斷並修正簡單的問題。 它無法取代 Microsoft 支援服務。 如果無法使用本文提供的指引來解決問題，請利用 [Microsoft 支援服務][Support]開啟支援票證。
>
>

## <a name="overview"></a>概觀
下圖顯示使用 ExpressRoute 從客戶網路連至 Microsoft 網路的邏輯連線。
[![1]][1]

在上圖中，數字指出重要的網路點。 本文會根據相關聯的數目，在本文中參考這些網路點。 根據 ExpressRoute 連線模型--雲端 Exchange 共置、點對點乙太網路連線或任意對任何 (IPVPN) --網路點3和4可能會切換 (第2層裝置) 或路由器 (第3層裝置) 。 在直接連線模型中，沒有網路點3和 4;相反地，CEs (2) 透過深色光纖直接連線至 Msee。 圖中的重要網路點分別是︰

1.  客戶計算裝置 (例如，伺服器或電腦)
2.  CE︰客戶邊緣路由器 
3.  PE (面對 CE)︰面對客戶邊緣路由器的提供者邊緣路由器/交換器。 在本文件中稱為 PE-CE。
4.  PE (面對 MSEE)︰面對 MSEE 的提供者邊緣路由器/交換器。 在本文件中稱為 PE-MSEE。
5.  MSEE：Microsoft Enterprise Edge (MSEE) ExpressRoute 路由器
6.  虛擬網路 (VNet) 閘道器
7.  Azure VNet 上的計算裝置

如果使用雲端交換共置、點對點乙太網路或直接連線模型，CEs (2) 使用 Msee (5) 建立 BGP 對等互連。 

如果使用任何 (IPVPN) 連線模型，PE-MSEEs (4) 使用 Msee (5) 建立 BGP 對等互連。 PE-MSEEs 透過 IPVPN 服務提供者網路，將接收自 Microsoft 的路由傳回到客戶網路。

> [!NOTE]
>為了達到高可用性，Microsoft 會在 Msee (5) 和 PE-MSEEs (4) 配對之間，建立完全重複的平行連線。 在客戶網路和 PE-CEs 配對之間，也鼓勵使用完全重複的平行網路路徑。 如需有關高可用性的詳細資訊，請參閱[使用 ExpressRoute 設計高可用性][HA]的文章
>
>

以下是針對 ExpressRoute 線路進行疑難排解的邏輯步驟：

* [確認線路布建和狀態](#verify-circuit-provisioning-and-state)
  
* [確認對等互連組態](#validate-peering-configuration)
  
* [驗證 ARP](#validate-arp)
  
* [確認 MSEE 上的 BGP 和路由](#validate-bgp-and-routes-on-the-msee)
  
* [確認流量流程](#confirm-the-traffic-flow)


## <a name="verify-circuit-provisioning-and-state"></a>確認線路布建和狀態
布建 ExpressRoute 線路會在 CEs/PE-Msee (2) / (4) 和 Msee (5) 之間建立冗余的第2層連線。 如需有關如何建立、修改、佈建、確認 ExpressRoute 路線的詳細資訊，請參閱[建立和修改 ExpressRoute 路線][CreateCircuit]一文。

>[!TIP]
>服務機碼可唯一識別 ExpressRoute 路線。 如果您需要來自 Microsoft 或 ExpressRoute 合作夥伴的協助，以針對 ExpressRoute 問題進行疑難排解，請提供服務金鑰來立即識別線路。
>
>

### <a name="verification-via-the-azure-portal"></a>透過 Azure 入口網站進行確認
在 Azure 入口網站中，開啟 [ExpressRoute 線路] 分頁。 在分頁的 ![3][3] 區段中，會列出 ExpressRoute 基本功能，如下列螢幕擷取畫面所示：

![4][4]    

在 ExpressRoute [基本資料] 中，[路線狀態] 指出 Microsoft 端路線的狀態。 [提供者狀態] 指出在服務提供者端是否「已佈建/未佈建」路線。 

[路線狀態] 必須是 [已啟用]，且[提供者狀態] 必須是 [已佈建]，ExpressRoute 路線才能運作。

> [!NOTE]
> 設定 ExpressRoute 線路之後，如果 *線路狀態* 為 [未啟用] 狀態，請聯絡 [Microsoft 支援服務][Support]。 另一方面，如果 *提供者狀態* 為 [未布建] 狀態，請洽詢您的服務提供者。
>
>

### <a name="verification-via-powershell"></a>透過 PowerShell 進行確認
若要列出資源群組中的所有 ExpressRoute 路線，使用下列命令：

```azurepowershell
Get-AzExpressRouteCircuit -ResourceGroupName "Test-ER-RG"
```

>[!TIP]
>如果您要尋找資源群組的名稱，您可以使用 *>new-azresourcegroup* 命令，列出訂用帳戶中的所有資源群組，以取得資源群組的名稱。
>


若要選取資源群組中的特定 ExpressRoute 路線，使用下列命令：

```azurepowershell
Get-AzExpressRouteCircuit -ResourceGroupName "Test-ER-RG" -Name "Test-ER-Ckt"
```

範例回應：

```output
Name                             : Test-ER-Ckt
ResourceGroupName                : Test-ER-RG
Location                         : westus2
Id                               : /subscriptions/***************************/resourceGroups/Test-ER-RG/providers/***********/expressRouteCircuits/Test-ER-Ckt
Etag                             : W/"################################"
ProvisioningState                : Succeeded
Sku                              : {
                                    "Name": "Standard_UnlimitedData",
                                    "Tier": "Standard",
                                    "Family": "UnlimitedData"
                                    }
CircuitProvisioningState         : Enabled
ServiceProviderProvisioningState : Provisioned
ServiceProviderNotes             : 
ServiceProviderProperties        : {
                                    "ServiceProviderName": "****",
                                    "PeeringLocation": "******",
                                    "BandwidthInMbps": 100
                                    }
ServiceKey                       : **************************************
Peerings                         : []
Authorizations                   : []
```

若要確認 ExpressRoute 路線是否在運作，請特別注意下列欄位︰

```output
CircuitProvisioningState         : Enabled
ServiceProviderProvisioningState : Provisioned
```

> [!NOTE]
> 設定 ExpressRoute 線路之後，如果 *線路狀態* 為 [未啟用] 狀態，請聯絡 [Microsoft 支援服務][Support]。 另一方面，如果 *提供者狀態* 為 [未布建] 狀態，請洽詢您的服務提供者。
>
>

## <a name="validate-peering-configuration"></a>確認對等互連組態
在服務提供者完成布建 ExpressRoute 線路之後，您可以在 CEs/MSEE-Pe (2) / (4) 和 Msee (5) 之間，透過 ExpressRoute 線路建立多個以 eBGP 為基礎的路由設定。 每個 ExpressRoute 線路可以有： Azure 私人對等互連 (Azure 中私人虛擬網路的流量) 和/或 Microsoft 對等互連 (PaaS 和 SaaS) 公用端點的流量。 如需有關如何建立及修改路由組態的詳細資訊，請參閱[建立和修改 ExpressRoute 路線的路由][CreatePeering]一文。

### <a name="verification-via-the-azure-portal"></a>透過 Azure 入口網站進行確認

> [!NOTE]
> 在 IPVPN 連接模型中，服務提供者負責設定對等互連 (第3層服務) 。 在這種模型中，服務提供者已設定對等互連，且入口網站中的對等互連是空白的，請嘗試使用入口網站上的 [重新整理] 按鈕來重新整理電路設定。 此操作會從您的線路提取目前的路由設定。 
>

在 Azure 入口網站中，ExpressRoute 線路對等互連的狀態可以在 ExpressRoute 線路分頁下進行檢查。 在分頁的 ![3][3] 區段中，會列出 ExpressRoute 對等互連，如下列螢幕擷取畫面所示：

![5][5]

在上述範例中，我們已布建 azure 私人對等互連，而不會布建 Azure 公用和 Microsoft 對等互連。 成功布建的對等互連內容也會列出主要和次要點對點子網。 /30 子網是用於 Msee 和 CEs/PE Msee 的介面 IP 位址。 針對布建的對等互連，清單也會指出上次修改設定的人員。 

> [!NOTE]
> 如果啟用對等互連失敗，請檢查指派的主要和次要子網是否符合連結之 CE/PE MSEE 上的設定。 也請檢查 Msee 上是否使用正確的 *VlanId* 、 *AzureASN* 和 *PeerASN* ，以及這些值是否對應至連結的 CE/PE-MSEE 上所使用的值。 如果選擇 MD5 雜湊，MSEE 和 MSEE/CE 配對上的共用金鑰應該相同。 基於安全考慮，將不會顯示先前設定的共用金鑰。 如果您需要變更 MSEE 路由器上的任何設定，請參閱 [建立和修改 ExpressRoute 線路的路由][CreatePeering]。  
>

> [!NOTE]
> 在指派給介面的/30 子網中，Microsoft 會為 MSEE 介面挑選子網的第二個可用 IP 位址。 因此，請確定子網的第一個可用 IP 位址已在對等互連 CE/PE MSEE 上指派。
>


### <a name="verification-via-powershell"></a>透過 PowerShell 進行確認
若要取得 Azure 私人對等互連的組態詳細資料，使用下列命令︰

```azurepowershell
$ckt = Get-AzExpressRouteCircuit -ResourceGroupName "Test-ER-RG" -Name "Test-ER-Ckt"
Get-AzExpressRouteCircuitPeeringConfig -Name "AzurePrivatePeering" -ExpressRouteCircuit $ckt
```

以下是已成功設定私人對等互連的回應範例︰

```output
Name                       : AzurePrivatePeering
Id                         : /subscriptions/***************************/resourceGroups/Test-ER-RG/providers/***********/expressRouteCircuits/Test-ER-Ckt/peerings/AzurePrivatePeering
Etag                       : W/"################################"
PeeringType                : AzurePrivatePeering
AzureASN                   : 12076
PeerASN                    : 123##
PrimaryPeerAddressPrefix   : 172.16.0.0/30
SecondaryPeerAddressPrefix : 172.16.0.4/30
PrimaryAzurePort           : 
SecondaryAzurePort         : 
SharedKey                  : 
VlanId                     : 200
MicrosoftPeeringConfig     : null
ProvisioningState          : Succeeded
```

 成功啟用的對等互連內容也會列出主要和次要位址前置詞。 /30 子網是用於 Msee 和 CEs/PE Msee 的介面 IP 位址。

若要取得 Azure 公用對等互連的組態詳細資料，使用下列命令︰

```azurepowershell
$ckt = Get-AzExpressRouteCircuit -ResourceGroupName "Test-ER-RG" -Name "Test-ER-Ckt"
Get-AzExpressRouteCircuitPeeringConfig -Name "AzurePublicPeering" -ExpressRouteCircuit $ckt
```

若要取得 Microsoft 對等互連的組態詳細資料，使用下列命令︰

```azurepowershell
$ckt = Get-AzExpressRouteCircuit -ResourceGroupName "Test-ER-RG" -Name "Test-ER-Ckt"
Get-AzExpressRouteCircuitPeeringConfig -Name "MicrosoftPeering" -ExpressRouteCircuit $ckt
```

如未設定對等互連，會顯示錯誤訊息。 未設定所述對等互連 (在此範例中是 Azure 公用互對等連) 的回應範例︰

```azurepowershell
Get-AzExpressRouteCircuitPeeringConfig : Sequence contains no matching element
At line:1 char:1
    + Get-AzExpressRouteCircuitPeeringConfig -Name "AzurePublicPeering ...
    + ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
        + CategoryInfo          : CloseError: (:) [Get-AzExpr...itPeeringConfig], InvalidOperationException
        + FullyQualifiedErrorId : Microsoft.Azure.Commands.Network.GetAzureExpressRouteCircuitPeeringConfigCommand
```

> [!NOTE]
> 如果啟用對等互連失敗，請檢查指派的主要和次要子網是否符合連結之 CE/PE MSEE 上的設定。 也請檢查 Msee 上是否使用正確的 *VlanId* 、 *AzureASN* 和 *PeerASN* ，以及這些值是否對應至連結的 CE/PE-MSEE 上所使用的值。 如果選擇 MD5 雜湊，MSEE 和 MSEE/CE 配對上的共用金鑰應該相同。 基於安全考慮，將不會顯示先前設定的共用金鑰。 如果您需要變更 MSEE 路由器上的任何設定，請參閱 [建立和修改 ExpressRoute 線路的路由][CreatePeering]。  
>
>

> [!NOTE]
> 在指派給介面的/30 子網中，Microsoft 會為 MSEE 介面挑選子網的第二個可用 IP 位址。 因此，請確定子網的第一個可用 IP 位址已在對等互連 CE/PE MSEE 上指派。
>

## <a name="validate-arp"></a>驗證 ARP

ARP 表格提供特定對等互連的 IP 位址和 MAC 位址對應。 適用於 ExpressRoute 線路對等互連的 ARP 表格能提供各個介面 (主要和次要) 的下列資訊：
* 內部部署路由器介面 IP 位址到 MAC 位址的對應
* ExpressRoute 路由器介面 IP 位址到 MAC 位址的對應
* 對應 ARP 資料表的存留期可協助驗證第2層設定，並針對基本第2層連線問題進行疑難排解。


請參閱 [Resource Manager 部署模型檔中的 arp 表格][ARP] 、如何查看 ExpressRoute 對等互連的 arp 表格，以及如何使用此資訊來針對第2層連線問題進行疑難排解。


## <a name="validate-bgp-and-routes-on-the-msee"></a>確認 MSEE 上的 BGP 和路由

若要從 MSEE 取得 *私人* 路由內容 *主要* 路徑上的路由表，請使用下列命令：

```azurepowershell
Get-AzExpressRouteCircuitRouteTable -DevicePath Primary -ExpressRouteCircuitName ******* -PeeringType AzurePrivatePeering -ResourceGroupName ****
```

以下是回應範例：

```output
Network : 10.1.0.0/16
NextHop : 10.17.17.141
LocPrf  : 
Weight  : 0
Path    : 65515

Network : 10.1.0.0/16
NextHop : 10.17.17.140*
LocPrf  : 
Weight  : 0
Path    : 65515

Network : 10.2.20.0/25
NextHop : 172.16.0.1
LocPrf  : 
Weight  : 0
Path    : 123##
```

> [!NOTE]
> 如果 MSEE 和 CE/PE MSEE 之間的 eBGP 對等互連狀態為 [作用中] 或 [閒置]，請檢查指派的主要和次要對等互連子網是否符合連結之 CE/PE MSEE 上的設定。 也請檢查 Msee 上是否使用正確的 *VlanId* 、 *AzureAsn* 和 *PeerAsn* ，以及這些值是否對應至連結的 PE-MSEE/CE 上所使用的值。 如果選擇 MD5 雜湊，MSEE 和 CE/PE MSEE 配對上的共用金鑰應該相同。 如果您需要變更 MSEE 路由器上的任何設定，請參閱 [建立和修改 ExpressRoute 線路的路由][CreatePeering]。
>


> [!NOTE]
> 如果無法透過對等互連連線到特定目的地，請檢查 Msee 的路由表以找出對應的對等互連內容。 如果對應的首碼 (可能是 NATed IP) 出現在路由表中，請檢查封鎖流量的路徑上是否有防火牆/NSG/Acl。
>


下列範例會顯示不存在對等互連的命令回應：

```azurepowershell
Get-AzExpressRouteCircuitRouteTable : The BGP Peering AzurePublicPeering with Service Key ********************* is not found.
StatusCode: 400
```

## <a name="confirm-the-traffic-flow"></a>確認流量流程
若要取得對等互連內容的主要和次要路徑的合併流量統計資料，包括進和出的位元組，使用下列命令︰

```azurepowershell
Get-AzExpressRouteCircuitStats -ResourceGroupName $RG -ExpressRouteCircuitName $CircuitName -PeeringType 'AzurePrivatePeering'
```

此命令的輸出範例如下：

```output
PrimaryBytesIn PrimaryBytesOut SecondaryBytesIn SecondaryBytesOut
-------------- --------------- ---------------- -----------------
     240780020       239863857        240565035         239628474
```

對等互連不存在時，命令的輸出範例如下︰

```azurepowershell
Get-AzExpressRouteCircuitRouteTable : The BGP Peering AzurePublicPeering with Service Key ********************* is not found.
StatusCode: 400
```

## <a name="next-steps"></a>後續步驟
如需詳細資訊或協助，請看看下列連結：

- [Microsoft 支援服務][Support]
- [建立和修改 ExpressRoute 線路][CreateCircuit]
- [建立和修改 ExpressRoute 路線的路由][CreatePeering]

<!--Image References-->
[1]: ./media/expressroute-troubleshooting-expressroute-overview/expressroute-logical-diagram.png "邏輯 Express 路由連線"
[2]: ./media/expressroute-troubleshooting-expressroute-overview/portal-all-resources.png "所有資源圖示"
[3]: ./media/expressroute-troubleshooting-expressroute-overview/portal-overview.png "概觀圖示"
[4]: ./media/expressroute-troubleshooting-expressroute-overview/portal-circuit-status.png "ExpressRoute 基本資料螢幕擷取畫面範例"
[5]: ./media/expressroute-troubleshooting-expressroute-overview/portal-private-peering.png "ExpressRoute 基本資料螢幕擷取畫面範例"

<!--Link References-->
[Support]: https://portal.azure.com/?#blade/Microsoft_Azure_Support/HelpAndSupportBlade
[CreateCircuit]: ./expressroute-howto-circuit-portal-resource-manager.md
[CreatePeering]: ./expressroute-howto-routing-portal-resource-manager.md
[ARP]: ./expressroute-troubleshooting-arp-resource-manager.md
[HA]: ./designing-for-high-availability-with-expressroute.md
[DR-Pvt]: ./designing-for-disaster-recovery-with-expressroute-privatepeering.md