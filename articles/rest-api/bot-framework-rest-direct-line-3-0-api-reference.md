---
title: API 參考 - Direct Line API 3.0 | Microsoft Docs
description: 深入了解 Direct Line API 3.0 中的標題、HTTP 狀態碼、結構描述、作業和物件。
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 12/13/2017
ms.openlocfilehash: 2e47591b04a91ce02cfeb6bd6485080426d201b5
ms.sourcegitcommit: f576981342fb3361216675815714e24281e20ddf
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/18/2018
ms.locfileid: "39299198"
---
# <a name="api-reference---direct-line-api-30"></a>API 參考 - Direct Line API 3.0

使用 Direct Line API 3.0，您就能讓用戶端應用程式與 Bot 進行通訊。 Direct Line API 3.0 會透過 HTTPS 使用業界標準的 REST 和 JSON。

## <a name="base-uri"></a>基底 URI

若要存取 Direct Line API 3.0，請將此基底 URI 用於所有 API 要求：

`https://directline.botframework.com`

## <a name="headers"></a>headers

除了標準的 HTTP 要求標題之外，Direct Line API 要求必須包含指定祕密或權杖的 `Authorization` 標題，藉此驗證發出要求的用戶端。 請使用此格式指定 `Authorization` 標頭：

```http
Authorization: Bearer SECRET_OR_TOKEN
```

如需如何取得您的用戶端用來驗證其 Direct Line API 要求之祕密或權杖的詳細資料，請參閱[驗證](bot-framework-rest-direct-line-3-0-authentication.md)。

## <a name="http-status-codes"></a>HTTP 狀態碼

與每個回應一起傳回的 <a href="http://www.w3.org/Protocols/rfc2616/rfc2616-sec10.html" target="_blank">HTTP 狀態碼</a>會指出對應要求的結果。 

| HTTP 狀態碼 | 意義 |
|----|----|
| 200 | 要求成功。 |
| 201 | 要求成功。 |
| 202 | 已接受要求進行處理。 |
| 204 | 要求成功，但未傳回任何內容。 |
| 400 | 要求的格式不正確，或有其他錯誤。 |
| 401 | 用戶端未獲授權，無法提出要求。 此狀態碼出現的原因通常是遺漏 `Authorization` 標題或格式不正確。 |
| 403 | 不允許用戶端執行要求的作業。 如果要求指定先前有效但已過期的權杖，在 [ErrorResponse](bot-framework-rest-connector-api-reference.md#errorresponse-object) 物件內傳回之[錯誤](bot-framework-rest-connector-api-reference.md#error-object)的 `code` 屬性會設定為 `TokenExpired`。 |
| 404 | 找不到要求的資源。 此狀態碼通常表示要求的 URI 無效。 |
| 500 | Direct Line 服務內發生內部伺服器錯誤。 |
| 502 | Bot 無法使用，或傳回錯誤。 **此為常見錯誤碼。** |

> [!NOTE]
> HTTP 狀態碼 101 用於 WebSocket 連線路徑中，雖然這可能由您的 WebSocket 用戶端處理。

### <a name="errors"></a>Errors

若回應顯示的 HTTP 狀態碼位於 4xx 或 5xx 範圍內，則會在回應本文中納入 [ErrorResponse](bot-framework-rest-connector-api-reference.md#errorresponse-object) 物件，提供錯誤相關資訊。 如果您收到 4xx 範圍的錯誤訊息，請檢查 **ErrorResponse** 物件以找出錯誤原因，並在重新提交要求之前先解決問題。

> [!NOTE]
> **ErrorResponse** 物件內 `code` 屬性指定的 HTTP 狀態碼和值很穩定。 **ErrorResponse** 物件內 `message` 屬性指定的值可能會隨著時間改變。

下列程式碼片段顯示範例要求和產生的錯誤回應。

#### <a name="request"></a>要求

```http
POST https://directline.botframework.com/v3/directline/conversations/abc123/activities
[detail omitted]
```

#### <a name="response"></a>Response
```http
HTTP/1.1 502 Bad Gateway
[other headers]
```
```json
{
    "error": {
        "code": "BotRejectedActivity",
        "message": "Failed to send activity: bot returned an error"
    }
}
```

## <a name="token-operations"></a>權杖作業 
透過這些作業可建立或重新整理用戶端用於存取單一對話的權杖。

| 作業 | 說明 |
|----|----|
| [產生權杖](#generate-token) | 產生新對話的權杖。 | 
| [重新整理權杖](#refresh-token) | 重新整理權杖。 | 

### <a name="generate-token"></a>產生權杖
產生適用於一個對話的權杖。 
```http 
POST /v3/directline/tokens/generate
```

| | |
|----|----|
| **要求本文** | n/a |
| **傳回** | [Conversation](#conversation-object) 物件 | 

### <a name="refresh-token"></a>重新整理權杖
重新整理此權杖。 
```http 
POST /v3/directline/tokens/refresh
```

| | |
|----|----|
| **要求本文** | n/a |
| **傳回** | [Conversation](#conversation-object) 物件 | 

## <a name="conversation-operations"></a>對話作業 
使用這些作業可開啟您與 Bot 的對話，並在用戶端與 Bot 之間交換訊息。

| 作業 | 說明 |
|----|----|
| [開始對話](#start-conversation) | 開啟與 Bot 的新對話。 | 
| [取得對話資訊](#get-conversation-information) | 取得現有對話的相關資訊。 此作業會產生新的 WebSocket 資料流 URL，用戶端可以用來[重新連線](bot-framework-rest-direct-line-3-0-reconnect-to-conversation.md)對話。 |
| [取得活動](#get-activities) | 從 Bot 擷取活動。 |
| [傳送活動](#send-an-activity) | 將活動傳送到 Bot。 | 
| [上傳與傳送檔案](#upload-send-files) | 上傳檔案後，以附件型式傳送檔案。 |

### <a name="start-conversation"></a>開始對話
開啟與 Bot 的新對話。 
```http 
POST /v3/directline/conversations
```

| | |
|----|----|
| **要求本文** | n/a |
| **傳回** | [Conversation](#conversation-object) 物件 | 

### <a name="get-conversation-information"></a>取得對話資訊
取得現有對話的相關資訊，並且產生新的 WebSocket 資料流 URL，用戶端可以用來[重新連線](bot-framework-rest-direct-line-3-0-reconnect-to-conversation.md)對話。 您可以選擇性地在要求 URI 中提供 `watermark` 參數，以指出最近期內用戶端所見到的訊息。
```http 
GET /v3/directline/conversations/{conversationId}?watermark={watermark_value}
```

| | |
|----|----|
| **要求本文** | n/a |
| **傳回** | [Conversation](#conversation-object) 物件 | 

### <a name="get-activities"></a>取得活動
針對指定對話從 Bot 擷取活動。 您可以選擇性地在要求 URI 中提供 `watermark` 參數，以指出最近期內用戶端所見到的訊息。 

```http 
GET /v3/directline/conversations/{conversationId}/activities?watermark={watermark_value}
```

| | |
|----|----|
| **要求本文** | n/a |
| **傳回** | [ActivitySet](#activityset-object) 物件。 回應中包含 `watermark` 作為 `ActivitySet` 物件的屬性。 用戶端應隨 `watermark` 值向前逐頁查詢可用的活動，直到沒有活動傳回。 | 

### <a name="send-an-activity"></a>傳送活動
將活動傳送到 Bot。 
```http 
POST /v3/directline/conversations/{conversationId}/activities
```

| | |
|----|----|
| **要求本文** | [Activity](bot-framework-rest-connector-api-reference.md#activity-object) 物件 |
| **傳回** | [ResourceResponse](bot-framework-rest-connector-api-reference.md#resourceresponse-object)，其中包含 `id` 屬性，指定已傳送給 Bot 的活動識別碼。 | 

### <a id="upload-send-files"></a> 上傳與傳送檔案
上傳檔案後，以附件型式傳送檔案。 設定要求 URI 中的 `userId` 參數以指定正在傳送附件之使用者的識別碼。
```http 
POST /v3/directline/conversations/{conversationId}/upload?userId={userId}
```

| | |
|----|----|
| **要求本文** | 若為單一附件，請在要求本文填入檔案內容。 若有多個附件，請建立多部分要求本文，包含一個部分是針對每個附件，以及 (選擇性) 一個部分是針對應該作為指定附件之容器的 [Activity](bot-framework-rest-connector-api-reference.md#activity-object) 物件。 如需詳細資訊，請參閱[將活動傳送到 Bot](bot-framework-rest-direct-line-3-0-send-activity.md)。 |
| **傳回** | [ResourceResponse](bot-framework-rest-connector-api-reference.md#resourceresponse-object)，其中包含 `id` 屬性，指定已傳送給 Bot 的活動識別碼。 | 

> [!NOTE]
> 上傳的檔案會在 24 小時後刪除。

## <a name="schema"></a>結構描述

Direct Line 3.0 結構描述包含 [Bot Framework v3 結構描述](bot-framework-rest-connector-api-reference.md#objects)所定義的所有物件，以及 `ActivitySet` 物件和 `Conversation` 物件。

### <a name="activityset-object"></a>ActivitySet 物件 
定義一組活動。<br/><br/>

| 屬性 | 類型 | 說明 |
|----|----|----|
| **activities** | [Activity](bot-framework-rest-connector-api-reference.md#activity-object)[] | **Activity** 物件的陣列。 |
| **watermark** | 字串 | 一組訊息中，活動的最大浮水印。 用戶端可以使用 `watermark` 值，指出它在[從 Bot 擷取活動](bot-framework-rest-direct-line-3-0-receive-activities.md#http-get)時，或在[產生新的 WebSocket 資料流 URL](bot-framework-rest-direct-line-3-0-reconnect-to-conversation.md) 時，已看到的最新訊息。 |

### <a name="conversation-object"></a>Conversation 物件
定義 Direct Line 對話。<br/><br/>

| 屬性 | 類型 | 說明 |
|----|----|----|
| **conversationId** | 字串 | 唯一識別指定權杖所適用對話的識別碼。 |
| **expires_in** | number | 權杖到期之前的秒數。 |
| **streamUrl** | 字串 | 對話的訊息資料流 URL。 |
| **token** | 字串 | 適用於指定對話的權杖。 |

### <a name="activities"></a>活動

針對用戶端透過 Direct Line 從 Bot 收到的每個 [Activity](bot-framework-rest-connector-api-reference.md#activity-object)：

- 會保留附件卡片。
- 已上傳附件的 URL 會以私密連結隱藏。
- `channelData` 屬性已保留而不修改。 

用戶端可能從 Bot [收到](bot-framework-rest-direct-line-3-0-receive-activities.md)多個活動，作為 [ActivitySet](#activityset-object) 的一部分。 

當用戶端透過 Direct Line 傳送 [Activity](bot-framework-rest-connector-api-reference.md#activity-object) 給 Bot：

- `type` 屬性會指定它傳送之活動的類型 (通常是**訊息**)。
- `from` 屬性必須填入使用者識別碼，由用戶端選擇。
- 附件可能包含現有資源的 URL，或透過 Direct Line 附件端點上傳的 URL。
- `channelData` 屬性已保留而不修改。

對於每個要求，用戶端可[傳送](bot-framework-rest-direct-line-3-0-send-activity.md)一個活動。 
