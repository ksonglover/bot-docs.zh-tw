---
title: 管理狀態資料 |Microsoft Docs
description: 了解如何使用 Bot 狀態服務儲存及擷取狀態資料。
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 12/13/2017
ms.openlocfilehash: b0d5ca6893d70a73bc005a949ef6cc2518d3862f
ms.sourcegitcommit: b78fe3d8dd604c4f7233740658a229e85b8535dd
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/24/2018
ms.locfileid: "50000025"
---
# <a name="manage-state-data"></a>管理狀態資料

Bot 狀態服務可讓您的 Bot 儲存及擷取與使用者、對話或特定對話內容中特定使用者相關聯的狀態資料。 您最多可以為頻道上的每個使用者、頻道上的每個對話，以及頻道上對話內容中的每個使用者儲存 32 KB 的資料。 狀態資料有許多用途，例如判斷先前離開對話的位置，或單純地以名字向回來的使用者問候。 如果您儲存使用者的喜好設定，下次聊天時就可以使用該資訊來自訂對話。 例如，您可能會在有使用者感興趣的主題之新聞文章，或有可用約會時通知使用者。 

> [!IMPORTANT]
> 建議您不要將 Bot Framework 狀態服務 API 用於生產環境，未來版本可能會將它淘汰。 建議更新您的 Bot 程式碼以使用記憶體內部儲存體進行測試，或將其中一個 **Azure 擴充功能**用於生產環境 Bot。 如需詳細資訊，請參閱 [.NET](~/dotnet/bot-builder-dotnet-state.md) 或 [Node](~/nodejs/bot-builder-nodejs-state.md) 實作的＜管理狀態資料＞主題。

## <a id="concurrency"></a> 資料並行

為控制使用 Bot 狀態服務儲存的資料並行，架構針對 `POST` 要求使用實體標記 (Etag)。 此架構不會使用標準的 Etag 標頭。 相反地，要求和回應的主體使用 `eTag` 屬性指定 Etag。 

例如，如果您發出 `GET` 要求以從存放區擷取使用者資料，回應將會包含 `eTag` 屬性。 如果您變更資料並發出 `POST` 要求以將更新的資料儲存到存放區，您的要求可以包含 `eTag` 屬性，並為此屬性指定與稍早在 `GET` 回應中收到值相同的值。 如您的 `POST` 要求中所指定的 Etag 符合儲存體中目前的值，伺服器就會儲存使用者的資料並回應 **HTTP 200 成功**，然後在回應的主體中指定新的 `eTag` 值。 如果您 `POST` 要求中指定的 Etag 不符合存放區中目前的值，伺服器將會回應「HTTP 412 前置條件失敗」，以表示自從您上次儲存或擷取之後，存放區中的使用者資料已變更。

> [!TIP]
> `GET` 回應將一律包含 `eTag` 屬性，但您不需要在 `GET` 要求中指定 `eTag` 屬性。 星號 (`*`) 的 `eTag` 屬性值表示您先前未儲存所指定頻道、使用者與對話之組合的資料。
>
> 在 `POST` 要求中指定 `eTag` 屬性是選擇性的。 
> 如果並行對於您的 Bot 不是問題，請勿在 `POST` 要求中包含 `eTag` 屬性。 

## <a id="save-user-data"></a> 儲存使用者資料

若要儲存特定頻道上使用者的狀態資料，請發出此要求：

```http
POST /v3/botstate/{channelId}/users/{userId}
```

在此要求 URI 中，將 **{channelId}** 取代為頻道的識別碼，並將 **{userId}** 取代為頻道上的使用者識別碼。 您的 Bot 先前從使用者收到的任何訊息中的 `channelId` 和 `from` 屬性都會包含這些識別碼。 您也可以將這些值快取在安全的位置，這樣就可以在未來存取使用者的資料，而不需要從訊息中擷取它們。

將要求的主體設定為 [BotData][BotData] 物件，其中 `data` 屬性指定您要儲存的資料。 如果您將實體標記用於[並行控制](#concurrency)，請將 `eTag` 屬性設定為您上次儲存或擷取使用者資料時收到的 Etag (視孰者較新)。 如果您不將實體標記用於並行，則請勿在要求中包含 `eTag` 屬性。

> [!NOTE]
> 只要當指定的 Etag 符合伺服器的 Etag，或未在要求中指定任何 Etag 時，此 `POST` 要求才會覆寫存放區中的使用者資料。

### <a name="request"></a>要求 

下列範例顯示儲存特定頻道上使用者資料的要求。 在此範例要求中，`https://smba.trafficmanager.net/apis` 表示基底 URI，您的 Bot 所發出之要求的基底 URI 可能和這個不同。 如需設定基底 URI 的詳細資料，請參閱 [API 參考](bot-framework-rest-connector-api-reference.md#base-uri)。

```http
POST https://smba.trafficmanager.net/apis/v3/botstate/abcd1234/users/12345678
Authorization: Bearer ACCESS_TOKEN
Content-Type: application/json
```

```json
{
    "data": [
        {
            "trail": "Lake Serene",
            "miles": 8.2,
            "difficulty": "Difficult",
        },
        {
            "trail": "Rainbow Falls",
            "miles": 6.3,
            "difficulty": "Moderate",
        }
    ],
    "eTag": "a1b2c3d4"
}
```

### <a name="response"></a>Response 

回應將會包含具有新 `eTag` 值的 [BotData][BotData] 物件。

## <a name="get-user-data"></a>取得使用者資料

若要取得先前為特定頻道上使用者儲存的狀態資料，請發出此要求：

```http
GET /v3/botstate/{channelId}/users/{userId}
```

在此要求 URI 中，將 **{channelId}** 取代為頻道的識別碼，並將 **{userId}** 取代為頻道上的使用者識別碼。 您的 Bot 先前從使用者收到的任何訊息中的 `channelId` 和 `from` 屬性都會包含這些識別碼。 您也可以將這些值快取在安全的位置，這樣就可以在未來存取使用者的資料，而不需要從訊息中擷取它們。

### <a name="request"></a>要求

下列範例顯示取得先前為使用者儲存之資料的要求。 在此範例要求中，`https://smba.trafficmanager.net/apis` 表示基底 URI，您的 Bot 所發出之要求的基底 URI 可能和這個不同。 如需設定基底 URI 的詳細資料，請參閱 [API 參考](bot-framework-rest-connector-api-reference.md#base-uri)。

```http
GET https://smba.trafficmanager.net/apis/v3/botstate/abcd1234/users/12345678
Authorization: Bearer ACCESS_TOKEN
Content-Type: application/json
```

### <a name="response"></a>Response

回應將會包含 [BotData][BotData] 物件，其中 `data` 屬性包含您先前為使用者儲存的資料，而 `eTag` 屬性包含您在後續儲存使用者資料的要求中使用的 Etag。 如果您先前沒有為使用者儲存資料，則 `data` 屬性將會是 `null`，而且 `eTag` 屬性將會包含星號 (`*`)。

此範例顯示 `GET` 要求的回應：

```json
{
    "data": [
        {
            "trail": "Lake Serene",
            "miles": 8.2,
            "difficulty": "Difficult",
        },
        {
            "trail": "Rainbow Falls",
            "miles": 6.3,
            "difficulty": "Moderate",
        }
    ],
    "eTag": "xyz12345"
}
```

## <a name="save-conversation-data"></a>儲存對話資料

若要儲存特定頻道上對話的狀態資料，請發出此要求：

```http
POST /v3/botstate/{channelId}/conversations/{conversationId}
```

在此要求 URI 中，將 **{channelId}** 取代為頻道的識別碼，並將 **{conversationId}** 取代為對話的識別碼。 您的 Bot 先前在對話內容中傳送或接收的任何訊息內的 `channelId` 和 `conversation` 屬性將會包含這些識別碼。 您也可以將這些值快取在安全的位置，這樣就可以在未來存取對話的資料，而不需要從訊息中擷取它們。

將要求主體設定為 [BotData][BotData] 物件，如[儲存使用者資料](#save-user-data)中所述。

> [!IMPORTANT]
> 因為[刪除使用者資料](#delete-user-data)作業不會刪除已經使用**儲存對話資料**作業儲存的資料，請勿使用此作業儲存使用者的個人識別資訊 (PII)。

### <a name="response"></a>Response

回應將會包含具有新 `eTag` 值的 [BotData][BotData] 物件。

## <a name="get-conversation-data"></a>取得對話資料

若要取得先前為特定頻道上對話儲存的狀態資料，請發出此要求：

```http
GET /v3/botstate/{channelId}/conversations/{conversationId}
```

在此要求 URI 中，將 **{channelId}** 取代為頻道的識別碼，並將 **{conversationId}** 取代為對話的識別碼。 您的 Bot 先前在對話內容中傳送或接收的任何訊息內的 `channelId` 和 `conversation` 屬性將會包含這些識別碼。 您也可以將這些值快取在安全的位置，這樣就可以在未來存取對話的資料，而不需要從訊息中擷取它們。

### <a name="response"></a>Response

回應將會包含具有新 `eTag` 值的 [BotData][BotData] 物件。

## <a name="save-private-conversation-data"></a>儲存私人對話資料

若要儲存特定對話內容中使用者的狀態資料，請發出此要求：

```http
POST /v3/botstate/{channelId}/conversations/{conversationId}/users/{userId}
```

在此要求 URI 中，將 **{channelId}** 取代成頻道的識別碼，將 **{conversationId}** 取代成對話的識別碼，並將 **{userId}** 取代成該頻道上的使用者識別碼。 您的 Bot 先前在對話內容中從使用者接收的任何訊息內的 `channelId`、`conversation` 和 `from` 屬性將會包含這些識別碼。 您也可以將這些值快取在安全的位置，這樣就可以在未來存取對話的資料，而不需要從訊息中擷取它們。

將要求主體設定為 [BotData][BotData] 物件，如[儲存使用者資料](#save-user-data)中所述。

### <a name="response"></a>Response

回應將會包含具有新 `eTag` 值的 [BotData][BotData] 物件。

## <a name="get-private-conversation-data"></a>取得私人對話資料

若要取得先前為特定對話內容中使用者儲存的狀態資料，請發出此要求： 

```http
GET /v3/botstate/{channelId}/conversations/{conversationId}/users/{userId}
```

在此要求 URI 中，將 **{channelId}** 取代為頻道的識別碼，將 **{conversationId}** 取代為對話的識別碼，並將 **{userId}** 取代為該頻道上的使用者識別碼。 您的 Bot 先前在對話內容中從使用者接收的任何訊息內的 `channelId`、`conversation` 和 `from` 屬性將會包含這些識別碼。 您也可以將這些值快取在安全的位置，這樣就可以在未來存取對話的資料，而不需要從訊息中擷取它們。

### <a name="response"></a>Response

回應將會包含具有新 `eTag` 值的 [BotData][BotData] 物件。

## <a name="delete-user-data"></a>刪除使用者資料

若要刪除特定頻道上使用者的狀態資料，請發出此要求：

```http
DELETE /v3/botstate/{channelId}/users/{userId}
```
在此要求 URI 中，將 **{channelId}** 取代為頻道的識別碼，並將 **{userId}** 取代為頻道上的使用者識別碼。 您的 Bot 先前從使用者收到的任何訊息中的 `channelId` 和 `from` 屬性都會包含這些識別碼。 您也可以將這些值快取在安全的位置，這樣就可以在未來存取使用者的資料，而不需要從訊息中擷取它們。

> [!IMPORTANT]
> 此作業會刪除先前使用[儲存使用者資料](#save-user-data)作業或[儲存私人對話資料](#save-private-conversation-data)作業來為使用者儲存的資料。 它不會刪除先前使用[儲存對話資料](#save-conversation-data)作業儲存的資料。 因此，請「不要」使用**儲存對話資料**作業來儲存使用者的個人識別資訊 (PII)。

當您的 Bot 接收 [deleteUserData](bot-framework-rest-connector-activities.md#deleteuserdata) 或 [contactRelationUpdate](bot-framework-rest-connector-activities.md#contactrelationupdate) 類型的 [Activity][Activity]，指出 Bot 已從使用者的連絡人清單中被移除時，Bot 應執行**刪除使用者資料**作業。

## <a name="additional-resources"></a>其他資源

- [重要概念](bot-framework-rest-connector-concepts.md)
- [活動概觀](bot-framework-rest-connector-activities.md)

[BotData]: bot-framework-rest-connector-api-reference.md#botdata-object
[Activity]: bot-framework-rest-connector-api-reference.md#activity-object
