---
title: 傳送主動訊息 | Microsoft Docs
description: 了解如何以使用適用於 Node.js 的 Bot Framework SDK 主動訊息中斷目前的交談流程
author: v-ducvo
ms.author: v-ducvo
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: 8ca8043c5680a993fa27e2febb9740206691884c
ms.sourcegitcommit: b15cf37afc4f57d13ca6636d4227433809562f8b
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 01/11/2019
ms.locfileid: "54225573"
---
# <a name="send-proactive-messages"></a>傳送主動訊息
[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]

> [!div class="op_single_selector"]
> - [.NET](../dotnet/bot-builder-dotnet-proactive-messages.md)
> - [Node.js](../nodejs/bot-builder-nodejs-proactive-messages.md)

[!INCLUDE [Introduction to proactive messages - part 1](../includes/snippet-proactive-messages-intro-1.md)]

## <a name="types-of-proactive-messages"></a>主動式訊息的類型

[!INCLUDE [Introduction to proactive messages - part 2](../includes/snippet-proactive-messages-intro-2.md)]

## <a name="send-an-ad-hoc-proactive-message"></a>傳送臨機操作的主動訊息

下列程式碼範例示範如何使用適用於 Node.js 的 Bot Framework SDK 傳送臨機操作的主動訊息。

為能將臨機操作的訊息傳送給使用者，Bot 必須先收集並儲存來自目前交談的使用者相關資訊。 訊息的 **address** 屬性包含 Bot 稍後需要傳送給使用者之臨機操作訊息的所有資訊。 

```javascript
bot.dialog('adhocDialog', function(session, args) {
    var savedAddress = session.message.address;

    // (Save this information somewhere that it can be accessed later, such as in a database, or session.userData)
    session.userData.savedAddress = savedAddress;

    var message = 'Hello user, good to meet you! I now know your address and can send you notifications in the future.';
    session.send(message);
})
```

> [!NOTE]
> Bot 可以任何方式儲存使用者資料，只要 Bot 以後能夠存取此資料。

Bot 收集完使用者相關資訊之後，它可以隨時向使用者傳送臨機操作的主動訊息。 若要這樣做，它只要擷取之前儲存的使用者資料、建構訊息，然後傳送訊息即可。

```javascript
var inMemoryStorage = new builder.MemoryBotStorage();

var bot = new builder.UniversalBot(connector)
                .set('storage', inMemoryStorage); // Register in-memory storage 

function sendProactiveMessage(address) {
    var msg = new builder.Message().address(address);
    msg.text('Hello, this is a notification');
    msg.textLocale('en-US');
    bot.send(msg);
}
```

> [!TIP]
> 您可從非同步的觸發程序 (例如 http 要求、計時器、佇列) 或開發人員選擇的任何其他地方，起始臨機操作主動訊息。

## <a name="send-a-dialog-based-proactive-message"></a>傳送對話方塊式的主動訊息

下列程式碼範例示範如何使用適用於 Node.js 的 Bot Framework SDK 傳送對話式主動訊息。 您可在 [Microsoft/BotBuilder-Samples/Node/core-proactiveMessages/startNewDialog](https://github.com/Microsoft/BotBuilder-Samples/tree/master/Node/core-proactiveMessages/startNewDialog) 資料夾中找到完整的運作範例。

為能將交談方式式的訊息傳送給使用者，Bot 必須先收集 (並儲存) 來自目前交談的相關資訊。 `session.message.address` 物件包含 Bot 需要傳送給使用者之對話方塊式主動訊息的所有資訊。 

```javascript
// proactiveDialog dialog
bot.dialog('proactiveDialog', function (session, args) {

    savedAddress = session.message.address;

    var message = 'Hey there, I\'m going to interrupt our conversation and start a survey in five seconds...';
    session.send(message);

    message = `You can also make me send a message by accessing: http://localhost:${server.address().port}/api/CustomWebApi`;
    session.send(message);

    setTimeout(() => {
        startProactiveDialog(savedAddress);
    }, 5000);
});
```

傳送訊息時，Bot 會建立新的對話方塊，並將它新增至對話方塊堆疊的最上層。 新的交談方塊會控制交談，傳遞主動訊息、關閉，然後將控制權傳回給堆疊中的上一個交談方塊。 

```javascript
// initiate a dialog proactively 
function startProactiveDialog(address) {
    bot.beginDialog(address, "*:survey");
}
```

> [!NOTE]
> 上述程式碼範例需要自訂的檔案 **botadapter.js**，您可以[從 GitHub 下載](https://github.com/Microsoft/BotBuilder-Samples/blob/master/Node/core-proactiveMessages/startNewDialog/botadapter.js)。

調查交談方塊會控制交談直到結束。 然後，它會關閉 (呼叫 `session.endDialog()`)，再將控制權傳回給之前的對話方塊。 


```javascript
// handle the proactive initiated dialog
bot.dialog('survey', function (session, args, next) {
  if (session.message.text === "done") {
    session.send("Great, back to the original conversation");
    session.endDialog();
  } else {
    session.send('Hello, I\'m the survey dialog. I\'m interrupting your conversation to ask you a question. Type "done" to resume');
  }
});
```

## <a name="sample-code"></a>範例程式碼

如需示範如何使用適用於 Node.js 的 Bot Framework SDK 傳送主動訊息的完整範例，請參閱 GitHub 中的<a href="https://github.com/Microsoft/BotBuilder-Samples/tree/master/Node/core-proactiveMessages" target="_blank">主動訊息範例</a>。 在主動訊息範例中，<a href="https://github.com/Microsoft/BotBuilder-Samples/tree/master/Node/core-proactiveMessages/simpleSendMessage" target="_blank">simpleSendMessage</a> 示範如何傳送臨機操作的主動訊息，而 <a href="https://github.com/Microsoft/BotBuilder-Samples/tree/master/Node/core-proactiveMessages/startNewDialog" target="_blank">startNewDialog</a> 示範如何傳送對話方塊式的主動訊息。

## <a name="additional-resources"></a>其他資源

- [設計交談流程](../bot-service-design-conversation-flow.md)
