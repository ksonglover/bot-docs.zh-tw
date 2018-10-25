---
title: 將建議的動作新增至訊息 | Microsoft Docs
description: 了解如何使用適用於 JavaScript 的 Bot Builder SDK 在訊息內傳送建議的動作。
keywords: 建訢動作, 按鈕, 額外輸入
author: Kaiqb
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 03/13/2018
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 1a42a127606ee72e16bd54eb507ecb4884996996
ms.sourcegitcommit: bd4f9669c0d26ac2a4be1ab8e508f163a1f465f3
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/28/2018
ms.locfileid: "47430317"
---
# <a name="add-suggested-actions-to-messages"></a>將建議的動作新增至訊息

[!INCLUDE [pre-release-label](../includes/pre-release-label.md)]

[!include[Introduction to suggested actions](../includes/snippet-suggested-actions-intro.md)] 

## <a name="send-suggested-actions"></a>傳送建議的動作

您可以建立一份將在交談單一回合顯示給使用者的建議動作清單 (也稱為「快速回覆」)： 

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

您可以從 [GitHub](https://aka.ms/SuggestedActionsCSharp) 存取這裡使用的原始程式碼

```csharp
using Microsoft.Bot.Builder;
using Microsoft.Bot.Builder.Core.Extensions;
using Microsoft.Bot.Schema;

```csharp
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
await turnContext.SendActivityAsync(reply, cancellationToken);
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
