---
title: 將複合式資訊卡附件新增至訊息 | Microsoft Docs
description: 了解如何使用 Bot Connector 服務，將複合式資訊卡新增至訊息。
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 12/13/2017
ms.openlocfilehash: 10246fda94932feb96e5faa0cdd8ca489c98c855
ms.sourcegitcommit: c200cc2db62dbb46c2a089fb76017cc55bdf26b0
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/27/2019
ms.locfileid: "70037462"
---
# <a name="add-rich-card-attachments-to-messages"></a>將複合式資訊卡 (Rich Card) 附件新增至訊息
> [!div class="op_single_selector"]
> - [.NET](../dotnet/bot-builder-dotnet-add-rich-card-attachments.md)
> - [Node.js](../nodejs/bot-builder-nodejs-send-rich-cards.md)
> - [REST](../rest-api/bot-framework-rest-connector-add-rich-cards.md)

Bot 和頻道通常會交換文字字串，但某些頻道也支援交換附件，讓您的 Bot 可將更豐富的訊息傳送給使用者。 例如，您的 Bot 可以傳送複合式資訊卡和媒體附件 (例如，影像、視訊、音訊、檔案)。 本文說明如何使用 Bot Connector 服務，將複合式資訊卡附件新增至訊息。

> [!NOTE]
> 如需如何將媒體附件新增至訊息的相關資訊，請參閱[將媒體附件新增至訊息](bot-framework-rest-connector-add-media-attachments.md)。

## <a id="types-of-cards"></a> 複合式資訊卡的類型

複合式資訊卡包含標題、描述、連結和影像。 訊息可包含多個複合式資訊卡，並以清單格式或浮動切換格式顯示。
Bot Framework 目前支援八種類型的複合式資訊卡： 

| 卡片類型 | 說明 |
|----|----|
| <a href="/adaptive-cards/get-started/bots">AdaptiveCard</a> | 可自訂的卡片，可包含文字、語音、影像、按鈕和輸入欄位的任意組合。 請參閱[個別頻道支援](/adaptive-cards/get-started/bots#channel-status)。  |
| [AnimationCard][] | 可以播放動畫 GIF 或短片的卡片。 |
| [AudioCard][] | 可播放音訊檔案的卡片。 |
| [HeroCard][] | 通常包含單一大型影像、一或多個按鈕和文字的卡片。 |
| [ThumbnailCard][] | 通常包含單一縮圖影像、一或多個按鈕和文字的卡片。 |
| [ReceiptCard][] | 讓 Bot 向使用者提供收據的卡片。 通常包含收據上的項目清單、稅金和總金額資訊，以及其他文字。 |
| [SignInCard][] | 可讓 Bot 要求使用者登入的卡片。 通常包含文字，以及一或多個使用者可以按一下以起始登入程序的按鈕。 |
| [VideoCard][] | 可播放視訊的卡片。 |

> [!TIP]
> 若要確認某個通道所支援的複合式資訊卡類型，以及該通道轉譯複合式資訊卡 (Rich Card) 的方式，請參閱[通道偵測器][ChannelInspector]。 如需卡片內容的限制 (例如，按鈕數目上限或標題長度上限) 的相關資訊，請參閱頻道的文件。

## <a name="process-events-within-rich-cards"></a>處理複合式資訊卡 (Rich Card) 內的事件

若要處理複合式資訊卡內的事件，請使用 [CardAction][] 物件，指定在使用者按一下按鈕或點選卡片的某區段時應發生的情況。 每個 `CardAction` 物件都包含下列屬性：

| 屬性 | 類型 | 說明 | 
|----|----|----|
| type | 字串 | 動作的類型 (下表中指定的其中一個值) |
| title | 字串 | 按鈕的標題 |
| image | 字串 | 按鈕的影像 URL |
| value | 字串 | 執行指定動作類型所需的值 |

> [!NOTE]
> 調適型卡片內的按鈕不會使用 `CardAction` 物件來建立，而會改用<a href="http://adaptivecards.io" target="_blank">調適型卡片</a>所定義的結構描述來建立。 如需示範如何將按鈕新增至調適型卡片的範例，請參閱[將調適型卡片新增至訊息](#adaptive-card)。

下表列出 `CardAction` 物件的 `type` 屬性適用的有效值，並說明每個類型的 `value` 屬性應有的內容：

| type | value | 
|----|----|
| openUrl | 要在內建瀏覽器中開啟的 URL |
| imBack | 要傳送至 Bot 的訊息文字 (來自於按一下按鈕或點選卡片的使用者)。 此訊息 (從使用者到 Bot) 會透過裝載對話的用戶端應用程式顯示給所有對話參與者。 |
| postBack | 要傳送至 Bot 的訊息文字 (來自於按一下按鈕或點選卡片的使用者)。 某些用戶端應用程式可能會在訊息摘要中顯示此文字，讓所有對話參與者都能看見。 |
| call | 以下列格式撥打電話的目的地：**tel:123123123123** |
| playAudio | 待播放音訊的 URL |
| playVideo | 待播放視訊的 URL |
| showImage | 待顯示影像的 URL |
| downloadFile | 待下載檔案的 URL |
| signin | 待起始 OAuth 流程的 URL |

## <a name="add-a-hero-card-to-a-message"></a>將主圖卡新增至訊息

若要將複合式資訊卡附件新增至訊息，請先建立對應至您想要新增至訊息之[卡片類型](#types-of-cards)的物件。 接著建立 [Attachment][] 物件、將其 `contentType` 屬性設定為卡片的媒體類型，並將其 `content` 屬性設定為您建立用來代表卡片的物件。 在訊息的 `attachments` 陣列內，指定您的 `Attachment` 物件。

> [!TIP]
> 包含複合式資訊卡附件的訊息通常不會指定 `text`。

某些頻道可讓您將多張複合式資訊卡新增至訊息內的 `attachments` 陣列。 此功能在您想要為使用者提供多個選項的情況下很有用。 例如，如果您的 Bot 讓使用者能夠預約飯店房間，即可為使用者呈現一份複合式資訊卡清單，以顯示各種類型的可用房間。 每張卡片都會包含一張圖片，以及與房間類型相對應的便利設施清單，而使用者可以藉由點選卡片或按一下按鈕來選取房間類型。

> [!TIP]
> 若要以清單格式顯示多張複合式資訊卡，請將 [Activity][] 物件的 `attachmentLayout` 屬性設定為 "list"。 若要以浮動切換格式顯示多張複合式資訊卡，請將 `Activity` 物件的 `attachmentLayout` 屬性設定為 "carousel"。 如果通道不支援浮動切換格式，則會以清單格式顯示複合式資訊卡 (Rich Card)，即使 `attachmentLayout` 屬性指定 [浮動切換] 亦然。

下列範例所顯示的要求會傳送包含單一主圖卡附件的訊息。 在此範例要求中，`https://smba.trafficmanager.net/apis` 表示基底 URI，您的 Bot 所發出之要求的基底 URI 可能和這個不同。 如需設定基底 URI 的詳細資訊，請參閱 [API 參考](bot-framework-rest-connector-api-reference.md#base-uri)。

```http
POST https://smba.trafficmanager.net/apis/v3/conversations/abcd1234/activities/5d5cdc723 
Authorization: Bearer ACCESS_TOKEN
Content-Type: application/json
```

```json
{
    "type": "message",
    "from": {
        "id": "12345678",
        "name": "sender's name"
    },
    "conversation": {
        "id": "abcd1234",
        "name": "conversation's name"
    },
    "recipient": {
        "id": "1234abcd",
        "name": "recipient's name"
    },
    "attachments": [
        {
            "contentType": "application/vnd.microsoft.card.hero",
            "content": {
                "title": "title goes here",
                "subtitle": "subtitle goes here",
                "text": "descriptive text goes here",
                "images": [
                    {
                        "url": "https://aka.ms/DuckOnARock",
                        "alt": "picture of a duck",
                        "tap": {
                            "type": "playAudio",
                            "value": "url to an audio track of a duck call goes here"
                        }
                    }
                ],
                "buttons": [
                    {
                        "type": "playAudio",
                        "title": "Duck Call",
                        "value": "url to an audio track of a duck call goes here"
                    },
                    {
                        "type": "openUrl",
                        "title": "Watch Video",
                        "image": "https://aka.ms/DuckOnARock",
                        "value": "url goes here of the duck in flight"
                    }
                ]
            }
        }
    ],
    "replyToId": "5d5cdc723"
}
```

## <a id="adaptive-card"></a> 將調適型卡片新增至訊息

調適型卡片可包含文字、語音、影像、按鈕和輸入欄位的任意組合。 調適型卡片會使用<a href="http://adaptivecards.io" target="_blank">調適型卡片</a>中指定的 JSON 格式來建立，讓您能夠完整控制卡片的內容和格式。 

請利用<a href="http://adaptivecards.io" target="_blank">調適型卡片</a>網站中的資訊，來了解調適型卡片的結構描述、探索調適型卡片的元素，並查看可用以建立具有不同組合和複雜度之卡片的 JSON 範例。 此外，您可以使用互動式視覺化檢視，來設計調適型卡片承載及預覽卡片輸出。

下列範例所顯示的要求會傳送包含單一調適型卡片作為行事曆提醒的訊息。 在此範例要求中，`https://smba.trafficmanager.net/apis` 表示基底 URI，您的 Bot 所發出之要求的基底 URI 可能和這個不同。 如需設定基底 URI 的詳細資訊，請參閱 [API 參考](bot-framework-rest-connector-api-reference.md#base-uri)。

```http
POST https://smba.trafficmanager.net/apis/v3/conversations/abcd1234/activities/5d5cdc723 
Authorization: Bearer ACCESS_TOKEN
Content-Type: application/json
```

```json
{
    "type": "message",
    "from": {
        "id": "12345678",
        "name": "sender's name"
    },
    "conversation": {
        "id": "abcd1234",
        "name": "conversation's name"
    },
    "recipient": {
        "id": "1234abcd",
        "name": "recipient's name"
    },
    "attachments": [
        {
            "contentType": "application/vnd.microsoft.card.adaptive",
            "content": {
                "type": "AdaptiveCard",
                "body": [
                    {
                        "type": "TextBlock",
                        "text": "Adaptive Card design session",
                        "size": "large",
                        "weight": "bolder"
                    },
                    {
                        "type": "TextBlock",
                        "text": "Conf Room 112/3377 (10)"
                    },
                    {
                        "type": "TextBlock",
                        "text": "12:30 PM - 1:30 PM"
                    },
                    {
                        "type": "TextBlock",
                        "text": "Snooze for"
                    },
                    {
                        "type": "Input.ChoiceSet",
                        "id": "snooze",
                        "style": "compact",
                        "choices": [
                            {
                                "title": "5 minutes",
                                "value": "5",
                                "isSelected": true
                            },
                            {
                                "title": "15 minutes",
                                "value": "15"
                            },
                            {
                                "title": "30 minutes",
                                "value": "30"
                            }
                        ]
                    }
                ],
                "actions": [
                    {
                        "type": "Action.Http",
                        "method": "POST",
                        "url": "http://foo.com",
                        "title": "Snooze"
                    },
                    {
                        "type": "Action.Http",
                        "method": "POST",
                        "url": "http://foo.com",
                        "title": "I'll be late"
                    },
                    {
                        "type": "Action.Http",
                        "method": "POST",
                        "url": "http://foo.com",
                        "title": "Dismiss"
                    }
                ]
            }
        }
    ],
    "replyToId": "5d5cdc723"
}
```

產生的卡片會包含三個文字區塊、一個輸入欄位 (選擇清單) 和三個按鈕：

![調適型卡片行事曆提醒](../media/adaptive-card-reminder.png)


## <a name="additional-resources"></a>其他資源

- [建立訊息](bot-framework-rest-connector-create-messages.md)
- [傳送及接收訊息](bot-framework-rest-connector-send-and-receive-messages.md)
- [將媒體附件新增至訊息](bot-framework-rest-connector-add-media-attachments.md)
- [Bot Framework -- 活動結構描述](https://aka.ms/botSpecs-activitySchema)
- [通道偵測器][ChannelInspector]
- <a href="http://adaptivecards.io" target="_blank">調適型卡片</a> \(英文\)

[ChannelInspector]: ../bot-service-channel-inspector.md

[Activity]: bot-framework-rest-connector-api-reference.md#activity-object
[Attachment]: bot-framework-rest-connector-api-reference.md#attachment-object
[CardAction]: bot-framework-rest-connector-api-reference.md#cardaction-object

[AnnimationCard]: bot-framework-rest-connector-api-reference.md#annimationcard-object
[AudioCard]: bot-framework-rest-connector-api-reference.md#audiocard-object
[HeroCard]: bot-framework-rest-connector-api-reference.md#herocard-object
[ThumbnailCard]: bot-framework-rest-connector-api-reference.md#thumbnailcard-object
[ReceiptCard]: bot-framework-rest-connector-api-reference.md#receiptcard-object
[SigninCard]: bot-framework-rest-connector-api-reference.md#signincard-object
[VideoCard]: bot-framework-rest-connector-api-reference.md#videocard-object
