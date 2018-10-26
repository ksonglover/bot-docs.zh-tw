---
title: 結束對話 | Microsoft Docs
description: 了解如何使用直接線路 API 3.0 版結束對話。
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 12/13/2017
ms.openlocfilehash: f0985f28fd1744bcfb6bf5cea1c2230254670e01
ms.sourcegitcommit: b78fe3d8dd604c4f7233740658a229e85b8535dd
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/24/2018
ms.locfileid: "50000205"
---
# <a name="end-a-conversation"></a>結束對話

用戶端或 Bot 可以透過傳送 **endOfConversation** [活動](bot-framework-rest-connector-activities.md)來表示直接對話的結束。 

> [!NOTE] 
> 僅 Cortana 通道可支援 endOfConversation 事件，其他通道不會實作這項功能。 每個通道會決定如何回應 endOfConversation 活動。 如果您要設計 DirectLine 用戶端，您會更新用戶端以做出正確行為，例如，如果 Bot 傳送活動給已經結束的對話，則會產生錯誤。

## <a name="send-an-endofconversation-activity"></a>傳送 endOfConversation 活動

**endOfConversation** 活動會結束 Bot 和用戶端之間的通訊。 在傳送 **endOfConversation** 活動之後，用戶端仍然可以使用 `HTTP GET` 來[擷取訊息](bot-framework-rest-direct-line-3-0-receive-activities.md#http-get)，但用戶端或 Bot 都不能將任何其他訊息傳送至對話。 

若要結束對話，只需發出 POST 要求以傳送 **endOfConversation** 活動即可。

### <a name="request"></a>要求

```http
POST https://directline.botframework.com/v3/directline/conversations/abc123/activities
Authorization: Bearer RCurR_XV9ZA.cwA.BKA.iaJrC8xpy8qbOF5xnR2vtCX7CZj0LdjAPGfiCpg4Fv0
[other headers]
```

```json
{
    "type": "endOfConversation",
    "from": {
        "id": "user1"
    }
}
```

### <a name="response"></a>Response

如果要求成功，則回應將包含已傳送之活動的識別碼。

```http
HTTP/1.1 200 OK
[other headers]
```

```json
{
  "id": "0004"
}
```

## <a name="additional-resources"></a>其他資源

- [重要概念](bot-framework-rest-direct-line-3-0-concepts.md)
- [驗證](bot-framework-rest-direct-line-3-0-authentication.md)
- [將活動傳送到 Bot](bot-framework-rest-direct-line-3-0-send-activity.md)
