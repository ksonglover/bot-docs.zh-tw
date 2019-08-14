---
title: API 參考資料 | Microsoft Docs
description: 了解 Bot 連接器服務和 Bot 狀態服務中的標頭、作業、物件及錯誤。
author: ivorb
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 08/02/2019
ms.openlocfilehash: 5a520c2fc5b9e94976e9a2618286aa184045bdeb
ms.sourcegitcommit: 6a83b2c8ab2902121e8ee9531a7aa2d85b827396
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/09/2019
ms.locfileid: "68866635"
---
# <a name="api-reference"></a>API 參考資料

> [!NOTE]
> REST API 不等同於 SDK。 REST API 是為了支援標準 REST 通訊，而 SDK 則是與 Bot Framework 通訊的慣用方法。 

在 Bot Framework 內，Bot 連接器服務可讓您的 Bot 透過在 Bot Framework 入口網站中設定的通道與使用者交換訊息。 此服務會透過 HTTPS 使用業界標準的 REST 和 JSON。

## <a name="base-uri"></a>基底 URI

使用者傳送訊息給 Bot 時，傳入要求將包含一個[活動](#bot-framework-activity-schema)物件，以及一個用於指定端點以接收 Bot 回應的 `serviceUrl` 屬性。 若要存取 Bot 連接器服務，請將 `serviceUrl` 值作為 API 要求的基底 URI。 

例如，假設使用者傳送訊息給您的 Bot 時，Bot 收到了下列活動。

```json
{
    "type": "message",
    "id": "bf3cc9a2f5de...",
    "timestamp": "2016-10-19T20:17:52.2891902Z",
    "serviceUrl": "https://smba.trafficmanager.net/apis",
    "channelId": "channel's name/id",
    "from": {
        "id": "1234abcd",
        "name": "user's name"
    },
    "conversation": {
        "id": "abcd1234",
        "name": "conversation's name"
    },
    "recipient": {
        "id": "12345678",
        "name": "bot's name"
    },
    "text": "Haircut on Saturday"
}
```

使用者訊息內的 `serviceUrl` 屬性指出 Bot 應將回應傳送至端點 `https://smba.trafficmanager.net/apis`；而這就會是 Bot 在該交談內容中發出任何後續要求時所用的基底 URI。 如果您的 Bot 需要傳送主動式訊息給使用者，請務必儲存 `serviceUrl` 的值。

下列範例說明由 Bot 發出的要求，該要求乃是用於回應使用者訊息。 

```http
POST https://smba.trafficmanager.net/apis/v3/conversations/abcd1234/activities/bf3cc9a2f5de... 
Authorization: Bearer eyJhbGciOiJIUzI1Ni...
Content-Type: application/json
```

```json
{
    "type": "message",
    "from": {
        "id": "12345678",
        "name": "bot's name"
    },
    "conversation": {
        "id": "abcd1234",
        "name": "conversation's name"
    },
   "recipient": {
        "id": "1234abcd",
        "name": "user's name"
    },
    "text": "I have several times available on Saturday!",
    "replyToId": "bf3cc9a2f5de..."
}
```

## <a name="headers"></a>headers

### <a name="request-headers"></a>要求標頭

除了標準的 HTTP 要求標頭，您發出的每個 API 要求都必須包含指定存取權杖的 `Authorization` 標頭以驗證您的 Bot。 使用此格式指定 `Authorization` 標頭：

```http
Authorization: Bearer ACCESS_TOKEN
```

如需有關如何取得 Bot 存取權杖的詳細資料，請參閱[驗證 Bot 向 Bot 連接器服務提出的要求](bot-framework-rest-connector-authentication.md#bot-to-connector)。

### <a name="response-headers"></a>回應標頭

除了標準的 HTTP 回應標頭，每個回應都將包含 `X-Correlating-OperationId` 標頭。 此標頭的值即為對應於 Bot Framework 記錄項目的識別碼，其中包含有關要求的詳細資料。 每次您收到錯誤訊息時，都應擷取該標頭的值。 如果您無法獨立解決問題，回報時請將此值和相關資訊一併提供給支援小組。

## <a name="http-status-codes"></a>HTTP 狀態碼

與每個回應一併傳回的 <a href="http://www.w3.org/Protocols/rfc2616/rfc2616-sec10.html" target="_blank">HTTP 狀態碼</a>會指出對應要求的結果。 

| HTTP 狀態碼 | 意義 |
|----|----|
| 200 | 要求成功。 |
| 201 | 要求成功。 |
| 202 | 已接受要求進行處理。 |
| 204 | 要求成功，但未傳回任何內容。 |
| 400 | 要求的格式不正確或有其他錯誤。 |
| 401 | Bot 未獲授權，無法提出要求。 |
| 403 | 不允許 Bot 執行要求的作業。 |
| 404 | 找不到要求的資源。 |
| 500 | 發生內部伺服器錯誤。 |
| 503 | 服務無法使用。 |

### <a name="errors"></a>Errors

若回應顯示的 HTTP 狀態碼位於 4xx 或 5xx 範圍內，則會在提供錯誤相關資訊的回應本文中加入 [ErrorResponse](#bot-framework-activity-schema) 物件。 如果您收到 4xx 範圍的錯誤訊息，請檢查 **ErrorResponse** 物件以找出錯誤原因，並在重新提交要求之前先解決問題。

## <a name="conversation-operations"></a>交談作業 
透過這些作業建立交談、傳送訊息 (活動)，以及管理交談內容。

| 作業 | 說明 |
|----|----|
| [建立交談](#create-conversation) | 建立新交談。 |
| [傳送至交談](#send-to-conversation) | 將活動 (訊息) 傳送至指定交談的結尾處。 |
| [回覆活動](#reply-to-activity) | 將活動 (訊息) 傳送至指定交談，以回覆指定活動。 |
| [取得交談](#get-conversations) | 取得聊天機器人所參與的交談清單。 |
| [取得交談成員](#get-conversation-members) | 取得指定交談的成員。 |
| [取得對話分頁成員](#get-conversation-paged-members) | 取得指定對話的成員 (一次一頁)。 |
| [取得活動成員](#get-activity-members) | 取得指定交談中指定活動的成員。 |
| [更新活動](#update-activity) | 更新現有作業。 |
| [刪除活動](#delete-activity) | 刪除現有活動。 |
| [刪除交談成員](#delete-conversation-member) | 從交談中移除成員。 |
| [傳送交談記錄](#send-conversation-history) | 將過往活動的文字記錄上傳至交談。 |
| [將附件上傳至通道](#upload-attachment-to-channel) | 將附件直接上傳到通道的 Blob 儲存體。 |

### <a name="create-conversation"></a>建立交談
建立新交談。
```http 
POST /v3/conversations
```

| | |
|----|----|
| **要求本文** | `ConversationParameters` 物件 |
| **傳回** | `ConversationResourceResponse` 物件 |

### <a name="send-to-conversation"></a>傳送至交談
將活動 (訊息) 傳送至指定交談。 系統將根據時間戳記或通道的語意，將活動附加至交談的結尾處。 若要回應交談內的特定訊息，請改為使用[回覆活動](#reply-to-activity)。
```http
POST /v3/conversations/{conversationId}/activities
```

| | |
|----|----|
| **要求本文** | [Activity](#bot-framework-activity-schema) 物件 |
| **傳回** | [Identification](#bot-framework-activity-schema) 物件 | 

### <a name="reply-to-activity"></a>回覆活動
將活動 (訊息) 傳送至指定交談，以回覆指定活動。 若通道支援，則系統會新增活動以回覆其他活動。 若通道不支援巢狀回覆，則此作業將以[傳送至交談](#send-to-conversation)的方式進行。
```http
POST /v3/conversations/{conversationId}/activities/{activityId}
```

| | |
|----|----|
| **要求本文** | [Activity](#bot-framework-activity-schema) 物件 |
| **傳回** | [Identification](#bot-framework-activity-schema) 物件 | 

### <a name="get-conversations"></a>取得交談
取得聊天機器人所參與的交談清單。
```http
GET /v3/conversations?continuationToken={continuationToken}
```

| | |
|----|----|
| **要求本文** | n/a |
| **傳回** | [ConversationsResult](#bot-framework-activity-schema) 物件 | 

### <a name="get-conversation-members"></a>取得交談成員
取得指定交談的成員。
```http
GET /v3/conversations/{conversationId}/members
```

| | |
|----|----|
| **要求本文** | n/a |
| **傳回** | [ChannelAccount](#bot-framework-activity-schema) 物件陣列 | 

### <a name="get-conversation-paged-members"></a>取得對話分頁成員
取得指定對話的成員 (一次一頁)。
```http
GET /v3/conversations/{conversationId}/pagedmembers?pageSize={pageSize}&continuationToken={continuationToken}
```

| | |
|----|----|
| **要求本文** | n/a |
| **傳回** | [ChannelAccount](#bot-framework-activity-schema)件陣列，以及可用來取得更多值的接續權杖 |

### <a name="get-activity-members"></a>取得活動成員
取得指定交談中指定活動的成員。
```http
GET /v3/conversations/{conversationId}/activities/{activityId}/members
```

| | |
|----|----|
| **要求本文** | n/a |
| **傳回** | [ChannelAccount](#bot-framework-activity-schema) 物件陣列 | 

### <a name="update-activity"></a>更新活動
部分通道可讓您編輯現有活動，以反映 Bot 交談的新狀態。 例如，使用者按下其中一個按鈕後，您或許會將按鈕從交談訊息中移除。 若成功執行此作業，指定交談內的指定活動將會更新。 
```http
PUT /v3/conversations/{conversationId}/activities/{activityId}
```

| | |
|----|----|
| **要求本文** | [Activity](#bot-framework-activity-schema) 物件 |
| **傳回** | [Identification](#bot-framework-activity-schema) 物件 | 

### <a name="delete-activity"></a>刪除活動
部分通道可讓您刪除現有活動。 若成功執行此作業，指定活動將會從指定交談移除。
```http
DELETE /v3/conversations/{conversationId}/activities/{activityId}
```

| | |
|----|----|
| **要求本文** | n/a |
| **傳回** | 用於指出作業結果的 HTTP 狀態碼。 回應本文中未指定任何項目。 | 

### <a name="delete-conversation-member"></a>刪除交談成員
從交談中移除成員。 如果該成員是交談內的最後一個成員，則該交談也會遭到刪除。
```http
DELETE /v3/conversations/{conversationId}/members/{memberId}
```

| | |
|----|----|
| **要求本文** | n/a |
| **傳回** | 用於指出作業結果的 HTTP 狀態碼。 回應本文中未指定任何項目。 | 

### <a name="send-conversation-history"></a>傳送交談記錄
將過往活動的文字記錄上傳至交談，以供用戶端呈現。
```http
POST /v3/conversations/{conversationId}/activities/history
```

| | |
|----|----|
| **要求本文** | [Transcript](#bot-framework-activity-schema) 物件。 |
| **傳回** | [ResourceResponse](#bot-framework-activity-schema) 物件。 | 

### <a name="upload-attachment-to-channel"></a>將附件上傳至通道
將指定交談的附件直接上傳至通道的 Blob 儲存體。 如此一來，可讓您將資料儲存在相容的存放區。
```http 
POST /v3/conversations/{conversationId}/attachments
```

| | |
|----|----|
| **要求本文** | [AttachmentUpload](#bot-framework-activity-schema) 物件。 |
| **傳回** | [ResourceResponse](#bot-framework-activity-schema) 物件。 **識別碼**屬性會指定可搭配[取得附件資訊](#get-attachment-info)作業和[取得附件](#get-attachment)作業使用的附件識別碼。 | 

## <a name="attachment-operations"></a>附件作業 
利用這些作業擷取附件相關資訊，以及該檔案本身的二進位資料。

| 作業 | 說明 |
|----|----|
| [取得附件資訊](#get-attachment-info) | 取得有關指定附件的資訊，包括檔案名稱、檔案類型及可用檢視 (例如：原稿或縮圖)。 |
| [取得附件](#get-attachment) | 取得指定附件的指定檢視做為二進位內容。 | 

### <a name="get-attachment-info"></a>取得附件資訊 
取得有關指定附件的資訊，包括檔案名稱、類型及可用檢視 (例如：原稿或縮圖)。
```http
GET /v3/attachments/{attachmentId}
```

| | |
|----|----|
| **要求本文** | n/a |
| **傳回** | [AttachmentInfo](#bot-framework-activity-schema) 物件 | 

### <a name="get-attachment"></a>取得附件
取得指定附件的指定檢視做為二進位內容。
```http
GET /v3/attachments/{attachmentId}/views/{viewId}
```

| | |
|----|----|
| **要求本文** | n/a |
| **傳回** | 代表指定附件之指定檢視的二進位內容 | 

## <a name="state-operations"></a>狀態作業
Microsoft Bot Framework 狀態服務已於 2018 年 3 月 30 日淘汰。 之前，建置於 Azure Bot 服務或 Bot Builder SDK 的 Bot 可依預設連線至 Microsoft 所裝載的這項服務，以儲存 Bot 狀態資料。 Bot 將需要更新才能使用其本身的狀態儲存體。

| 作業 | 說明 |
|----|----|
| `Set User Data` | 儲存通道上特定使用者的狀態資料。 |
| `Set Conversation Data` | 儲存某通道上特定交談的狀態資料。 |
| `Set Private Conversation Data` | 儲存通道上特定交談內容中特定使用者的狀態資料。 |
| `Get User Data` | 擷取先前為通道上所有交談的特定使用者儲存的狀態資料。 |
| `Get Conversation Data` | 擷取先前為通道上的特定交談儲存的狀態資料。 |
| `Get Private Conversation Data` | 擷取先前為通道上特定交談內容中特定使用者儲存的狀態資料。 |
| `Delete State For User` | 刪除先前為使用者儲存的狀態資料。 |

## <a name="bot-framework-activity-schema"></a>Bot Framework 活動結構描述

請參閱 [Bot Framework 活動結構描述](https://aka.ms/botSpecs-activitySchema)，以了解您的 Bot 可使用哪些物件和屬性與使用者通訊。
