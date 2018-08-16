---
title: 使用對話 (dialog) 管理對話 (conversation) 流程 | Microsoft Docs
description: 了解如何使用適用於 Node.js 的 Bot 建立器 SDK 中的對話 (dialog)，來管理 Bot 與使用者之間的對話 (conversation)。
author: v-ducvo
ms.author: v-ducvo
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: d6c8ad06b9fb198e684deae26e9cbad05a86a611
ms.sourcegitcommit: f576981342fb3361216675815714e24281e20ddf
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/18/2018
ms.locfileid: "39300279"
---
# <a name="manage-conversation-flow-with-dialogs"></a>使用對話 (dialog) 管理對話 (conversation) 流程
> [!div class="op_single_selector"]
> - [.NET](../dotnet/bot-builder-dotnet-manage-conversation-flow.md)
> - [Node.js](../nodejs/bot-builder-nodejs-dialog-manage-conversation-flow.md)

管理對話流程是建置 Bot 的必要工作。 Bot 必須能夠從容地執行核心工作，並在被中斷的情況下做出適當的因應措施。 在使用適用於 Node.js 的 Bot 建立器 SDK 時，您可以使用對話 (dialog) 來管理對話 (conversation) 流程。

對話類似於程式中的函式。 它通常是設計來執行特定作業，並可視需要隨時叫用。 您可以將多個對話 (dialog) 鏈結在一起，以處理幾乎任何您想要讓 Bot 處理的對話 (conversation) 流程。 適用於 Node.js 的 Bot 建立器 SDK 包含[提示](bot-builder-nodejs-dialog-prompt.md)和[瀑布圖](bot-builder-nodejs-dialog-waterfall.md)等內建功能，可協助您管理對話流程。

本文會提供一系列的範例來說明管理簡單和複雜對話 (conversation) 流程的方式，其中您的 Bot 可使用對話 (dialog) 來處理被中斷的情況，並從容地繼續該流程。 範例均以下列案例為基礎： 

1. 您的 Bot 將協助進行晚餐訂位事宜。
2. 您的 Bot 可以在訂位期間隨時處理 "Help" (說明) 要求。
3. 您的 Bot 可以針對目前的訂位步驟處理內容相關的 "Help" (說明)。
4. 您的 Bot 可以處理多個對話主題。

## <a name="manage-conversation-flow-with-a-waterfall"></a>使用瀑布圖管理對話流程

[瀑布圖](bot-builder-nodejs-dialog-waterfall.md)是一個對話，可讓 Bot 輕鬆地引導使用者執行一系列的工作。 在此範例中，訂位 Bot 會詢問使用者一連串的問題，讓 Bot 能夠處理訂位要求。 Bot 將提示使用者提供下列資訊：

1. 訂位日期和時間
2. 用餐人數
3. 訂位者的姓名

下列程式碼範例會示範如何使用瀑布圖來引導使用者完成一系列的提示。

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
        builder.Prompts.number(session, "How many people are in your party?");
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

這個 Bot 的核心功能會發生在預設對話中。 預設對話是在建立 Bot 時定義： 

```javascript
var bot = new builder.UniversalBot(connector, [..waterfall steps..]); 
```

此外，在這個建立過程中，您可以設定要使用的[資料儲存體](bot-builder-nodejs-state.md)。 例如，若要使用記憶體內部儲存體，您可以使用下列方式設定它：

```javascript
var inMemoryStorage = new builder.MemoryBotStorage();
var bot = new builder.UniversalBot(connector, [..waterfall steps..]).set('storage', inMemoryStorage); // Register in-memory storage 
```

預設對話會建立為定義瀑布圖之步驟的函式陣列。 此範例中有四個函式，因此瀑布圖會有四個步驟。 每個步驟均會執行單一工作，且其結果會在下一個步驟中加以處理。 此程序會繼續進行，直到最後一個步驟為止；該步驟將會確認訂位並結束對話。

下列螢幕擷取畫面會顯示在 [Bot Framework 模擬器](../bot-service-debug-emulator.md)中執行這個 Bot 的結果：

![使用瀑布圖管理對話流程](../media/bot-builder-nodejs-dialog-manage-conversation/waterfall-results.png)

### <a name="prompt-user-for-input"></a>提示使用者輸入

此範例的每個步驟都會使用提示來要求使用者輸入。 提示是一種特殊類型的對話，會要求使用者輸入，等候回應，並將該回應傳回至瀑布圖中的下一個步驟。 如需可在 Bot 中使用之各種不同類型提示的相關資訊，請參閱[提示使用者輸入](bot-builder-nodejs-dialog-prompt.md)。

在此範例中，Bot 會使用 `Prompts.text()` 來要求使用者以文字格式提供自由格式的回應。 使用者可以使用任何文字來回應，而 Bot 必須決定要如何處理回應。 `Prompts.time()` 會使用 [Chrono](https://github.com/wanasit/chrono) \(英文\) 程式庫，從字串中剖析日期和時間資訊。 這可讓您的 Bot 了解更多適用於指定日期與時間的自然語言。 例如："June 6th, 2017 at 9pm" (2017 年 6 月 6 日下午 9 點)、"Today at 7:30pm" (今天下午 7:30)、"next monday at 6pm" (下週一下午 6 點) 等等。

> [!TIP] 
> 使用者輸入的時間會根據裝載該 Bot 之伺服器的時區轉換成 UTC 時間。 由於伺服器可能位於與使用者不同的時區，因此，請務必將時區納入考量。 若要將日期和時間轉換成使用者的當地時間，請考慮詢問使用者位於哪個時區。

## <a name="manage-a-conversation-flow-with-multiple-dialogs"></a>使用多個對話 (dialog) 管理對話 (conversation) 流程

管理對話 (conversation) 流程的另一個技術，是結合使用瀑布圖和多個對話 (dialog)。 瀑布圖可讓您在對話 (dialog) 中鏈結多個函式，而對話 (dialog) 可讓您將對話 (conversation) 細分為可隨時重複使用的較小功能片段。

例如，讓我們以晚餐訂位 Bot 為例。 下列程式碼範例會已重新撰寫上一個範例，以使用瀑布圖和多個對話。

```javascript
var inMemoryStorage = new builder.MemoryBotStorage();

// This is a dinner reservation bot that uses multiple dialogs to prompt users for input.
var bot = new builder.UniversalBot(connector, [
    function (session) {
        session.send("Welcome to the dinner reservation.");
        session.beginDialog('askForDateTime');
    },
    function (session, results) {
        session.dialogData.reservationDate = builder.EntityRecognizer.resolveTime([results.response]);
        session.beginDialog('askForPartySize');
    },
    function (session, results) {
        session.dialogData.partySize = results.response;
        session.beginDialog('askForReserverName');
    },
    function (session, results) {
        session.dialogData.reservationName = results.response;

        // Process request and display reservation details
        session.send(`Reservation confirmed. Reservation details: <br/>Date/Time: ${session.dialogData.reservationDate} <br/>Party size: ${session.dialogData.partySize} <br/>Reservation name: ${session.dialogData.reservationName}`);
        session.endDialog();
    }
]).set('storage', inMemoryStorage); // Register in-memory storage 

// Dialog to ask for a date and time
bot.dialog('askForDateTime', [
    function (session) {
        builder.Prompts.time(session, "Please provide a reservation date and time (e.g.: June 6th at 5pm)");
    },
    function (session, results) {
        session.endDialogWithResult(results);
    }
]);

// Dialog to ask for number of people in the party
bot.dialog('askForPartySize', [
    function (session) {
        builder.Prompts.text(session, "How many people are in your party?");
    },
    function (session, results) {
        session.endDialogWithResult(results);
    }
])

// Dialog to ask for the reservation name.
bot.dialog('askForReserverName', [
    function (session) {
        builder.Prompts.text(session, "Who's name will this reservation be under?");
    },
    function (session, results) {
        session.endDialogWithResult(results);
    }
]);
```

執行此 Bot 的結果會與上一個僅使用瀑布圖的 Bot 完全相同。 不過，就程式設計方面而言，它們有兩個主要差異：

1. 預設對話 (dialog) 是專門用來管理對話 (conversation) 的流程。
2. 針對對話 (conversation) 之每個步驟的工作，是由個別的對話 (dialog) 所管理。 在此範例中，由於 Bot 需要三種資訊，因此它會提示使用者三次。 每個提示現在都會包含於自己的對話中。

透過使用此技術，您可以將對話流程從工作邏輯中分隔開來。 這可讓不同的對話 (conversation) 流程在必要的情況下重複使用該對話 (dialog)。 

## <a name="respond-to-user-input"></a>回應使用者輸入

在引導使用者完成一系列工作的過程中，如果使用者有問題，或者想要在回答之前要求其他資訊，您該如何處理那些要求？ 例如，不論使用者處於對話的哪個位置，當使用者輸入 "Help" (說明)、"Support" (支援) 或 "Cancel" (取消) 時，Bot 該如何回應？ 如果使用者想要取得關於某個步驟的其他資訊，該怎麼辦？ 如果使用者改變心意，並想要放棄目前的工作以開始全然不同的工作，會發生什麼事？

適用於 Node.js 的 Bot 建立器 SDK 可讓 Bot 接聽全域內容之內，或是位於目前對話範圍中的區域內容之內的特定輸入。 這些輸入稱為[動作](bot-builder-nodejs-dialog-actions.md)，其可讓 Bot 根據 `matches` 子句來接聽使用者輸入。 Bot 必須決定該如何回應特定的使用者輸入。

### <a name="handle-global-action"></a>處理全域動作

如果您想要 Bot 能夠在對話中的任何步驟處理動作，請使用 `triggerAction`。 觸發程序可讓 Bot 在輸入符合指定字詞時叫用特定的對話。 例如，如果您想要支援全域的 "Help" (說明) 選項，您可以建立說明對話，並附加能接聽符合 "Help" (說明) 之輸入的 `triggerAction`。

下列程式碼範例會示範如何將 `triggerAction` 附加到對話，以指定在使用者輸入 "Help" (說明) 時應該叫用對話。

```javascript
// The dialog stack is cleared and this dialog is invoked when the user enters 'help'.
bot.dialog('help', function (session, args, next) {
    session.endDialog("This is a bot that can help you make a dinner reservation. <br/>Please say 'next' to continue");
})
.triggerAction({
    matches: /^help$/i,
});
```

根據預設，當 `triggerAction` 執行時，系統會清除對話堆疊，而觸發的對話將會變成新的預設對話。 在此範例中，當 `triggerAction` 執行時，系統會清除對話堆疊，接著會將 `help` 對話新增至堆疊以作為新的預設對話。 如果這不是您想要的行為，您可以將 `onSelectAction` 選項新增至 `triggerAction`。 `onSelectAction` 選項可讓 Bot 在不清除對話堆疊的情況下開始新的對話 (dialog)，這可將對話 (conversation) 暫時重新導向，並於稍後再從相同的位置繼續。

下列程式碼範例會示範如何使用 `onSelectAction` 選項搭配 `triggerAction`，來將 `help` 對話新增至現有的對話堆疊 (且不會清除對話堆疊)。

```javascript
bot.dialog('help', function (session, args, next) {
    session.endDialog("This is a bot that can help you make a dinner reservation. <br/>Please say 'next' to continue");
})
.triggerAction({
    matches: /^help$/i,
    onSelectAction: (session, args, next) => {
        // Add the help dialog to the dialog stack 
        // (override the default behavior of replacing the stack)
        session.beginDialog(args.action, args);
    }
});
```

在此範例中，`help` 對話 (dialog) 會在使用者輸入 "Help" (說明) 時取得對話 (conversation) 的控制權。 由於 `triggerAction` 包含 `onSelectAction` 選項，因此系統會將 `help` 對話推送至現有的對話堆疊，且不會清除該堆疊。 當 `help` 對話 (dialog) 結束時，系統就會從對話 (dialog) 堆疊中移除它，而對話 (conversation) 則會從 `help` 命令先前中斷它的步驟繼續。

### <a name="handle-contextual-action"></a>處理內容相關的動作

在上述範例中，如果使用者在對話 (conversation) 的任何步驟輸入 "Help" (說明)，即會叫用 `help` 對話 (dialog)。 在此情況下，該對話只能提供一般的說明指導方針，因為使用者的說明要求並沒有針對任何特定內容。 但是，如果使用者想要針對對話的特定步驟要求說明，該怎麼辦？ 在該情況下，必須在目前對話的內容內觸發 `help` 對話。

例如，讓我們再以晚餐訂位 Bot 為例。 在詢問用餐人數時，如果使用者想要知道用餐人數上限，該怎麼辦？ 若要處理這種情況，您可以將 `beginDialogAction` 附加至 `askForPartySize` 對話，以接聽使用者輸入 "Help" (說明)。

下列程式碼範例會示範如何使用 `beginDialogAction`，將內容相關的說明附加至對話。

```javascript
// Dialog to ask for number of people in the party
bot.dialog('askForPartySize', [
    function (session) {
        builder.Prompts.text(session, "How many people are in your party?");
    },
    function (session, results) {
       session.endDialogWithResult(results);
    }
])
.beginDialogAction('partySizeHelpAction', 'partySizeHelp', { matches: /^help$/i });

// Context Help dialog for party size
bot.dialog('partySizeHelp', function(session, args, next) {
    var msg = "Party size help: Our restaurant can support party sizes up to 150 members.";
    session.endDialog(msg);
})
```

在此範例中，每當使用者輸入 "Help" (說明) 時，Bot 會將 `partySizeHelp` 對話推送至堆疊。 該對話會將說明訊息傳送給使用者，然後結束對話，並將控制權還給 `askForPartySize` 對話，而該對話將會重新提示使用者輸入用餐人數。

請務必注意，只有在使用者位於 `askForPartySize` 對話時，才會執行此內容相關說明。 否則，系統將會執行來自 `triggerAction` 的一般說明訊息。 換句話說，區域 `matches` 子句的優先順序，一律會優先於全域 `matches` 子句。 例如，如果 `beginDialogAction` 與 **help** 相符，系統便不會執行 `triggerAction` 中與 **help** 相符的項目。 如需詳細資訊，請參閱[動作優先順序](bot-builder-nodejs-dialog-actions.md#action-precedence)。

### <a name="change-the-topic-of-conversation"></a>變更對話的主題

根據預設，執行 `triggerAction` 會清除對話 (dialog) 堆疊並重設對話 (conversation)，並從指定的對話 (dialog) 開始。 當您的 Bot 需要從一個對話主題切換至另一個時 (例如，如果使用者在進行晚餐訂位時，決定要改成使用旅館的送餐服務並直接點晚餐)，通常會更適合使用此行為。 

下列範例是建置於上一個範例之上，但會讓 Bot 允許使用者進行晚餐訂位，或是使用送餐服務。 在這個 Bot 中，預設對話是一則問候語對話，其會向使用者顯示兩個選項：`Dinner Reservation` 和 `Order Dinner`。

```javascript
var inMemoryStorage = new builder.MemoryBotStorage();

// This bot enables users to either make a dinner reservation or order dinner.
var bot = new builder.UniversalBot(connector, function(session){
    var msg = "Welcome to the reservation bot. Please say `Dinner Reservation` or `Order Dinner`";
    session.send(msg);
}).set('storage', inMemoryStorage); // Register in-memory storage 
```

上一個範例之預設對話中的晚餐訂位邏輯，現在已是獨立的 `dinnerReservation` 對話。 `dinnerReservation` 的流程會與稍早所討論的多個對話版本保持一致。 唯一的差別在於對話具有附加的 `triggerAction`。 請注意，在此版本中，`confirmPrompt` 會在叫用新對話 (dialog) 之前，要求使用者確認他們想要變更對話 (conversation) 的主題。 在這類案例中，包含 `confirmPrompt` 是一個很好的作法，因為在清除對話 (dialog) 堆疊之後，系統會將使用者導向新的對話 (conversation) 主題，從而放棄先前所進行的對話 (conversation) 主題。

```javascript
// This dialog helps the user make a dinner reservation.
bot.dialog('dinnerReservation', [
    function (session) {
        session.send("Welcome to the dinner reservation.");
        session.beginDialog('askForDateTime');
    },
    function (session, results) {
        session.dialogData.reservationDate = builder.EntityRecognizer.resolveTime([results.response]);
        session.beginDialog('askForPartySize');
    },
    function (session, results) {
        session.dialogData.partySize = results.response;
        session.beginDialog('askForReserverName');
    },
    function (session, results) {
        session.dialogData.reservationName = results.response;

        // Process request and display reservation details
        session.send(`Reservation confirmed. Reservation details: <br/>Date/Time: ${session.dialogData.reservationDate} <br/>Party size: ${session.dialogData.partySize} <br/>Reservation name: ${session.dialogData.reservationName}`);
        session.endDialog();
    }
])
.triggerAction({
    matches: /^dinner reservation$/i,
    confirmPrompt: "This will cancel your current request. Are you sure?"
});
```

第二個對話 (conversation) 主題會使用瀑布圖定義於 `orderDinner` 對話 (dialog) 中。 這個對話單純會顯示晚餐菜單，並在使用者點餐之後提示使用者提供房間號碼。 `triggerAction` 會附加至該對話 (dialog)，以指定系統應在使用者輸入 "order dinner" (點晚餐) 時叫用它，並在使用者表示想要變更對話 (conversation) 主題時，確保系統會提示使用者確認其選項。

```javascript
// This dialog help the user order dinner to be delivered to their hotel room.
var dinnerMenu = {
    "Potato Salad - $5.99": {
        Description: "Potato Salad",
        Price: 5.99
    },
    "Tuna Sandwich - $6.89": {
        Description: "Tuna Sandwich",
        Price: 6.89
    },
    "Clam Chowder - $4.50":{
        Description: "Clam Chowder",
        Price: 4.50
    }
};

bot.dialog('orderDinner', [
    function(session){
        session.send("Lets order some dinner!");
        builder.Prompts.choice(session, "Dinner menu:", dinnerMenu);
    },
    function (session, results) {
        if (results.response) {
            var order = dinnerMenu[results.response.entity];
            var msg = `You ordered: ${order.Description} for a total of $${order.Price}.`;
            session.dialogData.order = order;
            session.send(msg);
            builder.Prompts.text(session, "What is your room number?");
        } 
    },
    function(session, results){
        if(results.response){
            session.dialogData.room = results.response;
            var msg = `Thank you. Your order will be delivered to room #${session.dialogData.room}`;
            session.endDialog(msg);
        }
    }
])
.triggerAction({
    matches: /^order dinner$/i,
    confirmPrompt: "This will cancel your order. Are you sure?"
});
```

在使用者開始對話並選取 `Dinner Reservation` 或 `Order Dinner` 之後，他們隨時都可能會改變心意。 例如，如果使用者在進行晚餐訂位的過程中輸入 "order dinner" (點晚餐)，Bot 將會說出 "This will cancel your current request. Are you sure?" (這將會取消您目前的要求。是否確定？) 來進行確認。 如果使用者輸入 "no" (否)，則會取消該要求，而使用者可以繼續進行晚餐訂位程序。 如果使用者輸入 "yes" (是)，Bot 將清除對話 (dialog) 堆疊，並將對話 (conversation) 的控制權轉給 `orderDinner` 對話 (dialog)。

## <a name="end-conversation"></a>結束對話

在上述範例中，對話是藉由使用 `session.endDialog` 或 `session.endDialogWithResult` 來關閉；這兩者均會結束對話，從堆疊中移除該對話，並將控制權還給呼叫對話。 在使用者已抵達對話結尾的情況下，您應該使用 `session.endConversation` 來指出該對話已結束。

[`session.endConversation`](https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.session#endconversation) \(英文\) 方法會結束對話，並能選擇性地傳送訊息給使用者。 例如，上一個範例中的 `orderDinner` 對話 (dialog) 可以使用 `session.endConversation` 來結束對話 (conversation)，如下列程式碼範例所示。

```javascript
bot.dialog('orderDinner', [
    //...waterfall steps...
    // Last step
    function(session, results){
        if(results.response){
            session.dialogData.room = results.response;
            var msg = `Thank you. Your order will be delivered to room #${session.dialogData.room}`;
            session.endConversation(msg);
        }
    }
]);
```

呼叫 `session.endConversation` 將會清除對話 (dialog) 堆疊並重設 [`session.conversationData`](https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.session#conversationdata) \(英文\) 儲存體來結束對話 (conversation)。 如需資料儲存體的詳細資訊，請參閱[管理狀態資料](bot-builder-nodejs-state.md)。

當使用者完成設計 Bot 以進行的對話流程時，呼叫 `session.endConversation` 是一件合乎邏輯的事。 您也可以在使用者於對話期間輸入"cancel" (取消) 或 "goodbye" (再見) 的情況下，使用 `session.endConversation` 來結束對話。 若要這樣做，只需將 `endConversationAction` 附加至對話，並讓此觸發程序接聽符合 "cancel" (取消) 或 "goodbye" (再見) 的輸入。

下列程式碼範例會示範如何將 `endConversationAction` 附加至對話 (dialog)，來在使用者輸入 "cancel" (取消) 或 "goodbye" (再見) 時結束對話 (conversation)。

```javascript
bot.dialog('dinnerOrder', [
    //...waterfall steps...
])
.endConversationAction(
    "endOrderDinner", "Ok. Goodbye.",
    {
        matches: /^cancel$|^goodbye$/i,
        confirmPrompt: "This will cancel your order. Are you sure?"
    }
);
```

由於使用 `session.endConversation` 或 `endConversationAction` 來結束對話 (conversation) 將會清除對話 (dialog) 堆疊，並強制使用者從頭開始，因此您應該包含 `confirmPrompt` 以確保使用者真的想要執行該動作。

## <a name="next-steps"></a>後續步驟

在本文中，您已探索管理具有循序本質之對話的方式。 如果您想要在對話 (conversation) 中重複某個對話 (dialog) 或使用迴圈模式，該怎麼辦？ 讓我們來看看如何藉由取代堆疊上的對話來達成該目的。

> [!div class="nextstepaction"]
> [取代對話](bot-builder-nodejs-dialog-replace.md)

