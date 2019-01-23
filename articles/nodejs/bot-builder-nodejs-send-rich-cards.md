---
title: 將複合式資訊卡 (Rich Card) 附件新增至訊息 | Microsoft Docs
description: 了解如何使用適用於 Node.js 的 Bot Framework SDK，傳送具互動性且吸引人的複合式資訊卡。
author: v-ducvo
ms.author: v-ducvo
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: e3bf4a6868702f24af08e69d5f07c036082ec3b6
ms.sourcegitcommit: b15cf37afc4f57d13ca6636d4227433809562f8b
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 01/11/2019
ms.locfileid: "54225233"
---
# <a name="add-rich-card-attachments-to-messages"></a>將複合式資訊卡 (Rich Card) 附件新增至訊息

[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]


> [!div class="op_single_selector"]
> - [.NET](../dotnet/bot-builder-dotnet-add-rich-card-attachments.md)
> - [Node.js](../nodejs/bot-builder-nodejs-send-rich-cards.md)
> - [REST](../rest-api/bot-framework-rest-connector-add-rich-cards.md)

許多頻道 (例如 Skype 和 Facebook) 均支援傳送複合式圖形化資訊卡給使用者，其中具有使用者可按一下來起始某個動作的互動式按鈕。 SDK 會提供數個訊息和卡片建立器類別，可用來建立和傳送卡片。 Bot Framework 連接器服務將使用頻道的原生結構描述來呈現這些卡片，以支援跨平台通訊。 如果頻道不支援卡片 (例如 SMS)，Bot Framework 將盡其所能為使用者呈現合理的體驗。 

## <a name="types-of-rich-cards"></a>複合式資訊卡的類型 
Bot Framework 目前支援八種類型的複合式資訊卡 (Rich Card)： 

| 卡片類型 | 說明 |
|------|------|
| <a href="/adaptive-cards/get-started/bots">調適型卡片</a> | 可包含文字、語音、影像、按鈕和輸入欄位之任意組合的可自訂卡片。  請參閱[個別頻道支援](/adaptive-cards/get-started/bots#channel-status)。 |
| [動畫卡片][animationCard] | 可播放動畫 GIF 或短片的卡片。 |
| [音訊卡片][audioCard] | 可以播放音訊檔案的卡片。 |
| [主圖卡片][heroCard] | 通常包含單一大型影像、一或多個按鈕和文字的卡片。 |
| [縮圖卡片][thumbnailCard] | 通常包含單一縮圖影像、一或多個按鈕和文字的卡片。|
| [收據卡片][receiptCard] | 讓 Bot 向使用者提供收據的卡片。 通常包含收據上的項目清單、稅金和總金額資訊，以及其他文字。 |
| [登入卡片][signinCard] | 可讓 Bot 要求使用者登入的卡片。 通常包含文字，以及一或多個使用者可以按一下以起始登入程序的按鈕。 |
| [視訊卡片][videoCard] | 可播放視訊的卡片。 |

## <a name="send-a-carousel-of-hero-cards"></a>傳送浮動切換的主圖卡片
下列範例會示範某家虛構 T 恤公司的 Bot，並示範如何在使用者說出 “show shirts” (顯示 T 恤) 時，傳送浮動切換的卡片來回應。 

```javascript
// Create your bot with a function to receive messages from the user
// Create bot and default message handler
var bot = new builder.UniversalBot(connector, function (session) {
    session.send("Hi... We sell shirts. Say 'show shirts' to see our products.");
});

// Add dialog to return list of shirts available
bot.dialog('showShirts', function (session) {
    var msg = new builder.Message(session);
    msg.attachmentLayout(builder.AttachmentLayout.carousel)
    msg.attachments([
        new builder.HeroCard(session)
            .title("Classic White T-Shirt")
            .subtitle("100% Soft and Luxurious Cotton")
            .text("Price is $25 and carried in sizes (S, M, L, and XL)")
            .images([builder.CardImage.create(session, 'http://petersapparel.parseapp.com/img/whiteshirt.png')])
            .buttons([
                builder.CardAction.imBack(session, "buy classic white t-shirt", "Buy")
            ]),
        new builder.HeroCard(session)
            .title("Classic Gray T-Shirt")
            .subtitle("100% Soft and Luxurious Cotton")
            .text("Price is $25 and carried in sizes (S, M, L, and XL)")
            .images([builder.CardImage.create(session, 'http://petersapparel.parseapp.com/img/grayshirt.png')])
            .buttons([
                builder.CardAction.imBack(session, "buy classic gray t-shirt", "Buy")
            ])
    ]);
    session.send(msg).endDialog();
}).triggerAction({ matches: /^(show|list)/i });
```
這個範例會使用 [Message][Message] 類別來建置浮動切換。  
浮動切換會由 [HeroCard][heroCard] 類別的清單所組成，其中包含影像、文字，以及會觸發購買該項目的單一按鈕。  
按一下 [Buy] \(購買\) 按鈕就會觸發傳送訊息，因此我們需要新增第二個對話來擷取按鈕點選動作。 

## <a name="handle-button-input"></a>處理按鈕輸入

無論在任何時候，只要接收到以 “buy” (購買) 或 “add” (新增) 作為開始，且後面接著包含 “shirt” (T 恤) 單字之內容的訊息時，就會觸發 `buyButtonClick` 對話。 對話一開始會使用幾個規則運算式，來尋找符合使用者所詢問之顏色及可選尺寸的 T 恤。
這個額外的彈性可讓您同時支援來自使用者的按鈕點選動作，以及自然語言訊息，例如 “please add a large gray shirt to my cart” (請將 L 號的灰色 T 恤新增至我的購物車)。
如果顏色是有效的，但是尺寸未知，Bot 就會提示使用者從清單挑選尺寸，然後才會將該項目新增至購物車。 一旦 Bot 擁有所需的所有資訊之後，就會將該項目新增至使用 **session.userData** 保存的購物車，並傳送確認訊息給使用者。

```javascript
// Add dialog to handle 'Buy' button click
bot.dialog('buyButtonClick', [
    function (session, args, next) {
        // Get color and optional size from users utterance
        var utterance = args.intent.matched[0];
        var color = /(white|gray)/i.exec(utterance);
        var size = /\b(Extra Large|Large|Medium|Small)\b/i.exec(utterance);
        if (color) {
            // Initialize cart item
            var item = session.dialogData.item = { 
                product: "classic " + color[0].toLowerCase() + " t-shirt",
                size: size ? size[0].toLowerCase() : null,
                price: 25.0,
                qty: 1
            };
            if (!item.size) {
                // Prompt for size
                builder.Prompts.choice(session, "What size would you like?", "Small|Medium|Large|Extra Large");
            } else {
                //Skip to next waterfall step
                next();
            }
        } else {
            // Invalid product
            session.send("I'm sorry... That product wasn't found.").endDialog();
        }   
    },
    function (session, results) {
        // Save size if prompted
        var item = session.dialogData.item;
        if (results.response) {
            item.size = results.response.entity.toLowerCase();
        }

        // Add to cart
        if (!session.userData.cart) {
            session.userData.cart = [];
        }
        session.userData.cart.push(item);

        // Send confirmation to users
        session.send("A '%(size)s %(product)s' has been added to your cart.", item).endDialog();
    }
]).triggerAction({ matches: /(buy|add)\s.*shirt/i });
```

<!-- 

> [!NOTE]
> When sending a message that contains images, keep in mind that some channels download images before displaying a message to the user.   
> As a result, a message containing an image followed immediately by a message without images may sometimes be flipped in the user's feed.
> For information on how to avoid messages being sent out of order, see [Message ordering][MessageOrder].  

-->
## <a name="add-a-message-delay-for-image-downloads"></a>針對影像下載新增訊息延遲
某些頻道會傾向先下載影像，然後才向使用者顯示訊息，因此如果您傳送一則包含影像的訊息，後面緊接著一則未包含影像的訊息，您有時會看見這兩則訊息的順序會在使用者的摘要中翻轉過來。 若要將發生此情況的機會降到最低，您可以試著確保您的影像是來自內容傳遞網路 (CDN)，並避免使用過大的影像。 在極端的案例中，您甚至可能需要在含有影像的訊息和後面接著的訊息之間插入 1-2 秒的延遲。 若要讓使用者對這個延遲感到更自然，您可以藉由呼叫 **session.sendTyping()** 以在開始延遲之前傳送「正在輸入」的指示。 

<!-- 
To learn more about sending a typing indicator, see [How to send a typing indicator](bot-builder-nodejs-send-typing-indicator.md).
-->

Bot Framework 會實作批次處理，以試著防止來自 Bot 的多則訊息會以不按照順序的方式顯示。 <!-- Unfortunately, not all channels can guarantee this. --> 當您的 Bot 傳送多個回覆給使用者時，個別的訊息將會自動分組為批次，並以集合的形式傳遞給使用者，以試圖保留訊息的原始順序。 這個自動批次處理會在每次呼叫 **session.send()** 之後，以及起始對 **send()** 的下一個呼叫之前，預設等待 250 毫秒的時間。

訊息批次處理延遲是可設定的。 若要停用 SDK 的自動批次處理邏輯，請將預設延遲設定為較大的數字，然後以手動方式呼叫 **sendBatch()**，並搭配會在傳遞批次之後叫用的回呼。

## <a name="send-an-adaptive-card"></a>傳送調適型卡片

調適型卡片可包含文字、語音、影像、按鈕和輸入欄位的任意組合。 調適型卡片會使用<a href="http://adaptivecards.io" target="_blank">調適型卡片</a> \(英文\) 中所指定的 JSON 格式來建立，讓您能夠完整控制卡片的內容和格式。 

若要使用 Node.js 建立調適型卡片，請利用<a href="http://adaptivecards.io" target="_blank">調適型卡片</a> \(英文\) 網站中的資訊，來了解調適型卡片的結構描述、探索調適型卡片的元素，以及查看可用來建立具有不同組合和複雜度之卡片的 JSON 範例。 此外，您可以使用互動式視覺化檢視，來設計調適型卡片承載及預覽卡片輸出。

下列程式碼範例示範如何建立包含調適型卡片作為行事曆提醒的訊息： 

[!code-javascript[Add Adaptive Card attachment](../includes/code/node-send-card-buttons.js#addAdaptiveCardAttachment)]

產生的卡片會包含三個文字區塊、一個輸入欄位 (選擇清單) 和三個按鈕：

![調適型卡片行事曆提醒](../media/adaptive-card-reminder.png)

## <a name="additional-resources"></a>其他資源

* [使用頻道偵測器來預覽功能][inspector]
* <a href="http://adaptivecards.io" target="_blank">調適型卡片</a> \(英文\)
* [AnimationCard][animationCard]
* [AudioCard][audioCard]
* [HeroCard][heroCard]
* [ThumbnailCard][thumbnailCard]
* [ReceiptCard][receiptCard]
* [SignInCard][signinCard]
* [VideoCard][videoCard]
* [Message][Message]
* [如何傳送附件](bot-builder-nodejs-send-receive-attachments.md)

[MessageOrder]: bot-builder-nodejs-manage-conversation-flow.md#message-ordering
[Message]: https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.message
[IMessage]: http://docs.botframework.com/en-us/node/builder/chat-reference/interfaces/_botbuilder_d_.imessage

[animationCard]: https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.animationcard.html 

[audioCard]: https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.audiocard.html 

[heroCard]: https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.herocard.html

[thumbnailCard]: https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.thumbnailcard.html 

[receiptCard]: https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.receiptcard.html 

[signinCard]: https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.signincard.html 

[videoCard]: https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.videocard.html

[inspector]: ../bot-service-channel-inspector.md
