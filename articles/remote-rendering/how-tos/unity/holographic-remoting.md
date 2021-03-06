---
title: 在 Unity 中使用全像攝影遠端處理和遠端轉譯
description: 如何搭配使用全像攝影遠端預覽的方式 Azure 遠端轉譯
author: christophermanthei
ms.author: chmant
ms.date: 03/23/2020
ms.topic: how-to
ms.openlocfilehash: bd69710b6e4404a76d3cca385b6c07ea7c1f3f5c
ms.sourcegitcommit: 957c916118f87ea3d67a60e1d72a30f48bad0db6
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/19/2020
ms.locfileid: "92201812"
---
# <a name="use-holographic-remoting-and-remote-rendering-in-unity"></a>在 Unity 中使用全像攝影遠端處理和遠端轉譯

全像是在一個應用程式內進行全像的[遠端處理](/windows/mixed-reality/holographic-remoting-player)和 Azure 遠端轉譯。 因此， [Unity play 模式](/windows/mixed-reality/unity-play-mode) 也無法使用。

每次執行 Unity 編輯器時，都只能使用其中一種。 若要使用另一個帳戶，請先重新開機 Unity。

## <a name="use-unity-play-mode-to-preview-on-hololens-2"></a>使用 Unity play 模式在 HoloLens 2 上預覽

 您仍然可以使用 Unity 播放模式，例如測試應用程式的 UI。 不過，ARR 絕對不會初始化。 否則將會損毀。

## <a name="use-a-wmr-vr-headset-to-preview-on-desktop"></a>使用 WMR VR 耳機在桌面上預覽

如果有 Windows Mixed Reality 的 VR 耳機，則可以用來在 Unity 內進行預覽。 在此情況下，您可以初始化 ARR，不過在使用 WMR 耳機時，將無法連接至會話。