---
title: 開始對話 | Microsoft Docs
description: 了解如何使用 Direct Line API 3.0 版開始對話。
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 12/13/2017
ms.openlocfilehash: 6754935070fefb890c3d5b7bd90f1c1a4c5d401d
ms.sourcegitcommit: b78fe3d8dd604c4f7233740658a229e85b8535dd
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/24/2018
ms.locfileid: "50000295"
---
# <a name="start-a-conversation"></a>開始對話

Direct Line 對話是由用戶端明確地開啟，且只要 Bot 和用戶端在參與且具有有效認證，對話就可以繼續執行。 對話開啟時，Bot 和用戶端可以傳送訊息。 可以有超過一個用戶端連線到指定對話，且每個用戶端都可以代表多個使用者參與對話。

## <a name="open-a-new-conversation"></a>開啟新的對話

若要與 Bot 開啟新的對話，請發出此要求：

```http
POST https://directline.botframework.com/v3/directline/conversations
Authorization: Bearer SECRET_OR_TOKEN
```

下列程式碼片段提供「開始對話」要求和回應範例。

### <a name="request"></a>要求

```http
POST https://directline.botframework.com/v3/directline/conversations
Authorization: Bearer RCurR_XV9ZA.cwA.BKA.iaJrC8xpy8qbOF5xnR2vtCX7CZj0LdjAPGfiCpg4Fv0y8qbOF5xPGfiCpg4Fv0y8qqbOF5x8qbOF5xn
```

### <a name="response"></a>Response

如果要求成功，回應將會包含對話識別碼、權杖、一個表示直到權杖過期前秒數的值，以及用戶端可用來[透過 WebSocket 資料流接收活動](bot-framework-rest-direct-line-3-0-receive-activities.md#connect-via-websocket)的資料流 URL。

```http
HTTP/1.1 201 Created
[other headers]
```

```json
{
  "conversationId": "abc123",
  "token": "RCurR_XV9ZA.cwA.BKA.iaJrC8xpy8qbOF5xnR2vtCX7CZj0LdjAPGfiCpg4Fv0y8qbOF5xPGfiCpg4Fv0y8qqbOF5x8qbOF5xn",
  "expires_in": 1800,
  "streamUrl": "https://directline.botframework.com/v3/directline/conversations/abc123/stream?t=RCurR_XV9ZA.cwA..."
}
```

一般而言，開始對話要求會用來開啟新的對話，而且如果已順利開始新的對話，便會傳回 **HTTP 201** 狀態碼。 不過，如果用戶端提交開始對話要求時，在 `Authorization` 標頭中使用了 Direct Line 權杖，該權杖先前已用來藉由「開始對話」作業開始對話，則會傳回 **HTTP 200** 狀態碼，表示這是可接受的要求，但未建立對話 (因為已經存在)。

> [!TIP]
> 您有 60 秒的時間可以連線到 WebSocket 資料流 URL。 如果無法在這段時間內建立連線，您可以[重新連線到對話](bot-framework-rest-direct-line-3-0-reconnect-to-conversation.md)，以產生新的資料流 URL。

## <a name="start-conversation-versus-generate-token"></a>開始對話與產生權杖

開始對話作業 (`POST /v3/directline/conversations`) 和[產生權杖](bot-framework-rest-direct-line-3-0-authentication.md#generate-token)作業 (`POST /v3/directline/tokens/generate`) 類似，兩個作業都會傳回可用於存取單一對話的 `token`。 不過，開始對話作業也會開始對話、連絡 Bot，並建立 WebSocket 資料流 URL，而產生權杖作業不會做這些事。 

如果您想要立即開始對話，請使用開始對話作業。 如果您計畫要將權杖散發給用戶端，並且要它們起始對話，請改為使用[產生權杖](bot-framework-rest-direct-line-3-0-authentication.md#generate-token)作業。 

## <a name="additional-resources"></a>其他資源

- [重要概念](bot-framework-rest-direct-line-3-0-concepts.md)
- [驗證](bot-framework-rest-direct-line-3-0-authentication.md)
- [透過 WebSocket 資料流接收活動](bot-framework-rest-direct-line-3-0-receive-activities.md#connect-via-websocket)
- [重新連線到對話](bot-framework-rest-direct-line-3-0-reconnect-to-conversation.md)