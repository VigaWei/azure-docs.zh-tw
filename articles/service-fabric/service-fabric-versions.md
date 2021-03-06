---
title: Azure Service Fabric 中支援的叢集版本
description: 瞭解 Azure Service Fabric 中的叢集版本，包括 Service Fabric team blog 中最新版本的連結。
ms.topic: troubleshooting
ms.date: 06/15/2020
ms.openlocfilehash: c2ea2b53649cf148a19df46835c8936345aa20e5
ms.sourcegitcommit: c7153bb48ce003a158e83a1174e1ee7e4b1a5461
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 01/15/2021
ms.locfileid: "98234336"
---
# <a name="supported-service-fabric-versions"></a>支援的 Service Fabric 版本

確定您的叢集一律執行支援的 Azure Service Fabric 版本。 在我們宣佈發行新版 Service Fabric 的最少60天后，就會結束對前一版的支援。 您可以在 [Service Fabric team blog](https://azure.microsoft.com/updates/?product=service-fabric)上找到新版本的公告。

針對特定版本的 Service Fabric 執行時間，您可以使用指定或較舊版本的 SDK/NuGet 套件。 不支援較新版本的套件，而且可能會有以較舊叢集為目標的問題，因為它們可能具有這些環境不支援的功能或通訊協定變更。

請參閱下列檔，以取得如何讓您的叢集保持執行支援的 Service Fabric 版本的詳細資料：

- [升級 Azure Service Fabric 叢集](service-fabric-cluster-upgrade.md)
- [升級在獨立 Windows Server 叢集上執行的 Service Fabric 版本](service-fabric-cluster-upgrade-windows-server.md)


## <a name="unsupported-versions"></a>不支援的版本

### <a name="upgrade-alert-for-versions-between-57-and-below-6363"></a>5.7 和以下6.3.63 之間版本的升級警示。 *
為了改善安全性和可用性，Azure 基礎結構將會進行變更，而這可能會影響 Service Fabric 客戶。 **從5.7 到6.3 的不支援版本上的所有 Service Fabric 叢集都會受到影響**。 解決此變更需要更新 Service Fabric 執行時間，而這些執行時間已可供所有區域中的所有支援 Service Fabric 版本使用。

我們會要求並建議您採取動作，以升級至最新支援的版本，並于 **1 月19日 2021** ，以避免服務中斷。如果您有支援方案，而且需要技術協助，請開啟 Azure Service Fabric 的支援要求，並在支援票證中提及此內容，以透過 Azure 支援通道與我們聯繫。

#### <a name="impact-if-not-upgraded-to-supported-versions"></a>如果未升級至支援的版本時的影響

**在不支援的版本（從5.7 到6.3.63）執行的 Azure Service Fabric 叢集。 \*** 如果您未在2021年1月19日升級為下列支援版本的其中一個，將無法啟動，也無法使用。

#### <a name="required-action"></a>必要的動作
升級至下列 Service Fabric 支援的版本，以避免停機或與此變更相關的功能遺失。 請確定您的叢集至少執行了這些版本，以防止您的環境發生問題。

  ###### <a name="supported-service-fabric-runtime-versions"></a>支援的 Service Fabric 執行階段版本
   如果您不在下面列出的 Service Fabric 支援版本，請升級為其中一個已包含必要變更的版本，以防止叢集停機。 **注意：** 7.2 的所有發行版本都包含必要的變更。
  
  | OS | 叢集中目前的 Service Fabric 執行時間 | CU/Patch 版本  | 
  | --- | --- |--- | 
  | Windows | 7.0. * | 7.0.478.9590 |
  | Windows | 7.1. * | 7.1.503.9590 |
  | Windows | 7.2. * | 7.2. * |
  | Ubuntu 16 | 7.0. * | 7.0.472.1  |
  | Linux Ubuntu 16.04 | 7.1. * | 7.1.455.1  |
  | Linux Ubuntu 18.04 | 7.1. * | 7.1.455.1804 |
  | Linux Ubuntu 16.04 | 7.2. * | 7.2. * |
  | Linux Ubuntu 18.04 | 7.2. * | 7.2. * |
 
### <a name="upgrade-alert-for-versions-greater-than-63"></a>版本超過6.3 的升級警示 
為了改善安全性和可用性，Azure 基礎結構將會進行變更，而這可能會影響 Service Fabric 客戶。 **所有使用 [容器的開放網路功能功能的](https://docs.microsoft.com/azure/service-fabric/service-fabric-networking-modes#set-up-open-networking-mode)Service Fabric 叢集，在不支援的版本（大於6.3）和低於7.0 的版本上執行，且不相容的支援版本從7.0 開始會受到影響**。 解決此變更需要更新 Service Fabric 執行時間，而這些執行時間已可供所有區域中的所有支援 Service Fabric 版本使用。

 #### <a name="impact-if-not-upgraded-to-supported-versions"></a>如果未升級至支援的版本時的影響
  **使用 [容器的開放網路功能功能](https://docs.microsoft.com/azure/service-fabric/service-fabric-networking-modes#set-up-open-networking-mode)的 Azure Service Fabric 叢集功能適用于容器，並在大於6.3 且** 不包含變更的版本上執行，如果未在 **2021 年1月 19** 日升級為下列支援版本的其中一個，則會遇到功能遺失或服務中斷的情況。
 
  - 如果叢集 **執行的 Service Fabric 大於6.3 但未使用開放式網路功能功能**，叢集將會保持運作，不過容器叢集的開放網路功能會停止運作，而導致工作負載的服務中斷。

 - 如果叢集 **執行的 Service Fabric 大於6.3，並使用 [適用于容器的開放網路功能功能](https://docs.microsoft.com/azure/service-fabric/service-fabric-networking-modes#set-up-open-networking-mode)** ，叢集可能會變成無法使用，而且將會停止運作，而導致工作負載的服務中斷。
  
#### <a name="required-action"></a>必要的動作
升級至下列 Service Fabric 支援的版本，以避免停機或與此變更相關的功能遺失。 請確定您的叢集至少執行了這些版本，以防止您的環境發生問題。 
 
 ###### <a name="supported-service-fabric-runtime-versions"></a>支援的 Service Fabric 執行階段版本
 如果您不在下面列出的 Service Fabric 支援版本，請升級為其中一個已包含必要變更的版本，以防止功能遺失。  **注意：** 7.2 的所有發行版本都包含必要的變更。
 
  | OS | 叢集中目前的 Service Fabric 執行時間 | CU/Patch 版本  | 
  | --- | --- |--- | 
  | Windows | 7.0. * | 7.0.478.9590 |
  | Windows | 7.1. * | 7.1.503.9590 |
  | Windows | 7.2. * | 7.2. * |
  | Linux Ubuntu 16.04 | 7.0. * | 7.0.472.1  |
  | Linux Ubuntu 16.04 | 7.1. * | 7.1.455.1  |
  | Linux Ubuntu 18.04 | 7.1. * | 7.1.455.1804 |
  | Linux Ubuntu 16.04 | 7.2. * | 7.2. * |
  | Linux Ubuntu 18.04 | 7.2. * | 7.2. * |

## <a name="supported-versions"></a>支援的版本
下表列出 Service Fabric 的版本及其支援結束日期。

| 叢集中的 Service Fabric 執行階段 | 可直接自叢集版本升級 |相容的 SDK 或 NuGet 套件版本 | 結束支援 |
| --- | --- |--- | --- |
| 5.3.121 前的所有叢集版本 | 5.1.158.* |小於或等於 2.3 版本 |2017 年 1 月 20 日 |
| 5.3.* | 5.1.158.* |小於或等於 2.3 版本 |2017 年 2 月 24 日 |
| 5.4.* | 5.1.158.* |小於或等於 2.4 版本 |2017 年 5 月 10 日       |
| 5.5.* | 5.4.164.* |小於或等於 2.5 版本 |2017 年 8 月 10 日    |
| 5.6.* | 5.4.164.* |小於或等於 2.6 版本 |2017 年 10 月 13 日   |
| 5.7.* | 5.4.164.* |小於或等於 2.7 版 |2017 年 12 月 15 日  |
| 6.0.* | 5.6.205.* |小於或等於 2.8 版 |2018年3月30日     |
| 6.1.* | 5.7.221.* |小於或等於 3.0 版 |2018年7月15日      |
| 6.2.* | 6.0.232.* |小於或等於 3.1 版 |2018年10月26日   |
| 6.3.* | 6.1.480.* |小於或等於 3.2 版 |2019年3月31日  |
| 6.4.* | 6.2.301.* |小於或等於 3.3 版 |2019年9月15日 |
| 6.5. * | 6.4.617.* |小於或等於3.4 版 |2020年8月1日 |
| 7.0.466.* | 6.4.664.* |小於或等於4.0 版|2021年1月31日  |
| 7.0.466.* | 6.5. * |小於或等於4.0 版|2021年1月31日 |
| 7.0.470.* | 7.0.466.* |小於或等於4.0 版 |2021年1月31日  |
| 7.0.472.* | 7.0.466.* |小於或等於4.0 版 |2021年1月31日  |
| 7.0.478.* | 7.0.466.* |小於或等於4.0 版 |2021年1月31日  |
| 7.1.409.* | 7.0.466.* |小於或等於4.1 版 |2021年3月31日 |
| 7.1.417.* | 7.0.466.* |小於或等於4.1 版 |2021年3月31日 |
| 7.1.428.* | 7.0.466.* |小於或等於4.1 版 |2021年3月31日 |
| 7.1.456.* | 7.0.466.* |小於或等於4.1 版 |2021年3月31日 |
| 7.1.458.* | 7.0.466.* |小於或等於4.1 版 |2021年3月31日 |
| 7.1.459.* | 7.0.466.* |小於或等於4.1 版 |2021年3月31日 |
| 7.1.503.* | 7.0.466.* |小於或等於4.1 版 |2021年3月31日 |
| 7.2.413.* | 7.0.470.* |小於或等於4.2 版 |目前的版本，因此沒有結束日期 |
| 7.2.432.* | 7.0.470.* |小於或等於4.2 版 |目前的版本，因此沒有結束日期 |
| 7.2.433.* | 7.0.470.* |小於或等於4.2 版 |目前的版本，因此沒有結束日期 |
| 7.2.445.* | 7.0.470.* |小於或等於4.2 版 |目前的版本，因此沒有結束日期 |

## <a name="supported-operating-systems"></a>支援的作業系統

下表列出支援的 Service Fabric 版本支援的作業系統。

| 作業系統 | 最早支援的 Service Fabric 版本 |
| --- | --- |
| Windows Server 2012 R2 | 所有版本 |
| Windows Server 2016 | 所有版本 |
| Windows Server 1709 | 6.0 |
| Windows Server 1803 | 6.4 |
| Windows Server 1809 | 6.4.654.9590 |
| Windows Server 2019 | 6.4.654.9590 |
| Linux Ubuntu 16.04 | 6.0 |
| Linux Ubuntu 18.04 | 7.1 |

## <a name="supported-version-names"></a>支援的版本名稱

下表列出 Service Fabric 的版本名稱及其對應的版本號碼。

| 版本名稱 | Windows 版本號碼 | Linux 版本號碼 |
| --- | --- | --- |
| 5.3 RTO | 5.3.121.9494 | NA |
| 5.3 CU1 | 5.3.204.9494 | NA |
| 5.3 CU2 | 5.3.301.9590 | NA |
| 5.3 CU3 | 5.3.311.9590 | NA |
| 5.4 CU2 | 5.4.164.9494 | NA |
| 5.5 CU1 | 5.5.216.0    | NA |
| 5.5 CU2 | 5.5.219.0    | NA |
| 5.5 CU3 | 5.5.227.0    | NA |
| 5.5 CU4 | 5.5.232.0 | NA |
| 5.6 RTO | 5.6.204.9494 以上 | NA |
| 5.6 CU2 | 5.6.210.9494 | NA |
| 5.6 CU3 | 5.6.220.9494 | NA |
| 5.7 RTO | 5.7.198.9494 | NA |
| 5.7 CU4 | 5.7.221.9494 | NA |
| 6.0 RTO | 6.0.211.9494 | 6.0.120.1 |
| 6.0 CU1 | 6.0.219.9494 | 6.0.127.1 |
| 6.0 CU2 | 6.0.232.9494 | 6.0.133.1 |
| 6.1 CU1 | 6.1.456.9494 | 6.1.183.1 |
| 6.1 CU2 | 6.1.467.9494 | 6.1.185.1 |
| 6.1 CU3 | 6.1.472.9494 | NA |
| 6.1 CU4 | 6.1.480.9494 | 6.1.187.1 |
| 6.2 RTO | 6.2.269.9494 | 6.2.184.1 | 
| 6.2 CU1 | 6.2.274.9494 | 6.2.191.1 |
| 6.2 CU2 | 6.2.283.9494 | 6.2.194.1 |
| 6.2 CU3 | 6.2.301.9494 | 6.2.199.1 |
| 6.3 RTO | 6.3.162.9494 | 6.3.119.1 |
| 6.3 CU1 | 6.3.176.9494 | 6.3.124.1 |
| 6.3 CU1 | 6.3.187.9494 | 6.3.129.1 |
| 6.4 RTO | 6.4.617.9590 | 6.4.625.1 |
| 6.4 CU2 | 6.4.622.9590 | NA |
| 6.4 CU3 | 6.4.637.9590 | 6.4.634.1 |
| 6.4 CU4 | 6.4.644.9590 | 6.4.639.1 |
| 6.4 CU5 | 6.4.654.9590 | 6.4.649.1 |
| 6.4 CU6 | 6.4.658.9590 | NA |
| 6.4 CU7 | 6.4.664.9590 | 6.4.661.1 |
| 6.4 CU8 | 6.4.670.9590 | NA |
| 6.5 RTO | 6.5.639.9590 | 6.5.435.1 版 |
| 6.5 CU1 | 6.5.641.9590 | 6.5.454.1 |
| 6.5 CU2 | 6.5.658.9590 | 6.5.460.1 |
| 6.5 CU3 | 6.5.664.9590 | 6.5.466.1 |
| 6.5 CU5 | 6.5.676.9590 | 6.5.467.1 |
| 7.0 RTO | 7.0.457.9590 | 7.0.457.1 |
| 7.0 CU2 | 7.0.464.9590 | 7.0.464.1 |
| 7.0 CU3 | 7.0.466.9590 | 7.0.465.1 |
| 7.0 CU4 | 7.0.470.9590 | 7.0.469.1 |
| 7.0 CU6 | 7.0.472.9590 | 7.0.471.1 |
| 7.0 CU9 | 7.0.478.9590 | 7.0.472.1 |
| 7.1 RTO | 7.1.409.9590 | 7.1.410.1 |
| 7.1 CU1 | 7.1.417.9590 | 7.1.418.1 |
| 7.1 CU2 | 7.1.428.9590 | 7.1.428.1 |
| 7.1 CU3 | 7.1.456.9590 | 7.1.452.1 |
| 7.1 CU5 | 7.1.458.9590 | 7.1.454.1 |
| 7.1 CU6 | 7.1.459.9590 | 7.1.455.1 |
| 7.1 CU8 | 7.1.503.9590 | 7.1.508.1 |
| 7.2 RTO | 7.2.413.9590 | NA |
| 7.2 CU2 | 7.2.432.9590 | 7.2.431.1 |
| 7.2 CU3 | 7.2.433.9590 | NA |
| 7.2 CU4 | 7.2.445.9590 | 7.2.447.1 |

