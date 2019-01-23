---
title: 即時媒體 Bot 的需求和考量 | Microsoft Docs
description: 了解使用適用於 .NET 的 Bot Framework SDK，來建立適用於 Skype 之即時媒體的重要需求和考量。
author: ssulzer
ms.author: ssulzer
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 12/13/17
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: d8cd326a3027fe5fcb440d9b205ba7d32a8b1640
ms.sourcegitcommit: b15cf37afc4f57d13ca6636d4227433809562f8b
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 01/11/2019
ms.locfileid: "54224933"
---
# <a name="requirements-and-considerations-for-real-time-media-bots"></a>即時媒體 Bot 的需求和考量

[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]

並非所有指導方針都適用於開發傳訊，而呼叫 Bot 的 IVR 同樣適用於建置即時媒體 Bot。 本文說明一些適用於開發即時媒體 Bot 的重要需求和考量。 

> [!NOTE]
> 由於適用於 Bot 的即時媒體平台為一項預覽技術，因此，本文中的指導方針可能會有所變化。

## <a name="real-time-media-bot-development-requires-cnet-and-windows-server"></a>開發即時媒體 Bot 需要 C#/.NET 和 Windows Server

- 即時媒體 Bot 需要 `Microsoft.Skype.Bots.Media` .NET 程式庫 (可透過 <a href="https://www.nuget.org/" target="_blank">NuGet</a> 取得) 來存取音訊和視訊媒體，而 Bot 必須部署於 Windows Server 電腦上 (或 Azure 中的 Windows Server 客體 OS)。 因此，Bot 應該使用 C# 和標準的 .NET Framework 來開發並部署於 Azure 中。 您無法使用 C++ 和 Node.js API 來存取即時媒體。

- 即時媒體 Bot 必須在 `Microsoft.Skype.Bots.Media` .NET 程式庫的最新版本上執行。 Bot 應該使用 <a href="https://www.nuget.org/" target="_blank">NuGet</a> 套件的最新可用版本，或者未超過三個月的版本。 較舊版本的程式庫將會被取代，而且將在數個月之後無法運作。

## <a name="real-time-media-calls-stay-on-the-machine-where-they-were-created"></a>即時媒體呼叫會保留在它們建立所在的電腦上

- 即時媒體 Bot 是極具狀態的。 即時媒體呼叫會釘選到接受傳入呼叫的虛擬機器 (VM) 執行個體：來自 Skype 呼叫端的語音和視訊媒體將流向該 VM 執行個體，而 Bot 要傳回給呼叫端的媒體也必須源自該 VM。

- 當 VM 停止時，如果有任何即時媒體呼叫正在進行，則那些呼叫將會突然終止。 如果 Bot 能夠得知擱置中的 VM 關機，它就可以嘗試「正常地」結束該呼叫。

## <a name="real-time-media-bots-must-be-directly-accessible-on-the-internet"></a>即時媒體 Bot 必須在網際網路上直接存取

- 裝載即時媒體 Bot 的每個 VM 執行個體都必須從網際網路直接存取。 針對 Azure 中裝載的即時媒體 Bot，每個 VM 執行個體都必須有一個執行個體層級的公用 IP 位址 (ILPIP)。 如需取得和設定 ILPIP 的相關資訊，請參閱<a href="/azure/virtual-network/virtual-networks-instance-level-public-ip" target="_blank">執行個體層級的公用 IP (傳統) 概觀</a>。 依預設，Azure 訂用帳戶可以取得 5 個 ILPIP 位址；請連絡 Azure 支援服務以提高您訂用帳戶的配額。

- 由於公用 IP 位址的需求，即時媒體 Bot 都必須裝載於 "IaaS" Azure 虛擬機器或「傳統」的 Azure 雲端服務中。 不支援其他執行階段環境 (例如 Bot 服務或含有 VM 擴展集的 Service Fabric)，因為這些不支援 ILPIP。

- [Bot Framework 模擬器](../bot-service-debug-emulator.md)也不支援即時媒體 Bot。

## <a name="scalability-and-performance-considerations"></a>延展性和效能考量

- 即時媒體 Bot 需要比傳訊 Bot 還多的計算和網路 (頻寬) 容量，而且可能造成明顯偏高的營運成本。 即時媒體 Bot 的開發人員必須仔細地衡量 Bot 的延展性，並確定該 Bot 不接受比它可管理還多的同時呼叫。 已啟用視訊的 Bot 或許可以維持每個 CPU 核心只有一或兩個並行即時媒體呼叫。

- 適用於 Bot 之即時媒體平台的目前預覽版本具有 Bot 開發人員必須知道的特定延展性限制 (這些可能會在未來版本中改善)： 
  1. 單一 VM 執行個體在任何時候都不能有 10 個以上已建立的視訊通訊端。
  2. 即時媒體平台目前不會利用可在 VM 上取得的任何圖形處理器 (GPU) 來卸載 H.264 視訊編碼/解碼。 相反地，視訊編碼和解碼都會以 CPU 上的軟體來完成。 如果 GPU 可供使用，Bot 就可利用它來進行自己的圖形轉譯 (例如，如果 Bot 使用 3D 圖形引擎)。

- 裝載即時媒體 Bot 的 VM 執行個體必須至少有 2 個 CPU 核心。 對於 Azure，建議使用 Dv2 系列的虛擬機器。 您可以在 <a href="/azure/virtual-machines/windows/sizes-general" target="_blank">Azure 文件</a>中找到 Azure VM 類型的詳細資訊。 
