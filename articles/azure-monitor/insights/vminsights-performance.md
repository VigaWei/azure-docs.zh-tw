---
title: 如何使用適用於 VM 的 Azure 監視器來繪製效能圖表
description: 效能是適用於 VM 的 Azure 監視器的一項功能，可自動探索 Windows 和 Linux 系統上的應用程式元件，並對應服務之間的通訊。 本文提供如何將它用於各種案例的詳細資料。
ms.subservice: ''
ms.topic: conceptual
author: bwren
ms.author: bwren
ms.date: 05/31/2020
ms.openlocfilehash: f9578fadfbe057b723af63e338bf8bda63cf6f21
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/09/2020
ms.locfileid: "91330905"
---
# <a name="how-to-chart-performance-with-azure-monitor-for-vms"></a>如何使用適用於 VM 的 Azure 監視器來繪製效能圖表

適用於 VM 的 Azure 監視器包含一組以關鍵效能指標 (KPI) 為目標的效能圖表，可協助您判斷虛擬機器的執行狀況。 那些圖表會顯示一段時間的資源使用率，讓您能夠找出瓶頸、異常狀況，或切換至列出每部機器的檢視方塊，以根據所選計量來檢視資源使用率。 雖然在處理效能時要考量許多元素，不過適用於 VM 的 Azure 監視器著重於監視與處理器、記憶體、網路介面卡和磁碟使用率相關的主要作業系統指標。 效能可彌補健康狀態監視功能，並有助於揭示可能導致系統元件故障的問題、支援調整和最佳化以實現效率，或支援容量規劃。  

## <a name="limitations"></a>限制
以下是適用於 VM 的 Azure 監視器的效能收集限制。

- **可用記憶體**不適用於執行 Red Hat Linux (RHEL) 6 的虛擬機器。 此計量是根據[核心版本 3.14](http://www.man7.org/linux/man-pages/man1/free.1.html) 引進的 **MemAvailable** 計算而得。
- 計量僅適用于使用 XFS filesystem 或 EXT filesystem 系列 (EXT2、EXT3、EXT4) 的 Linux 虛擬機器上的資料磁片。

## <a name="multi-vm-perspective-from-azure-monitor"></a>Azure 監視器的多 VM 檢視方塊

在 Azure 監視器中，效能功能提供在您訂閱或環境中的工作群組之間部署的所有受監視 VM 的檢視。 若要從 Azure 監視器存取，請執行下列步驟。 

1. 在 Azure 入口網站中，選取 [監視]。 
2. 選擇 [解決方案] 區段中的 [虛擬機器]。
3. 選取 [效能] 索引標籤。

![VM Insights 效能 N 大排行榜檢視](media/vminsights-performance/vminsights-performance-aggview-01.png)

在 [N 大排行榜] 索引標籤上，如果您具有多個 Log Analytics 工作區，請從頁面頂端的 [工作區] 選取器選擇已可處理解決方案的工作區。 [群組] 選取器會傳回與選取的工作區有關的訂用帳戶、資源群組、[電腦群組](../platform/computer-groups.md)與電腦的虛擬機器擴展集，供您進一步篩選此頁面上的圖表所顯示的結果，和其他頁面上的結果。 您的選取只會套用至「效能」功能，而不會擴及「健康情況」或「對應」。  

根據預設，圖表會顯示過去 24 小時。 使用 [TimeRange] 選取器，您可以查詢最多 30 天的歷程記錄時間範圍，以顯示過去的效能狀況。

頁面上顯示的五個容量使用率圖表包括：

* CPU 使用率 % - 顯示平均處理器使用率最高的前五部機器 
* 可用記憶體 - 顯示平均可用記憶體數量最低的前五部機器 
* 邏輯磁碟空間使用量 % - 顯示所有磁碟區中平均磁碟空間使用量 % 最高的前五部機器 
* 位元組傳送率 - 顯示已傳送位元組平均最高的前五部機器 
* 位元組接收率 - 顯示已接收位元組平均最高的前五部機器 

按一下五個圖表中任一圖表右上角的釘選圖示，即可將選取的圖表釘選至您上次查看的最後一個 Azure 儀表板。  您可以在儀表板中調整圖表大小，並重新置放圖表。 從儀表板選取圖表會將您重新導向適用於 VM 的 Azure 監視器，並載入正確的範圍和檢視。  

按一下位於五個圖表中任一圖表的釘選圖示左側的圖示，開啟 [N 大排行榜] 檢視。  此處，您可以在清單檢視中查看個別 VM 的該效能計量的資源使用率，以及哪一部機器趨勢最高。  

![所選效能計量的 N 大排行榜檢視](media/vminsights-performance/vminsights-performance-topnlist-01.png)

當您按一下虛擬機器時，[屬性] 窗格會在右側展開以顯示所選取項目的屬性，例如，作業系統所報告的系統資訊、Azure VM 的屬性等。按一下 [快速連結] 區段下的其中一個選項，會將您從所選 VM 直接重新導向到該功能。  

![虛擬機器屬性窗格](./media/vminsights-performance/vminsights-properties-pane-01.png)

切換到 [彙總圖表] 索引標籤可檢視依據平均或百分位數量值所篩選的效能計量。  

![VM Insights 效能彙總檢視](./media/vminsights-performance/vminsights-performance-aggview-02.png)

提供下列容量使用率圖表：

* CPU 使用率 % - 預設顯示的平均值和前 95 個百分位數 
* 可用記憶體 - 預設顯示平均值、前 5 個和前 10 個百分位數 
* 邏輯磁碟空間使用量 % - 預設顯示平均和前 95 個百分位數 
* 位元組傳送率 - 預設顯示平均已傳送位元組 
* 位元組接收率 - 預設顯示平均已接收位元組

您也可以在百分位數選取器中選取 [Avg]、[Min]、[Max]、[50th]、[90th] 和 [95th]，來變更時間範圍內的圖表細微度。

若要在清單檢視中檢視個別 VM 的資源使用率，並查看哪一部機器的使用率趨勢最高，請選取 [N 大排行榜] 索引標籤。[N 大排行榜] 頁面會顯示依照 [CPU 使用率 %] 計量前 95 個百分位數使用最多來排序的前 20 部機器。  您可以選取 [載入更多] 來檢視更多機器，而且其結果會展開以顯示前 500 部機器。 

>[!NOTE]
>清單無法一次顯示超過 500 部機器。  
>

![N 大排行榜頁面範例](./media/vminsights-performance/vminsights-performance-topnlist-01.png)

若要篩選清單中特定虛擬機器的結果，請在 [依名稱搜尋] 文字方塊中輸入其電腦名稱。  

如果您想要檢視不同效能計量的使用率，請從 [計量] 下拉式清單中選取 [可用記憶體]、[邏輯磁碟空間使用量 %]、[網路已接收位元組/s] 或 [網路已傳送位元組/s] 和清單更新，以顯示該計量範圍的使用率。  

從清單中選取虛擬機器會開啟頁面右側的 [屬性] 面板，您可以在此處選取 [效能詳細資料]。  [虛擬機器詳細資料] 頁面隨即開啟，而且範圍限定在該 VM，類似於直接從 Azure VM 存取 VM Insights 效能時的體驗。  

## <a name="view-performance-directly-from-an-azure-vm"></a>直接從 Azure VM 檢視效能

若要從虛擬機器直接存取，請執行下列步驟。

1. 在 Azure 入口網站中，選取 [虛擬機器]。 
2. 從清單中選擇 VM，然後在 [監視] 區段中選擇 [Insights]。  
3. 選取 [效能] 索引標籤。 

此頁面不僅包含效能使用率圖表，也是一個表格，可顯示已找到的每個邏輯磁碟、其容量、使用率，以及每個量值的總平均。  

提供下列容量使用率圖表：

* CPU 使用率 % - 預設顯示的平均值和前 95 個百分位數 
* 可用記憶體 - 預設顯示平均值、前 5 個和前 10 個百分位數 
* 邏輯磁碟空間使用量 % - 預設顯示平均和前 95 個百分位數 
* 邏輯磁碟 IOPS - 預設顯示平均和前 95 個百分位數
* 邏輯磁碟 MB/s - 預設顯示平均和前 95 個百分位數
* 最大邏輯磁碟使用量 - 預設顯示平均和前 95 個百分位數
* 位元組傳送率 - 預設顯示平均已傳送位元組 
* 位元組接收率 - 預設顯示平均已接收位元組

按一下任一圖表右上角的釘選圖示，即可將選取的圖表釘選至您查看的最後一個 Azure 儀表板。 您可以在儀表板中調整圖表大小，並重新置放圖表。 從儀表板選取圖表會將您重新導向適用於 VM 的 Azure 監視器，並載入 VM 的效能詳細資料檢視。  

![直接從 VM 檢視查看 VM insights 效能](./media/vminsights-performance/vminsights-performance-directvm-01.png)

## <a name="view-performance-directly-from-an-azure-virtual-machine-scale-set"></a>直接從 Azure 虛擬機器擴展集檢視效能

若要從 Azure 虛擬機器擴展集直接存取，請執行下列步驟。

1. 在 Azure 入口網站中，選取 [虛擬機器擴展集]。
2. 從清單中選擇 VM，並在 [監視] 區段中，選擇 [Insights]，以檢視 [效能] 索引標籤。

此頁面會載入 Azure 監視器效能檢視，範圍是選取的擴展集。 這可讓您在受監視計量的集合中，檢視擴展集的前 N 個執行個體、檢視擴展集之間的彙總效能，並查看擴展集中個別執行個體之間所選取計量的趨勢。 從清單檢視中選取執行個體，可讓您載入其對應，或導覽至該執行個體的詳細效能檢視。

按一下任一圖表右上角的釘選圖示，即可將選取的圖表釘選至您查看的最後一個 Azure 儀表板。 您可以在儀表板中調整圖表大小，並重新置放圖表。 從儀表板選取圖表會將您重新導向適用於 VM 的 Azure 監視器，並載入 VM 的效能詳細資料檢視。  

![直接從虛擬機器擴展集檢視 VM insights 效能](./media/vminsights-performance/vminsights-performance-directvmss-01.png)

>[!NOTE]
>您也可以從擴展集的執行個體檢視中，存取特定執行個體的詳細效能檢視。 導覽至 [設定]區段下的 [執行個體]，然後選取 [Insights]。



## <a name="next-steps"></a>後續步驟

- 了解如何使用適用於 VM 的 Azure 監視器隨附的[[活頁簿]](vminsights-workbooks.md)，進一步分析效能和網路計量。  

- 若要了解探索到的應用程式相依性，請參閱[檢視適用於 VM 的 Azure 監視器對應](vminsights-maps.md)。
