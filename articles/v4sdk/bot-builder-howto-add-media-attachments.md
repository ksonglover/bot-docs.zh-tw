---
title: 將媒體新增至訊息 | Microsoft Docs
description: 了解如何使用 Bot Framework SDK 將媒體新增至訊息。
keywords: 媒體, 訊息, 影像, 音訊, 視訊, 檔案, MessageFactory, 複合式資訊卡, 訊息, 調適型卡片, 主圖卡片, 建議動作
author: ivorb
ms.author: v-ivorb
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 12/17/2018
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 1ea9daeb35033e49232d64bfe98a223807dabf75
ms.sourcegitcommit: b94361234816e6b95459f142add936732fc40344
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 01/15/2019
ms.locfileid: "54317598"
---
# <a name="add-media-to-messages"></a>將媒體新增至訊息

[!INCLUDE [pre-release-label](../includes/pre-release-label.md)]

使用者與 Bot 之間交換的訊息可以包含媒體附件，例如影像、視訊、音訊和檔案等。 Bot Framework SDK 支援將豐富訊息傳送給使用者的工作。 若要判斷通道 (Facebook、Skype、Slack 等等) 支援的豐富訊息類型，請參閱通道文件中的限制相關資訊。 請參閱[設計使用者體驗](../bot-service-design-user-experience.md)，以取得一份可用的卡片。 

## <a name="send-attachments"></a>傳送附件

若要傳送影像或視訊之類的使用者內容，您可以將附件或附件清單新增至訊息。

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

`Activity` 物件的 `Attachments` 屬性包含 `Attachment` 物件的陣列，代表附加至訊息的媒體附件和複合式資訊卡 (Rich Card)。 若要將媒體附件新增至訊息，請建立 `message` 活動的 `Attachment` 物件，並設定 `ContentType`、`ContentUrl` 和 `Name` 屬性。
這裡顯示的原始程式碼是以[處理附件](https://aka.ms/bot-attachments-sample-code)範例為基礎。 

```csharp
using Microsoft.Bot.Builder;
using Microsoft.Bot.Builder.Core.Extensions;
using Microsoft.Bot.Schema;

var reply = turnContext.Activity.CreateReply();

// Create an attachment.
var attachment = new Attachment
    {
        ContentUrl = "imageUrl.png",
        ContentType = "image/png",
        Name = "imageName",
    };

// Add the attachment to our reply.
reply.Attachments = new List<Attachment>() { attachment };

// Send the activity to the user.
await turnContext.SendActivityAsync(reply, cancellationToken);
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

這裡顯示的原始程式碼是以 [JS 處理附件](https://aka.ms/bot-attachments-sample-code-js)範例為基礎。
若要將影像或影片等單項內容傳送給使用者，您可以傳送包含在 URL 中的媒體：

```javascript
const { ActionTypes, ActivityTypes, CardFactory } = require('botbuilder');

// Call function to get an attachment.
const reply = { type: ActivityTypes.Message };
reply.attachments = [this.getInternetAttachment()];
reply.text = 'This is an internet attachment.';
// Send the activity to the user.
await turnContext.sendActivity(reply);

/* function getInternetAttachment - Returns an attachment to be sent to the user from a HTTPS URL */
getInternetAttachment() {
        return {
            name: 'imageName.png',
            contentType: 'image/png',
            contentUrl: 'imageUrl.png'}
}
```

---

如果附件是影像、音訊或視訊，則連接器服務會以可讓[通道](bot-builder-channeldata.md)在對話內轉譯該附件的方式，將附件資料傳輸至通道。 如果附件是檔案，檔案 URL 將會轉譯為對話內的超連結。

## <a name="send-a-hero-card"></a>傳送主圖卡

除了簡單的影像或影片附件以外，您也可以附加**主圖卡**，這可讓您將影像和按鈕結合在單一物件中，並將其傳送給使用者。

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

若要撰寫含有主圖卡和按鈕的訊息，您可以將 `HeroCard` 附加至訊息。 這裡顯示的原始程式碼是以[處理附件](https://aka.ms/bot-attachments-sample-code)範例為基礎。 

```csharp
using Microsoft.Bot.Builder;
using Microsoft.Bot.Builder.Core.Extensions;
using Microsoft.Bot.Schema;

var reply = turnContext.Activity.CreateReply();

// Create a HeroCard with options for the user to choose to interact with the bot.
var card = new HeroCard
{
    Text = "You can upload an image or select one of the following choices",
    Buttons = new List<CardAction>()
    {
        new CardAction(ActionTypes.ImBack, title: "1. Inline Attachment", value: "1"),
        new CardAction(ActionTypes.ImBack, title: "2. Internet Attachment", value: "2"),
        new CardAction(ActionTypes.ImBack, title: "3. Uploaded Attachment", value: "3"),
    },
};

// Add the card to our reply.
reply.Attachments = new List<Attachment>() { card.ToAttachment() };

await turnContext.SendActivityAsync(reply, cancellationToken);
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

若要撰寫含有主圖卡和按鈕的訊息，您可以將 `HeroCard` 附加至訊息。 這裡顯示的原始程式碼是以 [JS 處理附件](https://aka.ms/bot-attachments-sample-code-js)範例為基礎：

```javascript
const { ActionTypes, ActivityTypes, CardFactory } = require('botbuilder');
// build buttons to display.
const buttons = [
            { type: ActionTypes.ImBack, title: '1. Inline Attachment', value: '1' },
            { type: ActionTypes.ImBack, title: '2. Internet Attachment', value: '2' },
            { type: ActionTypes.ImBack, title: '3. Uploaded Attachment', value: '3' }
];

// construct hero card.
const card = CardFactory.heroCard('', undefined,
buttons, { text: 'You can upload an image or select one of the following choices.' });

// add card to Activity.
const reply = { type: ActivityTypes.Message };
reply.attachments = [card];

// Send hero card to the user.
await turnContext.sendActivity(reply);
```

---

## <a name="process-events-within-rich-cards"></a>處理複合式資訊卡 (Rich Card) 內的事件

若要處理複合式資訊卡 (Rich Card) 內的事件，請使用_卡片動作_物件，指定在使用者按一下按鈕或點選卡片的某區段時，所應發生的情況。 每個卡片動作都有「類型」和「值」。

為達到正常運作，請為卡片上每個可點選的項目指派一個動作類型。 下表列出和說明可用的動作類型，以及相關聯的值屬性應有內容。

| 類型 | 說明 | 值 |
| :---- | :---- | :---- |
| openUrl | 在內建瀏覽器中開啟 URL。 | 要開啟的 URL。 |
| imBack | 將訊息傳送到 Bot，並在聊天中張貼可見的回應。 | 要傳送之訊息的文字。 |
| postBack | 將訊息傳送到 Bot，但無法在聊天中張貼可見的回應。 | 要傳送之訊息的文字。 |
| call | 開始撥打電話。 | 以下列格式撥打電話的目的地：`tel:123123123123`。 |
| playAudio | 播放音訊。 | 要播放的音訊 URL。 |
| playVideo | 播放影片。 | 要播放的影片 URL。 |
| showImage | 顯示影像。 | 要顯示的影像 URL。 |
| downloadFile | 下載檔案。 | 要下載的檔案 URL。 |
| signin | 起始 OAuth 登入程序。 | 要起始的 OAuth 流程 URL。 |

## <a name="hero-card-using-various-event-types"></a>使用各種不同事件類型的主圖卡

下列程式碼說明使用各種複合式資訊卡 (Rich Card) 事件的範例。

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

```csharp
using Microsoft.Bot.Builder;
using Microsoft.Bot.Builder.Core.Extensions;
using Microsoft.Bot.Schema;

var reply = turnContext.Activity.CreateReply();

var card = new HeroCard
{
    Buttons = new List<CardAction>()
    {
        new CardAction(title: "Much Quieter", type: ActionTypes.PostBack, value: "Shh! My Bot friend hears me."),
        new CardAction(ActionTypes.OpenUrl, title: "Azure Bot Service", value: "https://azure.microsoft.com/en-us/services/bot-service/"),
    },
};

```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

```javascript
const {ActionTypes} = require("botbuilder");

const hero = MessageFactory.attachment(
    CardFactory.heroCard(
        'Holler Back Buttons',
        ['https://example.com/whiteShirt.jpg'],
        [{
            type: ActionTypes.ImBack,
            title: 'ImBack',
            value: 'You can ALL hear me! Shout Out Loud'
        },
        {
            type: ActionTypes.PostBack,
            title: 'PostBack',
            value: 'Shh! My Bot friend hears me. Much Quieter'
        },
        {
            type: ActionTypes.OpenUrl,
            title: 'OpenUrl',
            value: 'https://en.wikipedia.org/wiki/{cardContent.Key}'
        }]
    )
);

await context.sendActivity(hero);

```

---

## <a name="send-an-adaptive-card"></a>傳送調適型卡片
調適型卡片和 MessageFactory 都用於傳送豐富訊息 (包括文字、影像、視訊、音訊和檔案) 來與使用者進行通訊。 但是，兩者之間有一些差異。 

首先，只有某些通道支援調適型卡片，而提供支援的通道可能部分支援調適型卡片。 例如，如果您在 Facebook 中傳送調適型卡片，當文字和影像正常運作時，按鈕將無法運作。 MessageFactory 只是 Bot Framework SDK 內的協助程式類別，可為您將建立步驟自動化，而且大部分通道都提供支援。 

再者，調適型卡片會以卡片格式傳遞訊息，而通道會決定卡片的版面配置。 MessageFactory 傳遞的訊息格式取決於通道，而且不一定是以卡片格式傳遞，除非調適型卡片是附件的一部分。 

如需調適型卡片通道支援的最新資訊，請參閱<a href="http://adaptivecards.io/visualizer/">調適型卡片視覺化檢視</a>。

若要使用調適型卡片，請務必新增 `Microsoft.AdaptiveCards`NuGet 套件。 


> [!NOTE]
> 您應對 Bot 所將使用的通道測試這項功能，以確認這些通道是否支援調適型卡片。

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

這裡顯示的原始程式碼是以[使用調適型卡片](https://aka.ms/bot-adaptive-cards-sample-code)範例為基礎：

```csharp
using AdaptiveCards;
using Microsoft.Bot.Builder;
using Microsoft.Bot.Builder.Core.Extensions;
using Microsoft.Bot.Schema;

// Creates an attachment that contains an adaptive card
// filePath is the path to JSON file
private static Attachment CreateAdaptiveCardAttachment(string filePath)
{
    var adaptiveCardJson = File.ReadAllText(filePath);
    var adaptiveCardAttachment = new Attachment()
    {
        ContentType = "application/vnd.microsoft.card.adaptive",
        Content = JsonConvert.DeserializeObject(adaptiveCardJson),
    };
    return adaptiveCardAttachment;
}

// Create adaptive card and attach it to the message 
var cardAttachment = CreateAdaptiveCardAttachment(adaptiveCardJsonFilePath);
var reply = turnContext.Activity.CreateReply();
reply.Attachments = new List<Attachment>() { cardAttachment };

await turnContext.SendActivityAsync(reply, cancellationToken);
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

這裡顯示的原始程式碼是以 [JS 使用調適型卡片](https://aka.ms/bot-adaptive-cards-js-sample-code)範例為基礎：

```javascript
const { BotFrameworkAdapter } = require('botbuilder');

// Import AdaptiveCard content.
const FlightItineraryCard = require('./resources/FlightItineraryCard.json');
const ImageGalleryCard = require('./resources/ImageGalleryCard.json');
const LargeWeatherCard = require('./resources/LargeWeatherCard.json');
const RestaurantCard = require('./resources/RestaurantCard.json');
const SolitaireCard = require('./resources/SolitaireCard.json');

// Create array of AdaptiveCard content, this will be used to send a random card to the user.
const CARDS = [
    FlightItineraryCard,
    ImageGalleryCard,
    LargeWeatherCard,
    RestaurantCard,
    SolitaireCard
];
// Select a random card to send.
const randomlySelectedCard = CARDS[Math.floor((Math.random() * CARDS.length - 1) + 1)];
// Send adaptive card.
await context.sendActivity({
      text: 'Here is an Adaptive Card:',
       attachments: [CardFactory.adaptiveCard(randomlySelectedCard)]
});
```

---

## <a name="send-a-carousel-of-cards"></a>傳送浮動切換的卡片

訊息也可以在浮動切換配置中包含多個附件，此時附件會並排顯示，讓使用者可加以捲動。

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

```csharp
using Microsoft.Bot.Builder;
using Microsoft.Bot.Builder.Core.Extensions;
using Microsoft.Bot.Schema;

// Create the activity and attach a set of Hero cards.
var activity = MessageFactory.Carousel(
    new Attachment[]
    {
        new HeroCard(
            title: "title1",
            images: new CardImage[] { new CardImage(url: "imageUrl1.png") },
            buttons: new CardAction[]
            {
                new CardAction(title: "button1", type: ActionTypes.ImBack, value: "item1")
            })
        .ToAttachment(),
        new HeroCard(
            title: "title2",
            images: new CardImage[] { new CardImage(url: "imageUrl2.png") },
            buttons: new CardAction[]
            {
                new CardAction(title: "button2", type: ActionTypes.ImBack, value: "item2")
            })
        .ToAttachment(),
        new HeroCard(
            title: "title3",
            images: new CardImage[] { new CardImage(url: "imageUrl3.png") },
            buttons: new CardAction[]
            {
                new CardAction(title: "button3", type: ActionTypes.ImBack, value: "item3")
            })
        .ToAttachment()
    });

// Send the activity as a reply to the user.
await context.SendActivity(activity);
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

```javascript
// require MessageFactory and CardFactory from botbuilder.
const {MessageFactory, CardFactory} = require('botbuilder');

//  init message object
let messageWithCarouselOfCards = MessageFactory.carousel([
    CardFactory.heroCard('title1', ['imageUrl1'], ['button1']),
    CardFactory.heroCard('title2', ['imageUrl2'], ['button2']),
    CardFactory.heroCard('title3', ['imageUrl3'], ['button3'])
]);

await context.sendActivity(messageWithCarouselOfCards);
```

---

<!-- TODO: Add a media card, such as video or audion. Revisit which examples we put here and link to the 06 through 08 samples. -->

## <a name="additional-resources"></a>其他資源

如需卡片結構描述的詳細資訊，請參閱 [Bot Framework 卡片結構描述](https://aka.ms/botSpecs-cardSchema)。

您可以在此找到卡片的範例程式碼：[C#](https://aka.ms/bot-cards-sample-code)/[JS](https://aka.ms/bot-cards-js-sample-code)、調適型卡片：[C#](https://aka.ms/bot-adaptive-cards-sample-code)/[JS](https://aka.ms/bot-adaptive-cards-js-sample-code)、附件：[C#](https://aka.ms/bot-attachments-sample-code)/[JS](https://aka.ms/bot-attachments-sample-code-js)，以及建議的動作：[C#](https://aka.ms/SuggestedActionsCSharp)/[JS](https://aka.ms/SuggestedActionsJS)。
如需其他範例，請參考 [GitHub](https://aka.ms/bot-samples-readme) 上的 Bot Builder 範例存放庫。
