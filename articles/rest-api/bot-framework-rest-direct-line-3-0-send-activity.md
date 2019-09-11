---
title: 將活動傳送至 Bot | Microsoft Docs
description: 了解如何使用 Direct Line API v3.0 將活動傳送至 Bot。
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 12/13/2017
ms.openlocfilehash: 5c7ac61da6c2e0d09fb2f8dc4cd0bf3961bcfc4f
ms.sourcegitcommit: e815e786413296deea0bd78e5a495df329a9a7cb
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/10/2019
ms.locfileid: "70876000"
---
# <a name="send-an-activity-to-the-bot"></a>將活動傳送至 Bot

使用 Direct Line 3.0 通訊協定時，用戶端與 Bot 可交換許多不同類型的[活動](https://aka.ms/botSpecs-activitySchema)，包括**訊息**活動、**輸入**活動，以及 Bot 所支援的自訂活動。 對於每個要求，用戶端可傳送一個活動。 

## <a name="send-an-activity"></a>傳送活動

若要將活動傳送至 Bot，用戶端必須建立[活動][]物件以定義活動，然後在要求本文中指定活動物件，並對 `https://directline.botframework.com/v3/directline/conversations/{conversationId}/activities` 發出 `POST` 要求。

下列程式碼片段提供「傳送活動」要求和回應的範例。

### <a name="request"></a>要求

```http
POST https://directline.botframework.com/v3/directline/conversations/abc123/activities
Authorization: Bearer RCurR_XV9ZA.cwA.BKA.iaJrC8xpy8qbOF5xnR2vtCX7CZj0LdjAPGfiCpg4Fv0
Content-Type: application/json
[other headers]
```

```json
{
    "type": "message",
    "from": {
        "id": "user1"
    },
    "text": "hello"
}
```

### <a name="response"></a>Response

當活動傳遞至 Bot 時，服務會以反映 Bot 狀態碼的 HTTP 狀態碼來回應。 如果 Bot 產生錯誤，系統會將 HTTP 502 回應 (「錯誤的閘道」) 傳回至用戶端，以回應其「傳送活動」要求。

> [!NOTE]
> 這可能是因為未使用正確的權杖所造成。 只有針對*開始對話*所接收的權杖可以用來傳送活動。

如果 POST 成功，則回應會包含 JSON 承載，指定已傳送至 Bot 的活動識別碼。

```http
HTTP/1.1 200 OK
[other headers]
```

```json
{
    "id": "0001"
}
```

### <a name="total-time-for-the-send-activity-requestresponse"></a>傳送活動要求/回應的時間總計

將訊息發佈至 Direct Line 交談的時間總計，是以下幾項的總和：

- HTTP 要求從用戶端到 Direct Line 服務的傳輸時間
- Direct Line 中的內部處理時間 (通常少於 120 毫秒)
- 從 Direct Line 服務到 Bot 的傳輸時間
- Bot 內的處理時間
- HTTP 回應回到用戶端的傳輸時間

## <a name="send-attachments-to-the-bot"></a>將附件傳送至 Bot

在某些情況下，用戶端可能需要將影像或文件之類的附件傳送至 Bot。 用戶端可藉由在它要使用 `POST /v3/directline/conversations/{conversationId}/activities` 傳送的[活動][]物件內為附件[指定 URL](#send-by-url)，或藉由使用 `POST /v3/directline/conversations/{conversationId}/upload` 來[上傳附件](#upload-attachments)，將附件傳送至 Bot。

## <a id="send-by-url"></a>依 URL 傳送附件

若要使用 `POST /v3/directline/conversations/{conversationId}/activities` 傳送隨附於[活動][]物件的一或多個附件，只需將一或多個[附件][]物件包含在活動物件內，並設定每個附件物件的 `contentUrl` 屬性以指定附件的 HTTP、HTTPS 或 `data` URI 即可。

## <a id="upload-attachments"></a>藉由上傳來傳送附件

常常用戶端的裝置上有影像或文件要傳送至 Bot，但卻沒有對應至這些檔案的 URL。 在此情況下，用戶端可以發出 `POST /v3/directline/conversations/{conversationId}/upload` 要求，以藉由上傳將附件傳送至 Bot。 要求的格式和內容，將取決於用戶端是要[傳送單一附件](#upload-one-attachment)還是[傳送多個附件](#upload-multiple-attachments)。

### <a id="upload-one-attachment"></a>藉由上傳來傳送單一附件

若要藉由上傳來傳送單一附件，請發出下列要求： 

```http
POST https://directline.botframework.com/v3/directline/conversations/{conversationId}/upload?userId={userId}
Authorization: Bearer SECRET_OR_TOKEN
Content-Type: TYPE_OF_ATTACHMENT
Content-Disposition: ATTACHMENT_INFO
[other headers]

[file content]
```

在此要求 URI 中，請將 **{conversationId}** 取代為交談的識別碼，並將 **{userId}** 取代為訊息傳送者的使用者識別碼。 `userId` 是必要參數。 在要求標頭中，請設定 `Content-Type` 以指定附件的類型，並設定 `Content-Disposition` 以指定附件的檔案名稱。

下列程式碼片段提供「傳送 (單一) 附件」要求和回應的範例。

#### <a name="request"></a>要求

```http
POST https://directline.botframework.com/v3/directline/conversations/abc123/upload?userId=user1
Authorization: Bearer RCurR_XV9ZA.cwA.BKA.iaJrC8xpy8qbOF5xnR2vtCX7CZj0LdjAPGfiCpg4Fv0
Content-Type: image/jpeg
Content-Disposition: name="file"; filename="badjokeeel.jpg"
[other headers]

[JPEG content]
```

#### <a name="response"></a>Response

如果要求成功，將會在上傳完成後將**訊息**活動傳送至 Bot，且用戶端收到的回應將會包含已傳送的活動識別碼。

```http
HTTP/1.1 200 OK
[other headers]
```

```json
{
  "id": "0003"
}
```

### <a id="upload-multiple-attachments"></a>藉由上傳來傳送多個附件

若要藉由上傳來傳送多個附件，請將有多個部分的要求 `POST` 至 `/v3/directline/conversations/{conversationId}/upload` 端點。 請將要求的 `Content-Type` 標頭設定為 `multipart/form-data`，並且為每個部分納入 `Content-Type` 標頭和 `Content-Disposition` 標頭，以指定每個附件的類型和檔案名稱。 在要求 URI 中，請將 `userId` 參數設定為訊息傳送者的使用者識別碼。 

您可以藉由新增一個指定 `Content-Type` 標頭值 `application/vnd.microsoft.activity` 的部分，在要求內納入 `Activity` 物件。 如果要求中包含活動，則承載的其他部分所指定的附件會先新增為該活動的附件，然後才會傳送。 如果要求未包含活動，則會建立空的活動，作為指定的附件藉以傳送的容器。

下列程式碼片段提供「傳送 (多個) 附件」要求和回應的範例。 在此範例中，要求會傳送包含一些文字和單一影像附件的訊息。 在要求中可以新增其他部分，以將多個附件包含在此訊息中。

#### <a name="request"></a>要求

```http
POST https://directline.botframework.com/v3/directline/conversations/abc123/upload?userId=user1
Authorization: Bearer RCurR_XV9ZA.cwA.BKA.iaJrC8xpy8qbOF5xnR2vtCX7CZj0LdjAPGfiCpg4Fv0
Content-Type: multipart/form-data; boundary=----DD4E5147-E865-4652-B662-F223701A8A89
[other headers]

----DD4E5147-E865-4652-B662-F223701A8A89
Content-Type: image/jpeg
Content-Disposition: form-data; name="file"; filename="badjokeeel.jpg"
[other headers]

[JPEG content]

----DD4E5147-E865-4652-B662-F223701A8A89
Content-Type: application/vnd.microsoft.activity
[other headers]

{
  "type": "message",
  "from": {
    "id": "user1"
  },
  "text": "Hey I just IM'd you\n\nand this is crazy\n\nbut here's my webhook\n\nso POST me maybe"
}

----DD4E5147-E865-4652-B662-F223701A8A89
```

#### <a name="response"></a>Response

如果要求成功，將會在上傳完成後將訊息活動傳送至 Bot，且用戶端收到的回應將會包含已傳送的活動識別碼。

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
- [開始對話](bot-framework-rest-direct-line-3-0-start-conversation.md)
- [重新連線至交談](bot-framework-rest-direct-line-3-0-reconnect-to-conversation.md)
- [從 Bot 接收活動](bot-framework-rest-direct-line-3-0-receive-activities.md)
- [結束對話](bot-framework-rest-direct-line-3-0-end-conversation.md)
- [Bot Framework -- 活動結構描述](https://aka.ms/botSpecs-activitySchema)

[活動]: bot-framework-rest-connector-api-reference.md#activity-object
[附件]: bot-framework-rest-connector-api-reference.md#attachment-object
