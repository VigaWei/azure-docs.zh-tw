---
title: NCv2 系列-Azure 虛擬機器
description: NCv2 系列 Vm 的規格。
author: vikancha-MSFT
ms.service: virtual-machines
ms.subservice: sizes
ms.topic: conceptual
ms.date: 02/03/2020
ms.author: jushiman
ms.openlocfilehash: 97fa5148d7de7cac67a69c8d9c21721cb57951d3
ms.sourcegitcommit: d2d1c90ec5218b93abb80b8f3ed49dcf4327f7f4
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 12/16/2020
ms.locfileid: "97585374"
---
# <a name="ncv2-series"></a>NCv2 系列

NCv2 系列 VM 是由 NVIDIA Tesla P100 GPU 提供技術支援。 這些 GPU 可提供 NC 系列 2 倍以上的計算效能。 客戶可針對儲槽模型、DNA 定序、蛋白質分析、蒙地卡羅模擬等傳統 HPC 工作負載，善用這些更新過的 GPU。 除了 Gpu 之外，NCv2 系列的 Vm 也會由 Intel Broadwell E5-2690 v4 () Cpu 提供技術支援。

NC24rs v2 組態提供低延遲且高輸送量網路介面，最適合用於緊密結合的平行計算工作負載。

[進階儲存體](premium-storage-performance.md)：支援<br>
[進階儲存體](premium-storage-performance.md)快取：支援<br>
[即時移轉](maintenance-and-updates.md)：不支援<br>
[記憶體保留更新](maintenance-and-updates.md)：不支援<br>
[VM 世代支援](generation-2.md)：第1代和第2代<br>
Nvidia NVLink 互連：不支援

> [!IMPORTANT]
> 針對此 VM 系列，訂用帳戶中的 vCPU (core) 配額一開始會在每個區域中設定為0。 在[可用區域](https://azure.microsoft.com/regions/services/)中，為此系列[要求增加 vCPU 配額](../azure-portal/supportability/resource-manager-core-quotas-request.md)。
>
| 大小 | vCPU | 記憶體：GiB | 暫存儲存體 (SSD) GiB | GPU | GPU 記憶體：GiB | 最大資料磁碟 | 最大取消快取的磁碟輸送量：IOPS/MBps | 最大 NIC |
|---|---|---|---|---|---|---|---|---|
| Standard_NC6s_v2    | 6  | 112 | 736  | 1 | 16 | 12 | 20000/200 | 4 |
| Standard_NC12s_v2   | 12 | 224 | 1474 | 2 | 32 | 24 | 40000/400 | 8 |
| Standard_NC24s_v2   | 24 | 448 | 2948 | 4 | 64 | 32 | 80000/800 | 8 |
| Standard_NC24rs_v2* | 24 | 448 | 2948 | 4 | 64 | 32 | 80000/800 | 8 |

1 GPU = 一張 P100 卡。

*支援 RDMA

[!INCLUDE [virtual-machines-common-sizes-table-defs](../../includes/virtual-machines-common-sizes-table-defs.md)]

## <a name="supported-operating-systems-and-drivers"></a>支援的作業系統和驅動程式

若要利用 Azure N 系列 Vm 的 GPU 功能，必須安裝 NVIDIA GPU 驅動程式。

[NVIDIA GPU 驅動程式擴充功能](./extensions/hpccompute-gpu-windows.md)會在 N 系列 VM 上安裝適當的 NVIDIA CUDA 或 GRID 驅動程式。 使用 Azure 入口網站或者 Azure PowerShell 或 Azure Resource Manager 範本之類的工具，安裝或管理擴充功能。 如需支援的作業系統和部署步驟，請參閱 [NVIDIA GPU 驅動程式擴充功能文件](./extensions/hpccompute-gpu-windows.md)。 如需有關虛擬機器擴充功能的一般資訊，請參閱 [Azure 虛擬機器擴充功能和功能](./extensions/overview.md)。

如果您選擇手動安裝 NVIDIA GPU 驅動程式，請參閱適用于 [Windows 的 n 系列 gpu 驅動程式設定](./windows/n-series-driver-setup.md) ，或適用于 [Linux 的 n 系列 gpu 驅動程式設定](./linux/n-series-driver-setup.md) ，以取得支援的作業系統、驅動程式、安裝和驗證步驟。

## <a name="other-sizes"></a>其他大小

- [一般用途](sizes-general.md)
- [記憶體最佳化](sizes-memory.md)
- [儲存體最佳化](sizes-storage.md)
- [GPU 最佳化](sizes-gpu.md)
- [高效能計算](sizes-hpc.md)
- [前幾代](sizes-previous-gen.md)

## <a name="next-steps"></a>後續步驟

深入了解 [Azure 計算單位 (ACU)](acu.md) 如何協助您比較各個 Azure SKU 的計算效能。
