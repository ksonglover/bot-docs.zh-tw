---
title: 結束對話 | Microsoft Docs
description: 了解如何使用直接線路 API 3.0 版結束對話。
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 12/13/2017
ms.openlocfilehash: ac984609acfdd8f85088bd47ccded1f45e953b2c
ms.sourcegitcommit: f576981342fb3361216675815714e24281e20ddf
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/18/2018
ms.locfileid: "39299678"
---
# <a name="end-a-conversation"></a>結束對話

用戶端或 Bot 可以透過傳送 **endOfConversation** [活動](bot-framework-rest-connector-activities.md)來表示直接對話的結束。 

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

### <a name="response"></a>回應

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
