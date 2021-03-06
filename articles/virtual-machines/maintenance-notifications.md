---
title: 維修通知
description: 針對在 Azure 中執行的虛擬機器進行維護通知的總覽。
author: shants123
ms.service: virtual-machines
ms.workload: infrastructure-services
ms.topic: conceptual
ms.date: 8/12/2020
ms.author: shants
ms.openlocfilehash: 53cde1178a4faae0fbd11222e4219f70be29145d
ms.sourcegitcommit: 04fb3a2b272d4bbc43de5b4dbceda9d4c9701310
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 11/12/2020
ms.locfileid: "94560803"
---
# <a name="handling-planned-maintenance-notifications"></a>處理預定的維修通知

為提升虛擬機器之主機基礎結構的可靠性、效能和安全性，Azure 會定期執行更新。 更新會變更，例如裝載環境的修補或將硬體升級與解除委任。 大部分的更新都會完成，而不會影響到託管的虛擬機器。 不過，更新在某些情況下確實會造成影響：

- 如果維護不需要重新開機，Azure 會在主機更新時暫停 VM 幾秒鐘。 這些類型的維護作業會依容錯網域套用容錯網域。 如果收到任何警告健康情況訊號，就會停止進度。

- 如果維護需要重新開機，您會在規劃維護時收到通知。 您會獲得大約35天的時間範圍，您可以在這段時間內自行開始維護。


預定進行的維護作業若需要重新開機，會排定在不同波段。 每一波段有不同的範圍 (區域)。

- 波段開始時會傳送通知給客戶。 根據預設，通知會傳送給訂用帳戶管理員和共同管理員。 您可以使用 [活動記錄警示](../service-health/alerts-activity-log-service-notifications-portal.md)來新增更多收件者和郵件選項，例如電子郵件、SMS 和 webhook。  
- 一旦通知消失，就會提供 *自助服務視窗* 。 在此視窗中，您可以查詢受影響的虛擬機器，並根據您自己的排程需求開始維護。 自助服務視窗通常大約是35天。
- 在自助期間之後，「排定維護期間」隨即開始。 在此期間的某個時間點，Azure 會排定並將必要的維護套用於您的虛擬機器。 

有兩個期間的目標是要讓您在知道 Azure 何時將會自動開始維修的同時，有足夠時間開始維修，並將虛擬機器重新啟動。


您可以使用 Azure 入口網站、PowerShell、REST API 和 CLI 來查詢您的 VM 的維修期間，並開始自助維修。

 
## <a name="should-you-start-maintenance-using-during-the-self-service-window"></a>您是否應該在自助期間開始維護？  

下列指導方針應該可以協助您決定是否要使用此功能，並且在自己的時間開始維護。 

> [!NOTE] 
> 自助式維護可能不適用於所有的虛擬機器。 若要判斷主動式重新部署是否可供您的虛擬機器使用，請在維護狀態中尋找 [立即開始]。 雲端服務 (Web/背景工作角色) 和 Service Fabric 目前無法使用自助維護。


使用 **可用性設定組** 進行部署時，不建議使用自助維護。 可用性設定組一次只能更新一個更新網域。 

- 讓 Azure 觸發維護工作。 針對需要重新開機的維護，將會依更新網域來更新網域。 更新網域不一定會依序接收維護，而且更新網域之間會暫停30分鐘。 
- 如果暫時遺失某些容量 (1 更新網域) 有問題，您可以在維護期間新增實例。 
- 針對不需要重新開機的維護，系統會在容錯網域層級套用更新。 

在下列情況， **請勿** 使用自助式維護： 
- 如果您經常關閉 VM (不論是手動、使用 DevTest Labs、使用自動關機，或依照排程)，都可能還原維護狀態，因而造成額外的停機時間。
- 在您知道將會在維護波段結束前被刪除的短期存留虛擬機器上。 
- 在具有大量狀態的工作負載中，其大量狀態儲存在想要於更新時進行維護的本機 (暫時) 磁碟中。 
- 在您時常調整虛擬機器大小的情況下，因為這可能會還原維護狀態。 
- 如果採用的排定事件能夠讓工作負載得以主動容錯移轉或順利關機，則在維護關機開始前 15 分鐘不應使用自助維護

如果您打算在排定維護階段連續不中斷地執行您的虛擬機器，而且上述的反指標都不適用，則 **使用** 自助式維護。 

在下列情況下，最好使用自助式維護：
- 您需要向管理階層或一般客戶傳達確切的維護期間。 
- 您需要在指定的日期前完成維護作業。 
- 您需要控制維護序列 (例如多層式應用程式) 以保證安全復原。
- 在兩個更新網域 (ud) 之間需要超過30分鐘的 VM 復原時間。 若要控制更新網域之間的時間，您必須在虛擬機器上觸發維護 (一次一個更新網域 (UD))。


## <a name="faq"></a>常見問題集


**問：為什麼需要立即重新啟動我的虛擬機器？**

**答：** 雖然對 Azure 平台的大部分更新與升級不會影響虛擬機器的可用性，但是有時候我們無法避免重新啟動裝載於 Azure 的虛擬機器。 我們已經累積數個變更，需要我們重新啟動伺服器，這會導致虛擬機器重新開機。

**問：如果我依照建議使用可用性設定組來獲得高可用性，是否安全？**

**答：** 部署在可用性設定組或虛擬機器擴展集的虛擬機器，具有更新網域 (UD) 的概念。 執行維護時，Azure 會接受 UD 條件約束，並且不會從不同的 UD (在相同的可用性設定組內) 重新啟動虛擬機器。  Azure 也會等候至少 30 分鐘，再移至下一個虛擬機器群組。 

如需高可用性的詳細資訊，請參閱 [Azure 中的虛擬機器可用性](availability.md)。

**問：我如何取得規劃維護的通知？**

**答：** 規劃的維護是從對一或多個 Azure 區域設定排程開始。 不久之後，就會將電子郵件通知傳送給訂用帳戶管理員、共同管理員、擁有者和參與者， (每個訂用帳戶) 一封電子郵件。 可以使用「活動記錄警示」來設定此通知的其他通道和收件者。 如果您將虛擬機器部署到已排程規劃的維護之區域，您不會收到通知，但是需要檢查 VM 的維護狀態。

**問：我在入口網站、PowerShell 或 CLI 中都看不到任何預定維護的指示。怎麼了？**

**答：** 只有在計劃性維護期間將受到該維護波段影響的 VM 可以取得與計劃性維護相關的資訊。 也就是說，如果您看不到資料，可能是維護已完成 (或尚未啟動)，或者您的虛擬機器是裝載在更新的伺服器上。

**問：是否有方法可以確切知道我的虛擬機器何時會受到影響？**

**答：** 設定排程時，我們會定義數天的時間期間。 不過，此期間內伺服器 (和 VM) 的確切順序則未知。 想要知道其 VM 確切時間的客戶，可以使用[排定的事件](./linux/scheduled-events.md)並且從虛擬機器內查詢，而後會在 VM 重新開機前 15 分鐘收到通知。

**問：重新啟動我的虛擬機器需要多久時間？**

**答：** 根據您的虛擬機器大小，在自助式維護期間重新啟動最長可能需要幾分鐘的時間。 在 Azure 於排定維護期間起始重新啟動期間，重新啟動通常需要大約 25 分鐘。 請注意，如果您使用雲端服務 (Web/背景工作角色)、虛擬機器擴展集或可用性設定組，在排定維護期間每個虛擬機器群組 (UD) 之間有 30 分鐘。

**問：虛擬機器擴展集的體驗是什麼？**

**答：** 規劃的維護現在可供虛擬機器擴展集使用。 如需有關如何起始自助維護的指示，請參閱 [虛擬機器擴展集檔的預定維修](../virtual-machine-scale-sets/virtual-machine-scale-sets-maintenance-notifications.md) 。

**問：雲端服務 (Web/背景工作角色) 和 Service Fabric 的體驗是什麼？**

**答：** 這些平台會受到規劃的維護影響，使用這些平台的客戶是安全的，因為只有單一升級網域 (UD) 中的 VM 會在任何指定時間受到影響。 雲端服務 (Web/背景工作角色) 和 Service Fabric 目前無法使用自助維護。

**問：我在 Vm 上看不到任何維護資訊。問題出在哪裡？**

**答：** 您在 VM 上看不到任何維護資訊有以下幾個原因：
1.  您使用標示為 Microsoft 內部的訂用帳戶。
2.  您的 VM 未排程進行維護。 可能是維護波段已經結束、取消或修改，所以您的 VM 不再受到它的影響。
3. 您已解除配置 VM，然後將它啟動。 這可能會導致 VM 移至未排定預定維護 wave 的位置。 因此，VM 將不會再顯示維護資訊。 
4.  您尚未將 [維護] 資料行新增至您的虛擬機器清單檢視。 雖然我們已將此資料行新增至預設檢視，但是設定為查看非預設資料行的客戶必須將 [維護] 資料行手動新增至其 VM 清單檢視。

**問：我的 VM 第二次排程進行維護。為什麼？**

**答：** 有數個使用案例，您會在您已完成維護重新部署之後，看到 VM 排程進行維護：
1.  我們已取消維護，並且使用不同的裝載重新啟動它。 可能是我們偵測到發生錯誤的裝載，只是需要部署額外承載。
2.  因為硬體故障，您的 VM 已 *服務修復* 到另一個節點。
3.  您已經選擇停止 (解除配置)，然後重新啟動 VM。
4.  您已經為 VM 開啟 **自動關機** 。



## <a name="next-steps"></a>後續步驟

您可以使用 [Azure CLI](maintenance-notifications-cli.md)、 [Azure PowerShell](maintenance-notifications-powershell.md) 或 [入口網站](maintenance-notifications-portal.md)來處理預定的維護。
