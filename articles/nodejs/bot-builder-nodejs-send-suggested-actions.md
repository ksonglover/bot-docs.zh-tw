---
title: 將建議的動作新增至訊息 | Microsoft Docs
description: 了解如何使用適用於 Node.js 的 Bot Framework SDK 在訊息內傳送建議的動作。
author: v-ducvo
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 02/19/2019
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: 73193b82c8f03a1a49df1927ced15684bd7af955
ms.sourcegitcommit: 05ddade244874b7d6e2fc91745131b99cc58b0d6
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 02/21/2019
ms.locfileid: "56591109"
---
# <a name="add-suggested-actions-to-messages"></a>將建議的動作新增至訊息

[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]

> [!div class="op_single_selector"]
> - [.NET](../dotnet/bot-builder-dotnet-add-suggested-actions.md)
> - [Node.js](../nodejs/bot-builder-nodejs-send-suggested-actions.md)
> - [REST](../rest-api/bot-framework-rest-connector-add-suggested-actions.md)

[!INCLUDE [Introduction to suggested actions](../includes/snippet-suggested-actions-intro.md)]

## <a name="suggested-actions-example"></a>建議的動作範例

若要將建議的動作新增至訊息中，請將訊息的 `suggestedActions` 屬性設為[卡片動作][ICardAction]的清單，這些動作可以代表要向使用者呈現的按鈕。

此程式碼範例示範如何傳送一則訊息，向使用者呈現三種建議的動作：

[!code-javascript[Send suggested actions](../includes/code/node-send-suggested-actions.js#sendSuggestedActions)]

當使用者點選其中一個建議的動作時，Bot 會收到來自使用者的訊息，其中包含對應動作的 `value`。

請注意，`imBack` 方法會將 `value` 張貼至您所使用通道的聊天視窗。 如果這不是想要的效果，您可以使用 `postBack` 方法；這個方法仍然會將選取項目張貼回您的 Bot，但是不會在聊天視窗中顯示選取項目。 部分通道不支援 `postBack`，但是，在這些執行個體中，方法的行為就像是 `imBack`。

## <a name="additional-resources"></a>其他資源

- [範例][samples]
- [IMessage][IMessage]
- [ICardAction][ICardAction]
- [session.send][SessionSend]

[IMessage]: http://docs.botframework.com/en-us/node/builder/chat-reference/interfaces/_botbuilder_d_.imessage

[SessionSend]: https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.session.html#send

[ICardAction]: https://docs.botframework.com/en-us/node/builder/chat-reference/interfaces/_botbuilder_d_.icardaction.html

<!-- The inspector is no longer supported: we're redirecting to the samples for now. -->
[samples]: https://github.com/Microsoft/BotBuilder-Samples/tree/v3-sdk-samples
