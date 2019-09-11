---
title: 使用瀑布定義交談步驟 |Microsoft Docs
description: 了解如何以適用於 Node.js 的 Bot Framework SDK 來使用瀑布以定義交談的步驟。
author: v-ducvo
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: 2a5ade5e6407537e72b520a22d74bc2c3943fce4
ms.sourcegitcommit: a6d02ec4738e7fc90b7108934740e9077667f3c5
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/04/2019
ms.locfileid: "70299838"
---
# <a name="define-conversation-steps-with-waterfalls"></a>使用瀑布定義交談步驟

[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]

交談是使用者與 Bot 之間交換的一系列訊息。 當 Bot 的目標是引導使用者完成一系列的步驟時，您可以使用瀑布來定義交談的步驟。

瀑布是特定的[對話](bot-builder-nodejs-dialog-overview.md)實作，最常用來向使用者收集資訊或引導使用者完成一系列的工作。 這些工作會實作為函式陣列，其中第一個函式的結果將作為輸入傳遞至下一個函式，依此類推。 每個函式通常代表整個程序的其中一個步驟。 在每個步驟中，Bot 會提示使用者提供輸入、等候回應，然後將結果傳遞給下一個步驟。

本文將協助您了解瀑布的運作方式，以及您如何使用它來[管理交談流程](bot-builder-nodejs-dialog-manage-conversation.md)。

## <a name="conversation-steps"></a>交談步驟

交談通常涉及使用者與 Bot 之間的數個提示/回應交換。 每個提示/回應交換都是交談的逐步前進。 您可以使用只有兩個步驟的瀑布建立交談。

例如，請考慮下列 `greetings` 對話。 瀑布的第一個步驟會提示輸入使用者名稱，而第二個步驟則使用回應來依名稱歡迎使用者。

```javascript
bot.dialog('greetings', [
    // Step 1
    function (session) {
        builder.Prompts.text(session, 'Hi! What is your name?');
    },
    // Step 2
    function (session, results) {
        session.endDialog(`Hello ${results.response}!`);
    }
]);
```

實現此目的的方式是使用提示。 適用於 Node.js 的 Bot Framework SDK 提供許多不同類型的內建[提示](bot-builder-nodejs-dialog-prompt.md)，您可以使用這些提示來要求使用者提供各種類型的資訊。

下列範例程式碼顯示一則對話，此對話會使用提示在含有 4 個步驟的瀑布中向使用者收集各項資訊。

```javascript
var inMemoryStorage = new builder.MemoryBotStorage();

// This is a dinner reservation bot that uses a waterfall technique to prompt users for input.
var bot = new builder.UniversalBot(connector, [
    function (session) {
        session.send("Welcome to the dinner reservation.");
        builder.Prompts.time(session, "Please provide a reservation date and time (e.g.: June 6th at 5pm)");
    },
    function (session, results) {
        session.dialogData.reservationDate = builder.EntityRecognizer.resolveTime([results.response]);
        builder.Prompts.text(session, "How many people are in your party?");
    },
    function (session, results) {
        session.dialogData.partySize = results.response;
        builder.Prompts.text(session, "Whose name will this reservation be under?");
    },
    function (session, results) {
        session.dialogData.reservationName = results.response;

        // Process request and display reservation details
        session.send(`Reservation confirmed. Reservation details: <br/>Date/Time: ${session.dialogData.reservationDate} <br/>Party size: ${session.dialogData.partySize} <br/>Reservation name: ${session.dialogData.reservationName}`);
        session.endDialog();
    }
]).set('storage', inMemoryStorage); // Register in-memory storage 
```

在此範例中，預設對話有四個函式，每個各代表瀑布中的一個步驟。 每個步驟會提示使用者提供輸入，並將結果傳送到要處理的下一個步驟。 此程序會繼續最後一個步驟執行，藉此確認預約並結束對話。

下列螢幕擷取畫面顯示在 [Bot Framework 模擬器](~/bot-service-debug-emulator.md)中執行此 Bot 的結果。

![使用瀑布管理交談流程](~/media/bot-builder-nodejs-dialog-manage-conversation/waterfall-results.png)

## <a name="create-a-conversation-with-multiple-waterfalls"></a>使用多個瀑布建立交談

您可以在交談內使用多個瀑布來定義 Bot 所需的任何交談結構。 例如，您可能會使用一個瀑布來管理交談流程，並使用其他瀑布向使用者收集資訊。 每個瀑布都封裝在一則對話中，可以透過呼叫 `session.beginDialog` 函式來叫用。

下列程式碼範例示範如何在交談中使用多個瀑布。 `greetings` 交談內的瀑布會管理交談的流程，而 `askName` 交談內的瀑布則會向使用者收集資訊。

```javascript
bot.dialog('greetings', [
    function (session) {
        session.beginDialog('askName');
    },
    function (session, results) {
        session.endDialog('Hello %s!', results.response);
    }
]);
bot.dialog('askName', [
    function (session) {
        builder.Prompts.text(session, 'Hi! What is your name?');
    },
    function (session, results) {
        session.endDialogWithResult(results);
    }
]);
```

## <a name="advance-the-waterfall"></a>推進瀑布

瀑布會依照陣列中函式的定義順序逐步進行。 瀑布內的第一個函式可以接收傳遞給對話的引數。 瀑布內的任何函式都可以使用 `next` 函式，以繼續進行下一個步驟，而不提示使用者提供輸入。

下列程式碼範例示範如何在對話中使用 `next` 函式，以逐步引導使用者完成提供其使用者設定檔資訊的程序。 在瀑布的每個步驟中，Bot 會提示使用者輸入一項資訊 (若必要)，而且瀑布的後續步驟會處理使用者的回應 (若有)。 `ensureProfile` 對話中瀑布的最後一個步驟會結束該對話，並將完成的設定檔資訊傳回給呼叫端對話，該對話接著會將個人化問候語傳送給使用者。

```javascript
var inMemoryStorage = new builder.MemoryBotStorage();

// This bot ensures user's profile is up to date.
var bot = new builder.UniversalBot(connector, [
    function (session) {
        session.beginDialog('ensureProfile', session.userData.profile);
    },
    function (session, results) {
        session.userData.profile = results.response; // Save user profile.
        session.send(`Hello ${session.userData.profile.name}! I love ${session.userData.profile.company}!`);
    }
]).set('storage', inMemoryStorage); // Register in-memory storage 

bot.dialog('ensureProfile', [
    function (session, args, next) {
        session.dialogData.profile = args || {}; // Set the profile or create the object.
        if (!session.dialogData.profile.name) {
            builder.Prompts.text(session, "What's your name?");
        } else {
            next(); // Skip if we already have this info.
        }
    },
    function (session, results, next) {
        if (results.response) {
            // Save user's name if we asked for it.
            session.dialogData.profile.name = results.response;
        }
        if (!session.dialogData.profile.company) {
            builder.Prompts.text(session, "What company do you work for?");
        } else {
            next(); // Skip if we already have this info.
        }
    },
    function (session, results) {
        if (results.response) {
            // Save company name if we asked for it.
            session.dialogData.profile.company = results.response;
        }
        session.endDialogWithResult({ response: session.dialogData.profile });
    }
]);
```

> [!NOTE]
> 此範例使用兩個不同的資料包來儲存資料：`dialogData` 和 `userData`。 如果 Bot 分散在多個計算節點上，則瀑布的每個步驟都可由不同的節點加以處理。 如需儲存 Bot 資料的詳細資訊，請參閱[管理狀態資料](bot-builder-nodejs-state.md)。

## <a name="end-a-waterfall"></a>結束瀑布

使用瀑布建立的對話必須明確地結束，否則 Bot 會無限次地重複瀑布。 您可以使用下列其中一個方法來結束瀑布：

* `session.endDialog`：如果沒有資料可傳回給呼叫端對話，請使用此方法來結束瀑布。

* `session.endDialogWithResult`：如果有資料可傳回給呼叫端對話，請使用此方法來結束瀑布。 傳回的 `response` 引數可以是 JSON 物件，或是任何 JavaScript 基本資料類型。 例如︰
  ```javascript
  session.endDialogWithResult({
    response: { name: session.dialogData.name, company: session.dialogData.company }
  });
  ```

* `session.endConversation`：如果瀑布的結尾代表交談的結尾，請使用此方法來結束交談。

作為使用這三種方法其中之一來結束瀑布的替代方式，您可以將 `endConversationAction` 觸發程序附加到對話中。 例如︰

```javascript
bot.dialog('dinnerOrder', [
    //...waterfall steps...,
    // Last step
    function(session, results){
        if(results.response){
            session.dialogData.room = results.response;
            var msg = `Thank you. Your order will be delivered to room #${session.dialogData.room}`;
            session.endConversation(msg);
        }
    }
])
.endConversationAction(
    "endOrderDinner", "Ok. Goodbye.",
    {
        matches: /^cancel$|^goodbye$/i,
        confirmPrompt: "This will cancel your order. Are you sure?"
    }
);
```

## <a name="next-steps"></a>後續步驟

使用瀑布，您可以透過「提示」  從使用者收集資訊。 讓我們深入了解您可以提示使用者提供輸入的方式。

> [!div class="nextstepaction"]
> [提示使用者提供輸入](bot-builder-nodejs-dialog-prompt.md)
