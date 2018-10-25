---
title: 傳送及接收訊息 | Microsoft Docs
description: 了解如何使用 Bot Connector 服務來傳送及接收訊息。
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 12/13/2017
ms.openlocfilehash: d0b7b3250a62a995113bc9c7e087e2e62af0f413
ms.sourcegitcommit: b78fe3d8dd604c4f7233740658a229e85b8535dd
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/24/2018
ms.locfileid: "49997065"
---
# <a name="send-and-receive-messages"></a>傳送及接收訊息

Bot Connector 服務可讓 Bot 跨多個通道 (例如 Skype、電子郵件、Slack 等) 進行通訊。 它可藉由轉接從 Bot 至通道和從通道至 Bot 的[活動](bot-framework-rest-connector-activities.md)，來協助 Bot 和使用者之間的通訊。 每個活動都包含資訊，可用於將訊息以及訊息建立者、訊息的內容和訊息收件者的相關資訊，路由傳送至適當的目的地。 本文說明如何使用 Bot Connector 服務來交換通道上的 Bot 和使用者之間的**訊息**活動。 

## <a id="create-reply"></a> 回覆訊息

### <a name="create-a-reply"></a>建立回覆 

當使用者將傳送訊息至您的 Bot 時，Bot 會以 **message** 類型的[活動][Activity]物件收到訊息。 若要建立使用者訊息的回覆，請建立新的[活動][Activity]物件，並從設定下列屬性開始：

| 屬性 | 值 |
|----|----|
| 對話 | 將此屬性設為使用者訊息中 `conversation` 屬性的內容。 |
| from | 將此屬性設為使用者訊息中 `recipient` 屬性的內容。 |
| 地區設定 | 將此屬性設為使用者訊息中 `locale` 屬性的內容 (若指定)。 |
| 收件者 | 將此屬性設為使用者訊息中 `from` 屬性的內容。 |
| replyToId | 將此屬性設為使用者訊息中 `id` 屬性的內容。 |
| type | 請將此屬性設定為 **message**。 |

接下來，設定屬性 (該屬性會指定您想要傳達給使用者的資訊)。 例如，您可以設定 `text` 屬性來指定要顯示在訊息中的文字、設定 `speak` 屬性來指定您 Bot 要說出的文字，並設定 `attachments` 屬性來指定要包含在訊息中的媒體附件或複合式資訊卡 (Rich Card)。 如需常用訊息屬性的詳細資訊，請參閱[建立訊息](bot-framework-rest-connector-create-messages.md)。

### <a name="send-the-reply"></a>傳送回覆

在連入活動中使用 `serviceUrl` 屬性以[識別基底 URI](bot-framework-rest-connector-api-reference.md#base-uri) (您的 Bot 應該使用該 URI 來發出其回應)。 

若要傳送回覆，請發出此要求： 

```http
POST /v3/conversations/{conversationId}/activities/{activityId}
```

在此要求 URI 中，將 **{conversationId}** 取代為您 (回覆) 活動內 `conversation` 物件的 `id` 屬性值，並將 **{activityId}** 取代為您 (回覆) 活動內 `replyToId` 屬性的值。 將要求的本文設定為您所建立要代表回覆的[活動][Activity]物件。

下列範例會示範將簡單純文字回覆傳送給使用者郵件的要求。 在此範例要求中，`https://smba.trafficmanager.net/apis` 表示基底 URI，和 Bot 所提出要求的基底 URI 可能不同。 如需設定基底 URI 的詳細資訊，請參閱 [API 參考](bot-framework-rest-connector-api-reference.md#base-uri)。

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
        "name": "Pepper's News Feed"
    },
    "conversation": {
        "id": "abcd1234",
        "name": "Convo1"
   },
   "recipient": {
        "id": "1234abcd",
        "name": "SteveW"
    },
    "text": "My bot's reply",
    "replyToId": "5d5cdc723"
}
```

## <a id="send-message"></a> 傳送 (非回覆) 訊息

您 Bot 所傳送的大部分訊息，會在它從使用者接收到的回覆訊息中。 不過，有時候您的 Bot 可能需要將訊息傳送到並非直接回覆給任何使用者訊息的對話。 例如，您的 Bot 可能需要啟動新的對話主題，或在對話結尾傳送再見訊息。 

若要將訊息傳送到並非直接回覆給任何使用者訊息的對話，請發出此要求： 

```http
POST /v3/conversations/{conversationId}/activities
```

在此要求 URI 中，將 **{conversationId}** 取代為對話的識別碼。 
    
將要求的本文設定為您所建立要代表回覆的[活動][Activity]物件。

> [!NOTE]
> Bot Framework 對於 Bot 可傳送的訊息數目並無任何限制。 不過，大部分的通道會強制執行節流限制，限制 Bot 在短時間內傳送大量的訊息。 此外，如果 Bot 連續快速地傳送多則訊息，通道可能不一定會依照正確順序轉譯訊息。

## <a name="start-a-conversation"></a>開始對話

您的 Bot 有時候可能需要起始與一或多位使用者的對話。 若要在通道上開始與使用者對話，您的 Bot 必須知道其帳戶資訊，以及該通道上的使用者帳戶資訊。 

> [!TIP]
> 如果您的 Bot 在未來可能需要開始與其使用者對話，請快取使用者帳戶資訊、其他相關資訊 (例如使用者喜好設定和地區設定) 和服務 URL (用以作為「開始對話」要求中的基底 URI)。 

若要開始對話，請發出此要求： 

```http
POST /v3/conversations
```

將要求的本文設定為[對話][Conversation]物件，該物件會指定您 Bot 的帳戶資訊，和您想要包含在對話中的使用者帳戶資訊。

> [!NOTE]
> 並非所有通道都支援群組對話。 請參閱通道的文件，判斷通道是否支援群組對話，並識別通道允許對話中的參與者數目上限。

下列範例顯示開始的對話要求。 在此範例要求中，`https://smba.trafficmanager.net/apis` 表示基底 URI，和 Bot 所提出要求的基底 URI 可能不同。 如需設定基底 URI 的詳細資訊，請參閱 [API 參考](bot-framework-rest-connector-api-reference.md#base-uri)。

```http
POST https://smba.trafficmanager.net/apis/v3/conversations 
Authorization: Bearer ACCESS_TOKEN
Content-Type: application/json
```

```json
{
    "bot": {
        "id": "12345678",
        "name": "bot's name"
    },
    "isGroup": false,
    "members": [
        {
            "id": "1234abcd",
            "name": "recipient's name"
        }
    ],
    "topicName": "News Alert"
}
```

如果對話是以指定的使用者所建立，回應就會包含識別對話的識別碼。 

```json
{
    "id": "abcd1234"
}
```

然後，您的 Bot 可以使用這個對話識別碼，[將訊息傳送](#send-message)給對話中的使用者。

## <a name="additional-resources"></a>其他資源

- [活動概觀](bot-framework-rest-connector-activities.md)
- [建立訊息](bot-framework-rest-connector-create-messages.md)

[Activity]: bot-framework-rest-connector-api-reference.md#activity-object
[ConversationAccount]: bot-framework-rest-connector-api-reference.md#conversationaccount-object
[Conversation]: bot-framework-rest-connector-api-reference.md#conversation-object

