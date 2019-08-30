---
title: 將建議的動作新增至訊息 | Microsoft Docs
description: 了解如何使用 Bot 連接器服務，將建議的動作新增至訊息。
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 12/13/2017
ms.openlocfilehash: b8b0eb454b77332a8b19104af78d9553582ab854
ms.sourcegitcommit: c200cc2db62dbb46c2a089fb76017cc55bdf26b0
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/27/2019
ms.locfileid: "70037139"
---
# <a name="add-suggested-actions-to-messages"></a>將建議的動作新增至訊息
> [!div class="op_single_selector"]
> - [.NET](../dotnet/bot-builder-dotnet-add-suggested-actions.md)
> - [Node.js](../nodejs/bot-builder-nodejs-send-suggested-actions.md)
> - [REST](../rest-api/bot-framework-rest-connector-add-suggested-actions.md)

[!INCLUDE [Introduction to suggested actions](../includes/snippet-suggested-actions-intro.md)]

## <a name="send-suggested-actions"></a>傳送建議的動作

若要將建議的動作新增至訊息，請設定 [Activity][] 的 `suggestedActions` 屬性，以指定 [CardAction][] 物件清單，以代表要向使用者呈現的按鈕。 

下列要求會傳送一則訊息，向使用者呈現三個建議的動作。 在此範例要求中，`https://smba.trafficmanager.net/apis` 表示基底 URI；與 Bot 所提出要求的基底 URI 可能不同。 如需設定基底 URI 的詳細資料，請參閱 [API 參考](bot-framework-rest-connector-api-reference.md#base-uri)。

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
    "text": "I have colors in mind, but need your help to choose the best one.",
    "inputHint": "expectingInput",
    "suggestedActions": {
        "actions": [
            {
                "type": "imBack",
                "title": "Blue",
                "value": "Blue"
            },
            {
                "type": "imBack",
                "title": "Red",
                "value": "Red"
            },
            {
                "type": "imBack",
                "title": "Green",
                "value": "Green"
            }
        ]
    },
    "replyToId": "5d5cdc723"
}
```

當使用者點選其中一個建議的動作時，Bot 會收到來自使用者的訊息，其中包含對應動作的 `value`。

## <a name="additional-resources"></a>其他資源

- [建立訊息](bot-framework-rest-connector-create-messages.md)
- [傳送及接收訊息](bot-framework-rest-connector-send-and-receive-messages.md)

[channelInspector]: ../bot-service-channel-inspector.md

[Activity]: bot-framework-rest-connector-api-reference.md#activity-object
[CardAction]: bot-framework-rest-connector-api-reference.md#cardaction-object
