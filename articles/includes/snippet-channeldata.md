---
ms.openlocfilehash: c19287b38a2c807e6675af2c3f7e1824eb7eab8e
ms.sourcegitcommit: fa6e775dcf95a4253ad854796f5906f33af05a42
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/16/2019
ms.locfileid: "68230774"
---
某些通道所提供的功能，無法只透過訊息文字和附件來實作。 若要實作通道特定的功能，您可以將原生中繼資料傳遞至活動物件「通道資料」  屬性中的通道。 例如，Bot 可以使用通道資料屬性，指示 Telegram 傳送貼圖或指示 Office365 傳送電子郵件。

此文章說明如何使用訊息活動的通道資料屬性，來實作此通道特定的功能：

| 通道 | 功能 |
|----|----|
| 電子郵件 | 傳送及接收電子郵件，其中包含本文、主旨和重要性中繼資料 |
| Slack | 傳送不失真的 Slack 訊息 |
| Facebook | 原生傳送 Facebook 通知 |
| Telegram | 執行 Telegram 特定動作，例如共用語音備忘或貼圖 |
| Kik | 傳送和接收原生 Kik 訊息 |

> [!NOTE]
> 活動物件的通道資料屬性值是一個 JSON 物件。
> 因此，本文中的範例會說明各種案例中 `channelData` JSON 屬性的預期格式。
> 若要使用 .NET 建立 JSON 物件，請使用 `JObject` (.NET) 類別。

## <a name="create-a-custom-email-message"></a>建立自訂的電子郵件訊息

若要建立電子郵件訊息，請將活動物件的通道資料屬性設定為包含這些屬性的 JSON 物件：

| 屬性 | 說明 |
|----|----|
| bccRecipients | 以分號 (;) 分隔的電子郵件地址字串，用來新增至訊息的 [Bcc]\(密件副本\)欄位。 |
| ccRecipients | 以分號 (;) 分隔的電子郵件地址字串，用來新增至訊息的 [Cc]\(副本\)欄位。 |
| htmlBody | 會指定電子郵件訊息本文的 HTML 文件。 請參閱通道的文件，以取得受支援 HTML 元素和屬性的資訊。 |
| importance | 電子郵件的重要性層級。 有效值為**高**、**一般**和**低**。 預設值為**一般**。 |
| subject | 電子郵件的主旨。請參閱通道的文件，以取得欄位需求的資訊。 |
| toRecipients | 以分號 (;) 分隔的電子郵件地址字串，用來新增至訊息的 [收件者] 欄位。 |

> [!NOTE]
> Bot 透過電子郵件通道從使用者收到的訊息，可能會包含通道資料屬性，且其中已填入類似上述的 JSON 物件。

此程式碼片段顯示自訂電子郵件訊息的 `channelData` 屬性範例。

```json
"channelData": {
    "type": "message",
    "locale": "en-Us",
    "channelID": "email",
    "from": { "id": "mybot@mydomain.com", "name": "My bot"},
    "recipient": { "id": "joe@otherdomain.com", "name": "Joe Doe"},
    "conversation": { "id": "123123123123", "topic": "awesome chat" },
    "channelData":
    {
        "htmlBody": "<html><body style = /"font-family: Calibri; font-size: 11pt;/" >This is more than awesome.</body></html>",
        "subject": "Super awesome message subject",
        "importance": "high",
        "ccRecipients": "Yasemin@adatum.com;Temel@adventure-works.com"
    }
}
```

## <a name="create-a-full-fidelity-slack-message"></a>建立不失真的 Slack 訊息

若要建立不失真的 Slack 訊息，請將活動物件的通道資料屬性設定為 JSON 物件，以指定 <a href="https://api.slack.com/docs/messages" target="_blank">Slack 訊息</a>、<a href="https://api.slack.com/docs/message-attachments" target="_blank">Slack 附件</a>和/或 <a href="https://api.slack.com/docs/message-buttons" target="_blank">Slack 按鈕</a>。

> [!NOTE]
> 若要在 Slack 訊息中支援按鈕，您必須在[將 Bot 連線](../bot-service-manage-channels.md)至 Slack 通道時啟用 [互動式訊息]  。

此程式碼片段顯示自訂 Slack 訊息的 `channelData` 屬性範例。

```json
"channelData": {
   "text": "Now back in stock! :tada:",
   "attachments": [
        {
            "title": "The Further Adventures of Slackbot",
            "author_name": "Stanford S. Strickland",
            "author_icon": "https://api.slack.com/img/api/homepage_custom_integrations-2x.png",
            "image_url": "http://i.imgur.com/OJkaVOI.jpg?1"
        },
        {
            "fields": [
                {
                    "title": "Volume",
                    "value": "1",
                    "short": true
                },
                {
                    "title": "Issue",
                    "value": "3",
                    "short": true
                }
            ]
        },
        {
            "title": "Synopsis",
            "text": "After @episod pushed exciting changes to a devious new branch back in Issue 1, Slackbot notifies @don about an unexpected deploy..."
        },
        {
            "fallback": "Would you recommend it to customers?",
            "title": "Would you recommend it to customers?",
            "callback_id": "comic_1234_xyz",
            "color": "#3AA3E3",
            "attachment_type": "default",
            "actions": [
                {
                    "name": "recommend",
                    "text": "Recommend",
                    "type": "button",
                    "value": "recommend"
                },
                {
                    "name": "no",
                    "text": "No",
                    "type": "button",
                    "value": "bad"
                }
            ]
        }
    ]
}
```

當使用者按一下 Slack 訊息中的按鈕時，Bot 會收到回應訊息，其中的通道資料屬性已填入 `payload` JSON 物件。
`payload` 物件會指定原始訊息的內容、識別已按下的按鈕，並識別按下按鈕的使用者。

此程式碼片段顯示當使用者按一下 Slack 訊息中的按鈕時，Bot 所收到的訊息中所含有的 `channelData` 屬性範例。

```json
"channelData": {
    "payload": {
        "actions": [
            {
                "name": "recommend",
                "value": "yes"
            }
        ],
        . . .
        "original_message": "{…}",
        "response_url": "https://hooks.slack.com/actions/..."
    }
}
```

Bot 可以透過正常方式回覆此訊息，也可以將其回應直接張貼到 `payload` 物件的 `response_url` 屬性所指定的端點。
如需何時及如何將回應張貼到 `response_url` 的資訊，請參閱 <a href="https://api.slack.com/docs/message-buttons" target="_blank">Slack 按鈕</a>。

您可以使用下列 JSON 來建立動態按鈕。

```json
{
    "text": "Would you like to play a game ? ",
    "attachments": [
        {
            "text": "Choose a game to play!",
            "fallback": "You are unable to choose a game",
            "callback_id": "wopr_game",
            "color": "#3AA3E3",
            "attachment_type": "default",
            "actions": [
                {
                    "name": "game",
                    "text": "Chess",
                    "type": "button",
                    "value": "chess"
                },
                {
                    "name": "game",
                    "text": "Falken's Maze",
                    "type": "button",
                    "value": "maze"
                },
                {
                    "name": "game",
                    "text": "Thermonuclear War",
                    "style": "danger",
                    "type": "button",
                    "value": "war",
                    "confirm": {
                        "title": "Are you sure?",
                        "text": "Wouldn't you prefer a good game of chess?",
                        "ok_text": "Yes",
                        "dismiss_text": "No"
                    }
                }
            ]
        }
    ]
}
```

若要建立互動式功能表，請使用下列 JSON：

```json
{
    "text": "Would you like to play a game ? ",
    "response_type": "in_channel",
    "attachments": [
        {
            "text": "Choose a game to play",
            "fallback": "If you could read this message, you'd be choosing something fun to do right now.",
            "color": "#3AA3E3",
            "attachment_type": "default",
            "callback_id": "game_selection",
            "actions": [
                {
                    "name": "games_list",
                    "text": "Pick a game...",
                    "type": "select",
                    "options": [
                        {
                            "text": "Hearts",
                            "value": "menu_id_hearts"
                        },
                        {
                            "text": "Bridge",
                            "value": "menu_id_bridge"
                        },
                        {
                            "text": "Checkers",
                            "value": "menu_id_checkers"
                        },
                        {
                            "text": "Chess",
                            "value": "menu_id_chess"
                        },
                        {
                            "text": "Poker",
                            "value": "menu_id_poker"
                        },
                        {
                            "text": "Falken's Maze",
                            "value": "menu_id_maze"
                        },
                        {
                            "text": "Global Thermonuclear War",
                            "value": "menu_id_war"
                        }
                    ]
                }
            ]
        }
    ]
}
```

## <a name="create-a-facebook-notification"></a>建立 Facebook 通知

若要建立 Facebook 通知，請將活動物件的通道資料屬性設定為指定這些屬性的 JSON 物件：

| 屬性 | 說明 |
|----|----|
| notification_type | 通知類型 (例如 **REGULAR**、**SILENT_PUSH**、**NO_PUSH**)。
| attachment | 指定影像、視訊或其他多媒體類型的附件，或是回條等樣板化附件。 |

> [!NOTE]
> 如需 `notification_type` 屬性和 `attachment` 屬性的格式和內容詳細資料，請參閱 <a href="https://developers.facebook.com/docs/messenger-platform/send-api-reference#guidelines" target="_blank">Facebook API 文件</a>。 

此程式碼片段顯示 Facebook 回條附件的 `channelData` 屬性範例。

```json
"channelData": {
    "notification_type": "NO_PUSH",
    "attachment": {
        "type": "template"
        "payload": {
            "template_type": "receipt",
            . . .
        }
    }
}
```

## <a name="create-a-telegram-message"></a>建立 Telegram 訊息

若要建立訊息來實作 Telegram 特定動作 (例如，共用語音備忘或貼圖)，請將活動物件的通道資料屬性設定為會指定這些屬性的 JSON 物件： 

| 屬性 | 說明 |
|----|----|
| method | 要呼叫的 Telegram Bot API 方法。 |
| parameters | 所指定方法的參數。 |

支援下列 Telegram 方法： 

- answerInlineQuery
- editMessageCaption
- editMessageReplyMarkup
- editMessageText
- forwardMessage
- kickChatMember
- sendAudio
- sendChatAction
- sendContact
- sendDocument
- sendLocation
- sendMessage
- sendPhoto
- sendSticker
- sendVenue
- sendVideo
- sendVoice
- unbanChateMember

如需這些 Telegram 方法和其參數的詳細資訊，請參閱 <a href="https://core.telegram.org/bots/api#available-methods" target="_blank">Telegram Bot API 文件</a>。

> [!NOTE]
> <ul><li><code>chat_id</code> 參數通用於所有 Telegram 方法。 如果您未指定 <code>chat_id</code> 作為參數，架構會為您提供識別碼。</li>
> <li>不要傳遞內嵌檔案內容，而是應該指定使用 URL 和媒體類型的檔案，如下列範例所示。</li>
> <li>在 Bot 從 Telegram 通道所收到的每則訊息內，<code>ChannelData</code> 屬性會包含 Bot 先前傳送的訊息。</li></ul>

此程式碼片段顯示 `channelData` 屬性範例，此屬性指定單一 Telegram 方法。

```json
"channelData": {
    "method": "sendSticker",
    "parameters": {
        "sticker": {
            "url": "https://domain.com/path/gif",
            "mediaType": "image/gif",
        }
    }
}
```

此程式碼片段顯示 `channelData` 屬性範例，此屬性指定 Telegram 方法陣列。

```json
"channelData": [
    {
        "method": "sendSticker",
        "parameters": {
            "sticker": {
                "url": "https://domain.com/path/gif",
                "mediaType": "image/gif",
            }
        }
    },
    {
        "method": "sendMessage",
        "parameters": {
            "text": "<b>This message is HTML formatted.</b>",
            "parse_mode": "HTML"
        }
    }
]
```

## <a name="create-a-native-kik-message"></a>建立原生的 Kik 訊息

若要建立原生的 Kik 訊息，請將活動物件的通道資料屬性設定為指定此屬性的 JSON 物件：

| 屬性 | 說明 |
|----|----|
| messages | Kik 訊息的陣列。如需 Kik 訊息格式的詳細資訊，請參閱 <a href="https://dev.kik.com/#/docs/messaging#message-formats" target="_blank">Kik 訊息格式</a>。 |


此程式碼片段顯示原生 Kik 訊息的 `channelData` 屬性範例。

```json
"channelData": {
    "messages": [
        {
            "chatId": "c6dd8165…",
            "type": "link",
            "to": "kikhandle",
            "title": "My Webpage",
            "text": "Some text to display",
            "url": "http://botframework.com",
            "picUrl": "http://lorempixel.com/400/200/",
            "attribution": {
                "name": "My App",
                "iconUrl": "http://lorempixel.com/50/50/"
            },
            "noForward": true,
            "kikJsData": {
                    "key": "value"
                }
        }
    ]
}
```

## <a name="create-a-line-message"></a>建立 LINE 訊息

若要建立會實作 LINE 特有訊息類型 (例如貼圖、範本或 LINE 特有動作類型，像是開啟手機相機) 的訊息，請將活動物件的通道資料屬性設定為會指定 LINE 訊息類型和動作類型的 JSON 物件。 

| 屬性 | 說明 |
|----|----|
| type | LINE 動作/訊息類型名稱 |

支援這些 LINE 訊息類型：
* 貼圖
* 影像地圖 
* 範本 (按鈕、確認、橫向捲軸) 
* Flex 

您可以在訊息類型 JSON 物件的動作欄位中，指定這些 LINE 動作： 
* Postback 
* 訊息 
* URI 
* Datetimerpicker 
* Camera 
* Camera roll 
* Location 

如需這些 LINE 方法和其參數的詳細資訊，請參閱 [LINE Bot API 文件](https://developers.line.biz/en/docs/messaging-api/)。 

此程式碼片段會舉例示範 `channelData` 屬性，此屬性會指定通道訊息類型 `ButtonTemplate` 和 3 個動作類型：camera、cameraRoll、Datetimepicker。 

```json
"channelData": { 
    "type": "ButtonsTemplate", 
    "altText": "This is a buttons template", 
    "template": { 
        "type": "buttons", 
        "thumbnailImageUrl": "https://example.com/bot/images/image.jpg", 
        "imageAspectRatio": "rectangle", 
        "imageSize": "cover", 
        "imageBackgroundColor": "#FFFFFF", 
        "title": "Menu", 
        "text": "Please select", 
        "defaultAction": { 
            "type": "uri", 
            "label": "View detail", 
            "uri": "http://example.com/page/123" 
        }, 
        "actions": [{ 
                "type": "cameraRoll", 
                "label": "Camera roll" 
            }, 
            { 
                "type": "camera", 
                "label": "Camera" 
            }, 
            { 
                "type": "datetimepicker", 
                "label": "Select date", 
                "data": "storeId=12345", 
                "mode": "datetime", 
                "initial": "2017-12-25t00:00", 
                "max": "2018-01-24t23:59", 
                "min": "2017-12-25t00:00" 
            } 
        ] 
    } 
}
```

## <a name="additional-resources"></a>其他資源

- [實體和活動類型](../bot-service-activities-entities.md)
- [Bot Framework -- 活動結構描述](https://aka.ms/botSpecs-activitySchema)
