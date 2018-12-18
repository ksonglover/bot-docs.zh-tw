---
title: 實體和活動類型 | Microsoft Docs
description: 實體和活動類型。
keywords: 提及實體, 活動類型, 取用實體
author: ivorb
ms.author: v-ivorb
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 03/01/2018
ms.openlocfilehash: 818017a81b497b13ee181dbb6b87c03a0182736d
ms.sourcegitcommit: 75f32b3325dd0fc4d8128dee6c22ebf91e5785b3
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 12/09/2018
ms.locfileid: "53120675"
---
# <a name="entities-and-activity-types"></a>實體和活動類型

實體是活動的一部分，會提供關於活動或對話的其他資訊。

[!include[Entity boilerplate](includes/snippet-entity-boilerplate.md)]

## <a name="entities"></a>實體

訊息的 Entities 屬性是開放式 <a href="http://schema.org/" target="_blank">schema.org</a> 物件的陣列，其允許交換通道與 Bot 之間的通用內容中繼資料。

### <a name="mention-entities"></a>提及實體

許多通道都支援 Bot 或使用者在交談內容中「提及」某人的功能。
若要在訊息中提及使用者，請以 mention 物件填入訊息的實體屬性。
提及物件包含下列屬性：

| 屬性 | 說明 |
|----|----|
| 類型 | 實體的類型 ("mention") |
| Mentioned | 通道帳戶物件，指出所提及的使用者 | 
| 文字 | activity.text 屬性中的文字，表示 mention 本身 (可以是空白或 null) |

此程式碼範例示範如何將 mention 實體新增至實體集合。

# <a name="ctabcs"></a>[C#](#tab/cs)
[!code-csharp[set Mention](includes/code/dotnet-create-messages.cs#setMention)]

> [!TIP]
> 嘗試判斷使用者意圖時，Bot 可以忽略提到它的訊息部分。 呼叫 `GetMentions` 方法，並評估 `Mention` 回應中傳回的物件。

# <a name="javascripttabjs"></a>[JavaScript](#tab/js)
```javascript
var entity = context.activity.entities;

const mention = {
    type: "Mention",
    text: "@johndoe",
    mentioned: {
        name: "John Doe",
        id: "UV341235"
    }
}

entity = [mention];
```

---

### <a name="place-objects"></a>放置物件

以 Place 物件或 GeoCoordinates 物件填入訊息的 entities 屬性，即可在訊息中傳達<a href="https://schema.org/Place" target="_blank">位置相關資訊</a>。

place 物件包含下列屬性：

| 屬性 | 說明 |
|----|----|
| 類型 | 實體的類型 ("Place") |
| 位址 | 說明或郵寄地址物件 (未來) |
| 地理區域 | GeoCoordinates |
| HasMap | 地圖的 URL 或地圖物件 (未來) |
| Name | 位置的名稱 |

geoCoordinates 物件包含下列屬性：

| 屬性 | 說明 |
|----|----|
| 類型 | 實體的類型 ("GeoCoordinates") |
| Name | 位置的名稱 |
| 經度 | 位置的經度 (<a href="https://en.wikipedia.org/wiki/World_Geodetic_System" target="_blank">WGS 84</a>) |
| 經度 | 位置的緯度 (<a href="https://en.wikipedia.org/wiki/World_Geodetic_System" target="_blank">WGS 84</a>) |
| Elevation | 位置的提升 (<a href="https://en.wikipedia.org/wiki/World_Geodetic_System" target="_blank">WGS 84</a>) |

此程式碼範例示範如何將 place 實體新增至實體集合：

# <a name="ctabcs"></a>[C#](#tab/cs)
[!code-csharp[set GeoCoordinates](includes/code/dotnet-create-messages.cs#setGeoCoord)]

# <a name="javascripttabjs"></a>[JavaScript](#tab/js)
```javascript
var entity = context.activity.entities;

const place = {
    elavation: 100,
    type: "GeoCoordinates",
    name : "myPlace",
    latitude: 123,
    longitude: 234
};

entity = [place];

```

---

### <a name="consume-entities"></a>取用實體

# <a name="ctabcs"></a>[C#](#tab/cs)

若要取用實體，請使用 `dynamic` 關鍵字或強型別類別。

此程式碼範例示範如何使用 `dynamic` 關鍵字來處理訊息 `Entities` 屬性內的實體：

[!code-csharp[examine entity using dynamic keyword](includes/code/dotnet-create-messages.cs#examineEntity1)]

此程式碼範例示範如何使用強型別類別來處理訊息 `Entities` 屬性內的實體：

[!code-csharp[examine entity using typed class](includes/code/dotnet-create-messages.cs#examineEntity2)]

# <a name="javascripttabjs"></a>[JavaScript](#tab/js)

此程式碼範例示範如何處理訊息 `entity` 屬性內的實體：

```javascript
if (entity[0].type === "GeoCoordinates" && entity[0].latitude > 34) {
    // do something
}
```

---

## <a name="activity-types"></a>活動類型

此程式碼範例示範如何處理 **message** 類型的活動：

# <a name="ctabcs"></a>[C#](#tab/cs)

```cs
if (context.Activity.Type == ActivityTypes.Message){
    // do something
}
```

# <a name="javascripttabjs"></a>[JavaScript](#tab/js)

```js
if(context.activity.type === 'message'){
    // do something
}
```

---

活動可以屬於數個會傳遞最常見**訊息**的不同類型。 有數種活動類型：

| Activity.Type | 介面 | 說明 |
|-----|-----|-----|
| [message](#message) | IMessageActivity (C#) <br> Activity (JS) | 代表 Bot 和使用者之間的通訊。 |
| [contactRelationUpdate](#contactrelationupdate) | IContactRelationUpdateActivity (C#) <br> Activity (JS) | 表示 Bot 已新增至或從使用者的連絡人清單中移除。 |
| [conversationUpdate](#conversationupdate) | IConversationUpdateActivity (C#) <br> Activity (JS) | 表示 Bot 新增至對話、其他成員新增至或從對話中移除，或對話中繼資料已變更。 |
| [deleteUserData](#deleteuserdata) | n/a | 表示使用者已要求 Bot 刪除任何可能已儲存的使用者資料。 |
| [endOfConversation](#endofconversation) | IEndOfConversationActivity (C#) <br> Activity (JS) | 表示對話結束。 |
| [event](#event) | IEventActivity (C#) <br> Activity (JS) | 代表傳送給 Bot 的通訊使用者看不到。 |
| [installationUpdate](#installationupdate) | IInstallationUpdateActivity (C#) <br> Activity (JS) | 代表在通道的組織單位內 (例如客戶租用戶或「小組」) 安裝或解除安裝 Bot。 |
| [invoke](#invoke) | IInvokeActivity (C#) <br> Activity (JS) | 代表傳送給 Bot 以要求它執行特定作業的通訊。 此活動類型由 Microsoft Bot Framework 保留供內部使用。 |
| [messageReaction](#messagereaction) | IMessageReactionActivity (C#) <br> Activity (JS) | 表示使用者已對現有活動做出回應。 例如，使用者按了訊息上的「讚」按鈕。 |
| [typing](#typing) | ITypingActivity (C#) <br> Activity (JS) | 表示使用者或在對話另一端的 Bot 正在編譯回應。 |
| messageUpdate | IMessageUpdateActivity (C#) <br> Activity (JS) | 表示要更新交談中先前訊息活動的要求。 |
| messageDelete | IMessageDeleteActivity (C#) <br> Activity (JS) | 表示要刪除交談中先前訊息活動的要求。 |
| 建議 | ISuggestionActivity (C#) <br> Activity (JS) | 表示對於另一個特定活動相關收件者的私人建議。 |
| trace | ITraceActivity (C#) <br> Activity (JS) | 一項活動，可供 Bot 將內部資訊記錄到已記錄的交談文字記錄中。 |
| handoff | IHandoffActivity (C#) <br> Activity (JS) | 已移轉的交談控制權，或是要移轉交談控制權的要求。 |

## <a name="message"></a>Message

<!-- Only the last link is different. -->

::: moniker range="azure-bot-service-3.0"

Bot 會傳送 message 活動以傳達資訊，及接收來自使用者的 message 活動。
某些訊息可能只包含純文字，而其他訊息可能包含更豐富的內容，例如要讀出的文字、[建議的動作](v4sdk/bot-builder-howto-add-suggested-actions.md)、[媒體附件](v4sdk/bot-builder-howto-add-media-attachments.md)、[複合式資訊卡 (Rich Card)](v4sdk/bot-builder-howto-add-media-attachments.md#send-a-hero-card) 和[通道特有資料](~/dotnet/bot-builder-dotnet-channeldata.md)。

::: moniker-end

::: moniker range="azure-bot-service-4.0"

Bot 會傳送 message 活動以傳達資訊，及接收來自使用者的 message 活動。
某些訊息可能只包含純文字，而其他訊息可能包含更豐富的內容，例如要讀出的文字、[建議的動作](v4sdk/bot-builder-howto-add-suggested-actions.md)、[媒體附件](v4sdk/bot-builder-howto-add-media-attachments.md)、[複合式資訊卡 (Rich Card)](v4sdk/bot-builder-howto-add-media-attachments.md#send-a-hero-card) 和[通道特有資料](~/v4sdk/bot-builder-channeldata.md)。

::: moniker-end

## <a name="contactrelationupdate"></a>contactRelationUpdate

每當 Bot 加入或從使用者的連絡人清單中移除時，Bot 會收到連絡人關係更新活動。 活動的 action 屬性值 (新增 | 移除) 會指出 Bot 是否已加入或已從使用者的連絡人清單中移除。

## <a name="conversationupdate"></a>conversationUpdate

每當 Bot 已加入對話、其他成員已加入或從對話中移除，或對話中繼資料已變更，Bot 會收到對話更新活動。

如果成員已加入對話，活動的「新增的成員」屬性會包含通道帳戶物件陣列以識別新成員。

若要判斷 Bot 是否已加入對話 (亦即成為其中一個新成員)，請評估活動的收件者識別碼值 (亦即 Bot 的識別碼) 是否符合成員所新增陣列中任何帳戶的 Id 屬性。

如果成員已從對話中移除，「移除的成員」屬性會包含通道帳戶物件陣列以識別移除的成員。

> [!TIP]
> 如果您的 Bot 收到對話更新活動指出使用者已加入對話，您可以選擇傳送歡迎訊息給使用者做為回應。

## <a name="deleteuserdata"></a>deleteUserData

當使用者要求刪除 Bot 先前為其保存的任何資料，Bot 會收到刪除使用者資料活動。 如果您的 Bot 收到此類型的活動，它會為提出要求的使用者刪除先前已儲存的任何個人識別資訊 (PII)。

## <a name="endofconversation"></a>endOfConversation

Bot 會收到結束對話活動，表示使用者已結束對話。 Bot 會傳送結束對話活動，向使用者表示對話已結束。

## <a name="event"></a>事件

Bot 可能會從外部處理序或服務收到事件活動，其想要傳達資訊給您的 Bot，而不向使用者顯示該資訊。 事件活動的傳送者通常不會預期 Bot 以任何方式確認收到活動。

## <a name="installationupdate"></a>installationUpdate

安裝更新活動代表在通道的組織單位內 (例如客戶租用戶或「小組」) 安裝或解除安裝 Bot。 安裝更新活動通常不代表新增或移除通道。 在通道內的租用戶、小組或其他組織單位中新增或移除 Bot 時，通道可以傳送安裝活動。 在通道內安裝或移除 Bot 時，通道不應該傳送安裝活動。

## <a name="invoke"></a>invoke

Bot 可能會收到叫用活動，代表執行特定作業的要求。
叫用活動的傳送者通常會預期 Bot 透過 HTTP 回應確認收到活動。
此活動類型由 Microsoft Bot Framework 保留供內部使用。

## <a name="messagereaction"></a>messageReaction

當使用者對現有活動做出回應，某些通道會傳送訊息回應活動給 Bot。 例如，使用者按了訊息上的「讚」按鈕。 replyToId 屬性會指出使用者回應了哪一個活動。

訊息回應活動可能對應到通道所定義任何數目的訊息回應類型。 比方說，「讚」或「PlusOne」為通道可能傳送的反應類型。

## <a name="typing"></a>typing

Bot 會收到輸入活動，表示使用者正在輸入回應。
Bot 會傳送輸入活動，向使用者表示 Bot 正在運作以完成要求或編譯回應。

::: moniker range="azure-bot-service-3.0"

## <a name="additional-resources"></a>其他資源

- <a href="https://docs.botframework.com/en-us/csharp/builder/sdkreference/dc/d2f/class_microsoft_1_1_bot_1_1_connector_1_1_activity.html" target="_blank">Activity 類別</a> \(英文\)
::: moniker-end
