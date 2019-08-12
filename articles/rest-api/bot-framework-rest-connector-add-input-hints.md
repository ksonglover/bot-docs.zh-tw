---
title: 將輸入提示新增至訊息 | Microsoft Docs
description: 了解如何使用 Bot Connector 服務將輸入提示新增至訊息。
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 12/13/2017
ms.openlocfilehash: 4eec5b84c05e170a9ed69d22dcabec7c1df055f9
ms.sourcegitcommit: a1eaa44f182a7210197bd793250907df00e9edab
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/03/2019
ms.locfileid: "68757080"
---
# <a name="add-input-hints-to-messages"></a>將輸入提示新增至訊息
> [!div class="op_single_selector"]
> - [.NET](../dotnet/bot-builder-dotnet-add-input-hints.md)
> - [Node.js](../nodejs/bot-builder-nodejs-send-input-hints.md)
> - [REST](../rest-api/bot-framework-rest-connector-add-input-hints.md)

您可藉由指定訊息的輸入提示，指出您的 Bot 在訊息傳遞給用戶端之後，會接受、需要或忽略使用者輸入。 這可讓用戶端為許多通道設定相應的使用者輸入控制項狀態。 例如，如果訊息的輸入提示指出 Bot 會忽略使用者輸入，則用戶端可關閉麥克風並停用輸入方塊，以防止使用者提供輸入。

## <a name="accepting-input"></a>接受輸入

若要指出您的 Bot 要被動接受輸入，但不等候使用者回應，請將代表訊息的 `Activity` 物件內的 `inputHint` 屬性設定為 **acceptingInput**。 在許多通道上，這會啟用用戶端的輸入方塊並關閉麥克風，但使用者仍可存取。 例如，如果使用者按住 [麥克風] 按鈕，Cortana 會開啟麥克風接受使用者輸入。 

下列範例示範傳送訊息並指定 Bot 接受輸入的要求。 在此範例要求中，`https://smba.trafficmanager.net/apis` 表示基底 URI，和您提出要求之 Bot 的基底 URI 可能不同。 如需設定基底 URI 的詳細資料，請參閱 [API 參考](bot-framework-rest-connector-api-reference.md#base-uri)。

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
    "text": "Here's a picture of the house I was telling you about.",
    "inputHint": "acceptingInput",
    "replyToId": "5d5cdc723"
}
```

## <a name="expecting-input"></a>需要輸入

若要指出您的 Bot 要等候使用者回應，請將代表訊息的 `Activity` 物件內的 `inputHint` 屬性設定為 **expectingInput**。 在許多通道上，這會啟用用戶端的輸入方塊並開啓麥克風。 

下列範例示範傳送訊息並指定 Bot 需要輸入的要求。 在此範例要求中，`https://smba.trafficmanager.net/apis` 表示基底 URI，和您提出要求之 Bot 的基底 URI 可能不同。 如需設定基底 URI 的詳細資料，請參閱 [API 參考](bot-framework-rest-connector-api-reference.md#base-uri)。

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
    "text": "What is your favorite color?",
    "inputHint": "expectingInput",
    "replyToId": "5d5cdc723"
}
```

## <a name="ignoring-input"></a>忽略輸入
 
若要指出您的 Bot 尚未準備接收使用者輸入，請將代表訊息的 `Activity` 物件內的 `inputHint` 屬性設定為 **ignoringInput**。 在許多通道上，這會停用用戶端的輸入方塊並關閉麥克風。 

下列範例示範傳送訊息並指定 Bot 忽略輸入的要求。 在此範例要求中，`https://smba.trafficmanager.net/apis` 表示基底 URI，和您提出要求之 Bot 的基底 URI 可能不同。 如需設定基底 URI 的詳細資料，請參閱 [API 參考](bot-framework-rest-connector-api-reference.md#base-uri)。

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
    "text": "Please hold while I perform the calculation.",
    "inputHint": "ignoringInput",
    "replyToId": "5d5cdc723"
}
```

## <a name="additional-resources"></a>其他資源

- [建立訊息](bot-framework-rest-connector-create-messages.md)
- [傳送及接收訊息](bot-framework-rest-connector-send-and-receive-messages.md)

