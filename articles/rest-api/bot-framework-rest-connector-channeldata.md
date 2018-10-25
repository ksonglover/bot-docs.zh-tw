---
title: 實作通道特定的功能 | Microsoft Docs
description: 了解如何使用 Bot Connector API 實作通道特定的功能。
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 12/13/2017
ms.openlocfilehash: d69013c721552483cfd38b204936cb1c7f508f82
ms.sourcegitcommit: b78fe3d8dd604c4f7233740658a229e85b8535dd
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/24/2018
ms.locfileid: "49996895"
---
# <a name="implement-channel-specific-functionality"></a>實作通道特定的功能

某些通道所提供的功能，無法只透過[訊息文字和附件](bot-framework-rest-connector-create-messages.md)來實作。 若要實作通道特定的功能，您可以將原生中繼資料傳遞至 [Activity][Activity] 物件之 `channelData` 屬性中的通道。 例如，Bot 可以使用 `channelData` 屬性，指示 Telegram 傳送貼圖或指示 Office365 傳送電子郵件。

此文章說明如何使用訊息活動的 `channelData` 屬性來實作此通道特定的功能：

| 通道 | 功能 |
|----|----|
| 電子郵件 | 傳送及接收電子郵件，其中包含本文、主旨和重要性中繼資料 |
| Slack | 傳送不失真的 Slack 訊息 |
| Facebook | 原生傳送 Facebook 通知 |
| Telegram | 執行 Telegram 特定動作，例如共用語音備忘或貼圖 |
| Kik | 傳送和接收原生 Kik 訊息 | 

> [!NOTE]
> [Activity][Activity] 物件的 `channelData` 屬性值是一個 JSON 物件。 JSON 物件的結構將根據正在實作的通道和功能而變化，如下所述。 

## <a name="create-a-custom-email-message"></a>建立自訂的電子郵件訊息

若要建立電子郵件訊息，請將 [Activity][Activity] 物件的 `channelData` 屬性設定為包含這些屬性的 JSON 物件：

[!INCLUDE [Email channelData table](~/includes/snippet-channelData-email.md)]

此程式碼片段顯示自訂電子郵件訊息的 `channelData` 屬性範例。

```json
"channelData":
{
    "htmlBody": "<html><body style = /"font-family: Calibri; font-size: 11pt;/" >This is more than awesome.</body></html>",
    "subject": "Super awesome message subject",
    "importance": "high",
    "ccRecipients": "Yasemin@adatum.com;Temel@adventure-works.com"
}
```

## <a name="create-a-full-fidelity-slack-message"></a>建立不失真的 Slack 訊息

若要建立不失真的 Slack 訊息，請將 [Activity][Activity] 物件的 `channelData` 屬性設定為 JSON 物件，以指定 <a href="https://api.slack.com/docs/messages" target="_blank">Slack 訊息</a>、<a href="https://api.slack.com/docs/message-attachments" target="_blank">Slack 附件</a>和/或 <a href="https://api.slack.com/docs/message-buttons" target="_blank">Slack 按鈕</a>。 

> [!NOTE]
> 若要在 Slack 訊息中支援按鈕，您必須在[將 Bot 連線](../bot-service-manage-channels.md)至 Slack 通道時啟用 [互動式訊息]。

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

當使用者按一下 Slack 訊息中的按鈕時，Bot 會收到回應訊息，其中的 `channelData` 屬性已填入 `payload` JSON 物件。 `payload` 物件會指定原始訊息的內容、識別已按下的按鈕，並識別按下按鈕的使用者。 

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

Bot 可以透過[正常方式](bot-framework-rest-connector-send-and-receive-messages.md#create-reply)回覆此訊息，也可以將其回應直接張貼到 `payload` 物件之 `response_url` 屬性所指定的端點。 如需何時及如何將回應張貼到 `response_url` 的資訊，請參閱 <a href="https://api.slack.com/docs/message-buttons" target="_blank">Slack 按鈕</a>。 

## <a name="create-a-facebook-notification"></a>建立 Facebook 通知

若要建立 Facebook 通知，請將 [Activity][Activity] 物件的 `channelData` 屬性設定為指定這些屬性的 JSON 物件： 

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

若要建立訊息來實作 Telegram 特定動作 (例如，共用語音備忘或貼圖)，請將 [Activity][Activity] 物件的 `channelData` 屬性設定為會指定這些屬性的 JSON 物件： 

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
> <li>在 Bot 從 Telegram 通道所收到的每則訊息內，<code>channelData</code> 屬性會包含 Bot 先前傳送的訊息。</li></ul>

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

若要建立原生的 Kik 訊息，請將 [Activity][Activity] 物件的 `channelData` 屬性設定為指定此屬性的 JSON 物件： 

| 屬性 | 說明 |
|----|----|
| 上限 | Kik 訊息的陣列。 如需 Kik 訊息格式的詳細資訊，請參閱 <a href="https://dev.kik.com/#/docs/messaging#message-formats" target="_blank">Kik 訊息格式</a>。 |

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

## <a name="additional-resources"></a>其他資源

- [活動概觀](bot-framework-rest-connector-activities.md)
- [建立訊息](bot-framework-rest-connector-create-messages.md)
- [傳送及接收訊息](bot-framework-rest-connector-send-and-receive-messages.md)
- [使用通道偵測器預覽功能](../bot-service-channel-inspector.md)

[Activity]: bot-framework-rest-connector-api-reference.md#activity-object
