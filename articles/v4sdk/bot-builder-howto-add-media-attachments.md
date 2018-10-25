---
title: 將媒體新增至訊息 | Microsoft Docs
description: 了解如何使用 Bot 建立器 SDK 將媒體新增至訊息。
keywords: 媒體, 訊息, 影像, 音訊, 視訊, 檔案, MessageFactory, 複合式資訊卡, 訊息, 調適型卡片, 主圖卡片, 建議動作
author: ivorb
ms.author: v-ivorb
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 09/10/2018
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 87862e8fcbfa357c27a1c8fac0e8dd71d9bc2998
ms.sourcegitcommit: bd4f9669c0d26ac2a4be1ab8e508f163a1f465f3
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/28/2018
ms.locfileid: "47430327"
---
# <a name="add-media-to-messages"></a>將媒體新增至訊息

[!INCLUDE [pre-release-label](../includes/pre-release-label.md)]

使用者與 Bot 之間交換的訊息可以包含媒體附件，例如影像、視訊、音訊和檔案等。 Bot Builder SDK 支援將豐富訊息傳送給使用者的工作。 若要判斷通道 (Facebook、Skype、Slack 等等) 支援的豐富訊息類型，請參閱通道文件中的限制相關資訊。 請參閱[設計使用者體驗](../bot-service-design-user-experience.md)，以取得一份可用的卡片。 

## <a name="send-attachments"></a>傳送附件

若要傳送影像或視訊之類的使用者內容，您可以將附件或附件清單新增至訊息。

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

`Activity` 物件的 `Attachments` 屬性包含 `Attachment` 物件的陣列，代表附加至訊息的媒體附件和複合式資訊卡 (Rich Card)。 若要將媒體附件新增至訊息，請建立 `message` 活動的 `Attachment` 物件，並設定 `ContentType`、`ContentUrl` 和 `Name` 屬性。 `Activity` 物件的 `Attachments` 屬性包含 `Attachment` 物件的陣列，代表附加至訊息的媒體附件和複合式資訊卡 (Rich Card)。 若要將媒體附件新增至訊息，請使用 `Attachment` 方法來建立 `message` 活動的 `Attachment` 物件，並設定 `ContentType`、`ContentUrl` 和 `Name` 屬性。 這裡顯示的原始程式碼是以[處理附件](https://aka.ms/bot-attachments-sample-code)範例為基礎。 

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
reply.attachments = [card];

// Send hero card to the user.
await turnContext.sendActivity(reply);
```
---

## <a name="process-events-within-rich-cards"></a>處理複合式資訊卡 (Rich Card) 內的事件

若要處理複合式資訊卡 (Rich Card) 內的事件，請使用_卡片動作_物件，指定在使用者按一下按鈕或點選卡片的某區段時，所應發生的情況。

為達到正常運作，請為卡片上每個可點選的項目指派一個動作類型。 下表列出卡片動作物件的類型屬性適用的有效值，並說明每個類型值屬性應有的預期內容。

| 類型 | 值 |
| :---- | :---- |
| openUrl | 要在內建瀏覽器中開啟的 URL。 對於「點選」或「按一下」動作，會以開啟 URL 來回應。 |
| imBack | 要傳送至 Bot 的訊息文字 (來自於按一下按鈕或點選卡片的使用者)。 此訊息 (從使用者到 Bot) 會透過裝載對話的用戶端應用程式顯示給所有對話參與者。 |
| postBack | 要傳送至 Bot 的訊息文字 (來自於按一下按鈕或點選卡片的使用者)。 某些用戶端應用程式可能會在訊息摘要中顯示此文字，讓所有對話參與者都能看見。 |
| call | 以下列格式撥打電話的目的地：`tel:123123123123`。對於「點選」或「按一下」動作，會以起始通話來回應。|
| playAudio | 待播放音訊的 URL。 對於「點選」或「按一下」動作，會以播放音訊來回應。 |
| playVideo | 待播放視訊的 URL。 對於「點選」或「按一下」動作，會以播放視訊來回應。 |
| showImage | 待顯示影像的 URL。 對於「點選」或「按一下」動作，會以顯示影像來回應。 |
| downloadFile | 待下載檔案的 URL。  對於「點選」或「按一下」動作，會以下載檔案來回應。 |
| signin | 待起始 OAuth 流程的 URL。 對於「點選」或「按一下」動作，會以起始登入來回應。 |

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

首先，只有某些通道支援調適型卡片，而提供支援的通道可能部分支援調適型卡片。 例如，如果您在 Facebook 中傳送調適型卡片，當文字和影像正常運作時，按鈕將無法運作。 MessageFactory 只是 Bot 建立器 SDK 內的協助程式類別，可為您將建立步驟自動化，而且大部分通道都提供支援。 

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

## <a name="additional-resources"></a>其他資源
這裡可以找到卡片的範例程式碼：[C#](https://aka.ms/bot-cards-sample-code)/[JS](https://aka.ms/bot-cards-js-sample-code) 調適型卡片：[C#](https://aka.ms/bot-adaptive-cards-sample-code)/[JS](https://aka.ms/bot-adaptive-cards-js-sample-code)，附件：[C#](https://aka.ms/bot-attachments-sample-code)/[JS](https://aka.ms/bot-attachments-sample-code-js)，以及建議的動作：[C#](https://aka.ms/SuggestedActionsCSharp)/[JS](https://aka.ms/SuggestedActionsJS)。 如需其他範例，請參考 [GitHub](https://github.com/Microsoft/BotBuilder-Samples) 上的 Bot Builder 範例存放庫。
