---
title: 使用 Bot Connector 服務建立 Bot |Microsoft Docs
description: 使用 Bot Connector 服務建立 Bot。
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 12/13/2017
ms.openlocfilehash: 751a5d8430bb675e8ad5e10d02f94ee5642672cb
ms.sourcegitcommit: c200cc2db62dbb46c2a089fb76017cc55bdf26b0
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/27/2019
ms.locfileid: "70037459"
---
# <a name="create-a-bot-with-the-bot-connector-service"></a>使用 Bot Connector 服務建立 Bot
> [!div class="op_single_selector"]
> - [.NET](../dotnet/bot-builder-dotnet-quickstart.md)
> - [Node.js](../nodejs/bot-builder-nodejs-quickstart.md)
> - [Bot Service](../bot-service-quickstart.md)
> - [REST](../rest-api/bot-framework-rest-connector-quickstart.md)

Bot Connector 服務可讓您的 Bot 藉由使用產業標準的 REST 和 JSON over HTTPS，與 <a href="https://dev.botframework.com/" target="_blank">Bot Framework 入口網站</a>中設定的通道交換訊息。 本教學課程會逐步引導您完成從 Bot Framework 取得存取權杖，並且使用 Bot Connector 服務與使用者交換訊息的程序。

## <a id="get-token"></a> 取得存取權杖

> [!IMPORTANT]
> 若您尚未這麼做，請務必向 Bot Framework [註冊您的 Bot](../bot-service-quickstart-registration.md)，以取得其應用程式識別碼和密碼。 您將會需要 Bot 的應用程式識別碼和密碼以取得存取權杖。

若要與 Bot Connector 服務通訊，您必須使用下列格式，在每個 API 要求的 `Authorization` 標頭中指定存取權杖： 

```http
Authorization: Bearer ACCESS_TOKEN
```

您可以藉由發出 API 要求，針對您的 Bot 取得存取權杖。

### <a name="request"></a>要求

如需要求存取權杖，以便用來對 Bot Connector 服務驗證要求，請發出下列要求，並將 **MICROSOFT-APP-ID** 和 **MICROSOFT-APP-PASSWORD**，取代為您向 Bot Framework [註冊](../bot-service-quickstart-registration.md)您的 Bot 時所取得的應用程式識別碼和密碼。

```http
POST https://login.microsoftonline.com/botframework.com/oauth2/v2.0/token
Host: login.microsoftonline.com
Content-Type: application/x-www-form-urlencoded

grant_type=client_credentials&client_id=MICROSOFT-APP-ID&client_secret=MICROSOFT-APP-PASSWORD&scope=https%3A%2F%2Fapi.botframework.com%2F.default
```

### <a name="response"></a>Response

如果要求成功，您會收到 HTTP 200 回應，指定存取權杖及其到期的相關資訊。 

```json
{
    "token_type":"Bearer",
    "expires_in":3600,
    "ext_expires_in":3600,
    "access_token":"eyJhbGciOiJIUzI1Ni..."
}
```

> [!TIP]
> 如需 Bot Connector 服務中驗證的詳細資訊，請參閱[驗證](bot-framework-rest-connector-authentication.md)。

## <a name="exchange-messages-with-the-user"></a>與使用者交換訊息

對話是使用者與 Bot 之間交換的一系列訊息。 

### <a name="receive-a-message-from-the-user"></a>從使用者處接收訊息

當使用者傳送訊息時，Bot Framework Connector 會將要求 POST 到您在[註冊](../bot-service-quickstart-registration.md) Bot 時指定的端點。 要求的本文是 [Activity][] 物件。 下列範例示範當使用者將簡單的訊息傳送給 Bot 時，Bot 收到的要求主體。 

```json
{
    "type": "message",
    "id": "bf3cc9a2f5de...",
    "timestamp": "2016-10-19T20:17:52.2891902Z",
    "serviceUrl": "https://smba.trafficmanager.net/apis",
    "channelId": "channel's name/id",
    "from": {
        "id": "1234abcd",
        "name": "user's name"
    },
    "conversation": {
        "id": "abcd1234",
        "name": "conversation's name"
    },
    "recipient": {
        "id": "12345678",
        "name": "bot's name"
    },
    "text": "Haircut on Saturday"
}
```

### <a name="reply-to-the-users-message"></a>回覆使用者的訊息

當您的 Bot 端點收到代表使用者傳入訊息的 `POST` 要求時 (亦即 `type` = **訊息**)，請使用該要求中的資訊，為回應建立 [Activity][] 物件。

1. 將 **conversation** 屬性設為使用者訊息中 **conversation** 屬性的內容。
2. 將 **from** 屬性設為使用者訊息中 **recipient** 屬性的內容。
3. 將 **recipient** 屬性設為使用者訊息中 **from** 屬性的內容。
4. 適當地設定 **text** 和 **attachments** 屬性。

在連入要求中使用 `serviceUrl` 屬性以[識別基底 URI](bot-framework-rest-connector-api-reference.md#base-uri)，您的 Bot 應該使用該 URI 來發出其回應。 

若要傳送回應，請將您的 `Activity` 物件 `POST` 到 `/v3/conversations/{conversationId}/activities/{activityId}`，如下列範例所示。 此要求的本文是 `Activity` 物件，該物件會提示使用者選取可用的約會時間。

```http
POST https://smba.trafficmanager.net/apis/v3/conversations/abcd1234/activities/bf3cc9a2f5de... 
Authorization: Bearer eyJhbGciOiJIUzI1Ni...
Content-Type: application/json
```

```json
{
    "type": "message",
    "from": {
        "id": "12345678",
        "name": "bot's name"
    },
    "conversation": {
        "id": "abcd1234",
        "name": "conversation's name"
   },
   "recipient": {
        "id": "1234abcd",
        "name": "user's name"
    },
    "text": "I have these times available:",
    "replyToId": "bf3cc9a2f5de..."
}
```

在此範例要求中，`https://smba.trafficmanager.net/apis` 表示基底 URI，您的 Bot 所發出之要求的基底 URI 可能和這個不同。 如需設定基底 URI 的詳細資訊，請參閱 [API 參考](bot-framework-rest-connector-api-reference.md#base-uri)。 

> [!IMPORTANT]
> 如同這個範例所示，您所傳送每個 API 要求的 `Authorization` 標頭都必須包含字詞 **Bearer**，後面加上您[從 Bot Framework 取得](#get-token)的存取權杖。

若要傳送另一則訊息，讓使用者藉由按一下按鈕來選取可用約會時間，請將另一個要求 `POST` 到相同端點：

```http
POST https://smba.trafficmanager.net/apis/v3/conversations/abcd1234/activities/bf3cc9a2f5de... 
Authorization: Bearer eyJhbGciOiJIUzI1Ni...
Content-Type: application/json
```

```json
{
    "type": "message",
    "from": {
        "id": "12345678",
        "name": "bot's name"
    },
    "conversation": {
        "id": "abcd1234",
        "name": "conversation's name"
   },
   "recipient": {
        "id": "1234abcd",
        "name": "user's name"
    },
    "attachmentLayout": "list",
    "attachments": [
      {
        "contentType": "application/vnd.microsoft.card.thumbnail",
        "content": {
          "buttons": [
            {
              "type": "imBack",
              "title": "10:30",
              "value": "10:30"
            },
            {
              "type": "imBack",
              "title": "11:30",
              "value": "11:30"
            },
            {
              "type": "openUrl",
              "title": "See more",
              "value": "http://www.contososalon.com/scheduling"
            }
          ]
        }
      }
    ],
    "replyToId": "bf3cc9a2f5de..."
}
```   

## <a name="next-steps"></a>後續步驟

在本教學課程中，您會從 Bot Framework 取得存取權杖，並且使用 Bot Connector 服務與使用者交換訊息。 您可以使用 [Bot Framework 模擬器](../bot-service-debug-emulator.md)來測試您的 Bot 並且進行偵錯。 如果您想要與其他人共用您的 Bot，您必須將它[設定](../bot-service-manage-channels.md)為在一或多個通道上執行。

[Activity]: bot-framework-rest-connector-api-reference.md#activity-object
