---
title: 對話概觀 | Microsoft Docs
description: 了解如何在適用於 Node.js 的 Bot Framework SDK 中使用對話，以建立交談模型並管理交談流程。
author: DucVo
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: 9ec846c5d464821538902dd726e8b9ee68d4bae1
ms.sourcegitcommit: a6d02ec4738e7fc90b7108934740e9077667f3c5
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/04/2019
ms.locfileid: "70299874"
---
# <a name="dialogs-in-the-bot-framework-sdk-for-nodejs"></a>適用於 Node.js 的 Bot Framework SDK 中的對話

[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]

> [!div class="op_single_selector"]
> - [.NET](../dotnet/bot-builder-dotnet-dialogs.md)
> - [Node.js](../nodejs/bot-builder-nodejs-dialog-overview.md)

適用於 Node.js 的 Bot Framework SDK 中的對話，可讓您建立交談模型及管理交談流程。 Bot 會透過對話與使用者通訊。 對話 (conversation) 會被組織為對話 (dialog)。 對話可以包含瀑布圖步驟和提示。 當使用者與 Bot 互動時，Bot 將會開始和停止對話，以及在對話之間切換，以回應使用者訊息。 了解對話的運作方式是成功設計和建立絕佳 Bot 的關鍵。 

本文將會介紹對話的概念。 在您讀完本文之後，請接著依照[後續步驟](#next-steps)一節中的連結深入探索這些概念。

## <a name="conversations-through-dialogs"></a>透過對話 (dialog) 進行對話 (conversation)

適用於 Node.js 的 Bot Framework SDK 將交談定義為 Bot 與使用者之間透過一或多個對話所進行的通訊。 對話就最基本層級而言是一個可重複使用的模組，可以執行作業或收集使用者的資訊。 您可以將 Bot 的複雜邏輯封裝於可重複使用的對話程式碼中。

對話可以數種方式進行結構化及變更：

- 它可以源自您的[預設對話](#default-dialog)。
- 它可以從一個對話重新導向到另一個對話。
- 它可以繼續。
- 它可以依循[瀑布圖](bot-builder-nodejs-dialog-waterfall.md)模式，其能引導使用者進行一系列的步驟，或是以一系列的問題來[提示](bot-builder-nodejs-dialog-prompt.md)使用者。
- 它可以使用[動作](bot-builder-nodejs-dialog-actions.md)來接聽會觸發不同對話的單字或片語。 

您可以將對話 (conversation) 想像成對話 (dialog) 的父代。 在此情況下，對話 (conversation) 會包含*對話 (dialog) 堆疊*，並維護一組自己的狀態資料，亦即 `conversationData` 和 `privateConversationData`。 另一方面，對話 (dialog) 會維護 `dialogData`。 如需狀態資料的詳細資訊，請參閱[管理狀態資料](bot-builder-nodejs-state.md)。

## <a name="dialog-stack"></a>對話堆疊

Bot 會透過一系列在對話堆疊上維護的對話來與使用者互動。 在對話 (conversation) 期間，系統會將對話 (dialog) 推送到堆疊上，或是從其中移除。 堆疊的運作方式類似一般的 LIFO 堆疊，這表示最後新增的對話將會第一個完成。 完成某個對話之後，系統會將控制權還給堆疊上的前一個對話。

在第一次啟動 Bot 對話 (conversation) 或對話 (conversation) 結束時，對話 (dialog) 堆疊會是空的。 此時，當使用者傳送訊息給 Bot 時，Bot 將以*預設對話*來回應。

## <a name="default-dialog"></a>預設對話

在 Bot Framework 3.5 版之前，*根*對話是藉由新增名為 `/` 的對話來定義，這導致命名慣例會與 URL 的類似。 此命名慣例並不適合用來為對話命名。 

> [!NOTE]
> 從 Bot Framework 3.5 版開始，*預設對話*會註冊為 [`UniversalBot`](https://docs.botframework.com/node/builder/chat-reference/classes/_botbuilder_d_.universalbot.html#constructor) \(英文\) 之建構函式中的第二個參數。  

下列程式碼片段會示範如何在建立 `UniversalBot` 物件時定義預設對話。

```javascript
var bot = new builder.UniversalBot(connector, [
    //...Default dialog waterfall steps...
    ]);
```

在對話堆疊是空的，且未透過 LUIS 或其他[辨識器](bot-builder-nodejs-recognize-intent-messages.md)[觸發](bot-builder-nodejs-dialog-actions.md)任何其他對話時，即會執行預設對話。 由於預設對話是 Bot 對使用者的第一個回應，因此預設對話應該要為使用者提供一些內容相關的資訊，例如可用命令的清單或 Bot 可執行項目的概觀。

## <a name="dialog-handlers"></a>對話處理常式

對話 (dialog) 處理常式會管理對話 (conversation) 的流程。 為了進行對話 (conversation)，對話 (dialog) 處理常式會藉由開始和結束對話 (dialog) 來引導流程。 

## <a name="starting-and-ending-dialogs"></a>開始和結束對話

若要開始新的對話 (並將它推送到堆疊上)，請使用 [`session.beginDialog()`](http://docs.botframework.com/node/builder/chat-reference/classes/_botbuilder_d_.session#begindialog) \(英文\)。 若要結束對話 (並將它從堆疊中移除，然後將控制權還給呼叫對話)，請使用 [`session.endDialog()`](http://docs.botframework.com/node/builder/chat-reference/classes/_botbuilder_d_.session#enddialog) \(英文\) 或 [`session.endDialogWithResult()`](http://docs.botframework.com/node/builder/chat-reference/classes/_botbuilder_d_.session#enddialogwithresult) \(英文\)。 

## <a name="using-waterfalls-and-prompts"></a>使用瀑布圖和提示

[瀑布圖](bot-builder-nodejs-dialog-waterfall.md)是建立對話流程模型並加以管理的簡單方式。 瀑布圖包含一系列的步驟。 在每個步驟中，您可以代表使用者完成某個動作，或[提示](bot-builder-nodejs-dialog-prompt.md)使用者輸入資訊。

瀑布圖是使用由函式集合所組成的對話來實作。 每個函式都會定義瀑布圖中的一個步驟。 下列程式碼範例會示範一個簡易對話，此對話會使用兩步驟的瀑布圖來提示使用者輸入其名字，並使用該名字問候他們。

```javascript
// Ask the user for their name and greet them by name.
bot.dialog('greetings', [
    function (session) {
        builder.Prompts.text(session, 'Hi! What is your name?');
    },
    function (session, results) {
        session.endDialog(`Hello ${results.response}!`);
    }
]);
```

當 Bot 抵達瀑布圖的結尾，但沒有結束對話時，來自使用者的下一個訊息會從瀑布圖的第一個步驟重新開始該對話。 這可能導致使用者覺得自己陷於迴圈中而感到挫折。 若要避免此情況，在對話 (conversation) 或對話 (dialog) 抵達結尾時，最佳做法是明確呼叫 `endDialog`、`endDialogWithResult` 或 `endConversation`。

## <a name="next-steps"></a>後續步驟

若要更深入探索對話，請務必了解瀑布圖模式的運作方式，以及如何用它來引導使用者完成程序。

> [!div class="nextstepaction"]
> [使用瀑布圖定義對話步驟](bot-builder-nodejs-dialog-waterfall.md)
