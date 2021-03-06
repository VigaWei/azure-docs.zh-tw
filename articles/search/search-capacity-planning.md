---
title: 預估查詢和索引工作負載的容量
titleSuffix: Azure Cognitive Search
description: 調整 Azure 認知搜尋中的磁碟分割和複本電腦資源，其中每個資源都是以計費搜尋單位來計價。
manager: nitinme
author: HeidiSteen
ms.author: heidist
ms.service: cognitive-search
ms.topic: conceptual
ms.date: 01/15/2021
ms.openlocfilehash: 4a9a6b61e392ed2efd68cdcb1cf7e53d6bde5ccd
ms.sourcegitcommit: 25d1d5eb0329c14367621924e1da19af0a99acf1
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 01/16/2021
ms.locfileid: "98249676"
---
# <a name="estimate-and-manage-capacity-of-an-azure-cognitive-search-service"></a>預估及管理 Azure 認知搜尋服務的容量

在 [提供搜尋服務](search-create-service-portal.md) 並鎖定特定定價層之前，請花幾分鐘的時間來瞭解容量的運作方式，以及如何調整複本和資料分割以因應工作負載波動。

容量是 [服務層](search-sku-tier.md)的功能。 階層會依最大儲存空間、每個資料分割儲存體，以及您可以建立之物件數目的最大限制來區分。 基本層是專為具有適度儲存需求的應用程式所設計， (一個資料分割) 但可以在高可用性設定中執行 (3 個複本) 。 其他層則是針對特定的工作負載或模式（例如多租使用者）所設計。 就內部而言，在這些層級上建立的服務會受益于有助於這些案例的硬體。

Azure 認知搜尋中的擴充性架構是以彈性的複本和資料分割組合為基礎，因此您可以根據您是否需要更多查詢或索引編制功能來改變容量。 一旦建立服務之後，您就可以單獨增加或減少複本或資料分割的數目。 成本會隨著每個額外的實體資源而增加，但在大型工作負載完成後，您可以減少調整以降低帳單。 視層級和調整大小而定，增加或減少容量可能需要15分鐘到數小時的時間。

修改複本和分割區的配置時，建議使用 Azure 入口網站。 入口網站會對允許的組合強制執行限制，而這些組合仍低於層級的最大限制。 但是，如果您需要以腳本為基礎或以程式碼為基礎的布建方法， [Azure PowerShell](search-manage-powershell.md) 或 [管理 REST API](/rest/api/searchmanagement/services) 是替代的解決方案。

## <a name="concepts-search-units-replicas-partitions-shards"></a>概念：搜尋單位、複本、資料分割、分區

容量是以可透過資料 *分割和**複本* 組合來配置的 *搜尋單位* 來表示，並使用基礎 *分區化* 機制來支援彈性設定：

| 概念  | 定義|
|----------|-----------|
|*搜尋單位* | 總可用容量的單一增量 (36 單位) 。 這也是 Azure 認知搜尋服務的計費單位。 至少需要一個單位才能執行服務。|
|*複本* | 搜尋服務的執行個體，主要用來讓查詢作業達到負載平衡。 每個複本都會裝載一份索引。 如果您配置三個複本，則會有三個索引複本可用於服務查詢要求。|
|*資料分割* | 讀取/寫入作業的實體儲存體和 i/o (例如，在重建或重新整理索引) 時。 每個分割區都有總計索引的配量。 如果您配置三個數據分割，則索引會分成三分之二。 |
|*碎片* | 索引的區塊。 Azure 認知搜尋會將每個索引分割成分區，藉由將分區移至新的搜尋單位) ，讓新增資料分割的過程 (更快速。|

下圖顯示覆本、磁碟分割、分區和搜尋單位之間的關聯性。 它會顯示一個範例，說明如何在具有兩個複本和兩個數據分割的服務中，跨越四個搜尋單位的單一索引。 這四個搜尋單位的每一個都只會儲存索引的一半分區。 左欄中的搜尋單位會儲存第一半的分區，其中包含第一個資料分割，而右邊資料行中的資料行則是由第二個數據分割組成的分區後半部。 由於有兩個複本，因此每個索引分區都有兩個複本。 頂端資料列中的搜尋單位會儲存一個複本，其中包含第一個複本，而底部資料列中的資料會儲存另一個複本，其中包含第二個複本。

:::image type="content" source="media/search-capacity-planning/shards.png" alt-text="搜尋索引會跨資料分割分區化。":::

上圖只是一個範例。 有許多分割區和複本的組合可供使用，最多可達36的搜尋單位總數。

在認知搜尋中，分區管理是一個執行詳細資料和不可設定的，但瞭解索引分區化有助於瞭解排名和自動完成行為中偶爾發生的異常狀況：

+ 排名異常：先在分區層級計算搜尋分數，然後匯總成單一結果集。 根據分區內容的特性而定，一個分區的相符專案可能會高於另一個的相符專案。 如果您注意到搜尋結果中有直覺排名的計數器，最有可能是因為分區化的影響，尤其是當索引較小時。 您可以選擇在 [整個索引的全域計算分數](index-similarity-and-scoring.md#scoring-statistics-and-sticky-sessions)，以避免這些排名異常，但是這樣做會導致效能的負面影響。

+ 自動完成異常：自動完成查詢，在部分輸入詞彙的前幾個字元上進行相符專案的情況下，接受模糊參數以 forgives 模糊的微小偏差。 針對自動完成，模糊比對會限制為目前分區內的詞彙。 例如，如果分區包含 "Microsoft" 且輸入了 "micor" 的部分詞彙，則搜尋引擎會比對該分區中的 "Microsoft"，但不會比對剩餘索引部分的其他分區。

## <a name="how-to-evaluate-capacity-requirements"></a>如何評估容量需求

開始執行服務的容量和成本。 層級會限制兩個層級：儲存體和內容 (服務上索引的計數，例如) 。 請務必考慮這兩者，因為您最先達到的限制是有效的限制。

索引和其他物件的數量通常是由商務和工程需求所決定。 例如，您可能會有相同索引的多個版本，以供主動開發、測試和生產環境使用。

儲存體需求取決於您預期要建立的索引大小。 沒有可協助評估的穩固啟發學習法或 generalities。 判斷索引大小的唯一方式是 [建立一個](search-what-is-an-index.md)。 其大小會以匯入的資料、文字分析和索引設定為基礎，例如您是否啟用建議工具、篩選和排序。

針對全文檢索搜尋，主要資料結構是 [反向索引](https://en.wikipedia.org/wiki/Inverted_index) 結構，其具有與來源資料不同的特性。 對於反向索引，大小和複雜度是由內容所決定，而不一定是您饋送至其中的資料量。 具有高冗余性的大型資料來源可能會產生比包含高度變數內容的較小資料集更小的索引。 因此很少可以根據原始資料集的大小推斷索引大小。

> [!NOTE] 
> 雖然預估索引和儲存空間的未來需求可能會像是猜測，但還是值得一試。 如果層的容量太低，您將需要在較高的層級布建新的服務，然後 [重載您的索引](search-howto-reindex.md)。 沒有將服務從一個階層就地升級到另一個層級。
>

### <a name="estimate-with-the-free-tier"></a>使用免費層進行評估

其中一個預估容量的方法是從免費層開始。 請記住，免費服務提供最多三個索引、50 MB 的儲存空間，以及2分鐘的索引時間。 使用這些條件約束來估計投影的索引大小可能會很困難，但這些步驟如下：

+ [建立免費服務](search-create-service-portal.md)。
+ 準備小型、具代表性的資料集。
+ 建立索引並載入您的資料。 如果資料集可以裝載于索引子支援的 Azure 資料來源中，您可以使用 [入口網站中的 [匯入資料] 嚮導](search-get-started-portal.md) 來建立和載入索引。 否則，您應該使用 REST 和 [Postman](search-get-started-rest.md) 或 [Visual Studio Code](search-get-started-vs-code.md) 來建立索引並推送資料。 推送模型要求資料的格式為 JSON 檔，檔中的欄位會對應至索引中的欄位。
+ 收集索引的相關資訊，例如大小。 功能和屬性會影響儲存體。 例如，新增建議工具 (搜尋即輸入查詢) 將會增加儲存需求。 

  使用相同的資料集，您可能會嘗試在每個欄位上建立具有不同屬性的多個索引版本，以查看儲存需求的差異。 如需詳細資訊，請參閱 [建立基本索引中的「存放裝置含意](search-what-is-an-index.md#index-size)」。

有了粗略的估計值，您可以將兩個索引的數量加倍 (開發和生產) ，然後據以選擇您的定價層。

### <a name="estimate-with-a-billable-tier"></a>使用可計費層進行估計

專用資源可以容納較大型的取樣和處理時間，以在開發期間更實際估計索引數量、大小和查詢量。 某些客戶會直接使用計費層，然後在開發專案成熟時重新評估。

1. [查看每一層的服務限制](./search-limits-quotas-capacity.md#index-limits) ，判斷較低的層級是否可以支援您需要的索引數目。 在基本、S1 和 S2 層中，索引限制分別為15、50和200。 儲存體優化層有10個索引的限制，因為它的設計是為了支援少量的極大型索引。

1. [在可計費的定價層建立服務](search-create-service-portal.md)：

    + 如果您不確定預計的負載，請在 [基本] 或 [S1] 上啟動。
    + 如果測試包含大規模的索引和查詢負載，請從 S2 或甚至 S3 開始。
    + 如果您要編制大量資料的索引，且查詢負載相對較低（如同內部商務應用程式），請從 L1 或 L2 的儲存體進行優化。

1. [建置初始索引](search-what-is-an-index.md)，以判斷來源資料轉譯為索引的情形。 這是唯一能預估索引大小的方式。

1. 在入口網站中[監視儲存體、服務限制、查詢量和延遲](search-monitor-usage.md)。 入口網站會顯示每秒查詢數、節流查詢和搜尋延遲。 所有這些值都可以協助您決定是否選取了正確的層級。

1. 如果您需要高可用性，或如果您遇到查詢效能緩慢的情況，請新增複本。

   對於需要多少個複本來容納查詢負載，並沒有任何相關指導方針。 查詢效能取決於查詢和競爭工作負載的複雜性。 雖然新增複本會明顯獲得更好的效能，但是最終結果並不會完全地呈線性關係：新增 3 個複本並不保證有 3 倍的輸送量。 如需評估解決方案 QPS 的指導方針，請參閱 [針對效能](search-performance-optimization.md)和 [監視查詢](search-monitor-queries.md)進行調整。

> [!NOTE]
> 如果您包含永遠不會搜尋的資料，則可以擴大儲存體需求。 在理想情況下，檔只包含搜尋體驗所需的資料。 二進位資料無法搜尋，而且應該個別儲存 (可能在 Azure 資料表或 blob 儲存體) 中。 接著，應該在索引中加入欄位，以保存外部資料的 URL 參考。 如果您要在單一) 要求中大量上傳多份檔，則個別搜尋檔的大小上限為 16 MB (或較少。 如需詳細資訊，請參閱 [Azure 認知搜尋中的服務限制](search-limits-quotas-capacity.md)。
>

**查詢量的考量**

每秒查詢數 (QPS) 是效能調整期間的重要計量，但如果您在一開始就預期會有大量的查詢量，通常只會考慮階層。

標準層可以提供複本和資料分割的平衡。 您可以藉由新增負載平衡的複本或加入資料分割以進行平行處理，來增加查詢週期。 然後，您可以在布建服務之後調整效能。

如果您預期從一開始就會有高度持續的查詢磁片區，您應該考慮較高的標準層，並支援更強大的硬體。 然後，您可以讓分割區和複本離線，或甚至切換至較低層的服務（如果沒有發生這些查詢磁片區）。 如需如何計算查詢輸送量的詳細資訊，請參閱 [Azure 認知搜尋效能和優化](search-performance-optimization.md)。

儲存體優化層適用于大型資料工作負載，當查詢延遲需求較不重要時，支援更多的整體可用索引儲存體。 您仍應使用額外的複本來進行負載平衡，並使用額外的資料分割來進行平行處理。 然後，您可以在布建服務之後調整效能。

**服務等級協定**

免費層和預覽功能不提供 [服務等級協定 (sla) ](https://azure.microsoft.com/support/legal/sla/search/v1_0/)。 在所有可計費層中，SLA 會在您為您的服務佈建足夠的備援性時生效。 您需要有兩個以上的複本，查詢 (讀取) Sla。 您需要有三個以上的複本，以便查詢和編制索引 (讀寫) Sla。 磁碟分割數目不會影響 Sla。

## <a name="tips-for-capacity-planning"></a>容量規劃的秘訣

+ 允許計量以查詢為依據，並收集在上班時間 (查詢的使用模式資料，在離峰時段編制索引) 。 使用此資料來通知服務布建決策。 雖然不是每小時或每日的頻率，您可以動態調整分割區和資源，以配合查詢磁片區中的規劃變更。 如果層級保有足夠的時間足以保證採取動作，您也可以容納未計畫但持續的變更。

+ 請記住，在布建下的唯一缺點是，如果實際需求大於您的預測，您可能必須卸載服務。 為了避免服務中斷，您會在較高的層級建立新的服務，並並存執行，直到所有應用程式和要求都以新端點為目標為止。

## <a name="when-to-add-partitions-and-replicas"></a>何時加入分割區和複本

一開始，服務會配置由一個資料分割和一個複本組成的基本資源層級。

單一服務必須具有足夠的資源，才能處理所有工作負載 (編製索引和查詢)。 這兩個工作負載都不會在背景執行。 您可以針對查詢要求自然較不頻繁的時間排程索引編制，但服務也不會優先處理其中一個工作的優先順序。 此外，當服務或節點在內部更新時，特定數量的冗余會將查詢效能變得更平滑。

一般而言，搜尋應用程式所需的複本比分割區多，特別是當服務作業偏差查詢工作負載時。 [高可用性](#HA) 一節將會說明原因。

[您選擇的層級](search-sku-tier.md)會決定資料分割的大小和速度，而每一層都會針對符合各種案例的一組特性進行優化。 如果您選擇較高階的層級，您可能需要比使用 S1 更少的資料分割。 您必須透過自我導向測試回答的問題之一，就是，較大且較昂貴的資料分割是否可在布建于較低層的服務上，產生比兩個較便宜的資料分割更佳的效能。

要求近乎即時資料重新整理的搜尋應用程式，按比例需要比複本更多的資料分割數。 新增資料分割可將讀取/寫入作業分配到更大量的計算資源。 它也提供更多磁碟空間來儲存額外的索引和文件。

索引越大，查詢所需的時間就越長。 這麼一來，您可能會發現每個資料分割中每個累加式的增加在複本中都需要有較小但按比例的增加。 要將查詢執行速度提高到何種程度，需將您的查詢和查詢磁碟區複雜性納入考量。

> [!NOTE]
> 加入更多複本或資料分割會增加執行服務的成本，而且可能會在排序結果的方式中帶來些許變化。 請務必檢查 [定價計算機](https://azure.microsoft.com/pricing/calculator/) ，以瞭解新增更多節點的計費含意。 [下圖](#chart)可協助您交叉參考特定設定所需的搜尋單位數目。 如需其他複本如何影響查詢處理的詳細資訊，請參閱 [排序結果](search-pagination-page-layout.md#ordering-results)。

## <a name="how-to-allocate-replicas-and-partitions"></a>如何配置複本和磁碟分割

1. 登入 [Azure 入口網站](https://portal.azure.com/)，然後選取搜尋服務。

1. 在 [ **設定**] 底下，開啟 [ **調整** ] 頁面以修改複本和資料分割。 

   下列螢幕擷取畫面顯示已布建一個複本和分割區的基本標準。 底部的公式指出 (1) 所使用的搜尋單位數目。 如果單價為 $100 (不是真正的價格) ，執行這項服務的每月成本平均都會是 $100。

   :::image type="content" source="media/search-capacity-planning/1-initial-values.png" alt-text="顯示目前值的縮放頁面" border="true":::

1. 使用滑杆來增加或減少分割區數目。 底部的公式會指出正在使用的搜尋單位數目。 選取 [儲存]。

   此範例會新增第二個複本和資料分割。 請注意搜尋單位元數目;這現在是四個，因為計費公式是複本乘以資料分割 (2 x 2) 。 比起執行服務的成本倍倍以上的容量。 如果搜尋單位成本為 $100，則新的每月帳單現在會是 $400。

   如需每一層的目前每個單位成本，請造訪 [定價頁面](https://azure.microsoft.com/pricing/details/search/)。

   :::image type="content" source="media/search-capacity-planning/2-add-2-each.png" alt-text="新增複本和磁碟分割" border="true":::

1. 儲存之後，您可以檢查通知來確認動作是否成功。

   :::image type="content" source="media/search-capacity-planning/3-save-confirm.png" alt-text="儲存變更" border="true":::

   容量變更最多可能需要數小時的時間才能完成。 一旦啟動程式之後，您就無法取消，且不會對複本和資料分割調整進行即時監視。 不過，在進行變更時，仍會顯示下列訊息。

   :::image type="content" source="media/search-capacity-planning/4-updating.png" alt-text="入口網站中的狀態訊息" border="true":::

> [!NOTE]
> 服務在布建之後，即無法升級至較高的層級。 您必須在新層中建立搜尋服務，然後重新載入您的索引。 請參閱 [入口網站中的建立 Azure 認知搜尋服務](search-create-service-portal.md) ，以取得服務布建方面的協助。
>
> 此外，分割區和複本會以獨佔方式以及由服務在內部進行管理。 沒有處理器親和性的概念，或將工作負載指派給特定的節點。
>

<a id="chart"></a>

## <a name="partition-and-replica-combinations"></a>資料分割與複本組合

「基本」服務可以有不多不少 1 個分割區及最多 3 個複本，上限為 3 個 SU。 唯一可調整的資源是複本。 您至少需要 2 個複本，才能在查詢上達到高可用性。

所有標準和儲存體優化搜尋服務都可採用下列複本和資料分割組合，受限於這些層級所允許的 36-SU 限制。 

|   | **1個分割區** | **2個磁碟分割** | **3 個資料分割** | **4 個資料分割** | **6個磁碟分割** | **12 個資料分割** |
| --- | --- | --- | --- | --- | --- | --- |
| **1 個複本** |1 SU |2 SU |3 SU |4 SU |6 SU |12 SU |
| **2 個複本** |2 SU |4 SU |6 SU |8 SU |12 SU |24 SU |
| **3 個複本** |3 SU |6 SU |9 SU |12 SU |18 SU |36 SU |
| **4 個複本** |4 SU |8 SU |12 SU |16 SU |24 SU |N/A |
| **5 個複本** |5 SU |10 SU |15 SU |20 SU |30 SU |N/A |
| **6 個複本** |6 SU |12 SU |18 SU |24 SU |36 SU |N/A |
| **12 個複本** |12 SU |24 SU |36 SU |N/A |N/A |N/A |

SU、定價和容量會在 Azure 網站上詳細說明。 如需詳細資訊，請參閱 [定價詳細資料](https://azure.microsoft.com/pricing/details/search/)。

> [!NOTE]
> 複本數和資料分割數可整除 12 (明確來說就是 1、2、3、4、6、12)。 這是因為 Azure 認知搜尋會將每個索引預先分割成12個分區，以便在所有分割區的相等部分中散佈。 例如，如果您的服務有三個資料分割，而您建立了索引，則每個資料分割將會包含 4 個該索引的分區。 Azure 認知搜尋分區索引的方式是執行的詳細資料，未來的版本可能會變更。 雖然現在分區數為 12，但您不應預期未來該數字永遠都會是 12。
>

<a id="HA"></a>

## <a name="high-availability"></a>高可用性

由於相應增加非常快速簡單，我們通常建議您從一個資料分割和一或兩個複本開始，然後於查詢量增長時再相應增加。 查詢工作負載主要是在複本上執行。 如果您需要更多的輸送量或高可用性，可能會需要額外的複本。

針對高可用性的一般建議為：

+ 針對唯讀工作負載 (查詢)，需有 2 個複本才能達到高可用性

+ 針對讀寫工作負載 (查詢再加上新增、更新或刪除個別文件時的索引編製)，需有 3 個或更多個複本才能達到高可用性

Azure 認知搜尋的服務等級協定 (SLA) 的目標是查詢作業，以及包含新增、更新或刪除檔的索引更新。

基本層最多一個分割區和三個複本。 如果想要具備立即回應檢索和查詢輸送量需求波動的彈性，請考慮標準層的其中一個。  如果您發現儲存體需求的成長速度比查詢輸送量更快，請考慮其中一個儲存體優化層級。

## <a name="next-steps"></a>後續步驟

> [!div class="nextstepaction"]
> [如何預估和管理成本](search-sku-manage-costs.md)