---
title: 攔截訊息 | Microsoft Docs
description: 了解如何藉由使用適用於 Node.js 的 Bot 建立器 SDK 來攔截和處理資訊交換，來建立記錄檔或其他記錄。
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: 9160e81f9086f88f808b41e8eb01745776e0fc9f
ms.sourcegitcommit: f576981342fb3361216675815714e24281e20ddf
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/18/2018
ms.locfileid: "39299394"
---
# <a name="intercept-messages"></a>攔截訊息
> [!div class="op_single_selector"]
> - [.NET](../dotnet/bot-builder-dotnet-middleware.md)
> - [Node.js](../nodejs/bot-builder-nodejs-intercept-messages.md)

[!INCLUDE [Introduction to message logging](../includes/snippet-message-logging-intro.md)]

## <a name="example"></a>範例

下列程式碼範例會示範如何使用適用於 Node.js 的 Bot 建立器 SDK 中的**中介軟體**概念，來攔截使用者與 Bot 之間交換的訊息。 

首先，設定適用於傳入訊息 (`botbuilder`) 和傳出訊息 (`send`) 的處理常式。

```javascript
server.post('/api/messages', connector.listen());
var bot = new builder.UniversalBot(connector);
bot.use({
    botbuilder: function (session, next) {
        myMiddleware.logIncomingMessage(session, next);
    },
    send: function (event, next) {
        myMiddleware.logOutgoingMessage(event, next);
    }
})
```

接著，實作每個處理常式以定義要針對每個被攔截之訊息採取的動作。

```javascript
module.exports = {
    logIncomingMessage: function (session, next) {
        console.log(session.message.text);
        next();
    },
    logOutgoingMessage: function (event, next) {
        console.log(event.text);
        next();
    }
}
```

現在每個輸入訊息 (從使用者到 Bot) 都會觸發 `logIncomingMessage`，而每個輸出訊息 (從 Bot 到使用者) 則都會觸發 `logOutgoingMessage`。
在此範例中，Bot 只會列印有關每個訊息的部分資訊，但您可以視需要更新 `logIncomingMessage` 和 `logOutgoingMessage`，以定義您想要針對每個訊息採取的動作。 

## <a name="sample-code"></a>範例程式碼

如需示範如何使用適用於 Node.js 的 Bot 建立器 SDK 來攔截和記錄訊息的完整範例，請參閱 GitHub 中的<a href="https://github.com/Microsoft/BotBuilder-Samples/tree/master/Node/capability-middlewareLogging" target="_blank">中介軟體和記錄範例</a> \(英文\)。
