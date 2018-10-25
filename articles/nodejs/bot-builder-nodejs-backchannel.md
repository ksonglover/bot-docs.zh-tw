---
title: 使用 Web 控制項交換資訊 | Microsoft Docs
description: 了解如何使用適用於 Node.js 的 Bot 建立器 SDK 在 Bot 與網頁之間交換資訊。
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: dc77a09f1eb7d6ccbdf98d1bee22000c56fe8e6b
ms.sourcegitcommit: b78fe3d8dd604c4f7233740658a229e85b8535dd
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/24/2018
ms.locfileid: "49999685"
---
# <a name="use-the-backchannel-mechanism"></a>使用 backchannel 機制

[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]

[!INCLUDE [Introduction to backchannel mechanism](../includes/snippet-backchannel.md)]

## <a name="walk-through"></a>逐步解說

開放原始碼網路聊天控制項會使用稱為 <a href="https://github.com/microsoft/botframework-DirectLinejs" target="_blank">DirectLineJS</a> 的 JavaScript 類別來存取 Direct Line API。 控制項可以建立自己 Direct Line 的執行個體，或可與主控頁面共用同一個執行個體。 如果控制項與主控頁面共用 Direct Line 的執行個體，則控制項和頁面都能夠傳送及接收活動。 下圖顯示網站的高階架構，可使用開放原始碼網路 (聊天) 控制項和 Direct Line API 來支援 Bot 功能。 

![Backchannel](../media/designing-bots/patterns/back-channel.png)

### <a name="sample-code"></a>範例程式碼 

在此範例中，Bot 和網頁會使用 backchannel 機制來交換使用者不會察覺到的資訊。 Bot 會要求網頁變更其背景色彩，且當使用者按一下網頁上的按鈕時，網頁會通知 Bot。 

> [!NOTE]
> 本文中的程式碼片段是來自 <a href="https://github.com/Microsoft/BotFramework-WebChat/blob/master/samples/backchannel/index.html" target="_blank">backchannel 範例</a> (英文) 和 <a href="https://github.com/ryanvolum/backChannelBot" target="_blank">backchannel Bot</a> (英文)。 

#### <a name="client-side-code"></a>用戶端程式碼

首先，網頁會建立 **DirectLine** 物件。

```javascript
var botConnection = new BotChat.DirectLine(...);
```

然後，它在建立 WebChat 執行個體時，會共用 **DirectLine** 物件。

```javascript
BotChat.App({
    botConnection: botConnection,
    user: user,
    bot: bot
}, document.getElementById("BotChatGoesHere"));
```

當使用者按一下網頁上的按鈕時，網頁會張貼「活動」類型的通知，以通知 Bot 有人按一下按鈕。

```javascript
const postButtonMessage = () => {
    botConnection
        .postActivity({type: "event", value: "", from: {id: "me" }, name: "buttonClicked"})
        .subscribe(id => console.log("success"));
    }
```

> [!TIP]
> 使用 `name` 和 `value` 屬性，來通訊 Bot 要正確解譯及/或回應事件時，可能需要的任何資訊。 

最後，網頁也會接聽來自 Bot 的特定事件。
在此範例中，網頁會接聽其中 type="event" 和 name="changeBackground" 的活動。 當它收到這種類型的活動時，會將網頁的背景色彩變更為活動所指定的 `value`。 

```javascript
botConnection.activity$
    .filter(activity => activity.type === "event" && activity.name === "changeBackground")
    .subscribe(activity => changeBackgroundColor(activity.value))
```

#### <a name="server-side-code"></a>伺服器端程式碼

<a href="https://github.com/ryanvolum/backChannelBot" target="_blank">Backchannel Bot</a> 會使用 helper 函式來建立事件。

```javascript
var bot = new builder.UniversalBot(connector, 
    function (session) {
        var reply = createEvent("changeBackground", session.message.text, session.message.address);
        session.endDialog(reply);
    }
);

const createEvent = (eventName, value, address) => {
    var msg = new builder.Message().address(address);
    msg.data.type = "event";
    msg.data.name = eventName;
    msg.data.value = value;
    return msg;
}
```

同樣地，Bot 也會接聽來自用戶端的事件。 在此範例中，如果 Bot 收到包含 `name="buttonClicked"` 的事件，會傳送訊息給使用者，說明「我看到您已按一下按鈕」。

```javascript
bot.on("event", function (event) {
    var msg = new builder.Message().address(event.address);
    msg.data.textLocale = "en-us";
    if (event.name === "buttonClicked") {
        msg.data.text = "I see that you clicked a button.";
    }
    bot.send(msg);
})
```

## <a name="additional-resources"></a>其他資源

- [Direct Line API][directLineAPI]
- <a href="https://github.com/Microsoft/BotFramework-WebChat" target="_blank">Microsoft Bot Framework WebChat 控制項</a>
- <a href="https://github.com/Microsoft/BotFramework-WebChat/blob/master/samples/backchannel/index.html" target="_blank">Backchannel 範例</a>
- <a href="https://github.com/ryanvolum/backChannelBot" target="_blank">Backchannel Bot</a>

[directLineAPI]: https://docs.botframework.com/en-us/restapi/directline3/#navtitle
