---
title: Azure Cache for Redis 的高可用性
description: 瞭解 Azure Cache for Redis 高可用性功能和選項
author: yegu-ms
ms.service: cache
ms.topic: conceptual
ms.date: 10/28/2020
ms.author: yegu
ms.openlocfilehash: e44aed1415f85bf4ea597eac6720207301946b97
ms.sourcegitcommit: 3bdeb546890a740384a8ef383cf915e84bd7e91e
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/30/2020
ms.locfileid: "93076906"
---
# <a name="high-availability-for-azure-cache-for-redis"></a>Azure Cache for Redis 的高可用性

Azure Cache for Redis 具有內建的高可用性。 其高可用性架構的目標是確保您的受控 Redis 實例可以正常運作，即使其基礎虛擬機器 (Vm) 受到規劃或非計畫中斷的影響。 它提供的百分比比在單一 VM 上裝載 Redis 可達成的速率高出許多。

Azure Cache for Redis 使用多個 Vm （稱為 *節點* ）來執行快取，以獲得高可用性。 它會設定這些節點，以便在協調形式中進行資料複寫和容錯移轉。 它也會協調維護作業，例如 Redis 軟體修補。 標準、高階和企業層都提供各種高可用性選項：

| 選項 | 描述 | 可用性 | 標準 | Premium | Enterprise |
| ------------------- | ------- | ------- | :------: | :---: | :---: |
| [標準複寫](#standard-replication)| 單一資料中心或可用性區域中的雙重節點複寫設定 (AZ) ，具有自動容錯移轉 | 99.9% |✔|✔|-|
| [企業叢集](#enterprise-cluster) | 具有自動容錯移轉的兩個區域中的連結快取實例 | 99.9% |-|-|✔|
| [區域備援](#zone-redundancy) | 跨 Az 的多重節點複寫設定，具有自動容錯移轉 | 99.95% (標準複寫) ，99.99% (企業叢集)  |-|✔|✔|
| [異地複寫](#geo-replication) | 兩個區域中的連結快取實例，具有使用者控制的容錯移轉 | 適用于單一區域的 99.9% ()  |-|✔|-|

## <a name="standard-replication"></a>標準複寫

標準層或進階層中的 Azure Cache for Redis 預設會在一對 Redis 伺服器上執行。 這兩部伺服器裝載于專用 Vm 上。 開放原始碼 Redis 只允許一部伺服器處理資料寫入要求。 此伺服器是 *主要* 節點，另一個則是另一個 *複本* 。 在布建伺服器節點之後，Azure Cache for Redis 會將主要和複本角色指派給它們。 主要節點通常負責維護寫入以及來自 Redis 用戶端的讀取要求。 在寫入作業中，它會將新的金鑰和金鑰更新認可至其內部記憶體，並立即回復至用戶端。 它會以非同步方式將作業轉送至複本。

:::image type="content" source="media/cache-high-availability/replication.png" alt-text="資料複寫設定":::
   
>[!NOTE]
>一般來說，Redis 用戶端會與 Redis 快取中的主要節點通訊，以取得所有的讀取和寫入要求。 某些 Redis 用戶端可以設定為從複本節點讀取。
>
>

如果 Redis 快取中的主要節點無法使用，複本就會自動升級成為新的主節點。 此進程稱為 *容錯移轉* 。 複本會先等待足夠的時間，然後再接管主要節點以快速復原。 發生容錯移轉時，Azure Cache for Redis 會布建新的 VM，並將其聯結至快取作為複本節點。 複本會執行與主要複本的完整資料同步處理，使其具有另一個快取資料複本。

主要節點可能會在規劃的維護活動（例如 Redis 軟體或作業系統更新）中離開服務。 它也可能因為未規劃的事件（例如基礎硬體、軟體或網路失敗）而停止運作。 [Azure Cache for Redis 的容錯移轉和修補](cache-failover.md) 提供 Redis 容錯移轉類型的詳細說明。 Azure Cache for Redis 將會在其存留期內經歷許多容錯移轉。 高可用性架構的設計，是為了讓快取內的這些變更盡可能透明地在其用戶端上進行。

>[!NOTE]
>以下是預覽版本。
>
>

此外，Azure Cache for Redis 允許進階層中的額外複本節點。 [多複本](cache-how-to-multi-replicas.md)快取最多可以設定三個複本節點。 有更多複本通常會改善復原能力，因為有其他節點備份主要複本。 即使有更多複本，Azure Cache for Redis 實例還是可能會嚴重受到資料中心或 AZ 個中斷的影響。 您可以使用多個複本搭配 [區域冗余](#zone-redundancy)來提高快取可用性。

## <a name="enterprise-cluster"></a>企業叢集

>[!NOTE]
>這會以預覽形式提供。
>
>

任一企業層中的快取都是在 Redis Enterprise 叢集上執行。 它需要有奇數數目的伺服器節點，才能形成仲裁。 依預設，它是由三個節點所組成，每個節點都裝載于專用 VM 上。 企業快取有兩個相同大小的 *資料節點* 和一個較小的 *仲裁節點* 。 企業 Flash 快取有三個相同大小的資料節點。 企業叢集會將 Redis 資料分成內部的資料分割。 每個分割區都有一個 *主要* 複本和至少一個 *複本* 。 每個資料節點都會保存一或多個資料分割。 企業叢集可確保任何資料分割的主要和複本 (的) 都不會共置於相同的資料節點上。 資料分割會以非同步方式將資料從主要複本複寫到其對應的複本。

當資料節點變成無法使用或發生網路分割時，會發生類似于 [標準](#standard-replication) 複寫中所述的容錯移轉。 企業叢集使用以仲裁為基礎的模型，來判斷哪些存活的節點將參與新的仲裁。 它也會視需要將這些節點內的複本分割區升級為主要複本。

## <a name="zone-redundancy"></a>區域備援

>[!NOTE]
>這會以預覽形式提供。
>
>

Azure Cache for Redis 支援 Premium 和 Enterprise 層中的區域多餘設定。 [區域冗余](cache-how-to-zone-redundancy.md)快取可將其節點放在相同區域中的不同[Azure 可用性區域](../availability-zones/az-overview.md)。 它會將資料中心或 AZ 中斷排除為單一失敗點，並提高快取的整體可用性。

下圖說明區域冗余設定：

:::image type="content" source="media/cache-high-availability/zone-redundancy.png" alt-text="資料複寫設定":::
   
Azure Cache for Redis 會透過您所選取的 Az，以迴圈配置資源的方式，在區域冗余快取中分散節點。 它也會決定哪一個節點一開始做為主要節點。

區域冗余快取會提供自動容錯移轉。 當目前的主要節點無法使用時，其中一個複本將會接管。 如果新的主要節點位於不同的 AZ 中，則您的應用程式可能會遇到較高的快取回應時間。 Az 是以地理區隔開。 從一個 AZ 切換到另一個 AZ，改變了您的應用程式和快取裝載位置之間的實體距離。 此變更會影響您應用程式到快取的來回網路延遲。 額外的延遲預期會落在大部分應用程式可接受的範圍內。 建議您測試應用程式，以確保它可以透過區域冗余快取順利執行。

## <a name="geo-replication"></a>異地複寫

地理複寫的設計主要是為了進行嚴重損壞修復。 它能讓您在不同的 Azure 區域中設定 Azure Cache for Redis 實例，以備份您的主要快取。 [針對 Azure Cache for Redis 設定異地複寫可](cache-how-to-geo-replication.md) 提供地理複寫運作方式的詳細說明。

## <a name="next-steps"></a>下一步

深入瞭解如何設定 Azure Cache for Redis 高可用性選項。

* [Azure Cache for Redis Premium 服務層級](cache-overview.md#service-tiers)
* [將複本新增至 Azure Cache for Redis](cache-how-to-multi-replicas.md)
* [啟用 Azure Cache for Redis 的區域冗余](cache-how-to-zone-redundancy.md)
* [設定 Azure Cache for Redis 的異地複寫](cache-how-to-geo-replication.md)