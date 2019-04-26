---
title: 使用按鈕進行輸入 | Microsoft Docs
description: 了解如何使用適用於 JavaScript 的 Bot Framework SDK 在訊息內傳送建議的動作。
keywords: 建訢動作, 按鈕, 額外輸入
author: Kaiqb
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 4/18/2019
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: bf0c5c0bba335c41a268d43014e925f6a9289d75
ms.sourcegitcommit: aea57820b8a137047d59491b45320cf268043861
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 04/22/2019
ms.locfileid: "59904971"
---
# <a name="use-button-for-input"></a>使用按鈕進行輸入

[!INCLUDE[applies-to](../includes/applies-to.md)]

您可以讓 Bot 顯示可供使用者點選，以提供輸入的按鈕。 按鈕讓使用者只要點選按鈕，即可回答問題或進行選取，而無需使用鍵盤輸入回應，藉以提升使用者體驗。 不同於複合式資訊卡 (Rich Card) 中出現的按鈕 (即使在點選之後，使用者仍可看見並可存取)，出現在建議動作窗格內的按鈕會在使用者進行選取後消失。 這可以避免使用者在對話中點選過時的按鈕，並簡化 Bot 的開發 (因為您不需要再說明該情境)。 

## <a name="suggest-action-using-button"></a>使用按鈕建議動作

「建議動作」可讓 Bot 顯示按鈕。 您可以建立一份將在交談單一回合顯示給使用者的建議動作清單 (也稱為「快速回覆」)： 

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

您可以從 [GitHub](https://aka.ms/SuggestedActionsCSharp) 存取這裡使用的原始程式碼

```csharp
using Microsoft.Bot.Builder;
using Microsoft.Bot.Schema;

var reply = turnContext.Activity.CreateReply("What is your favorite color?");

reply.SuggestedActions = new SuggestedActions()
{
    Actions = new List<CardAction>()
    {
        new CardAction() { Title = "Red", Type = ActionTypes.ImBack, Value = "Red" },
        new CardAction() { Title = "Yellow", Type = ActionTypes.ImBack, Value = "Yellow" },
        new CardAction() { Title = "Blue", Type = ActionTypes.ImBack, Value = "Blue" },
    },

};
await turnContext.SendActivityAsync(reply, cancellationToken: cancellationToken);
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)
您可以從 [GitHub](https://aka.ms/SuggestActionsJS) 存取這裡使用的原始程式碼。

```javascript
const { ActivityTypes, MessageFactory, TurnContext } = require('botbuilder');

async sendSuggestedActions(turnContext) {
    var reply = MessageFactory.suggestedActions(['Red', 'Yellow', 'Blue'], 'What is the best color?');
    await turnContext.sendActivity(reply);
}
```

---

## <a name="additional-resources"></a>其他資源

您可以從 GitHub [[C#](https://aka.ms/SuggestedActionsCSharp) | [JS](https://aka.ms/SuggestActionsJS)] 存取這裡顯示的完整原始程式碼。
