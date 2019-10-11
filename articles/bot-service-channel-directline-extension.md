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
ms.openlocfilehash: c4c54e50450ae81098992c880e23a049229fa09f
ms.sourcegitcommit: 7e901f5f39a0cfb0d37e532321b90a1dcf4baadd
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/08/2019
ms.locfileid: "72039753"
---
# <a name="direct-line-app-service-extension"></a>Direct Line App Service 擴充功能

[!INCLUDE[applies-to-v4](includes/applies-to.md)]

> [!WARNING]
> **Direct Line App Service 擴充功能**已公開**預覽**。  

Direct Line App Service 擴充功能可讓用戶端直接與 Bot 所在的主機連線。 這會提供工作負載隔離，而在某些情況下可改善效能。 下圖顯示整體架構：

![Direct Line 擴充功能架構](./media/channels/direct-line-extension-architecture.png)

Direct Line App Service 擴充功能會將一組新的串流擴充功能新增至 Bot Framework 通訊協定，取代用於與傳輸交換訊息的 HTTP，該傳輸允許透過**持續性 WebSocket** 傳送雙向要求。

在串流擴充功能之前，Direct Line API 提供了一種方式讓用戶端將活動傳送至 Direct Line，並提供兩種方式讓用戶端從 Direct Line 擷取活動。 訊息會透過 HTTP POST 進行傳送，並由 HTTP GET (輪詢) 來接收，或藉由開啟 WebSocket 以接收 ActivitySets，來接收訊息。
串流擴充功能擴充了 WebSocket 的用途，且允許**所有訊息通訊**都在該 WebSocket 上傳送。 串流擴充功能也可在通道服務與 Bot 之間使用。

Direct Line App Service 擴充功能會預先安裝在全世界各個資料中心的所有 Azure App Service 執行個體上。 此擴充功能由 Microsoft 維護及管理，客戶不需要進行額外的部署工作。 Azure App Service 依預設會加以停用，但您可以輕鬆地將其開啟，讓其能夠連線至您裝載的 Bot。


## <a name="see-also"></a>另請參閱

|名稱|說明|
|---|---|
|[設定 .NET Bot 擴充功能](bot-service-channel-directline-extension-net-bot.md)|更新 Bot 以使用**具名管道**，並在裝載 Bot 的 **Azure App Service** 資源中啟用 Direct Line App Service 擴充功能。  |
|[建立具有擴充功能的 .NET 用戶端](bot-service-channel-directline-extension-net-client.md)|以 C# 建立連線至 Direct Line App Service 擴充功能的 .NET 用戶端|
|[使用 WebChat 擴充功能](bot-service-channel-directline-extension-webchat-client.md)|搭配使用 WebChat 與 Direct Line App Service 擴充功能|
|[在 VNET 中使用擴充功能](bot-service-channel-directline-extension-vnet.md)|搭配使用 Direct Line App Service 擴充功能與 Azure 虛擬網路 (VNET)|

## <a name="addtional-resources"></a>其他資源

- [將 Bot 連線至 Direct Line](bot-service-channel-connect-directline.md)
- [將 Bot 連線至 Direct Line Speech](bot-service-channel-connect-directlinespeech.md)
