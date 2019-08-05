---
title: 傳送和接收附件 | Microsoft Docs
description: 了解如何使用適用於 Node.js 的 Bot Framework SDK 來傳送和接收含有附件的訊息。
author: DeniseMak
ms.author: v-demak
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: ac74fff5fa7635bf0ef585423b0f8663a1df41c4
ms.sourcegitcommit: 23a1808e18176f1704f2f6f2763ace872b1388ae
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/25/2019
ms.locfileid: "67404830"
---
# <a name="send-and-receive-attachments"></a>傳送和接收附件

[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]

> [!div class="op_single_selector"]
> - [.NET](../dotnet/bot-builder-dotnet-add-media-attachments.md)
> - [Node.js](../nodejs/bot-builder-nodejs-send-receive-attachments.md)
> - [REST](../rest-api/bot-framework-rest-connector-add-media-attachments.md)

使用者與 Bot 之間的訊息交換可以包含媒體附件，例如影像、視訊、音訊和檔案等。 可傳送的附件類型隨通道而異，但以下是基本類型：

* **媒體和檔案**：您可以傳送影像、音訊和視訊等檔案，方法是將 **contentType** 設定為 [IAttachment 物件][IAttachment] 的 MIME 類型，然後在 **contentUrl** 中傳遞檔案的連結。
* **卡**：您可以傳送一組豐富的視覺化卡片 <!-- and custom keyboards --> (方法是將 **contentType** 設定為所需的卡類型)，然後再傳遞卡片的 JSON。 如果您使用其中一個複合式資訊卡 (Rich Card) 建立器類別 (例如 **HeroCard**)，系統會自動為您填入附件。 如需這方面的範例，請參閱[傳送複合式資訊卡 (Rich Card)](bot-builder-nodejs-send-rich-cards.md)。

## <a name="add-a-media-attachment"></a>新增媒體附件
訊息物件預期會是 [IMessage][IMessage] 的執行個體，當您想要包含影像等附件時，最適合以物件形式來傳送訊息給使用者。 請使用 [session.send()][SessionSend] 方法，以 JSON 物件的形式傳送訊息。 

## <a name="example"></a>範例

下列範例會檢查使用者是否已傳送附件，如果是，則會回傳附件中包含的任何影像。 您可以傳送影像給 Bot ，以使用 Bot Framework 模擬器來進行這方面的測試。

```javascript
// Create your bot with a function to receive messages from the user
var bot = new builder.UniversalBot(connector, function (session) {
    var msg = session.message;
    if (msg.attachments && msg.attachments.length > 0) {
     // Echo back attachment
     var attachment = msg.attachments[0];
        session.send({
            text: "You sent:",
            attachments: [
                {
                    contentType: attachment.contentType,
                    contentUrl: attachment.contentUrl,
                    name: attachment.name
                }
            ]
        });
    } else {
        // Echo back users text
        session.send("You said: %s", session.message.text);
    }
});
```
## <a name="additional-resources"></a>其他資源

* [使用通道偵測器預覽功能][inspector]
* [IMessage][IMessage]
* [傳送複合式資訊卡 (Rich Card)][SendRichCard]
* [session.send][SessionSend]

[IMessage]: http://docs.botframework.com/node/builder/chat-reference/interfaces/_botbuilder_d_.imessage
[SendRichCard]: bot-builder-nodejs-send-rich-cards.md
[SessionSend]: https://docs.botframework.com/node/builder/chat-reference/classes/_botbuilder_d_.session.html#send
[IAttachment]: https://docs.botframework.com/node/builder/chat-reference/interfaces/_botbuilder_d_.iattachment.html
[inspector]: ../bot-service-channel-inspector.md
