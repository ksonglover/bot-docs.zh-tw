---
title: API 參照 - 直接線路 API 1.1 | Microsoft Docs
description: 深入了解標頭、HTTP 狀態碼、結構描述、作業以及直接線路 API 1.1 中的物件。
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 12/13/2017
ms.openlocfilehash: 3607957cd5cb8738e8268ece6eba4417250bc596
ms.sourcegitcommit: b78fe3d8dd604c4f7233740658a229e85b8535dd
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/24/2018
ms.locfileid: "49997956"
---
# <a name="api-reference---direct-line-api-11"></a>API 參照 - 直接線路 API 1.1

> [!IMPORTANT]
> 本文包含直接線路 API 1.1 的參照資訊。 如果要在用戶端應用程式與 Bot 之間建立新連線，請改為使用[直接線路 API 3.0](bot-framework-rest-direct-line-3-0-api-reference.md)。

使用直接線路 API 1.1，您就能讓用戶端應用程式與 Bot 進行通訊。 直接線路 API 1.1 會透過 HTTPS 使用業界標準的 REST 和 JSON。

## <a name="base-uri"></a>基底 URI

若要存取直接線路 API 1.1，請將此基底 URI 使用於所有 API 要求：

`https://directline.botframework.com`

## <a name="headers"></a>headers

除了標準的 HTTP 要求標頭之外，直接線路 API 要求必須包含指定密碼或權杖的 `Authorization` 標頭，藉此驗證發出要求的用戶端。 您可以指定讓 `Authorization` 標頭使用「持有人」配置或「BotConnector」配置。 

**持有人配置**：
```http
Authorization: Bearer SECRET_OR_TOKEN
```

**BotConnector 配置**：
```http
Authorization: BotConnector SECRET_OR_TOKEN
```

如需如何取得您的用戶端用來驗證其 Direct Line API 要求之祕密或權杖的詳細資料，請參閱[驗證](bot-framework-rest-direct-line-1-1-authentication.md)。

## <a name="http-status-codes"></a>HTTP 狀態碼

與每個回應一併傳回的 <a href="http://www.w3.org/Protocols/rfc2616/rfc2616-sec10.html" target="_blank">HTTP 狀態碼</a>會指出對應要求的結果。 

| HTTP 狀態碼 | 意義 |
|----|----|
| 200 | 要求成功。 |
| 204 | 要求成功，但未傳回任何內容。 |
| 400 | 要求的格式不正確或有其他錯誤。 |
| 401 | 用戶端未獲授權，無法提出要求。 此狀態碼出現的原因通常是遺漏 `Authorization` 標頭或格式不正確。 |
| 403 | 不允許用戶端執行要求的作業。 此狀態碼出現的原因通常是 `Authorization` 標頭指定了無效的權杖或密碼。 |
| 404 | 找不到要求的資源。 此狀態碼通常表示要求的 URI 無效。 |
| 500 | Direct Line 服務內發生內部伺服器錯誤 |
| 502 | Bot 內發生錯誤；Bot 無法使用，或傳回錯誤。  **此為常見錯誤碼。** |

## <a name="token-operations"></a>權杖作業 
透過這些作業可建立或重新整理用戶端用於存取單一對話的權杖。

| 作業 | 說明 |
|----|----|
| [產生權杖](#generate-token) | 產生新對話的權杖。 | 
| [重新整理權杖](#refresh-token) | 重新整理權杖。 | 

### <a name="generate-token"></a>產生權杖
產生適用於一個對話的權杖。 
```http 
POST /api/tokens/conversation
```

| | |
|----|----|
| **要求本文** | n/a |
| **傳回** | 代表權杖的字串 | 

### <a name="refresh-token"></a>重新整理權杖
重新整理此權杖。 
```http 
GET /api/tokens/{conversationId}/renew
```

| | |
|----|----|
| **要求本文** | n/a |
| **傳回** | 代表新權杖的字串 | 

## <a name="conversation-operations"></a>交談作業 
使用這些作業可開啟您與 Bot 的交談，並在用戶端與 Bot 之間交換訊息。

| 作業 | 說明 |
|----|----|
| [開始對話](#start-conversation) | 開啟與 Bot 的新對話。 | 
| [取得訊息](#get-messages) | 接收來自 Bot 的訊息。 |
| [傳送訊息](#send-a-message) | 將訊息傳送至 Bot。 | 
| [上傳與傳送檔案](#upload-send-files) | 上傳檔案後，以附件型式傳送檔案。 |

### <a name="start-conversation"></a>開始對話
開啟與 Bot 的新對話。 
```http 
POST /api/conversations
```

| | |
|----|----|
| **要求本文** | n/a |
| **傳回** | [Conversation](#conversation-object) 物件 | 

### <a name="get-messages"></a>取得訊息
針對指定交談擷取來自 Bot 的訊息。 設定 `watermark` 要求 URI 中的參數，以指出最近期內用戶端所見到的訊息。 

```http
GET /api/conversations/{conversationId}/messages?watermark={watermark_value}
```

| | |
|----|----|
| **要求本文** | n/a |
| **傳回** | [MessageSet](#messageset-object) 物件。 回應中包含 `watermark` 做為 `MessageSet` 物件的屬性。 用戶端應持續送出 `watermark` 值來瀏覽整頁的可用訊息，直至不再收到任何訊息。 | 

### <a name="send-a-message"></a>傳送訊息
將訊息傳送至 Bot。 
```http 
POST /api/conversations/{conversationId}/messages
```

| | |
|----|----|
| **要求本文** | [訊息](#message-object)物件 |
| **傳回** | 回應的本文中沒有任何傳回的資料。 訊息成功傳送時，服務會以 HTTP 204 狀態碼回應。 藉由運用[取得訊息](#get-messages)作業，用戶端可能會取得其所傳送的訊息 (以及任何 Bot 傳送至用戶端的訊息)。 |

### <a id="upload-send-files"></a>上傳與傳送檔案
上傳檔案後，以附件型式傳送檔案。 設定要求 URI 中的 `userId` 參數以定傳送附件之使用者的識別碼。
```http 
POST /api/conversations/{conversationId}/upload?userId={userId}
```

| | |
|----|----|
| **要求本文** | 若為單一附件，請在要求本文填入檔案內容。 若有多個附件，請建立多部分要求本文，每個附件均包含其各自的一部份；(選擇性) 亦可採用屬於做為指定附件之容器的[訊息](#message-object)物件的單一部分的方式。 如需詳細資訊，請參閱[將訊息傳送至 Bot](bot-framework-rest-direct-line-1-1-send-message.md)。 |
| **傳回** | 回應的本文中沒有任何傳回的資料。 訊息成功傳送時，服務會以 HTTP 204 狀態碼回應。 藉由運用[取得訊息](#get-messages)作業，用戶端可能會取得其所傳送的訊息 (以及任何 Bot 傳送至用戶端的訊息)。 | 

> [!NOTE]
> 上傳的檔案會在 24 小時後刪除。

## <a name="schema"></a>結構描述

直接線路 1.1 結構描述是 Bot Framework v1 結構描述的簡化版本，其中包含下列物件。

### <a name="message-object"></a>訊息物件

用於定義用戶端與 Bot 之間來回傳送的訊息。

| 屬性 | 類型 | 說明 |
|----|----|----|
| **id** | 字串 | 唯一識別由直接路線所指派之訊息的識別碼。 | 
| **conversationId** | 字串 | 可識別交談的識別碼。  | 
| **created** | 字串 | 訊息建立的日期和時間，以 <a href="https://en.wikipedia.org/wiki/ISO_8601" target="_blank">ISO-8601</a> 格式表示。 | 
| **from** | 字串 | 可識別使用者是否為訊息寄件者的識別碼。 用戶端在建立訊息時，必須對固定使用者識別碼設定這項屬性。 雖然直接線路會在未提供使用者識別碼時指派一個識別碼，不過，如此一來，通常會導致非預期的行為。 | 
| **text** | 字串 | Bot 與使用者之間傳送的訊息文字。 | 
| **channelData** | 物件 | 包含通道專用內容的物件。 某些通道提供的功能，需要使用無法以附件結構描述來呈現的其他資訊。 在該等情況下，請將此屬性設定為通道文件中所定義的通道專用內容。 此資料會在未經修改的狀態下，在用戶端和 Bot 之間傳送。 這個屬性必須設為複雜物件或保留空白。 請勿設為字串、數字或其他簡單類型。 | 
| **images** | string[] | 字串陣列，當中包含訊息中所含影像的 URL。 某些情況下，此陣列中的字串可能是相對 URL。 如果此陣列中的任何字串並未以「http」或「https」開頭，請在字串前加上 `https://directline.botframework.com` 以構成完整 URL。 | 
| **attachments** | [Attachment](#attachment-object)[] | 呈現訊息中所含非影像附件之**附件**物件陣列。 陣列中的每個物件均具有 `url` 屬性和 `contentType` 屬性。 用戶端從 Bot 接收的訊息中，`url` 屬性有時會指定相對 URL。 對於任何未以「http」或「https」開頭的 `url` 屬性值，請在字串前加上 `https://directline.botframework.com` 以構成完整 URL。 | 

下列範例為含有所有可能屬性的訊息物件。 大部分情況下，建立一則訊息時，用戶端只需提供 `from` 屬性和至少一個內容屬性 (例如 `text`、`images`、`attachments` 或 `channelData`)。

```json
{
    "id": "CuvLPID4kDb|000000000000000004",
    "conversationId": "CuvLPID4kDb",
    "created": "2016-10-28T21:19:51.0357965Z",
    "from": "examplebot",
    "text": "Hello!",
    "channelData": {
        "examplefield": "abc123"
    },
    "images": [
        "/attachments/CuvLPID4kDb/0.jpg?..."
    ],
    "attachments": [
        {
            "url": "https://example.com/example.docx",
            "contentType": "application/vnd.openxmlformats-officedocument.wordprocessingml.document"
        }, 
        {
            "url": "https://example.com/example.doc",
            "contentType": "application/msword"
        }
    ]
}
```

### <a name="messageset-object"></a>MessageSet 物件 
用於定義一組訊息。<br/><br/>

| 屬性 | 類型 | 說明 |
|----|----|----|
| **messages** | [訊息](#message-object)[] | **訊息**物件陣列。 |
| **watermark** | 字串 | 一組訊息中，訊息的最大浮水印。 用戶端可以在[擷取 Bot 的訊息時](bot-framework-rest-direct-line-1-1-receive-messages.md)，使用 `watermark` 值指出從 Bot 接收的最新訊息。 |

### <a name="attachment-object"></a>Attachment 物件
用於定義非影像附件。<br/><br/> 

| 屬性 | 類型 | 說明 |
|----|----|----|
| **contentType** | 字串 | 附件中內容的媒體類型。 |
| **url** | 字串 | 附件內容的 URL。 |

### <a name="conversation-object"></a>交談物件
定義 Direct Line 對話。<br/><br/>

| 屬性 | 類型 | 說明 |
|----|----|----|
| **conversationId** | 字串 | 唯一識別指定權杖所適用對話的識別碼。 |
| **token** | 字串 | 適用於指定對話的權杖。 |
| **expires_in** | number | 權杖到期之前的秒數。 |

### <a name="error-object"></a>錯誤物件
定義錯誤。<br/><br/> 

| 屬性 | 類型 | 說明 |
|----|----|----|
| **code** | 字串 | 錯誤碼。 下列其中一項值：**MissingProperty**、**MalformedData**、**NotFound**、**ServiceError**、**Internal**、**InvalidRange**、**NotSupported**、**NotAllowed**、**BadCertificate**。 |
| **message** | 字串 | 錯誤的描述。 |
| **statusCode** | number | 狀態碼。 |

### <a name="errormessage-object"></a>ErrorMessage 物件
標準化訊息的錯誤承載。<br/><br/> 


|        屬性        |          類型          |                                 說明                                 |
|------------------------|------------------------|-----------------------------------------------------------------------------|
| <strong>error</strong> | [錯誤](#error-object) | 含有錯誤相關資訊的 <strong>Error</strong> 物件。 |

