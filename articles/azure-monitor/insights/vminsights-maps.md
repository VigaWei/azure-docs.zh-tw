---
title: 使用適用於 VM 的 Azure 監視器檢視應用程式相依性
description: Map 是適用於 VM 的 Azure 監視器的功能。 它會自動探索 Windows 和 Linux 系統上的應用程式元件，並對應服務之間的通訊。 本文提供如何在各種案例中使用地圖功能的詳細資料。
ms.subservice: ''
ms.topic: conceptual
author: bwren
ms.author: bwren
ms.date: 03/20/2020
ms.openlocfilehash: 32ad92b333eda8cd429303177dba160f0b58af54
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 11/25/2020
ms.locfileid: "95985793"
---
# <a name="use-the-map-feature-of-azure-monitor-for-vms-to-understand-application-components"></a>使用適用於 VM 的 Azure 監視器的地圖功能來瞭解應用程式元件
在適用於 VM 的 Azure 監視器中，您可以在 Windows 和 Linux 虛擬機器上，查看在 Azure 或您的環境中執行 (Vm) 上探索到的應用程式元件。 您可以透過兩種方式來觀察 Vm。 直接從 VM 查看對應，或從 Azure 監視器查看地圖，以查看跨 Vm 群組的元件。 本文將協助您瞭解這兩種視圖方法，以及如何使用地圖功能。 

如需設定適用於 VM 的 Azure 監視器相關資訊，請參閱[啟用適用於 VM 的 Azure 監視器](vminsights-enable-overview.md)。

## <a name="sign-in-to-azure"></a>登入 Azure
登入 [Azure 入口網站](https://portal.azure.com)。

## <a name="introduction-to-the-map-experience"></a>地圖體驗簡介
深入瞭解地圖體驗之前，您應該先瞭解它如何呈現和視覺化資訊。 無論您是直接從 VM 或從 Azure 監視器選取對應功能，地圖功能都會提供一致的體驗。 唯一的差別在於，從 Azure 監視器，一個對應會顯示多層式應用程式或叢集的所有成員。

地圖功能會藉由探索執行中的處理常式，以視覺化方式呈現 VM 相依性： 

- 伺服器之間的作用中網路連接。
- 輸入和輸出連線延遲。
- 在指定的時間範圍內，跨任何 TCP 連接架構的埠。  
 
展開 VM 以顯示處理常式詳細資料，以及僅顯示與 VM 通訊的處理常式。 用戶端群組會顯示連接到 VM 的前端用戶端計數。 伺服器埠群組會顯示 VM 連接的後端伺服器計數。 展開伺服器埠群組，以查看透過該埠連接的詳細伺服器清單。  

當您選取 VM 時，右側的 [ **屬性** ] 窗格會顯示 vm 的屬性。 屬性包含作業系統所報告的系統資訊、Azure VM 的屬性，以及摘要探索到的連接的環圈圖。 

![[屬性] 窗格](./media/vminsights-maps/properties-pane-01.png)

在窗格的右邊，選取 [ **記錄事件** ] 以顯示 VM 傳送給 Azure 監視器的資料清單。 這項資料可供查詢。  選取 [任何記錄類型] 以開啟 **[記錄] 頁面，** 您可以在其中看到該記錄類型的結果。 您也會看到針對 VM 進行篩選的預先設定查詢。  

![[記錄事件] 窗格](./media/vminsights-maps/properties-pane-logs-01.png)

關閉 [ **記錄** 檔] 頁面，並返回 [ **屬性** ] 窗格。 在該處選取 [ **警示** ]，以查看 VM 健全狀況-準則警示。 地圖功能會與 Azure 警示整合，以顯示所選時間範圍內所選伺服器的警示。 伺服器會顯示目前警示的圖示，且 [ **機器警示** ] 窗格會列出警示。 

![警示窗格](./media/vminsights-maps/properties-pane-alerts-01.png)

若要讓對應功能顯示相關警示，請建立適用于特定電腦的警示規則：

- 包含依電腦分組警示的子句 (例如， **依電腦間隔1分鐘**) 。
- 以計量為基礎的警示。

如需有關 Azure 警示和建立警示規則的詳細資訊，請參閱 [Azure 監視器中的整合警示](../platform/alerts-overview.md)。

在右上角， **圖例** 選項描述地圖上的符號和角色。 若要深入瞭解地圖並四處移動，請使用右下角的縮放控制項。 您可以設定縮放層級，並將地圖調整為頁面大小。  

## <a name="connection-metrics"></a>連接計量
[ **連線] 窗格** 會顯示從 VM 透過 TCP 埠所選取連線的標準計量。 計量包括回應時間、每分鐘的要求、流量輸送量及連結。  

![[連線] 窗格上的網路連接圖](./media/vminsights-maps/map-group-network-conn-pane-01.png)  

### <a name="failed-connections"></a>失敗的連線
地圖會顯示處理常式和電腦的失敗連接。 紅色虛線表示用戶端系統無法連線到進程或埠。 針對使用相依性代理程式的系統，代理程式會報告連接嘗試失敗。 地圖功能會監觀察無法建立連線的 TCP 通訊端來監視處理常式。 此失敗的原因可能是防火牆、用戶端或伺服器的設定不正確，或遠端服務無法使用。

![地圖上的連接失敗](./media/vminsights-maps/map-group-failed-connection-01.png)

瞭解失敗的連接可協助您進行疑難排解、驗證遷移、分析安全性，以及瞭解服務的整體架構。 失敗的連線有時無害，但它們通常會指向問題。 例如，當容錯移轉環境突然變成無法連線，或兩個應用層在雲端遷移之後無法彼此通訊時，連接可能會失敗。

### <a name="client-groups"></a>用戶端群組
在對應上，用戶端群組代表連接至對應電腦的用戶端電腦。 單一用戶端群組代表個別進程或電腦的用戶端。

![地圖上的用戶端群組](./media/vminsights-maps/map-group-client-groups-01.png)

若要查看受監視的用戶端和用戶端群組中系統的 IP 位址，請選取群組。 群組的內容如下所示。  

![用戶端群組在對應上的 IP 位址清單](./media/vminsights-maps/map-group-client-group-iplist-01.png)

如果群組包含受監視和未受監視的用戶端，您可以選取群組環圈圖的適當區段來篩選用戶端。

### <a name="server-port-groups"></a>伺服器埠群組
伺服器埠群組代表伺服器上具有來自對應電腦之輸入連接的埠。 此群組包含伺服器埠，以及與該埠連線的伺服器數目計數。 選取群組以查看個別的伺服器和連接。 

![地圖上的伺服器埠群組](./media/vminsights-maps/map-group-server-port-groups-01.png)  

如果群組包含受監視和未受監視的伺服器，您可以選取群組環圈圖的適當區段來篩選伺服器。

## <a name="view-a-map-from-a-vm"></a>從 VM 查看對應 

若要直接從 VM 存取適用於 VM 的 Azure 監視器：

1. 在 Azure 入口網站中，選取 [虛擬機器]。 
2. 從清單中選擇 VM。 在 [ **監視** ] 區段中，選擇 [ **見解**]。  
3. 選取 [對應] 索引標籤。

地圖會藉由探索在指定時間範圍內有作用中網路連線的執行中進程群組和進程，以視覺化方式呈現 VM 的相依性。  

根據預設，對應會顯示過去 30 分鐘。 如果您想要查看相依性在過去的樣子，可以查詢最多一小時的歷程記錄時間範圍。 若要執行查詢，請使用左上角的 **TimeRange** 選取器。 例如，您可以在事件期間執行查詢，或在變更前查看狀態。  

![顯示虛擬機器之間相依性圖表的 Azure 入口網站的 [對應] 索引標籤螢幕擷取畫面。](./media/vminsights-maps/map-direct-vm-01.png)

## <a name="view-a-map-from-a-virtual-machine-scale-set"></a>從虛擬機器擴展集查看對應

若要直接從虛擬機器擴展集存取適用於 VM 的 Azure 監視器：

1. 在 Azure 入口網站中，選取 [虛擬機器擴展集]。
2. 從清單中選擇 VM。 然後，在 [ **監視** ] 區段中，選擇 [ **見解**]。  
3. 選取 [對應] 索引標籤。

地圖會將擴展集中的所有實例視覺化為群組節點，以及群組的相依性。 展開的節點會列出擴展集中的實例。 您可以一次滾動這些實例10。 

若要載入特定實例的對應，請先在地圖上選取該實例。 然後選取 **省略號** 按鈕 ( ... ) 至右側，然後選擇 [ **載入伺服器對應**]。 在出現的對應中，您會看到進程群組和處理常式，其在指定的時間範圍內具有作用中的網路連線。 

根據預設，對應會顯示過去 30 分鐘。 如果您想要查看相依性在過去的樣子，可以查詢最多一小時的歷程記錄時間範圍。 若要執行查詢，請使用 **TimeRange** 選取器。 例如，您可以在事件期間執行查詢，或在變更前查看狀態。

![顯示虛擬機器擴展集之間相依性圖表的 Azure 入口網站的 [對應] 索引標籤螢幕擷取畫面。](./media/vminsights-maps/map-direct-vmss-01.png)

>[!NOTE]
>您也可以從虛擬機器擴展集的 **實例** 視圖，存取特定實例的對應。 在 [**設定**] 區段中，移至 [**實例**  >  **見解**]。

## <a name="view-a-map-from-azure-monitor"></a>從 Azure 監視器觀看地圖

在 Azure 監視器中，地圖功能可提供 Vm 的全域觀點及其相依性。 若要存取 Azure 監視器中的對應功能：

1. 在 Azure 入口網站中，選取 [監視]。 
2. 在 [ **見解** ] 區段中，選擇 [ **虛擬機器**]。
3. 選取 [對應] 索引標籤。

   ![Azure 監視器多個 Vm 的總覽圖](./media/vminsights-maps/map-multivm-azure-monitor-01.png)

使用頁面頂端的 **工作區** 選取器，選擇工作區。 如果您有多個 Log Analytics 工作區，請選擇已啟用解決方案的工作區，並為其報告 Vm。 

**群組** 選取器會傳回與選取的工作區相關的訂用帳戶、資源群組、[電腦群組](../platform/computer-groups.md)，以及電腦的虛擬機器擴展集。 您的選取專案只適用于地圖功能，不會延續到效能或健全狀況。

根據預設，對應會顯示過去 30 分鐘。 如果您想要查看相依性在過去的樣子，可以查詢最多一小時的歷程記錄時間範圍。 若要執行查詢，請使用 **TimeRange** 選取器。 例如，您可以在事件期間執行查詢，或在變更前查看狀態。  

## <a name="next-steps"></a>後續步驟

若要找出瓶頸、檢查效能，並瞭解 Vm 的整體使用率，請參閱適用於 VM 的 Azure 監視器的 [ [查看效能狀態](vminsights-performance.md)]。 

