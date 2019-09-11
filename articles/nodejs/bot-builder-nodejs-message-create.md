---
title: 建立訊息 | Microsoft Docs
description: 了解如何使用適用於 Node.js 的 Bot Framework SDK 來建立訊息。
author: v-ducvo
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 09/7/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: 7e2f48a3450154de9e2465f9d0d992ace4f3996f
ms.sourcegitcommit: a6d02ec4738e7fc90b7108934740e9077667f3c5
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/04/2019
ms.locfileid: "70299801"
---
# <a name="create-messages"></a>建立訊息

[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]

Bot 和使用者會透過訊息來通訊。 Bot 會傳送訊息活動以傳達資訊給使用者，同樣也會接收來自使用者的訊息活動。 某些訊息可能只包含純文字，而其他訊息可能包含更豐富的內容，例如要讀出的文字、建議的動作、媒體附件、複合式資訊卡 (Rich Card)，和通道特定資料。

本文說明一些可供您用來增強使用者體驗的常用訊息方法。

## <a name="default-message-handler"></a>預設訊息處理常式

適用於 Node.js 的 Bot Framework SDK 隨附內建於 [`session`](https://docs.botframework.com/node/builder/chat-reference/classes/_botbuilder_d_.session.html) 物件的預設訊息處理常式。 此訊息處理常式可讓您在 Bot 和使用者之間傳送和接收文字訊息。

### <a name="send-a-text-message"></a>傳送文字訊息

使用預設訊息處理常式來傳送文字訊息非常簡單，只要呼叫 `session.send` 並傳入**字串**即可。

下列範例說明如何傳送文字訊息來歡迎使用者。
```javascript
session.send("Good morning.");
```

下列範例說明如何使用 JavaScript 字串範本來傳送文字訊息。
```javascript
var msg = `You ordered: ${order.Description} for a total of $${order.Price}.`;
session.send(msg); //msg: "You ordered: Potato Salad for a total of $5.99."
```

### <a name="receive-a-text-message"></a>接收文字訊息

當使用者傳送訊息給 Bot 時，Bot 會透過 `session.message` 屬性接收到訊息。

下列範例說明如何存取使用者的訊息。
```javascript
var userMessage = session.message.text;
```

## <a name="customizing-a-message"></a>自訂訊息

若要更充分地掌控訊息的文字格式設定，您可以建立自訂 [`message`](https://docs.botframework.com/node/builder/chat-reference/classes/_botbuilder_d_.message.html) 物件並先設定所需屬性，再將此物件傳送給使用者。

下列範例說明如何建立自訂 `message` 物件，並設定 `text`、`textFormat` 和 `textLocale` 屬性。

```javascript
var customMessage = new builder.Message(session)
    .text("Hello!")
    .textFormat("plain")
    .textLocale("en-us");
session.send(customMessage);
```

若您在範圍內沒有 `session` 物件，則可以使用 `bot.send` 方法將已格式化的訊息傳送給使用者。

訊息的 `textFormat` 屬性可用來指定文字格式。 `textFormat` 屬性可以設定為 **plain**、**markdown** 或 **xml**。 `textFormat` 的預設值為 **markdown**。 

## <a name="message-property"></a>訊息屬性

[`Message`](https://docs.botframework.com/node/builder/chat-reference/classes/_botbuilder_d_.message.html) 物件具有內部**資料**屬性，並使用此屬性來管理所傳送的訊息。 您所設定的其他屬性，則會透過此物件向您公開的不同方法。 

## <a name="message-methods"></a>訊息方法

訊息屬性會透過物件的方法來加以設定並擷取。 下表提供您所能呼叫以便設定/取得不同**訊息**屬性的方法清單。

| 方法 | 說明 |
| ---- | ---- | 
| [`addAttachment(attachment:AttachmentType)`](https://docs.botframework.com/node/builder/chat-reference/classes/_botbuilder_d_.message.html#addattachment) | 將附件新增至訊息|
| [`addEntity(obj:Object)`](https://docs.botframework.com/node/builder/chat-reference/classes/_botbuilder_d_.message.html#addentity) | 將實體新增至訊息。 |
| [`address(adr:IAddress)`](https://docs.botframework.com/node/builder/chat-reference/classes/_botbuilder_d_.message.html#address) | 寫下訊息的路由資訊位址。 若要傳送主動訊息給使用者，請將訊息的位址儲存在 userData 包中。 |
| [`attachmentLayout(style:string)`](https://docs.botframework.com/node/builder/chat-reference/classes/_botbuilder_d_.message.html#attachmentlayout) | 提示用戶端應如何配置多個附件。 預設值為 'list'。 |
| [`attachments(list:AttachmentType)`](https://docs.botframework.com/node/builder/chat-reference/classes/_botbuilder_d_.message.html#attachments) | 要傳送給使用者的卡片或影像清單。 |
| [`compose(prompts:string[], ...args:any[])`](https://docs.botframework.com/node/builder/chat-reference/classes/_botbuilder_d_.message.html#compose) | 撰寫要給使用者的複雜、隨機回覆。 |
| [`entities(list:Object[])`](https://docs.botframework.com/node/builder/chat-reference/classes/_botbuilder_d_.message.html#entities) | 已傳遞給 Bot 或使用者的結構化物件。 |
| [`inputHint(hint:string)`](https://docs.botframework.com/node/builder/chat-reference/classes/_botbuilder_d_.message.html#inputhint) | 傳送給使用者的提示，讓他們知道 Bot 是否需要進一步的輸入。 內建提示會自動為外寄訊息填入這個值。 |
| [`localTimeStamp((optional)time:string)`](https://docs.botframework.com/node/builder/chat-reference/classes/_botbuilder_d_.message.html#localtimestamp) | 傳送訊息時的當地時間 (由用戶端或 Bot 設定，例如：2016-09-23T13:07:49.4714686-07:00)。 |
| [`originalEvent(event:any)`](https://docs.botframework.com/node/builder/chat-reference/classes/_botbuilder_d_.message.html#originalevent) | 內送訊息通道的原始/原生格式訊息。 |
| [`sourceEvent(map:ISourceEventMap)`](https://docs.botframework.com/node/builder/chat-reference/classes/_botbuilder_d_.message.html#sourceevent) | 因為外寄訊息可以用來傳遞來源特定的事件資料，例如自訂附件。 |
| [`speak(ssml:TextType, ...args:any[])`](https://docs.botframework.com/node/builder/chat-reference/classes/_botbuilder_d_.message.html#speak) | 將訊息的發音領域設定為「語音合成標記語言 (SSML)」  。 會在受支援裝置上向使用者說出此內容。 |
| [`suggestedActions(suggestions:ISuggestedActions `&#124;` IIsSuggestedActions)`](https://docs.botframework.com/node/builder/chat-reference/classes/_botbuilder_d_.message.html#suggestedactions) | 要傳送給使用者的選擇性建議動作。 只會在支援建議動作的通道上顯示建議動作。 |
| [`summary(text:TextType, ...argus:any[])`](https://docs.botframework.com/node/builder/chat-reference/classes/_botbuilder_d_.message.html#summary) | 要在其中顯示為後援以及顯示為訊息內容簡短描述的文字 (例如：近期交談清單)。 |
| [`text(text:TextType, ...args:any[])`](https://docs.botframework.com/node/builder/chat-reference/classes/_botbuilder_d_.message.html#text) | 設定訊息文字。 |
| [`textFormat(style:string)`](https://docs.botframework.com/node/builder/chat-reference/classes/_botbuilder_d_.message.html#textformat) | 設定文字格式。 預設格式為 **markdown**。 |
| [`textLocale(locale:string)`](https://docs.botframework.com/node/builder/chat-reference/classes/_botbuilder_d_.message.html#textlocale) | 設定訊息的目標語言。 |
| [`toMessage()`](https://docs.botframework.com/node/builder/chat-reference/classes/_botbuilder_d_.message.html#tomessage) | 取得訊息的 JSON。 |
| [`composePrompt(session:Session, prompts:string[], args?:any[])`](https://docs.botframework.com/node/builder/chat-reference/classes/_botbuilder_d_.message.html#composeprompt-1) | 將提示陣列結合成單一的當地語系化提示，然後選擇性地以傳入的引數填滿提示範本位置。 |
| [`randomPrompt(prompts:TextType)`](https://docs.botframework.com/node/builder/chat-reference/classes/_botbuilder_d_.message.html#randomprompt) | 從傳入的 *prompts  陣列中取得隨機提示。 |

## <a name="next-step"></a>後續步驟

> [!div class="nextstepaction"]
> [傳送和接收附件](bot-builder-nodejs-send-receive-attachments.md)

