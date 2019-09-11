---
title: 重新連線到對話 | Microsoft Docs
description: 了解如何使用 Direct Line API v3.0 來重新連線到對話。
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 2/09/2019
ms.openlocfilehash: 3197b4f0e3d8d2cc07cea967f4ddf0738e3e27e9
ms.sourcegitcommit: a6d02ec4738e7fc90b7108934740e9077667f3c5
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/04/2019
ms.locfileid: "70299556"
---
# <a name="reconnect-to-a-conversation"></a>重新連線到對話

如果用戶端使用 [WebSocket 介面](bot-framework-rest-direct-line-3-0-receive-activities.md#connect-via-websocket)來接收訊息，但卻失去連線，此時可能需要重新連線。 在此情況下，用戶端必須產生新的 WebSocket 串流 URL，用它來重新連線到對話。

## <a name="generate-a-new-websocket-stream-url"></a>產生新的 WebSocket 串流 URL

若要產生新的 WebSocket 串流 URL，以用來重新連線到現有對話，請發出此要求： 

```http
GET https://directline.botframework.com/v3/directline/conversations/{conversationId}?watermark={watermark_value}
Authorization: Bearer SECRET_OR_TOKEN
```

在此要求 URI 中，使用對話識別碼取代 **{conversationId}** ，以及使用 watermark 值來取代 **{watermark_value}** (如果有提供 `watermark` 參數的話)。 `watermark` 是選用參數。 如果要求 URI 中有指定 `watermark` 參數，對話會從浮水印 重新執行，以保證沒有任何訊息遺失。 如果 `watermark` 參數從要求 URI 中省略，則只會收到重新連線要求執行之後的訊息。

下列程式碼片段提供重新連線要求和回應的範例。

### <a name="request"></a>要求

```http
GET https://directline.botframework.com/v3/directline/conversations/abc123?watermark=0000a-42
Authorization: Bearer RCurR_XV9ZA.cwA.BKA.iaJrC8xpy8qbOF5xnR2vtCX7CZj0LdjAPGfiCpg4Fv0y8qbOF5xPGfiCpg4Fv0y8qqbOF5x8qbOF5xn
```

### <a name="response"></a>Response

如果要求成功，回應會包含對話識別碼、語彙基元和新的 WebSocket 串流 URL。

```http
HTTP/1.1 200 OK
[other headers]
```

```json
{
  "conversationId": "abc123",
  "token": "RCurR_XV9ZA.cwA.BKA.iaJrC8xpy8qbOF5xnR2vtCX7CZj0LdjAPGfiCpg4Fv0y8qbOF5xPGfiCpg4Fv0y8qqbOF5x8qbOF5xn",
  "streamUrl": "https://directline.botframework.com/v3/directline/conversations/abc123/stream?watermark=000a-4&amp;t=RCurR_XV9ZA.cwA..."
}
```

## <a name="reconnect-to-the-conversation"></a>重新連線到對話

用戶端必須使用新的 WebSocket 串流 URL，以在 60 秒內[重新連線到對話](bot-framework-rest-direct-line-3-0-receive-activities.md#connect-via-websocket)。 如果無法在這段時間內建立連線，用戶端必須發出另一個重新連線要求，以產生新的串流 URL。

如果您已在 Direct Line 設定中啟用「增強型驗證選項」，則可能會收到 400 "MissingProperty" 錯誤，表示未指定任何使用者識別碼。

## <a name="additional-resources"></a>其他資源

- [重要概念](bot-framework-rest-direct-line-3-0-concepts.md)
- [驗證](bot-framework-rest-direct-line-3-0-authentication.md)
- [透過 WebSocket 資料流接收活動](bot-framework-rest-direct-line-3-0-receive-activities.md#connect-via-websocket)
