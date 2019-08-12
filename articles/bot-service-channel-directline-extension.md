---
title: Direct Line App Service 擴充功能
titleSuffix: Bot Service
description: Direct Line App Service 擴充功能
services: bot-service
manager: kamrani
ms.service: bot-service
ms.topic: conceptual
ms.author: kamrani
ms.date: 07/09/2019
ms.openlocfilehash: 27067cf2582de63cf67785cfaaa70b9a33685894
ms.sourcegitcommit: a1eaa44f182a7210197bd793250907df00e9edab
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/03/2019
ms.locfileid: "68757760"
---
## <a name="direct-line-app-service-extension"></a>Direct Line App Service 擴充功能

[!INCLUDE[applies-to-v4](includes/applies-to.md)]

Direct Line App Service 擴充功能會建立一組**永續性具名管道**，以透過客戶可新增的 **BotAdapter** 連線至 Bot。

擴充功能會將一組新的串流擴充功能新增至 Bot Framework 通訊協定。 這些擴充功能會將用來交換訊息的 HTTP 傳輸機制，取代為允許透過**永續性 WebSocket** 傳送雙向要求的傳輸機制。 這樣可以提高效能，並且可增加資訊交換期間的隔離性。

在串流擴充功能之前，Direct Line API 提供了一種方式讓用戶端將活動傳送至 Direct Line，並提供兩種方式讓用戶端從 Direct Line 擷取活動。 訊息會透過 HTTP POST 進行傳送，並由 HTTP GET (輪詢) 來接收，或藉由開啟 WebSocket 以接收 ActivitySets，來接收訊息。
串流擴充功能擴充了 WebSocket 的用途，且允許**所有訊息通訊**都在該 WebSocket 上傳送。 串流擴充功能也可在通道服務與 Bot 之間使用。


Direct Line App Service 擴充功能會預先安裝在全世界各個資料中心的所有 Azure App Service 執行個體上。 此擴充功能由 Microsoft 維護及管理，客戶不需要進行額外的部署工作。
Azure App Service 依預設會加以停用，但您可以輕鬆地將其開啟，讓其能夠連線至您裝載的 Bot。

下圖顯示整體架構：

![Direct Line 擴充功能架構](./media/channels/direct-line-extension-architecture.png)

## <a name="see-also"></a>另請參閱

|Name|說明|
|---|---|
|[具有擴充功能的 .NET bot](bot-service-channel-directline-extension-net-bot.md)|更新 Bot 以使用**具名管道**，並在裝載 Bot 的 **Azure App Service** 資源中啟用 Direct Line App Service 擴充功能。  |
|[具有擴充功能的 .NET 用戶端](bot-service-channel-directline-extension-net-client.md)|以 C# 建立連線至 Direct Line App Service 擴充功能的 .NET 用戶端|
|[具有擴充功能的 WebChat](bot-service-channel-directline-extension-webchat-client.md)|搭配使用 WebChat 與 Direct Line App Service 擴充功能|
|[在 VNET 中使用擴充功能](bot-service-channel-directline-extension-vnet.md)|搭配使用 Direct Line App Service 擴充功能與 Azure 虛擬網路 (VNET)|

## <a name="addtional-resources"></a>其他資源

- [將 Bot 連線至 Direct Line](bot-service-channel-connect-directline.md)
- [將 Bot 連線至 Direct Line Speech](bot-service-channel-connect-directlinespeech.md)
