---
title: 包含檔案
description: 包含檔案
services: virtual-machines
author: albecker1
ms.service: virtual-machines
ms.topic: include
ms.date: 10/12/2020
ms.author: albecker1
ms.custom: include file
ms.openlocfilehash: 086ebf71e2da19a96433f32cfb1bae133e875400
ms.sourcegitcommit: 59f506857abb1ed3328fda34d37800b55159c91d
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/24/2020
ms.locfileid: "92518044"
---
![顯示 D s v 3 規格的圖表。](media/vm-disk-performance/dsv3-documentation.jpg)

- 最大未快取 *的磁片輸送量* 是虛擬機器可以處理的預設儲存體上限。
- 當您啟用主機快取時， *最大快* 取儲存體輸送量限制是個別的限制。

主機快取的運作方式是讓儲存體更接近可供寫入或讀取的 VM。 適用于主機快取的 VM 可用的儲存體數量是在檔中。 例如，您可以看到 Standard_D8s_v3 隨附于200快取儲存體 GiB。

當您建立虛擬機器並連接磁片時，可以啟用主機快取。 您也可以在現有的 VM 上，開啟和關閉磁片上的主機快取。

![顯示主機快取的螢幕擷取畫面。](media/vm-disk-performance/host-caching.jpg)

您可以調整主機快取，以符合每個磁片的工作負載需求。 您可以將主機快取設定為：

- **唯讀**：適用于只執行讀取作業的工作負載
- **讀取/寫入**：執行讀取和寫入作業平衡的工作負載

如果您的工作負載未遵循上述任一種模式，則不建議使用主機快取。

讓我們執行一些不同主機快取設定的範例，以查看它對資料流程和效能的影響。 在第一個範例中，我們將探討當主機快取設定設為 **唯讀**時，IO 要求會發生什麼事。

**設置：**

- Standard_D8s_v3
  - 快取的 IOPS：16000
  - 未快取的 IOPS：12800
- P30 資料磁片
  - IOPS：5000
  - 主機快取： **唯讀**

執行讀取並在快取上提供所需的資料時，快取會傳回要求的資料。 不需要從磁片讀取。 這項讀取會計算 VM 的快取限制。

![顯示讀取主機快取讀取計數的圖表。](media/vm-disk-performance/host-caching-read-hit.jpg)

當執行讀取，且快取上 *未* 提供所需的資料時，會將讀取要求轉送至磁片。 然後磁片會將它呈現給快取和 VM。 這項讀取會計算 VM 的未快取限制和 VM 的快取限制。

![顯示讀取主機快取讀取遺漏的圖表。](media/vm-disk-performance/host-caching-read-miss.jpg)

執行寫入時，必須先將寫入寫入至快取和磁片，然後才會被視為已完成。 此寫入會計算 VM 的未快取限制和 VM 的快取限制。

![顯示讀取主機快取寫入的圖表。](media/vm-disk-performance/host-caching-write.jpg)

接下來讓我們看看當主機快取設定為 [ **讀取/寫入**] 時，IO 要求會發生什麼事。

**設置：**

- Standard_D8s_v3
  - 快取的 IOPS：16000
  - 未快取的 IOPS：12800
- P30 資料磁片
  - IOPS：5000
  - 主機快取： **讀取/寫入**

讀取的處理方式與唯讀相同。 寫入是唯一與讀取/寫入快取不同的東西。 當使用主機快取進行寫入設定為 **讀取/寫入**時，寫入只需要寫入主機快取，才能被視為已完成。 然後寫入會以背景進程的形式，延遲寫入磁片。 這表示寫入至快取時，會將寫入計算到快取 IO。 當它延遲寫入磁片時，它會計算到未快取的 IO。

![顯示讀取/寫入主機快取寫入的圖表。](media/vm-disk-performance/host-caching-read-write.jpg)

讓我們繼續 Standard_D8s_v3 的虛擬機器。 但這次我們將在磁片上啟用主機快取。 此外，現在 VM 的 IOPS 限制為 16000 IOPS。 連結至 VM 的三個基礎 P30 磁片可以處理 5000 IOPS。

**設置：**

- Standard_D8s_v3
  - 快取的 IOPS：16000
  - 未快取的 IOPS：12800
- P30 OS 磁片
  - IOPS：5000
  - 主機快取： **讀取/寫入**
- 兩個 P30 資料磁片×2
  - IOPS：5000
  - 主機快取： **讀取/寫入**

![顯示主機快取範例的圖表。](media/vm-disk-performance/host-caching-example-without-remote.jpg)

應用程式會使用已啟用快取的 Standard_D8s_v3 虛擬機器。 它會提出 15000 IOPS 的要求。 這些要求會細分為每個連接的基礎磁片的 5000 IOPS。 不會發生效能上限。

## <a name="combined-uncached-and-cached-limits"></a>結合未快取和快取的限制

虛擬機器的快取限制與其未快取的限制不同。 這表示您可以在連接至 VM 的磁片上啟用主機快取，而不會在其他磁片上啟用主機快取。 這項設定可讓您的虛擬機器取得快取限制的總儲存體 IO 加上未快取的限制。

讓我們逐步執行範例，以協助您瞭解這些限制如何一起運作。 我們將繼續進行 Standard_D8s_v3 的虛擬機器和 premium 磁片連接設定。

**設置：**

- Standard_D8s_v3
  - 快取的 IOPS：16000
  - 未快取的 IOPS：12800
- P30 OS 磁片
  - IOPS：5000
  - 主機快取： **讀取/寫入**
- 兩個 P30 資料磁片×2
  - IOPS：5000
  - 主機快取： **讀取/寫入**
- 兩個 P30 資料磁片×2
  - IOPS：5000
  - 主機快取： **已停用**

![此圖顯示具有遠端存放的主機快取範例。](media/vm-disk-performance/host-caching-example-with-remote.jpg)

在此情況下，在 Standard_D8s_v3 虛擬機器上執行的應用程式會提出 25000 IOPS 的要求。 要求會細分為每個連接磁片的 5000 IOPS。 三個磁片使用主機快取，而兩個磁片不使用主機快取。

- 由於三個使用主機快取的磁片都在16000的快取限制內，因此這些要求已成功完成。 未發生儲存體效能上限。
- 由於未使用主機快取的兩個磁片是在未快取的限制12800內，因此也會成功完成這些要求。 未發生任何上限。

## <a name="disk-performance-metrics"></a>磁片效能計量

我們在 Azure 上有一些計量，可讓您深入瞭解虛擬機器和磁片的執行情況。 您可以透過 Azure 入口網站來查看這些計量。 也可以透過 API 呼叫來抓取。 計量會以一分鐘的間隔計算。 您可以使用下列計量來取得 VM 和磁片 IO 的深入解析，也可以取得輸送量效能：

- **Os 磁片佇列深度**：目前正在等候讀取或寫入 OS 磁片的未完成 IO 要求數目。
- **Os Disk Read Bytes/Sec**：從 OS 磁片讀取的位元組數目。
- **Os 磁片讀取作業數/秒**：從 os 磁片讀取的輸入作業數（秒）。
- **Os Disk Write Bytes/Sec**：以秒為單位從 os 磁片寫入的位元組數目。
- **Os 磁片寫入作業數/秒**：從 OS 磁片寫入的輸出作業數（秒）。
- **資料磁片佇列深度**：目前正在等候讀取或寫入資料磁片的未完成 IO 要求數目 (s) 。
- **資料磁片讀取位元組數/秒**：從資料磁片 (s) 讀取的位元組數目。
- **資料磁片讀取作業數/秒**：從資料磁片 (s) 第二次讀取的輸入作業數目。
- **資料磁片寫入位元組數/秒**：從資料磁片 (s) 以秒為單位寫入的位元組數目。
- **資料磁片寫入作業數/秒：每**秒從資料磁片 (s 寫入的輸出作業數目) 。
- **磁片讀取位元組數/秒**：從所有連接至 VM 的磁片讀取的位元組總數。
- **磁片讀取作業數/秒**：從連結至 VM 的所有磁片中，第二次讀取的輸入作業數目。
- **磁片寫入位元組數/秒**：從連接至 VM 的所有磁片以秒為單位寫入的位元組數目。
- **磁片寫入作業數/秒：每**秒從連結至 VM 的所有磁片寫入的輸出作業數目。

## <a name="storage-io-utilization-metrics"></a>儲存體 IO 使用量計量

協助診斷磁片 IO 上限的計量：

- **資料磁片 Iops 耗用百分比**：透過布建的資料磁片 iops 完成的資料磁片 iops 所計算的百分比。 如果此數量為100%，則您的應用程式執行的 IO 上限為您資料磁片的 IOPS 限制。
- **使用的資料磁片頻寬百分比**：透過布建的資料磁片輸送量完成的資料磁片輸送量所計算的百分比。 如果此數量為100%，則您的應用程式執行的 IO 會限制為數據磁片的頻寬限制。
- **作業系統磁片 Iops 耗用百分比**：作業系統磁片 iops 所計算的百分比，由布建的 OS 磁片 iops 所完成。 如果此數量為100%，則您的應用程式執行的 IO 上限為作業系統磁片的 IOPS 限制。
- **作業系統磁片頻寬耗用百分比**：透過布建的 os 磁片輸送量完成的 os 磁片輸送量所計算的百分比。 如果此數量為100%，則您的應用程式執行的 IO 上限為作業系統磁片的頻寬限制。

協助診斷 VM IO 上限的計量：

- VM 快取的**Iops 耗用百分比**：依最大快取的虛擬機器 iops 限制完成的總 IOPS 所計算的百分比。 如果此數量為100%，則您的應用程式執行的 IO 會受限於 VM 的快取 IOPS 限制。
- VM 快取的**頻寬使用量百分比**：依最大快取的虛擬機器輸送量完成的總磁片輸送量所計算的百分比。 如果此數量為100%，則您的應用程式執行的 IO 上限為 VM 的快取頻寬限制。
- VM 未快取的已**耗用 IOPS 百分比**：虛擬機器上的 IOPS 總計所計算的百分比，已達最大未快取的虛擬機器 iops 限制。 如果此數量為100%，則您的應用程式執行的 IO 上限為 VM 未快取的 IOPS 限制。
- VM 未快取的**頻寬使用量百分比**：虛擬機器上的總磁片輸送量所計算的百分比，已達已布建的虛擬機器輸送量上限。 如果此數量為100%，則您的應用程式執行的 IO 會受限於 VM 未快取的頻寬限制。

## <a name="storage-io-utilization-metrics-example"></a>儲存體 IO 使用計量範例

讓我們來看看如何使用這些新的儲存體 IO 使用量計量的範例，以協助我們在系統中的瓶頸。 系統設定與上一個範例相同，但這次 *不* 會快取連接的 OS 磁片。

**設置：**

- Standard_D8s_v3
  - 快取的 IOPS：16000
  - 未快取的 IOPS：12800
- P30 OS 磁片
  - IOPS：5000
  - 主機快取： **已停用**
- 兩個 P30 資料磁片×2
  - IOPS：5000
  - 主機快取： **讀取/寫入**
- 兩個 P30 資料磁片×2
  - IOPS：5000
  - 主機快取： **已停用**
