---
title: API 參考資料 | Microsoft Docs
description: 了解 Bot 連接器服務和 Bot 狀態服務中的標頭、作業、物件及錯誤。
author: ivorb
ms.author: v-ivorb
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 10/25/2018
ms.openlocfilehash: 41aceaa20613d9b6b7ac95a7837b4ae197d1dd4a
ms.sourcegitcommit: dbbfcf45a8d0ba66bd4fb5620d093abfa3b2f725
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/28/2019
ms.locfileid: "67464787"
---
# <a name="api-reference"></a>API 參考資料

> [!NOTE]
> REST API 不等同於 SDK。 REST API 是為了支援標準 REST 通訊，而 SDK 則是與 Bot Framework 通訊的慣用方法。 

在 Bot Framework 內，Bot 連接器服務可讓 Bot 透過您在 Bot Framework 入口網站中設定的通道和使用者交換訊息；而 Bot 狀態服務則可讓 Bot 儲存和擷取其使用 Bot 連接器服務進行交談時的相關狀態資料。 這兩種服務皆透過 HTTPS 使用業界標準的 REST 和 JSON。

> [!IMPORTANT]
> 不建議將 Bot Framework State Service API 用於生產環境，未來版本可能會加以取代。 建議更新您的 Bot 程式碼以使用記憶體內部儲存體進行測試，或將其中一個 **Azure 擴充功能**用於生產環境 Bot。 如需詳細資訊，請參閱 [.NET](~/dotnet/bot-builder-dotnet-state.md) 或 [Node](~/nodejs/bot-builder-nodejs-state.md) 實作的＜管理狀態資料＞  主題。

## <a name="base-uri"></a>基底 URI

使用者傳送訊息給 Bot 時，傳入要求將包含一個[活動](#activity-object)物件，以及一個用於指定端點以接收 Bot 回應的 `serviceUrl` 屬性。 若要存取 Bot 連接器服務或 Bot 狀態服務，請將 `serviceUrl`值當作 API 要求的基底 URI 使用。 

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

若回應顯示的 HTTP 狀態碼位於 4xx 或 5xx 範圍內，則會在提供錯誤相關資訊的回應本文中加入 [ErrorResponse](#errorresponse-object) 物件。 如果您收到 4xx 範圍的錯誤訊息，請檢查 **ErrorResponse** 物件以找出錯誤原因，並在重新提交要求之前先解決問題。

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
| **要求本文** | [Conversation](#conversation-object) 物件 |
| **傳回** | [ConversationResourceResponse](#conversationresourceresponse-object) 物件 | 

### <a name="send-to-conversation"></a>傳送至交談
將活動 (訊息) 傳送至指定交談。 系統將根據時間戳記或通道的語意，將活動附加至交談的結尾處。 若要回應交談內的特定訊息，請改為使用[回覆活動](#reply-to-activity)。
```http
POST /v3/conversations/{conversationId}/activities
```

| | |
|----|----|
| **要求本文** | [Activity](#activity-object) 物件 |
| **傳回** | [Identification](#identification-object) 物件 | 

### <a name="reply-to-activity"></a>回覆活動
將活動 (訊息) 傳送至指定交談，以回覆指定活動。 若通道支援，則系統會新增活動以回覆其他活動。 若通道不支援巢狀回覆，則此作業將以[傳送至交談](#send-to-conversation)的方式進行。
```http
POST /v3/conversations/{conversationId}/activities/{activityId}
```

| | |
|----|----|
| **要求本文** | [Activity](#activity-object) 物件 |
| **傳回** | [Identification](#identification-object) 物件 | 

### <a name="get-conversations"></a>取得交談
取得聊天機器人所參與的交談清單。
```http
GET /v3/conversations?continuationToken={continuationToken}
```

| | |
|----|----|
| **要求本文** | n/a |
| **傳回** | [ConversationsResult](#conversationsresult-object) 物件 | 

### <a name="get-conversation-members"></a>取得交談成員
取得指定交談的成員。
```http
GET /v3/conversations/{conversationId}/members
```

| | |
|----|----|
| **要求本文** | n/a |
| **傳回** | [ChannelAccount](#channelaccount-object) 物件陣列 | 

### <a name="get-conversation-paged-members"></a>取得對話分頁成員
取得指定對話的成員 (一次一頁)。
```http
GET /v3/conversations/{conversationId}/pagedmembers?pageSize={pageSize}&continuationToken={continuationToken}
```

| | |
|----|----|
| **要求本文** | n/a |
| **傳回** | [ChannelAccount](#channelaccount-object)件陣列，以及可用來取得更多值的接續權杖 |

### <a name="get-activity-members"></a>取得活動成員
取得指定交談中指定活動的成員。
```http
GET /v3/conversations/{conversationId}/activities/{activityId}/members
```

| | |
|----|----|
| **要求本文** | n/a |
| **傳回** | [ChannelAccount](#channelaccount-object) 物件陣列 | 

### <a name="update-activity"></a>更新活動
部分通道可讓您編輯現有活動，以反映 Bot 交談的新狀態。 例如，使用者按下其中一個按鈕後，您或許會將按鈕從交談訊息中移除。 若成功執行此作業，指定交談內的指定活動將會更新。 
```http
PUT /v3/conversations/{conversationId}/activities/{activityId}
```

| | |
|----|----|
| **要求本文** | [Activity](#activity-object) 物件 |
| **傳回** | [Identification](#identification-object) 物件 | 

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
| **要求本文** | [Transcript](#transcript-object) 物件。 |
| **傳回** | [ResourceResponse](#resourceresponse-object) 物件。 | 

### <a name="upload-attachment-to-channel"></a>將附件上傳至通道
將指定交談的附件直接上傳至通道的 Blob 儲存體。 如此一來，可讓您將資料儲存在相容的存放區。
```http 
POST /v3/conversations/{conversationId}/attachments
```

| | |
|----|----|
| **要求本文** | [AttachmentUpload](#attachmentupload-object) 物件。 |
| **傳回** | [ResourceResponse](#resourceresponse-object) 物件。 **識別碼**屬性會指定可搭配[取得附件資訊](#get-attachment-info)作業和[取得附件](#get-attachment)作業使用的附件識別碼。 | 

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
| **傳回** | [AttachmentInfo](#attachmentinfo-object) 物件 | 

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
利用這些作業儲存和擷取狀態資料。

| 作業 | 說明 |
|----|----|
| [設定使用者資料](#set-user-data) | 儲存通道上特定使用者的狀態資料。 | 
| [設定交談資料](#set-conversation-data) | 儲存某通道上特定交談的狀態資料。 | 
| [設定私人交談資料](#set-private-conversation-data) | 儲存通道上特定交談內容中特定使用者的狀態資料。 | 
| [取得使用者資料](#get-user-data) | 擷取先前為通道上所有交談的特定使用者儲存的狀態資料。 | 
| [取得交談資料](#get-conversation-data) | 擷取先前為通道上的特定交談儲存的狀態資料。 | 
| [取得私人交談資料](#get-private-conversation-data) | 擷取先前為通道上特定交談內容中特定使用者儲存的狀態資料。 | 
| [刪除使用者狀態](#delete-state-for-user) | 運用[設定使用者資料](#set-user-data)作業或[設定私人交談資料](#set-private-conversation-data)作業，刪除先前為使用者儲存的狀態資料。 | 

### <a name="set-user-data"></a>設定使用者資料
儲存指定通道上指定使用者的狀態資料。
```http
POST /v3/botstate/{channelId}/users/{userId} 
```

| | |
|----|----|
| **要求本文** | [BotData](#botdata-object) 物件 |
| **傳回** | [BotData](#botdata-object) 物件 | 

### <a name="set-conversation-data"></a>設定交談資料
儲存指定通道上指定交談的狀態資料。
```http
POST /v3/botstate/{channelId}/conversations/{conversationId}
```

| | |
|----|----|
| **要求本文** | [BotData](#botdata-object) 物件 |
| **傳回** | [BotData](#botdata-object) 物件 | 

### <a name="set-private-conversation-data"></a>設定私人交談資料
針對指定通道上指定交談內容中的指定使用者儲存狀態資料。
```http
POST /v3/botstate/{channelId}/conversations/{conversationId}/users/{userId} 
```

| | |
|----|----|
| **要求本文** | [BotData](#botdata-object) 物件 |
| **傳回** | [BotData](#botdata-object) 物件 | 

### <a name="get-user-data"></a>取得使用者資料
擷取先前針對指定通道上所有交談的指定使用者儲存的狀態資料。
```http
GET /v3/botstate/{channelId}/users/{userId} 
```

| | |
|----|----|
| **要求本文** | n/a |
| **傳回** | [BotData](#botdata-object) 物件 | 

### <a name="get-conversation-data"></a>取得交談資料
擷取先前針對指定通道的指定交談儲存的狀態資料。
```http
GET /v3/botstate/{channelId}/conversations/{conversationId} 
```

| | |
|----|----|
| **要求本文** | n/a |
| **傳回** | [BotData](#botdata-object) 物件 | 

### <a name="get-private-conversation-data"></a>取得私人交談資料
擷取先前針對指定通道的指定交談內容之指定使用者儲存的狀態資料。
```http
GET /v3/botstate/{channelId}/conversations/{conversationId}/users/{userId} 
```

| | |
|----|----|
| **要求本文** | n/a |
| **傳回** | [BotData](#botdata-object) 物件 | 

### <a name="delete-state-for-user"></a>刪除使用者狀態
使用[設定使用者資料](#set-user-data)作業或[設定私人交談資料](#set-private-conversation-data)作業，刪除先前為指定通道的指定使用者儲存的狀態資料。
```http
DELETE /v3/botstate/{channelId}/users/{userId} 
```

| | |
|----|----|
| **要求本文** | n/a |
| **傳回** | 字串陣列 (識別碼) | 

## <a id="objects"></a> 結構描述

結構描述可定義能讓 Bot 用來與使用者通訊的物件及其屬性。 

| Object | 說明 |
| ---- | ---- |
| [Activity 物件](#activity-object) | 定義 Bot 和使用者之間交換的訊息。 |
| [AnimationCard 物件](#animationcard-object) | 定義可播放動畫 GIF 或短片的資訊卡。 |
| [Attachment 物件](#attachment-object) | 定義要包含在訊息中的其他資訊。 附件可能是媒體檔案 (例如：音訊、影片、影像、檔案) 或豐富資訊卡。 |
| [AttachmentData 物件](#attachmentdata-object) | 描述附件資料。 |
| [AttachmentInfo 物件](#attachmentinfo-object) | 描述附件。 |
| [AttachmentView 物件](#attachmentview-object) | 定義附件檢視。 |
| [AttachmentUpload 物件](#attachmentupload-object) | 定義要上傳的附件。 |
| [AudioCard 物件](#audiocard-object) | 定義可播放音訊檔案的資訊卡。 |
| [BotData 物件](#botdata-object) | 定義使用者狀態資料、交談或利用 Bot 狀態服務所儲存的特定交談內容中的使用者。 |
| [CardAction 物件](#cardaction-object) | 定義要執行的動作。 |
| [CardImage 物件](#cardimage-object) | 定義要在資訊卡上顯示的影像。 |
| [ChannelAccount 物件](#channelaccount-object) | 定義通道上的 Bot 或使用者帳戶。 |
| [Conversation 物件](#conversation-object) | 定義交談，包含交談中的 Bot 及使用者。 |
| [ConversationAccount 物件](#conversationaccount-object) | 定義通道中的交談。 |
| [ConversationMembers 物件](#conversationmembers-object) | 定義交談的成員。 |
| [ConversationParameters 物件](#conversationparameters-object) | 定義用來建立新交談的參數 |
| [ConversationReference 物件](#conversationreference-object) | 定義交談中的特定要點。 |
| [ConversationResourceResponse 物件](#conversationresourceresponse-object) | 定義[建立交談](#create-conversation)的回應。 |
| [ConversationsResult 物件](#conversationsresult-object) | 定義[取得交談](#get-conversations)呼叫的結果。 |
| [Entity 物件](#entity-object) | 定義實體物件。 |
| [Error 物件](#error-object) | 定義錯誤。 |
| [ErrorResponse 物件](#errorresponse-object) | 定義 HTTP API 回應。 |
| [Fact 物件](#fact-object) | 定義包含事實的機碼值組。 |
| [GeoCoordinates 物件](#geocoordinates-object) | 定義採用「世界大地坐標系統 (WSG84)」座標的地理位置。 |
| [HeroCard 物件](#herocard-object) | 定義具有大型影像、標題、文字及動作按鈕的資訊卡。 |
| [Identification 物件](#identification-object) | 識別資源。 |
| [MediaEventValue 物件](#mediaeventvalue-object) | 媒體事件的增補參數。 |
| [MediaUrl 物件](#mediaurl-object) | 定義媒體檔案來源的 URL。 |
| [Mention 物件](#mention-object) | 定義在交談中提及的使用者或 Bot。 |
| [MessageReaction 物件](#messagereaction-object) | 定義對訊息的回應。 |
| [Place 物件](#place-object) | 定義在交談中提及的地點。 |
| [ReceiptCard 物件](#receiptcard-object) | 定義內含購買收據的資訊卡。 |
| [ReceiptItem 物件](#receiptitem-object) | 定義收據內的明細項目。 |
| [ResourceResponse 物件](#resourceresponse-object) | 定義資源。 |
| [SemanticAction 物件](#semanticaction-object) | 定義程式設計動作的參考。 |
| [SignInCard 物件](#signincard-object) | 定義可讓使用者登入至服務的資訊卡。 |
| [SuggestedActions 物件](#suggestedactions-object) | 定義可讓使用者從中選擇的選項。 |
| [ThumbnailCard 物件](#thumbnailcard-object) | 定義具有縮圖影像、標題、文字和動作按鈕的資訊卡。 |
| [ThumbnailUrl 物件](#thumbnailurl-object) | 定義影像來源的 URL。 |
| [Transcript 物件](#transcript-object) | 要使用[傳送交談記錄](#send-conversation-history)上傳的活動集合。 |
| [VideoCard 物件](#videocard-object) | 定義可播放影片的資訊卡。 |

### <a name="activity-object"></a>活動物件
定義 Bot 和使用者之間交換的訊息。<br/><br/> 

| 屬性 | 類型 | 說明 |
|----|----|----|
| **action** | 字串 | 要套用的動作或是已套用的動作。 使用**類型**屬性判斷動作的內容。 例如，在**類型**是 **contactRelationUpdate** 的情況下，若使用者將 Bot 新增至其連絡人清單，則 **action** 屬性的值會是 **add**；而若使用者將 Bot 從連絡人清單中移除，其值則為 **remove**。 |
| **attachments** | [Attachment](#attachment-object)[] | **Attachment** 物件陣列，用於定義要加入訊息的其他資訊。 每個附件可能是媒體檔案 (例如：音訊、影片、影像、檔案) 或豐富資訊卡。 |
| **attachmentLayout** | 字串 | 包含在訊息中的豐富資訊卡**附件**的版面配置。 下列任一值：**carousel**、**list**。 如需有關豐富資訊卡附件的詳細資訊，請參閱[將豐富資訊卡附件新增至訊息](bot-framework-rest-connector-add-rich-cards.md)。 |
| **channelData** | 物件 | 包含通道專用內容的物件。 某些通道提供的功能，需要使用無法以附件結構描述來呈現的其他資訊。 在該等情況下，請將此屬性設定為通道文件中所定義的通道專用內容。 如需詳細資訊，請參閱[實作通道專屬功能](bot-framework-rest-connector-channeldata.md)。 |
| **channelId** | 字串 | 可唯一識別通道的識別碼。 由通道設定。 | 
| 交談  | [ConversationAccount](#conversationaccount-object) | 定義活動所屬交談的 **ConversationAccount** 物件。 |
| **code** | 字串 | 指出交談終止原因的代碼。 |
| **entities** | object[] | 代表訊息中提及之實體的物件陣列。 此陣列中的物件可能是任何 <a href="http://schema.org/" target="_blank">Schema.org</a> 物件。 例如，陣列可能包含 [Mention](#mention-object) 物件，可識別交談中提及的使用者；而 [Place](#place-object) 物件則可識別交談中提及的地點。 |
| **from** | [ChannelAccount](#channelaccount-object) | 用於指出訊息傳送者的 **ChannelAccount** 物件。 |
| **historyDisclosed** | 布林值 | 用於指出是否公開歷程記錄的旗標。 預設值為 **false**。 |
| **id** | 字串 | 可唯一識別通道活動的識別碼。 | 
| **inputHint** | 字串 | 此值指出您的 Bot 在訊息傳遞給用戶端之後是否要接受、等候或忽略使用者輸入。 下列任一值：**acceptingInput**、**expectingInput**、**ignoringInput**。 |
| **locale** | 字串 | 訊息內顯示文字所使用之語言的地區設定 (格式為：`<language>-<country>`)。 通道使用此屬性指出使用者語言，如此 Bot 即可以該語言指定顯示字串。 預設值為 **en-US**。 |
| **localTimestamp** | 字串 | 訊息傳送時的當地時區日期及時間，以 <a href="https://en.wikipedia.org/wiki/ISO_8601" target="_blank">ISO-8601</a> 格式表示。 |
| **membersAdded** | [ChannelAccount](#channelaccount-object)[] | 代表已加入交談之使用者清單的 **ChannelAccount** 物件陣列。 僅在活動**類型**為「conversationUpdate」，且使用者已加入交談的情況下才會顯示。 | 
| **membersRemoved** | [ChannelAccount](#channelaccount-object)[] | 代表已離開交談之使用者清單的 **ChannelAccount** 物件陣列。 僅在活動**類型**為「conversationUpdate」，且使用者已離開交談的情況下才會顯示。 | 
| **name** | 字串 | 要叫用之作業的名稱或事件的名稱。 |
| **recipient** | [ChannelAccount](#channelaccount-object) | 用於指出訊息收件者的 **ChannelAccount** 物件。 |
| **relatesTo** | [ConversationReference](#conversationreference-object) | 用於定義交談中特定要點的 **ConversationReference** 物件。 |
| **replyToId** | 字串 | 此訊息要回覆之訊息的識別碼。 若要回覆使用者傳送的訊息，請將此屬性設為該使用者訊息的識別碼。 並非所有通道皆支援執行緒回覆。 在這些情況下，通道會忽略此屬性，並使用依照時間排序的語意 (時間戳記)，將訊息附加至交談。 | 
| **serviceUrl** | 字串 | 用於指定通道服務端點的 URL。 由通道設定。 | 
| **speak** | 字串 | 要讓 Bot 在支援語音功能的通道上以語音讀出的文字。 若要控制 Bot 語音的聲音、速率、音量、發音和音高等各種特性，請以<a href="https://msdn.microsoft.com/library/hh378377(v=office.14).aspx" target="_blank">語音合成標記語言 (SSML)</a> 格式指定此屬性。 |
| **suggestedActions** | [SuggestedActions](#suggestedactions-object) | 用於定義選項以讓使用者從中選擇的 **SuggestedActions** 物件。 |
| **summary** | 字串 | 訊息包含的資訊摘要。 例如，若是透過電子郵件通道傳送的訊息，此屬性可指定電子郵件訊息的前 50 個字元。 |
| **text** | 字串 | Bot 與使用者之間傳送的訊息文字。 請參閱通道文件，以了解加諸於此屬性內容的限制。 |
| **textFormat** | 字串 | 訊息**文字**的格式。 下列任一值：**markdown**、**plain**、**xml**。 如需有關文字格式的詳細資料，請參閱[建立訊息](bot-framework-rest-connector-create-messages.md)。 |
| **timestamp** | 字串 | 訊息傳送時的 UTC 時區日期及時間，以 <a href="https://en.wikipedia.org/wiki/ISO_8601" target="_blank">ISO-8601</a> 格式表示。 |
| **topicName** | 字串 | 活動所屬交談的主題。 |
| **type** | 字串 | 活動的類型。 下列任一值：**contactRelationUpdate**、**conversationUpdate**、**deleteUserData**、**message**、**typing**、**event** 和 **endOfConversation**。 如需有關活動類型的詳細資料，請參閱[活動概觀](bot-framework-rest-connector-activities.md)。 |
| **value** | 物件 | 開放端點的值。 |
| **semanticAction** |[SemanticAction](#semanticaction-object) | **SemanticAction** 物件，代表程式設計動作的參考。 |

<a href="#objects">回到結構描述資料表</a>

### <a name="animationcard-object"></a>AnimationCard 物件
定義可播放動畫 GIF 或短片的資訊卡。<br/><br/> 

| 屬性 | 類型 | 說明 |
|----|----|----|
| **autoloop** | 布林值 | 用於指出最後一個動畫 GIF 播放完畢後是否要重播動畫 GIF 清單的旗標。 將此屬性設為 **true** 可自動重播動畫；如不重播則設為 **false**。 預設值為 **true**。 |
| **autostart** | 布林值 | 用於指出資訊卡顯示時是否要自動播放動畫的旗標。 將此屬性設為 **true** 可自動播放動畫；如不播放則設為 **false**。 預設值為 **true**。 |
| **buttons** | [CardAction](#cardaction-object)[] | 可讓使用者執行一或多個動作的 **CardAction** 物件陣列。 此通道可決定您能指定的按鈕數目。 |
| **duration** | 字串 | 媒體內容的長度，採用 [ISO 8601 持續時間格式](https://www.iso.org/iso-8601-date-and-time-format.html)。 |
| **映像** | [ThumbnailUrl](#thumbnailurl-object) | 可指定要在資訊卡上顯示之影像的 **ThumbnailUrl** 物件。 |
| **media** | [MediaUrl](#mediaurl-object)[] | 用於指定要播放之動畫 GIF 清單的 **MediaUrl** 物件陣列。 |
| **shareable** | 布林值 | 用於指出是否要與他人共用動畫的旗標。 將此屬性設為 **true** 可與他人共用動畫；如不共用則設為 **false**。 預設值為 **true**。 |
| **subtitle** | 字串 | 顯示在資訊卡標題下方的子標題。 |
| **text** | 字串 | 顯示在資訊卡標題或子標題下方的描述或提示。 |
| **title** | 字串 | 資訊卡的標題。 |
| **value** | 物件 | 此資訊卡的增補參數 |

<a href="#objects">回到結構描述資料表</a>

### <a name="attachment-object"></a>Attachment 物件
定義要包含在訊息中的其他資訊。 附件可能是媒體檔案 (例如：音訊、影片、影像、檔案) 或豐富資訊卡。<br/><br/> 

| 屬性 | 類型 | 說明 |
|----|----|----|
| **contentType** | 字串 | 附件中內容的媒體類型。 若為媒體檔案，請將此屬性設為已知的媒體類型如：**image/png**、**audio/wav** 和 **video/mp4**。 若為豐富資訊卡，請將此屬性設為下列任一個廠商特有類型：<ul><li>**application/vnd.microsoft.card.adaptive**：豐富卡片可包含文字、語音、影像、按鈕和輸入欄位的任何組合。 將**內容**屬性設為 <a href="http://adaptivecards.io/documentation/#create-cardschema" target="_blank">AdaptiveCard</a> 物件。</li><li>**application/vnd.microsoft.card.animation**：可播放動畫的豐富介面卡。 將**內容**屬性設為 [AnimationCard](#animationcard-object) 物件。</li><li>**application/vnd.microsoft.card.audio**：可播放音訊檔案的豐富介面卡。 將**內容**屬性設為 [AudioCard](#audiocard-object) 物件。</li><li>**application/vnd.microsoft.card.video**：可播放影片的豐富介面卡。 將**內容**屬性設為 [VideoCard](#videocard-object) 物件。</li><li>**application/vnd.microsoft.card.hero**：主圖卡片。 將**內容**屬性設為 [HeroCard](#herocard-object) 物件。</li><li>**application/vnd.microsoft.card.thumbnail**：縮圖卡片。 將**內容**屬性設為 [ThumbnailCard](#thumbnailcard-object) 物件。</li><li>**application/vnd.microsoft.com.card.receipt**：收據卡片。 將**內容**屬性設為 [ReceiptCard](#receiptcard-object) 物件。</li><li>**application/vnd.microsoft.com.card.signin**：使用者登入卡片。 將**內容**屬性設為 [SignInCard](#signincard-object) 物件。</li></ul> |
| **contentUrl** | 字串 | 附件內容的 URL。 例如，若附件為影像，則將 **contentUrl** 設為代表該影像位置的 URL。 支援的通訊協定：HTTP、HTTPS、檔案和資料。 |
| **content** | 物件 | 附件的內容。 若附件是豐富資訊卡，則將此屬性設為豐富資訊卡物件。 此屬性與 **contentUrl** 屬性互斥。 |
| **name** | 字串 | 附件的名稱。 |
| **thumbnailUrl** | 字串 | 若通道支援使用小型的**內容**或 **contentUrl**，通道可使用之縮圖影像的 URL。 例如，若您將 **contentType** 設為 **application/word**，並且將 **contentUrl** 設為 Word 文件的位置，則可加入代表該文件的縮圖影像。 通道會顯示縮圖影像而非該文件。 只要使用者按一下影像，通道便會開啟該文件。 |

<a href="#objects">回到結構描述資料表</a>

### <a name="attachmentdata-object"></a>AttachmentData 物件 
描述附件資料。<br/><br/> 

| 屬性 | 類型 | 說明 |
|----|----|----|
| **name** | 字串 | 附件的名稱。 |
| **originalBase64** | 字串 | 附件內容。 |
| **thumbnailBase64** | 字串 | 附件縮圖內容。 |
| **type** | 字串 | 附件的內容類型。 |

<a href="#objects">回到結構描述資料表</a>

### <a name="attachmentinfo-object"></a>AttachmentInfo 物件
描述附件。<br/><br/> 

| 屬性 | 類型 | 說明 |
|----|----|----|
| **name** | 字串 | 附件的名稱。 |
| **type** | 字串 | 附件的 ContentType。 |
| **檢視** | [AttachmentView](#attachmentview-object)[] | 表示附件可用檢視的 **AttachmentView** 物件陣列。 |

<a href="#objects">回到結構描述資料表</a>

### <a name="attachmentview-object"></a>AttachmentView 物件
定義附件檢視。<br/><br/> 

| 屬性 | 類型 | 說明 |
|----|----|----|
| **viewId** | 字串 | 檢視識別碼。 |
| **大小** | number | 檔案的大小。 |

<a href="#objects">回到結構描述資料表</a>

<!-- TODO - can't find in swagger file -->
### <a name="attachmentupload-object"></a>AttachmentUpload 物件
定義要上傳的附件。<br/><br/> 

| 屬性 | 類型 | 說明 |
|----|----|----|
| **type** | 字串 | 附件的 ContentType。 | 
| **name** | 字串 | 附件的名稱。 | 
| **originalBase64** | 字串 | 代表檔案原始版本內容的二進位資料。 |
| **thumbnailBase64** | 字串 | 代表檔案縮圖版本內容的二進位資料。 |

<a href="#objects">回到結構描述資料表</a>

### <a name="audiocard-object"></a>AudioCard 物件
定義可播放音訊檔案的資訊卡。<br/><br/> 

| 屬性 | 類型 | 說明 |
|----|----|----|
| **aspect** | 字串 | 在 **image** 屬性中指定之縮圖的外觀比例。 有效值為 **16:9** 和 **9:16**。 |
| **autoloop** | 布林值 | 用於指出最後一個音訊檔案播放完畢後是否要重播音訊清單的旗標。 將此屬性設為 **true** 可自動重播音訊檔案；如不重播則設為 **false**。 預設值為 **true**。 |
| **autostart** | 布林值 | 用於指出資訊卡顯示時是否要自動播放音訊的旗標。 將此屬性設為 **true** 可自動播放音訊；如不播放則設為 **false**。 預設值為 **true**。 |
| **buttons** | [CardAction](#cardaction-object)[] | 可讓使用者執行一或多個動作的 **CardAction** 物件陣列。 此通道可決定您能指定的按鈕數目。 |
| **duration** | 字串 | 媒體內容的長度，採用 [ISO 8601 持續時間格式](https://www.iso.org/iso-8601-date-and-time-format.html)。 |
| **映像** | [ThumbnailUrl](#thumbnailurl-object) | 可指定要在資訊卡上顯示之影像的 **ThumbnailUrl** 物件。 |
| **media** | [MediaUrl](#mediaurl-object)[] | 用於指定要播放之音訊檔案清單的 **MediaUrl** 物件陣列。 |
| **shareable** | 布林值 | 用於指出是否要與他人共用音訊檔案的旗標。 將此屬性設為 **true** 可與他人共用音訊；如不共用則設為 **false**。 預設值為 **true**。 |
| **subtitle** | 字串 | 顯示在資訊卡標題下方的子標題。 |
| **text** | 字串 | 顯示在資訊卡標題或子標題下方的描述或提示。 |
| **title** | 字串 | 資訊卡的標題。 |
| **value** | 物件 | 此資訊卡的增補參數 |

<a href="#objects">回到結構描述資料表</a>

<!-- TODO - can't find in swagger file -->
### <a name="botdata-object"></a>BotData 物件
定義使用者狀態資料、交談或利用 Bot 狀態服務所儲存的特定交談內容中的使用者。<br/><br/>

| 屬性 | 類型 | 說明 |
|----|----|----|
| **資料** | 物件 | 在要求中，使用 Bot 狀態服務指定要儲存之屬性和值的 JSON 物件。 在回應中，使用  Bot 狀態服務指定已經儲存之屬性和值的 JSON 物件。 | 
| **eTag** | 字串 | 此為實體標記值，可讓您針對使用 Bot 狀態服務儲存的資料，控制資料並行存取。 如需詳細資訊，請參閱[管理狀態資料](bot-framework-rest-state.md)。 | 

<a href="#objects">回到結構描述資料表</a>

### <a name="cardaction-object"></a>CardAction 物件
定義要執行的動作。<br/><br/> 

| 屬性 | 類型 | 說明 |
|----|----|----|
| **映像** | 字串 | 顯示於以下項目之影像的 URL： | 
| **text** | 字串 | 動作的文字 |
| **title** | 字串 | 按鈕的文字。 僅適用於按鈕的動作。 |
按鈕。 僅適用於按鈕的動作。 |
| **type** | 字串 | 要執行之動作的類型。 如需有效值的清單，請參閱[將豐富資訊卡附件新增至訊息](bot-framework-rest-connector-add-rich-cards.md)。 |
| **value** | 物件 | 動作的增補參數。 此屬性的值將因動作**類型**而有所不同。 如需詳細資訊，請參閱[將豐富資訊卡附件新增至訊息](bot-framework-rest-connector-add-rich-cards.md)。 |

<a href="#objects">回到結構描述資料表</a>

### <a name="cardimage-object"></a>CardImage 物件
定義要在資訊卡上顯示的影像。<br/><br/> 

| 屬性 | 類型 | 說明 |
|----|----|----|
| **alt** | 字串 | 影像的描述。 您應加入描述以支援可存取性。 |
| **點選** | [CardAction](#cardaction-object) | 用於指定使用者點選或按一下影像後要執行之動作的 **CardAction** 物件。 |
| **url** | 字串 | 影像來源的 URL 或影像的 base64 二進位 (例如：`data:image/png;base64,iVBORw0KGgo...`)。 |

<a href="#objects">回到結構描述資料表</a>

### <a name="channelaccount-object"></a>ChannelAccount 物件
定義通道上的 Bot 或使用者帳戶。<br/><br/>

| 屬性 | 類型 | 說明 |
|----|----|----|
| **id** | 字串 | 可唯一識別通道上 Bot 或使用者的識別碼。 |
| **name** | 字串 | Bot 或使用者的名稱。 |

<a href="#objects">回到結構描述資料表</a>

<!--TODO can't find-->
### <a name="conversation-object"></a>交談物件
定義交談，包含交談中的 Bot 及使用者。<br/><br/> 

| 屬性 | 類型 | 說明 |
|----|----|----|
| **Bot** | [ChannelAccount](#channelaccount-object) | 用於識別 Bot 的 **ChannelAccount** 物件。 |
| **isGroup** | 布林值 | 用於指出此項目是否為群組交談的旗標。 設為 **true** 代表此為群組交談，反之則設為 **false**。 預設值為 **false**。 若要開啟群組交談，通道必須支援群組交談功能。 |
| **members** | [ChannelAccount](#channelaccount-object)[] | 用於識別交談成員的 **ChannelAccount** 物件陣列。 此清單必須包含單一使用者，除非 **isGroup** 設為 **true**。 此清單可能包含其他 Bot。 |
| **topicName** | 字串 | 交談的標題。 |
| **activity** | [活動](#activity-object) | 在[建立交談](#create-conversation)要求中，用於定義要發佈至新交談之第一個訊息的 **Activity** 物件。 |

<a href="#objects">回到結構描述資料表</a>

### <a name="conversationaccount-object"></a>ConversationAccount 物件
定義通道中的交談。<br/><br/>

| 屬性 | 類型 | 說明 |
|----|----|----|
| **id** | 字串 | 可識別交談的識別碼。 此識別碼在每個通道均不重複。 通道開啟交談時便會設定此識別碼；反之，則由 Bot 將此屬性設為其開啟交談時，Bot 從回應中取回的識別碼 (請參閱「開啟交談」一節)。 |
| **isGroup** | 布林值 | 用於指出活動產生時交談是否要包含兩位以上參與者的旗標。 設為 **true** 代表此為群組交談，反之則設為 **false**。 預設值為 **false**。 |
| **name** | 字串 | 可用來識別交談的顯示名稱。 |
| **conversationType** | 字串 | 指出可區別交談類型 (例如：群組或私人) 之通道中的交談類型。 |

<a href="#objects">回到結構描述資料表</a>

### <a name="conversationmembers-object"></a>ConversationMembers 物件
定義交談的成員。<br/><br/>

| 屬性 | 類型 | 說明 |
|----|----|----|
| **id** | 字串 | 交談識別碼。 |
| **members** | array | [ChannelAccount](#channelaccount-object) 物件的陣列。 |

<a href="#objects">回到結構描述資料表</a>

### <a name="conversationparameters-object"></a>ConversationParameters 物件
定義用來建立新交談的參數。<br/><br/> 

| 屬性 | 類型 | 說明 |
|----|----|----|
| **isGroup** | 布林值 | 指出此項目是否為群組交談。 |
| **Bot** | [ChannelAccount](#channelaccount-object) | 交談中 Bot 的位址。 |
| **members** | array | 要新增至交談的成員清單。 |
| **topicName** | 字串 | 交談的主題標題。 僅在通道提供支援時，才可使用此屬性。 |
| **activity** | [活動](#activity-object) | (選用) 建立新交談時，以此活動做為交談的初始訊息。 |
| **channelData** | 物件 | 建立交談時的通道特定酬載。 |

<a href="#objects">回到結構描述資料表</a>

### <a name="conversationreference-object"></a>ConversationReference 物件
定義交談中的特定要點。<br/><br/>

| 屬性 | 類型 | 說明 |
|----|----|----|
| **activityId** | 字串 | 可唯一識別此物件參照之活動的識別碼。 | 
| **Bot** | [ChannelAccount](#channelaccount-object) | 用於識別此物件參照之交談內 Bot 的 **ChannelAccount** 物件。 |
| **channelId** | 字串 | 可唯一識別此物件參照之交談內通道的識別碼。 | 
| 交談  | [ConversationAccount](#conversationaccount-object) | 用於定義此物件參照之交談的 **ChannelAccount** 物件。 |
| **serviceUrl** | 字串 | 用於指定此物件參照之交談中通道服務端點的 URL。 | 
| **user** | [ChannelAccount](#channelaccount-object) | 用於指出此物件參照之交談內使用者的 **ChannelAccount** 物件。 |

<a href="#objects">回到結構描述資料表</a>

### <a name="conversationresourceresponse-object"></a>ConversationResourceResponse 物件
定義[建立交談](#create-conversation)的回應。<br/><br/> 

| 屬性 | 類型 | 說明 |
|----|----|----|
| **activityId** | 字串 | 活動的識別碼。 |
| **id** | 字串 | 資源的識別碼。 |
| **serviceUrl** | 字串 | 服務端點。 |

<a href="#objects">回到結構描述資料表</a>

### <a name="conversationsresult-object"></a>ConversationsResult 物件
定義[取得交談](#get-conversations)的結果。<br/><br/> 

| 屬性 | 類型 | 說明 |
|----|----|----|
| **continuationToken** | 字串 | 可在[取得交談](#get-conversations)的後續呼叫中使用的接續權杖。 |
| **交談** | array | [ConversationMembers](#conversationmembers-object) 物件的陣列 |

<a href="#objects">回到結構描述資料表</a>

### <a name="error-object"></a>錯誤物件
定義錯誤。<br/><br/> 

| 屬性 | 類型 | 說明 |
|----|----|----|
| **code** | 字串 | 錯誤碼。 |
| **message** | 字串 | 錯誤的描述。 |

<a href="#objects">回到結構描述資料表</a>

### <a name="entity-object"></a>實體物件
定義實體物件。<br/><br/> 

| 屬性 | 類型 | 說明 |
|----|----|----|
| **type** | 字串 | 實體類型。 通常包含來自 schema.org 的類型。 |

<a href="#objects">回到結構描述資料表</a>

### <a name="errorresponse-object"></a>ErrorResponse 物件
定義 HTTP API 回應。<br/><br/> 

| 屬性 | 類型 | 說明 |
|----|----|----|
| **error** | [錯誤](#error-object) | 含有錯誤相關資訊的 **Error** 物件。 |

<a href="#objects">回到結構描述資料表</a>

### <a name="fact-object"></a>Fact 物件
定義包含事實的機碼值組。<br/><br/> 

| 屬性 | 類型 | 說明 |
|----|----|----|
| **key** | 字串 | 事實的名稱。 例如：**簽入**。 此索引鍵在顯示事實的值時做為標籤使用。 |
| **value** | 字串 | 事實的值。 例如：**2016 年 10 月 10 日**。 |

<a href="#objects">回到結構描述資料表</a>

### <a name="geocoordinates-object"></a>GeoCoordinates 物件
定義採用「世界大地坐標系統 (WSG84)」座標的地理位置。<br/><br/> 

| 屬性 | 類型 | 說明 |
|----|----|----|
| **海拔** | number | 位置的海拔。 |
| **name** | 字串 | 位置的名稱。 |
| **緯度** | number | 位置的緯度。 |
| **經度** | number | 位置的經度。 |
| **type** | 字串 | 此物件的類型。 一律設為 **GeoCoordinates**。 |

<a href="#objects">回到結構描述資料表</a>

### <a name="herocard-object"></a>HeroCard 物件
定義具有大型影像、標題、文字及動作按鈕的資訊卡。<br/><br/> 

| 屬性 | 類型 | 說明 |
|----|----|----|
| **buttons** | [CardAction](#cardaction-object)[] | 可讓使用者執行一或多個動作的 **CardAction** 物件陣列。 此通道可決定您能指定的按鈕數目。 |
| **images** | [CardImage](#cardimage-object)[] | 用於指定要在資訊卡上顯示之影像的 **CardImage** 物件陣列。 主圖卡僅包含一個影像。 |
| **subtitle** | 字串 | 顯示在資訊卡標題下方的子標題。 |
| **點選** | [CardAction](#cardaction-object) | 用於指定使用者點選或按一下資訊卡後要執行之動作的 **CardAction** 物件。 此值可與任一按鈕動作相同或選用其他動作。 |
| **text** | 字串 | 顯示在資訊卡標題或子標題下方的描述或提示。 |
| **title** | 字串 | 資訊卡的標題。 |

<a href="#objects">回到結構描述資料表</a>

<!--TODO can't find-->
### <a name="identification-object"></a>識別物件
識別資源。<br/><br/> 

| 屬性 | 類型 | 說明 |
|----|----|----|
| **id** | 字串 | 可唯一識別資源的識別碼。 |

<a href="#objects">回到結構描述資料表</a>

### <a name="mediaeventvalue-object"></a>MediaEventValue 物件 
媒體事件的增補參數。<br/><br/> 

| 屬性 | 類型 | 說明 |
|----|----|----|
| **cardValue** | 物件 | 在產生此事件之媒體卡上，於 [值]  欄位中指定的回呼參數。 |

<a href="#objects">回到結構描述資料表</a>

### <a name="mediaurl-object"></a>MediaUrl 物件
定義媒體檔案來源的 URL。<br/><br/> 

| 屬性 | 類型 | 說明 |
|----|----|----|
| **設定檔** | 字串 | 用於描述媒體內容的提示。 |
| **url** | 字串 | 媒體檔案來源的 URL。 |

<a href="#objects">回到結構描述資料表</a>

<!--TODO can't find-->
### <a name="mention-object"></a>Mention 物件
定義在交談中提及的使用者或 Bot。<br/><br/> 


|          屬性          |                   類型                   |                                                                                                                                                                                                                           說明                                                                                                                                                                                                                            |
|----------------------------|------------------------------------------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| <strong>提及</strong> | [ChannelAccount](#channelaccount-object) | 用於指定提及之使用者或 Bot 的 <strong>ChannelAccount</strong> 物件。 請注意，部分通道 (如 Slack) 會按交談指派名稱，因此在訊息<strong>收件者</strong>屬性中提及的 Bot 名稱，可能與您[註冊](../bot-service-quickstart-registration.md) Bot 時指定的控制代碼不同。 不過，這兩者的帳戶識別碼是相同的。 |
|   <strong>text</strong>    |                  字串                  |                                                                                                                         在交談中提及的使用者或 Bot。 例如，若訊息是「@ColorBot幫我挑一個新顏色」，則此屬性便會設為 <strong>@ColorBot</strong>。 並非所有通道都能設定此屬性。                                                                                                                          |
|   <strong>type</strong>    |                  字串                  |                                                                                                                                                                                                   此物件的類型。 一律設為<strong>提及</strong>。                                                                                                                                                                                                    |

<a href="#objects">回到結構描述資料表</a>

### <a name="messagereaction-object"></a>MessageReaction 物件
定義對訊息的回應。<br/><br/> 

| 屬性 | 類型 | 說明 |
|----|----|----|
| **type** | 字串 | 反應的類型。 |

<a href="#objects">回到結構描述資料表</a>

### <a name="place-object"></a>Place 物件
定義在交談中提及的地點。<br/><br/> 

| 屬性 | 類型 | 說明 |
|----|----|----|
| **address** | 物件 |  地點的地址。 此屬性可為 `string` 或類型 `PostalAddress` 的複雜物件。 |
| **地理區域** | [GeoCoordinates](#geocoordinates-object) | 用於指定地點之地理座標的 **GeoCoordinates** 物件。 |
| **hasMap** | 物件 | 對應至地點。 此屬性可為 `string` (URL) 或類型 `Map` 的複雜物件。 |
| **name** | 字串 | 地點的名稱。 |
| **type** | 字串 | 此物件的類型。 一律設為**地點**。 |

<a href="#objects">回到結構描述資料表</a>

### <a name="receiptcard-object"></a>ReceiptCard 物件
定義內含購買收據的資訊卡。<br/><br/>

| 屬性 | 類型 | 說明 |
|----|----|----|
| **buttons** | [CardAction](#cardaction-object)[] | 可讓使用者執行一或多個動作的 **CardAction** 物件陣列。 此通道可決定您能指定的按鈕數目。 |
| **facts** | [Fact](#fact-object)[] | 用於指定購買相關資訊的 **Fact** 物件陣列。 例如，飯店住宿收據的明細清單將列出入住日期和退房日期。 此通道可決定您能指定的事實數目。 |
| **items** | [ReceiptItem](#receiptitem-object)[] | 用於指定已購項目的 **ReceiptItem** 物件陣列 |
| **點選** | [CardAction](#cardaction-object) | 用於指定使用者點選或按一下資訊卡後要執行之動作的 **CardAction** 物件。 此值可與任一按鈕動作相同或選用其他動作。 |
| **tax** | 字串 | 用於指定該筆購買金額須支付之稅金的貨幣格式字串。 |
| **title** | 字串 | 顯示在收據頂端的標題。 |
| **total** | 字串 | 用於指定購買總額 (包括適用稅金) 的貨幣格式字串。 |
| **vat** | 字串 | 用於指定該筆購買金額須支付之加值稅 (VAT) 的貨幣格式字串。 |

<a href="#objects">回到結構描述資料表</a>

### <a name="receiptitem-object"></a>ReceiptItem 物件
定義收據內的明細項目。<br/><br/> 

| 屬性 | 類型 | 說明 |
|----|----|----|
| **映像** | [CardImage](#cardimage-object) | 用於指定要顯示於明細項目旁之縮圖影像的 **CardImage** 物件。  |
| **price** | 字串 | 用於指定所有已購單位之總價的貨幣格式字串。 |
| **quantity** | 字串 | 用於指定已購單位數量的數值字串。 |
| **subtitle** | 字串 | 顯示於明細項目標題下方的子標題。 |
| **點選** | [CardAction](#cardaction-object) | 用於指定使用者點選或按一下明細項目後要執行之動作的 **CardAction** 物件。 |
| **text** | 字串 | 明細項目的描述。 |
| **title** | 字串 | 明細項目的標題。 |

<a href="#objects">回到結構描述資料表</a>

### <a name="resourceresponse-object"></a>ResourceResponse 物件
定義包含資源識別碼的回應。<br/><br/>

|      屬性       |  類型  |                說明                |
|---------------------|--------|-------------------------------------------|
| <strong>id</strong> | 字串 | 可唯一識別資源的識別碼。 |

<a href="#objects">回到結構描述資料表</a>

### <a name="semanticaction-object"></a>SemanticAction 物件
定義程式設計動作的參考。<br/><br/>

| 屬性 | 類型 | 說明 |
|----|----|----|
| **id** | 字串 | 此動作的識別碼 |
| **entities** | [實體](#entity-object) | 與此動作相關聯的實體 |

<a href="#objects">回到結構描述資料表</a>

### <a name="signincard-object"></a>SignInCard 物件
定義可讓使用者登入至服務的資訊卡。<br/><br/>

| 屬性 | 類型 | 說明 |
|----|----|----|
| **buttons** | [CardAction](#cardaction-object)[] | 可讓使用者登入服務的 **CardAction** 物件。 此通道可決定您能指定的按鈕數目。 |
| **text** | 字串 | 要加入登入卡的描述或提示。 |

<a href="#objects">回到結構描述資料表</a>

### <a name="suggestedactions-object"></a>SuggestedActions 物件
定義可讓使用者從中選擇的選項。<br/><br/> 

| 屬性 | 類型 | 說明 |
|----|----|----|
| **actions** | [CardAction](#cardaction-object)[] | 用於定義建議動作的 **CardAction** 物件。 |
| **to** | string[] | 此字串陣列包含可檢視建議動作之收件者的識別碼。 |

<a href="#objects">回到結構描述資料表</a>

### <a name="thumbnailcard-object"></a>ThumbnailCard 物件
定義具有縮圖影像、標題、文字和動作按鈕的資訊卡。<br/><br/>

| 屬性 | 類型 | 說明 |
|----|----|----|
| **buttons** | [CardAction](#cardaction-object)[] | 可讓使用者執行一或多個動作的 **CardAction** 物件陣列。 此通道可決定您能指定的按鈕數目。 |
| **images** | [CardImage](#cardimage-object)[] | 用於指定要在資訊卡上顯示之縮圖影像的 **CardImage** 物件。 此通道可決定您能指定的縮圖影像數目。 |
| **subtitle** | 字串 | 顯示在資訊卡標題下方的子標題。 |
| **點選** | [CardAction](#cardaction-object) | 用於指定使用者點選或按一下資訊卡後要執行之動作的 **CardAction** 物件。 此值可與任一按鈕動作相同或選用其他動作。 |
| **text** | 字串 | 顯示在資訊卡標題或子標題下方的描述或提示。 |
| **title** | 字串 | 資訊卡的標題。 |

<a href="#objects">回到結構描述資料表</a>

### <a name="thumbnailurl-object"></a>ThumbnailUrl 物件
定義影像來源的 URL。<br/><br/> 

| 屬性 | 類型 | 說明 |
|----|----|----|
| **alt** | 字串 | 影像的描述。 您應加入描述以支援可存取性。 |
| **url** | 字串 | 影像來源的 URL 或影像的 base64 二進位 (例如：`data:image/png;base64,iVBORw0KGgo...`)。 |

<a href="#objects">回到結構描述資料表</a>

### <a name="transcript-object"></a>Transcript 物件
要使用[傳送交談記錄](#send-conversation-history)上傳的活動集合。<br/><br/> 

| 屬性 | 類型 | 說明 |
|----|----|----|
| **活動** | array | [Activity](#activity-object) 物件的陣列。 這些物件各自都應該擁有唯一的識別碼和時間戳記。 |

<a href="#objects">回到結構描述資料表</a>

### <a name="videocard-object"></a>VideoCard 物件
定義可播放影片的資訊卡。<br/><br/>

| 屬性 | 類型 | 說明 |
|----|----|----|
| **aspect** | 字串 | 影片的外觀比例 (例如：16:9、4:3)。|
| **autoloop** | 布林值 | 用於指出最後一個影片播放完畢後是否要重播影片清單的旗標。 將此屬性設為 **true** 可自動重播影片；如不重播則設為 **false**。 預設值為 **true**。 |
| **autostart** | 布林值 | 用於指出資訊卡顯示時是否要自動播放影片的旗標。 將此屬性設為 **true** 可自動播放影片；如不播放則設為 **false**。 預設值為 **true**。 |
| **buttons** | [CardAction](#cardaction-object)[] | 可讓使用者執行一或多個動作的 **CardAction** 物件陣列。 此通道可決定您能指定的按鈕數目。 |
| **duration** | 字串 | 媒體內容的長度，採用 [ISO 8601 持續時間格式](https://www.iso.org/iso-8601-date-and-time-format.html)。 |
| **映像** | [ThumbnailUrl](#thumbnailurl-object) | 可指定要在資訊卡上顯示之影像的 **ThumbnailUrl** 物件。 |
| **media** | [MediaUrl](#mediaurl-object)[] | 用於指定要播放之影片清單的 **MediaUrl** 物件。 |
| **shareable** | 布林值 | 用於指出是否要與他人共用影片的旗標。 將此屬性設為 **true** 可與他人共用影片；如不共用則設為 **false**。 預設值為 **true**。 |
| **subtitle** | 字串 | 顯示在資訊卡標題下方的子標題。 |
| **text** | 字串 | 顯示在資訊卡標題或子標題下方的描述或提示。 |
| **title** | 字串 | 資訊卡的標題。 |
| **value** | 物件 | 此資訊卡的增補參數|

<a href="#objects">回到結構描述資料表</a>
