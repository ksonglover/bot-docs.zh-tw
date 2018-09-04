---
title: 使用事件處理常式 | Microsoft Docs
description: 了解如何使用事件處理常式。
author: ivorb
ms.author: v-ivorb
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 06/14/2018
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 36bd2638c599de1662a37dd85790b6126184d51b
ms.sourcegitcommit: 2dc75701b169d822c9499e393439161bc87639d2
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/24/2018
ms.locfileid: "42905687"
---
# <a name="using-event-handlers"></a>使用事件處理常式

[!INCLUDE [pre-release-label](../includes/pre-release-label.md)]

事件處理常式是可新增至 [turn](bot-builder-basics.md#defining-a-turn) 內未來活動事件的函式。 這些活動是 `SendActivity`、`UpdateActivity` 和 `DeleteActivity`，各有其自己的處理常式。 當您需要對目前內容物件該類型的每一個未來活動做些什麼的時候，這些處理常式會很有用。

您可以將多個事件處理常式新增至每個活動事件，它們會在新增**之後**針對每個事件處理；因此，它們新增至程式碼何處很重要。

> [!NOTE]
> 本文中的「事件處理常式」不遵守「事件」和「處理常式」的傳統語言特定定義。 它們反而更接近「事件」和「處理常式」的概念性程式設計理念。

## <a name="using-the-next-delegate"></a>使用「下一個」委派

每個處理常式藉由呼叫「下一個」委派，遵循處理常式的新增順序將執行傳送至下列處理常式。 最後的「下一個」委派是對實際傳送、更新或刪除活動之配接器的呼叫。

除非另有目的，否則在您的處理常式中呼叫 *next()* 十分重要，若不然您會[最少運算](bot-builder-create-middleware.md#short-circuit-routing)活動。 最少運算活動和中介軟體之間的差異是，最少運算完全取消活動，而中介軟體則變更允許執行的程式碼，但活動繼續執行。

在傳送動活時，如果成功，「下一個」委派會傳回指派給傳送訊息的識別碼。

## <a name="expected-return-value"></a>預期的傳回值

事件處理常式預期的傳回值是下一個委派的結果。 如有需要，您可以先行檢視及儲存該結果再傳回，不然就會直接傳回如下的內容。

`SendActivity` 處理常式的傳回值是透過鏈結備份通過為對應下一個委派傳回值的傳回值。

# <a name="ctabcseventhandler"></a>[C#](#tab/cseventhandler)

所提供的三種事件處理常式類型為 `OnSendActivities()`、`OnUpdateActivity()` 和 `OnDeleteActivity()`。 在這裡，我們會在Bot 程式碼內新增 `OnSendActivities()` 處理常式，但處理常式也可新增至中介軟體程式碼。

處理常式通常會新增為 Lambda 運算式，您在此範例中會看到。 在這裡，我們會在寫入**help** 時接聽使用者的輸入活動。

```cs
public Task OnTurn(ITurnContext context)
{
    if (context.Activity.Type == ActivityTypes.Message)
    {
        // Get the conversation state from the turn context
        
        context.OnSendActivities(async (handlerContext, activities, handlerNext) =>
        {
            if (handlerContext.Activity.Text == "help")
            {
                Console.WriteLine("help!");
                // Do whatever logging you want to do for this help message
            }

            return await handlerNext();
        });
        // ...
    }
}
```

# <a name="javascripttabjseventhandler"></a>[JavaScript](#tab/jseventhandler)

所提供的三種事件處理常式類型為 `onSendActivities()`、`onUpdateActivity()` 和 `onDeleteActivity()`。 在這裡，我們會在 Bot 程式碼內新增 `onSendActivities()` 處理常式，但處理常式也可新增至中介軟體程式碼。

此範例會在寫入**help** 時接聽使用者的輸入活動。

```js
adapter.processActivity(req, res, async (context) => {

    if (context.activity.type === 'message') {

        context.onSendActivities(async (handlerContext, activities, handlerNext) => { 
            
            if(handlerContext.activity.text === 'help'){
                console.log('help!')
                // Do whatever logging you want to do for this help message
            }
            // Add handler logic here
        
            await handlerNext(); 
        });
        await context.sendActivity(`you said ${context.activity.text}`);
    }
});
```

---

請務必區分「傳送活動」和「更新或刪除活動」事件，前者會建立全新的活動事件，後者則作用於過去的活動。 此外， 並非所有的通道都支援「更新」或「刪除」活動。 建議新增這些活動呼叫及其處理常式的適當例外狀況處理，以將此可能性列入考量。

