---
title: 將建議的動作新增至訊息 | Microsoft Docs
description: 了解如何使用適用於 .NET 的 Bot Builder SDK，將建議的動作新增至訊息。
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 03/13/2018
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: b03003d4822c657f9b606de36087ddf4d5cbee7a
ms.sourcegitcommit: b78fe3d8dd604c4f7233740658a229e85b8535dd
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/24/2018
ms.locfileid: "49997646"
---
# <a name="add-suggested-actions-to-messages"></a>將建議的動作新增至訊息

[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]

> [!div class="op_single_selector"]
> - [.NET](../dotnet/bot-builder-dotnet-add-suggested-actions.md)
> - [Node.js](../nodejs/bot-builder-nodejs-send-suggested-actions.md)
> - [REST](../rest-api/bot-framework-rest-connector-add-suggested-actions.md)

[!INCLUDE [Introduction to suggested actions](../includes/snippet-suggested-actions-intro.md)]

> [!TIP]
> 使用[通道偵測器][channelInspector]來查看建議動作的外觀，以及在各種通道上的運作方式。

## <a name="send-suggested-actions"></a>傳送建議的動作

若要將建議的動作新增至訊息，請將活動的 `SuggestedActions` 屬性設為 [CardAction][cardAction] 物件清單，這些物件代表要向使用者呈現的按鈕。 

此程式碼範例示範如何建立一則訊息，向使用者呈現三種建議的動作：

[!code-csharp[Add suggested actions](../includes/code/dotnet-add-suggested-actions.cs#addSuggestedActions)]

當使用者點選其中一個建議的動作時，Bot 會收到來自使用者的訊息，其中包含對應動作的 `Value`。

## <a name="additional-resources"></a>其他資源

- [使用頻道偵測器來預覽功能][inspector]
- [活動概觀](bot-builder-dotnet-activities.md)
- [建立訊息](bot-builder-dotnet-create-messages.md)
- <a href="https://docs.botframework.com/en-us/csharp/builder/sdkreference/dc/d2f/class_microsoft_1_1_bot_1_1_connector_1_1_activity.html" target="_blank">Activity 類別</a> \(英文\)
- <a href="/dotnet/api/microsoft.bot.connector.imessageactivity" target="_blank">IMessageActivity 介面</a>
- <a href="/dotnet/api/microsoft.bot.connector.cardaction" target="_blank">CardAction 類別</a>
- <a href="/dotnet/api/microsoft.bot.connector.suggestedactions" target="_blank">SuggestedActions 類別</a>

[cardAction]: /dotnet/api/microsoft.bot.connector.cardaction

[inspector]: ../bot-service-channel-inspector.md

[channelInspector]: ../bot-service-channel-inspector.md


