---
title: 將媒體附件新增至訊息 | Microsoft Docs
description: 了解如何使用 Bot 連接器服務將媒體附件新增至訊息。
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 12/13/2017
ms.openlocfilehash: 39247ec3be4da7129989041269e930de8fa766ae
ms.sourcegitcommit: f576981342fb3361216675815714e24281e20ddf
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/18/2018
ms.locfileid: "39299410"
---
# <a name="add-media-attachments-to-messages"></a>將媒體附件新增至訊息
> [!div class="op_single_selector"]
> - [.NET](../dotnet/bot-builder-dotnet-add-media-attachments.md)
> - [Node.js](../nodejs/bot-builder-nodejs-send-receive-attachments.md)
> - [REST](../rest-api/bot-framework-rest-connector-add-media-attachments.md)

Bot 和通道通常會交換文字字串，但某些通道也支援交換附件，讓您的 Bot 可將更豐富的訊息傳送給使用者。 例如，您的 Bot 可以傳送媒體附件 (例如，影像、影片、音訊、檔案) 和[複合式資訊卡](bot-framework-rest-connector-add-rich-cards.md)。 本文描述如何使用 Bot 連接器服務，將媒體附件新增至訊息。

> [!TIP]
> 若要確認某個通道所支援的附件類型和數目，以及該通道呈現附件的方式，請參閱[通道偵測器][ChannelInspector]。

## <a name="add-a-media-attachment"></a>新增媒體附件  

若要將媒體附件新增至訊息，請建立[附件][Attachment]物件，設定 `name` 屬性，將 `contentUrl` 屬性設為媒體檔案的 URL，並將 `contentType` 屬性設為適當的媒體類型 (例如 **image/png**、**audio/wav**、**video/mp4**)。 然後，在表示訊息的[活動][Activity]物件中，於 `attachments` 陣列內指定您的[附件][Attachment]物件。 

下列範例所顯示的要求會傳送包含文字和單一影像附件的訊息。 在此範例要求中，`https://smba.trafficmanager.net/apis` 表示基底 URI；與 Bot 所提出要求的基底 URI 可能不同。 如需設定基底 URI 的詳細資料，請參閱 [API 參考](bot-framework-rest-connector-api-reference.md#base-uri)。

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
    "text": "Here's a picture of the duck I was telling you about.",
    "attachments": [
        {
            "contentType": "image/png",
            "contentUrl": "http://aka.ms/Fo983c",
            "name": "duck-on-a-rock.jpg"
        }
    ],
    "replyToId": "5d5cdc723"
}
```

對於支援影像內嵌二進位檔的通道，您可以將 `Attachment` 的 `contentUrl` 屬性設定為影像的 base64 二進位檔 (例如，**data:image/png;base64,iVBORw0KGgo…**)。 該通道會在訊息的文字字串旁顯示影像或影像的 URL。

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
    "text": "Here's a picture of the duck I was telling you about.",
    "attachments": [
        {
            "contentType": "image/png",
            "contentUrl": "data:image/png;base64,iVBORw0KGgo…",
            "name": "duck-on-a-rock.jpg"
        }
    ],
    "replyToId": "5d5cdc723"
}
```

您可以使用與影像檔上述處理相同的處理序，將影片檔或音訊檔附加到訊息。 視通道而定，影片及音訊可以內嵌播放，也可能會顯示為連結。

> [!NOTE] 
> 您的 Bot 也可能會收到包含媒體附件的訊息。
> 比方說，如果通道可讓使用者上傳要分析的相片或要儲存的文件，Bot 所收到的訊息可能會包含附件。

## <a name="add-an-audiocard-attachment"></a>新增 AudioCard 附件

新增 [AudioCard](bot-framework-rest-connector-api-reference.md#audiocard-object) 或 [VideoCard](bot-framework-rest-connector-api-reference.md#videocard-object) 附件等同於新增媒體附件。 例如，下列 JSON 顯示如何在媒體附件中新增音訊卡。

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
      "contentType": "application/vnd.microsoft.card.audio",
      "content": {
        "title": "Allegro in C Major",
        "subtitle": "Allegro Duet",
        "text": "No Image, No Buttons, Autoloop, Autostart, Sharable",
        "media": [
          {
            "url": "https://contoso.com/media/AllegrofromDuetinCMajor.mp3"
          }
        ],
        "shareable": true,
        "autoloop": true,
        "autostart": true,
        "value": {
            // Supplementary parameter for this card
        }
      }
    }],
    "replyToId": "5d5cdc723"
}
```

一旦通道收到此附件，它就會開始播放音訊檔案。 比方說，如果使用者按一下 [暫停] 按鈕與音訊互動，通道將使用如下所示的 JSON 傳送回呼給 Bot：

```json
{
    ...
    "type": "event",
    "name": "media/pause",
    "value": {
        "url": // URL for media
        "cardValue": {
            // Supplementary parameter for this card
        }
    }
}
```

媒體事件名稱 **media/pause** 會出現在 `activity.name` 欄位中。 請參考下方的資料表，以取得所有媒體事件名稱的清單。

| Event | 說明 |
| ---- | ---- |
| **media/next** | 用戶端已跳到下一個媒體 |
| **media/pause** | 用戶端已暫停播放媒體 |
| **media/play** | 用戶端已開始播放媒體 |
| **media/previous** | 用戶端已跳到上一個媒體 |
| **media/resume** | 用戶端已繼續播放媒體 |
| **media/stop** | 用戶端已停止播放媒體 |

## <a name="additional-resources"></a>其他資源

- [建立訊息](bot-framework-rest-connector-create-messages.md)
- [傳送及接收訊息](bot-framework-rest-connector-send-and-receive-messages.md)
- [將複合式資訊卡 (Rich Card) 新增至訊息](bot-framework-rest-connector-add-rich-cards.md)
- [通道偵測器][ChannelInspector]

[ChannelInspector]: ../bot-service-channel-inspector.md

[Activity]: bot-framework-rest-connector-api-reference.md#activity-object
[Attachment]: bot-framework-rest-connector-api-reference.md#attachment-object
