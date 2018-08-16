---
title: 從 Bot 接收訊息 | Microsoft Docs
description: 了解如何從使用 Direct Line API v1.1 的 Bot 接收訊息。
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 12/13/2017
ms.openlocfilehash: d9f1821767e5bd26c9a8bfdf3927f257077f0e79
ms.sourcegitcommit: f576981342fb3361216675815714e24281e20ddf
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/18/2018
ms.locfileid: "39299931"
---
# <a name="receive-messages-from-the-bot"></a>從 Bot 接收訊息

> [!IMPORTANT]
> 本文描述如何從使用 Direct Line API 1.1 的 Bot 接收訊息。 如果要在用戶端應用程式與 Bot 之間建立新連線，請改用 [Direct Line API 3.0](bot-framework-rest-direct-line-3-0-receive-activities.md)。

使用 Direct Line 1.1 通訊協定，用戶端必須輪詢 `HTTP GET` 介面以接收訊息。 

## <a name="retrieve-messages-with-http-get"></a>使用 HTTP GET 接收訊息

若要擷取特定交談的訊息，請向 `api/conversations/{conversationId}/messages` 端點發出 `GET` 要求，選擇性指定 `watermark` 參數以指出用戶端看到的最新訊息。 JSON 回應中會傳回已更新的 `watermark` 值，即使其未包含任何訊息。

下列程式碼片段提供 Get Messages 要求和回應的範例。 Get Messages 回應包含作為 [MessageSet](bot-framework-rest-direct-line-1-1-api-reference.md#messageset-object) 屬性的 `watermark`。 用戶端應持續送出 `watermark` 值來瀏覽整頁的可用訊息，直至不再收到任何訊息。 

### <a name="request"></a>要求

```http
GET https://directline.botframework.com/api/conversations/abc123/messages?watermark=0001a-94
Authorization: Bearer RCurR_XV9ZA.cwA.BKA.iaJrC8xpy8qbOF5xnR2vtCX7CZj0LdjAPGfiCpg4Fv0
```

### <a name="response"></a>Response

```http
HTTP/1.1 200 OK
[other headers]
```

```json
{
    "messages": [
        {
            "conversation": "abc123",
            "id": "abc123|0000",
            "text": "hello",
            "from": "user1"
        }, 
        {
            "conversation": "abc123",
            "id": "abc123|0001",
            "text": "Nice to see you, user1!",
            "from": "bot1"
        }
    ],
    "watermark": "0001a-95"
}
```

## <a name="timing-considerations"></a>計時考量

雖然 Direct Line 是有潛在計時間隔的多方通訊協定，但通訊協定和服務的設計旨在更容易建置可靠的用戶端。 Get Messages 回應中傳送的 `watermark` 屬性是可靠的。 只要用戶端逐字重新執行浮水印，就不會錯過任何訊息。

用戶端應該選擇較符合其預計用途的輪詢間隔。

- 服務對服務應用程式通常會使用 5 秒或 10 秒的輪詢間隔。

- 用戶端對應應用程式通常使用 1 秒的輪詢間隔，並在用戶端傳送的每則訊息後，發出約 300 毫秒的其他要求 (以快速擷取 Bot 的回應)。 此 300 毫秒的延遲應根據 Bot 的速度和傳輸時間加以調整。

## <a name="additional-resources"></a>其他資源

- [重要概念](bot-framework-rest-direct-line-1-1-concepts.md)
- [驗證](bot-framework-rest-direct-line-1-1-authentication.md)
- [開始交談](bot-framework-rest-direct-line-1-1-start-conversation.md)
- [將訊息傳送至 Bot](bot-framework-rest-direct-line-1-1-send-message.md)