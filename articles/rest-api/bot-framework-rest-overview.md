---
title: Bot 建立器 REST API | Microsoft Docs
description: 開始使用 Bot Framework REST API，此 REST API 可用來建置 Bot 和連線到 Bot 的用戶端。
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 12/13/2017
ms.openlocfilehash: d6d83edb390933cb8895b26efeb9775cafdb1acb
ms.sourcegitcommit: f576981342fb3361216675815714e24281e20ddf
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/18/2018
ms.locfileid: "39299339"
---
# <a name="bot-framework-rest-apis"></a>Bot Framework REST API
> [!div class="op_single_selector"]
> - [.NET](../dotnet/bot-builder-dotnet-overview.md)
> - [Node.js](../nodejs/bot-builder-nodejs-overview.md)
> - [REST](../rest-api/bot-framework-rest-overview.md)

Bot Framework REST API 可讓您建置 Bot，以便與 <a href="https://dev.botframework.com/" target="_blank">Bot Framework 入口網站</a>中設定的通道交換訊息、儲存和擷取狀態資料，以及將自己的用戶端應用程式連線到 Bot。 所有 Bot Framework 服務皆透過 HTTPS 使用業界標準的 REST 和 JSON。

## <a name="build-a-bot"></a>建置 Bot

您可以使用 Bot 連接器服務與 Bot Framework 入口網站中設定的通道交換訊息，藉以使用任何程式設計語言建立 Bot。 

> [!TIP]
> Bot Framework 提供可用於在 C# 或 Node.js 中建置 Bot 的用戶端程式庫。 若要使用 C# 建置 Bot，請使用[適用於 C# 的 Bot 建立器 SDK](../dotnet/bot-builder-dotnet-overview.md)。 若要使用 Node.js 建置 Bot，請使用[適用於 Node.js 的 Bot 建立器 SDK](../nodejs/index.md)。 

若要深入了解如何使用 Bot 連接器服務建置 Bot，請參閱[重要概念](bot-framework-rest-connector-concepts.md)。

## <a name="build-a-client"></a>建置用戶端

您可以使用 Direct Line API，讓自己的用戶端應用程式與 Bot 進行通訊。 Direct Line API 可實作使用標準密碼/權杖模式，並提供穩定結構描述的驗證機制，即使您的 Bot 會變更其通訊協定版本也一樣。 若要深入了解如何使用 Direct Line API 來啟用用戶端與 Bot 之間的通訊，請參閱[重要概念](bot-framework-rest-direct-line-3-0-concepts.md)。 