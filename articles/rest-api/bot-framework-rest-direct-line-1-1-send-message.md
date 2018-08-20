---
title: 將訊息傳送至 Bot | Microsoft Docs
description: 了解如何使用 Direct Line API v1.1 將訊息傳送至 Bot。
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 12/13/2017
ms.openlocfilehash: 3bc56d08f45ffd1e389a2dca1868a788d65e087e
ms.sourcegitcommit: f576981342fb3361216675815714e24281e20ddf
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/18/2018
ms.locfileid: "39300214"
---
# <a name="send-a-message-to-the-bot"></a>將訊息傳送至 Bot

> [!IMPORTANT]
> 本文說明如何使用 Direct Line API 1.1，將訊息傳送至 Bot。 如果要在用戶端應用程式與 Bot 之間建立新連線，請改為使用 [Direct Line API 3.0](bot-framework-rest-direct-line-3-0-send-activity.md)。

使用 Direct Line 1.1 通訊協定，用戶端可以與 Bot 交換訊息。 這些訊息會轉換成 Bot 支援 (Bot Framework v1 或 Bot Framework v3) 的結構描述。 對於每個要求，用戶端可傳送一個訊息。 

## <a name="send-a-message"></a>傳送訊息

若要將訊息傳送至 Bot，用戶端必須建立[訊息](bot-framework-rest-direct-line-1-1-api-reference.md#message-object)物件以定義訊息，然後在要求本文中指定訊息物件，並對 `https://directline.botframework.com/api/conversations/{conversationId}/messages` 發出 `POST` 要求。

下列程式碼片段提供「傳送訊息」要求和回應的範例。

### <a name="request"></a>要求

```http
POST https://directline.botframework.com/api/conversations/abc123/messages
Authorization: Bearer RCurR_XV9ZA.cwA.BKA.iaJrC8xpy8qbOF5xnR2vtCX7CZj0LdjAPGfiCpg4Fv0
[other headers]
```

```json
{
  "text": "hello",
  "from": "user1"
}
```

### <a name="response"></a>Response

當訊息傳遞至 Bot 時，服務會以可反映 Bot 狀態碼的 HTTP 狀態碼來回應。 如果 Bot 產生錯誤，系統會將 HTTP 500 回應 (「內部伺服器錯誤」) 傳回至用戶端，以回應其「傳送訊息」要求。 如果 POST 成功，服務就會傳回 HTTP 204 狀態碼。 回應的本文中沒有任何傳回的資料。 用戶端的訊息和來自 Bot 的任何訊息都，可以透過[輪詢](bot-framework-rest-direct-line-1-1-receive-messages.md)取得。 

```http
HTTP/1.1 204 No Content
[other headers]
```

### <a name="total-time-for-the-send-message-requestresponse"></a>傳送訊息要求/回應的時間總計

將訊息 POST 至 Direct Line 對話的時間總計，是以下幾項的總和：

- HTTP 要求從用戶端到 Direct Line 服務的傳輸時間
- Direct Line 內的內部處理時間 (通常少於 120 毫秒)
- 從 Direct Line 服務到 Bot 的傳輸時間
- Bot 內的處理時間
- HTTP 回應回到用戶端的傳輸時間

## <a name="send-attachments-to-the-bot"></a>將附件傳送至 Bot

在某些情況下，用戶端可能需要將影像或文件之類的附件傳送至 Bot。 用戶端可藉由在它要使用 `POST /api/conversations/{conversationId}/messages` 傳送的[訊息](bot-framework-rest-direct-line-1-1-api-reference.md#message-object)物件內為附件[指定 URL](#send-by-url)，或藉由使用 `POST /api/conversations/{conversationId}/upload` 來[上傳附件](#upload-attachments)，將附件傳送至 Bot。

## <a id="send-by-url"></a>依 URL 傳送附件

若要使用 `POST /api/conversations/{conversationId}/messages` 傳送一或多個附件作為[訊息](bot-framework-rest-direct-line-1-1-api-reference.md#message-object)物件的一部分，請在訊息的 `images` 陣列和/或 `attachments` 陣列內指定附件 URL。

## <a id="upload-attachments"></a>藉由上傳來傳送附件

常常用戶端的裝置上有影像或文件要傳送至 Bot，但卻沒有對應至這些檔案的 URL。 在此情況下，用戶端可以發出 `POST /api/conversations/{conversationId}/upload` 要求，以藉由上傳將附件傳送至 Bot。 要求的格式和內容，將取決於用戶端是要[傳送單一附件](#upload-one-attachment)還是[傳送多個附件](#upload-multiple-attachments)。

### <a id="upload-one-attachment"></a>藉由上傳來傳送單一附件

若要藉由上傳來傳送單一附件，請發出下列要求： 

```http
POST https://directline.botframework.com/api/conversations/{conversationId}/upload?userId={userId}
Authorization: Bearer SECRET_OR_TOKEN
Content-Type: TYPE_OF_ATTACHMENT
Content-Disposition: ATTACHMENT_INFO
[other headers]

[file content]
```

在此要求 URI 中，請將 **{conversationId}** 取代為對話的識別碼，並將 **{userId}** 取代為訊息傳送者的使用者識別碼。 在要求標頭中，請設定 `Content-Type` 以指定附件的類型，並設定 `Content-Disposition` 以指定附件的檔案名稱。

下列程式碼片段提供「傳送 (單一) 附件」要求和回應的範例。

#### <a name="request"></a>要求

```http
POST https://directline.botframework.com/api/conversations/abc123/upload?userId=user1
Authorization: Bearer RCurR_XV9ZA.cwA.BKA.iaJrC8xpy8qbOF5xnR2vtCX7CZj0LdjAPGfiCpg4Fv0
Content-Type: image/jpeg
Content-Disposition: name="file"; filename="badjokeeel.jpg"
[other headers]

[JPEG content]
```

#### <a name="response"></a>Response

如果要求成功，會在上傳完成時傳送訊息至 Bot，且服務會傳回 HTTP 204 狀態碼。

```http
HTTP/1.1 204 No Content
[other headers]
```

### <a id="upload-multiple-attachments"></a>藉由上傳來傳送多個附件

若要藉由上傳來傳送多個附件，請將有多個部分的要求 `POST` 至 `/api/conversations/{conversationId}/upload` 端點。 請將要求的 `Content-Type` 標頭設定為 `multipart/form-data`，並且為每個部分納入 `Content-Type` 標頭和 `Content-Disposition` 標頭，以指定每個附件的類型和檔案名稱。 在要求 URI 中，請將 `userId` 參數設定為訊息傳送者的使用者識別碼。 

您可以藉由新增一個會指定 `Content-Type` 標頭值 `application/vnd.microsoft.bot.message` 的部分，在要求內納入[訊息](bot-framework-rest-direct-line-1-1-api-reference.md#message-object)物件。 這可讓用戶端自訂包含附件的訊息。 如果要求中包含訊息，則承載的其他部分所指定的附件會先新增為該訊息的附件，然後才會傳送。 

下列程式碼片段提供「傳送 (多個) 附件」要求和回應的範例。 在此範例中，要求會傳送包含一些文字和單一影像附件的訊息。 在要求中可以新增其他部分，以將多個附件包含在此訊息中。

#### <a name="request"></a>要求

```http
POST https://directline.botframework.com/api/conversations/abc123/upload?userId=user1
Authorization: Bearer RCurR_XV9ZA.cwA.BKA.iaJrC8xpy8qbOF5xnR2vtCX7CZj0LdjAPGfiCpg4Fv0
Content-Type: multipart/form-data; boundary=----DD4E5147-E865-4652-B662-F223701A8A89
[other headers]

----DD4E5147-E865-4652-B662-F223701A8A89
Content-Type: image/jpeg
Content-Disposition: form-data; name="file"; filename="badjokeeel.jpg"
[other headers]

[JPEG content]

----DD4E5147-E865-4652-B662-F223701A8A89
Content-Type: application/vnd.microsoft.bot.message
[other headers]

{
  "text": "Hey I just IM'd you\n\nand this is crazy\n\nbut here's my webhook\n\nso POST me maybe",
  "from": "user1"
}

----DD4E5147-E865-4652-B662-F223701A8A89
```

#### <a name="response"></a>Response

如果要求成功，會在上傳完成時傳送訊息至 Bot，且服務會傳回 HTTP 204 狀態碼。

```http
HTTP/1.1 204 No Content
[other headers]
```

## <a name="additional-resources"></a>其他資源

- [重要概念](bot-framework-rest-direct-line-1-1-concepts.md)
- [驗證](bot-framework-rest-direct-line-1-1-authentication.md)
- [開始對話](bot-framework-rest-direct-line-1-1-start-conversation.md)
- [從 Bot 接收訊息](bot-framework-rest-direct-line-1-1-receive-messages.md)