---
title: 接收來自 Bot 的活動 | Microsoft Docs
description: 了解如何使用直接線路 API v3.0 接收來自 Bot 的活動。
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 12/13/2017
ms.openlocfilehash: c2d4b9a8e2b8ffc1656df44e04ee1bde912e36ea
ms.sourcegitcommit: b78fe3d8dd604c4f7233740658a229e85b8535dd
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/24/2018
ms.locfileid: "49998155"
---
# <a name="receive-activities-from-the-bot"></a>接收來自 Bot 的活動

透過使用直接線路 3.0 通訊協定，用戶端可以透過 `WebSocket` 串流來接收活動，或透過發出 `HTTP GET` 要求來擷取活動。 

## <a name="websocket-vs-http-get"></a>WebSocket 與 HTTP GET

串流 WebSocket 會有效率地將訊息推送到用戶端，而 GET 介面可讓用戶端明確地要求訊息。 雖然 WebSocket 機制因其效率而較常被使用，但 GET 機制對於無法使用 WebSockets 的用戶端或想要擷取對話記錄的用戶端而言非常實用。 

並非所有[活動類型](bot-framework-rest-connector-activities.md)都可透過 WebSocket 和 HTTP GET 這兩者使用。 下表說明對於使用直接線路通訊協定的用戶端之各種活動類型的可用性。

| 活動類型 | 可用性 | 
|----|----|
| Message | HTTP GET 和 WebSocket |
| typing | 只有 WebSocket |
| conversationUpdate | 不透過用戶端傳送/接收 |
| contactRelationUpdate | 直接線路中不支援 |
| endOfConversation | HTTP GET 和 WebSocket |
| 所有其他活動類型 | HTTP GET 和 WebSocket |

## <a id="connect-via-websocket"></a> 透過 WebSocket 串流接收活動

當用戶端傳送[開始對話](bot-framework-rest-direct-line-3-0-start-conversation.md)要求以開啟和 Bot 的對話時，服務的回應包含用戶端可以在後續用於透過 WebSocket 連線的 `streamUrl` 屬性。 串流 URL 是預先授權的，因此用戶端透過 WebSocket 的連線要求「不」需要 `Authorization` 標頭。

下列範例顯示使用 `streamUrl` 透過 WebSocket 來連線的要求。

```http
-- connect to wss://directline.botframework.com --
GET /v3/directline/conversations/abc123/stream?t=RCurR_XV9ZA.cwA..."
Upgrade: websocket
Connection: upgrade
[other headers]
```

服務回應狀態碼 HTTP 101 (「正在切換通訊協定」)。

```http
HTTP/1.1 101 Switching Protocols
[other headers]
```

### <a name="receive-messages"></a>接收訊息

透過 WebSocket 連線之後，用戶端可能會收到來自直接線路服務的這些訊息類型：

- 包含 [ActivitySet](bot-framework-rest-direct-line-3-0-api-reference.md#activityset-object) 的訊息，其中包括一或多個活動和浮水印 (如下所述)。
- 空白訊息，直接線路服務用來確認連線仍然有效。
- 其他類型 (在稍後定義)。 這些類型由 JSON 根中的屬性識別。

`ActivitySet` 包含由 Bot 及對話中所有使用者所傳送的訊息。 下列範例顯示包含單一訊息的 `ActivitySet`。

```json
{
    "activities": [
        {
            "type": "message",
            "channelId": "directline",
            "conversation": {
                "id": "abc123"
            },
            "id": "abc123|0000",
            "from": {
                "id": "user1"
            },
            "text": "hello"
        }
    ],
    "watermark": "0000a-42"
}
```

### <a name="process-messages"></a>處理訊息

用戶端應該持續追蹤收到的每個 [ActivitySet](bot-framework-rest-direct-line-3-0-api-reference.md#activityset-object) 中的 `watermark` 值，這樣就能使用浮水印來保證在連線中斷時不會有任何訊息遺失而需要[重新連線到對話](bot-framework-rest-direct-line-3-0-reconnect-to-conversation.md)。 如果用戶端接收到的 `ActivitySet` 中的 `watermark` 屬性為 `null` 或已遺失，則它應該忽略該值且不覆寫之前收到的浮水印。

用戶端應該忽略從直接線路服務收到的空白訊息。

用戶端可能會傳送空白訊息至直接線路服務以驗證連線能力。 直接線路服務將會忽略從用戶端收到的空白訊息。

直接線路服務在某些情況下可能會強制關閉 WebSocket 連線。 如果用戶端尚未收到 `endOfConversation` 活動，它可能會[產生新的 WebSocket 串流 URL](bot-framework-rest-direct-line-3-0-reconnect-to-conversation.md)，以用於重新連線到對話。 

WebSocket 串流包含即時更新和最近的訊息 (因為已發出透過 WebSocket 連線的呼叫)，但它不包含在最近的 `POST` 之前傳送至 `/v3/directline/conversations/{id}` 的訊息。 若要擷取對話中稍早之前的訊息，請使用下述的 `HTTP GET`。

## <a id="http-get"></a> 使用 HTTP GET 擷取活動

無法使用 WebSockets 的用戶端，或想要取得對話記錄的用戶端可以使用 `HTTP GET` 來擷取活動。

若要擷取特定對話的訊息，請向 `/v3/directline/conversations/{conversationId}/activities` 端點發出 `GET` 要求，選擇性指定 `watermark` 參數以指出用戶端看到的最新訊息。 

下列程式碼片段提供「取得對話活動」的要求和回應範例。 「取得對話活動」回應包含 `watermark` 做為 [ActivitySet](bot-framework-rest-direct-line-3-0-api-reference.md#activityset-object) 的屬性。 用戶端應隨 `watermark` 值向前逐頁查詢可用的活動，直到沒有活動傳回。

### <a name="request"></a>要求

```http
GET https://directline.botframework.com/v3/directline/conversations/abc123/activities?watermark=0001a-94
Authorization: Bearer RCurR_XV9ZA.cwA.BKA.iaJrC8xpy8qbOF5xnR2vtCX7CZj0LdjAPGfiCpg4Fv0
```

### <a name="response"></a>Response

```http
HTTP/1.1 200 OK
[other headers]
```

```json
{
    "activities": [
        {
            "type": "message",
            "channelId": "directline",
            "conversation": {
                "id": "abc123"
            },
            "id": "abc123|0000",
            "from": {
                "id": "user1"
            },
            "text": "hello"
        }, 
        {
            "type": "message",
            "channelId": "directline",
            "conversation": {
                "id": "abc123"
            },
            "id": "abc123|0001",
            "from": {
                "id": "bot1"
            },
            "text": "Nice to see you, user1!"
        }
    ],
    "watermark": "0001a-95"
}
```

## <a name="timing-considerations"></a>計時考量

大部分的用戶端都希望建立完整的訊息記錄。 雖然直接線路是有潛在計時間隔的多方通訊協定，但通訊協定和服務的設計旨在讓您更容易建置可靠的用戶端。

- 在 WebSocket 串流和「取得對話活動」回應中傳送的 `watermark` 屬性很可靠。 只要用戶端逐字重新執行浮水印，就不會錯過任何訊息。

- 當用戶端啟動對話並連線到 WebSocket 串流時，在 POST 之後且在通訊端開啟之前傳送的任何活動，都會在新的活動前重新執行。

- 當用戶端已連線到 WebSocket 串流並發出「取得對話活動」要求 (以重新整理記錄) 時，活動可能會在兩個頻道之間重複。 用戶端應持續追蹤所有已知的活動識別碼，這樣它們就能拒絕重複的活動 (如果發生重複)。

使用 `HTTP GET` 輪詢的用戶端應選擇符合其使用目的的輪詢間隔。

- 服務對服務應用程式通常會使用 5 秒或 10 秒的輪詢間隔。

- 面向用戶端的應用程式通常使用 1 秒的輪詢間隔，並在用戶端傳送的每則訊息後，發出約 300 毫秒的其他要求 (以快速擷取 Bot 的回應)。 此 300 毫秒的延遲應根據 Bot 的速度和傳輸時間加以調整。

## <a name="additional-resources"></a>其他資源

- [重要概念](bot-framework-rest-direct-line-3-0-concepts.md)
- [驗證](bot-framework-rest-direct-line-3-0-authentication.md)
- [開始對話](bot-framework-rest-direct-line-3-0-start-conversation.md)
- [重新連線到對話](bot-framework-rest-direct-line-3-0-reconnect-to-conversation.md)
- [將活動傳送到 Bot](bot-framework-rest-direct-line-3-0-send-activity.md)
- [結束對話](bot-framework-rest-direct-line-3-0-end-conversation.md)