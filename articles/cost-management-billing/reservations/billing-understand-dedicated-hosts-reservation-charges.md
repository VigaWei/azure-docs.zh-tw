---
title: 了解 Azure 專用主機保留執行個體折扣
description: 了解 Azure 保留 VM 執行個體的折扣如何套用至 Azure 專用主機。
author: yashesvi
ms.service: cost-management-billing
ms.subservice: reservations
ms.topic: conceptual
ms.date: 02/28/2020
ms.author: banders
ms.openlocfilehash: 8d273aae3588a006f7b0a55d2798b7a43c040c9b
ms.sourcegitcommit: dbe434f45f9d0f9d298076bf8c08672ceca416c6
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/17/2020
ms.locfileid: "92148378"
---
# <a name="how-the-azure-reservation-discount-is-applied-to-azure-dedicated-hosts"></a>Azure 保留折扣如何套用至 Azure 專用主機

購買 Azure 保留專用主機執行個體之後，保留折扣會自動套用至符合保留屬性和數量的專用主機。 保留會涵蓋專用主機的計算成本。

## <a name="how-reservation-discount-is-applied"></a>保留折扣的套用方式

保留折扣採「不用則作廢」  的原則。 因此，如果您有任何一小時沒有相符的資源，就會失去該小時的保留數量。 您無法遞轉未使用的保留時數。

當您刪除專用主機時，保留折扣會自動套用至指定範圍中另一個相符的資源。 如果在指定的範圍內找不到相符的資源，則會「失去」  保留時數。

## <a name="reservation-discount-for-dedicated-hosts"></a>專用主機的保留折扣

Azure 保留專用主機執行個體為您的專用主機所使用的計算基礎結構成本保留折扣。 此折扣適用於您的專用主機，不論虛擬機器是否正在使用這些主機。 保留並不涵蓋額外的成本，例如授權、網路使用量，或專用主機上所部署虛擬機器的儲存體耗用量。

## <a name="need-help-contact-us"></a>需要協助嗎？ 與我們連絡

如有問題或需要協助，請 [建立支援要求](https://go.microsoft.com/fwlink/?linkid=2083458)。

## <a name="next-steps"></a>後續步驟

若要深入了解 Azure 保留項目，請參閱下列文章：

- [什麼是 Azure 的保留？](./save-compute-costs-reservations.md)

- [使用 Azure 專用主機](../../virtual-machines/dedicated-hosts.md)

- [專用主機定價](https://azure.microsoft.com/pricing/details/virtual-machines/dedicated-host/)

- [管理 Azure 的保留](./manage-reserved-vm-instance.md)

- [了解隨用隨付訂用帳戶的保留使用量](./understand-reserved-instance-usage.md)

- [了解 Enterprise 註冊的保留項目使用量](./understand-reserved-instance-usage-ea.md)

- [了解 CSP 訂用帳戶的保留使用量](/partner-center/azure-reservations)