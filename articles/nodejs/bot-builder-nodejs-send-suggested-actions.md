---
title: 將建議的動作新增至訊息中 | Microsoft Docs
description: 了解如何使用適用於 Node.js 的 Bot 建立器 SDK，在訊息內傳送建議的動作。
author: v-ducvo
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 06/06/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: 1be5423de3859b32416b5acf9ba7b9d8e92ecb14
ms.sourcegitcommit: b78fe3d8dd604c4f7233740658a229e85b8535dd
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/24/2018
ms.locfileid: "49997245"
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

## <a name="suggested-actions-example"></a>建議的動作範例

若要將建議的動作新增至訊息中，請將訊息的 `suggestedActions` 屬性設為[卡片動作][ICardAction]的清單，這些動作可以代表要向使用者呈現的按鈕。

此程式碼範例示範如何傳送一則訊息，向使用者呈現三種建議的動作：

[!code-javascript[Send suggested actions](../includes/code/node-send-suggested-actions.js#sendSuggestedActions)]

當使用者點選其中一個建議的動作時，Bot 會收到來自使用者的訊息，其中包含對應動作的 `value`。

請注意，`imBack` 方法會將 `value` 張貼至您所使用通道的聊天視窗。 如果這不是想要的效果，您可以使用 `postBack` 方法；這個方法仍然會將選取項目張貼回您的 Bot，但是不會在聊天視窗中顯示選取項目。 部分通道不支援 `postBack`，但是，在這些執行個體中，方法的行為就像是 `imBack`。

## <a name="additional-resources"></a>其他資源

* [使用頻道偵測器來預覽功能][inspector]
* [IMessage][IMessage]
* [ICardAction][ICardAction]
* [session.send][SessionSend]

[IMessage]: http://docs.botframework.com/en-us/node/builder/chat-reference/interfaces/_botbuilder_d_.imessage

[SessionSend]: https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.session.html#send

[ICardAction]: https://docs.botframework.com/en-us/node/builder/chat-reference/interfaces/_botbuilder_d_.icardaction.html

[inspector]: ../bot-service-channel-inspector.md

[channelInspector]: ../bot-service-channel-inspector.md
