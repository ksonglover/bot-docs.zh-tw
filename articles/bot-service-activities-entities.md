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
ms.openlocfilehash: 9fa9a23f4d14667aeb97d304498b415f2c8041d1
ms.sourcegitcommit: f84b56beecd41debe6baf056e98332f20b646bda
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 05/03/2019
ms.locfileid: "65033065"
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

活動類型有好幾種；活動可以屬於數個會傳遞最常見**訊息**的不同類型。 說明和進一步的資訊可於[活動結構描述頁面](https://aka.ms/botSpecs-activitySchema)中找到。

::: moniker range="azure-bot-service-3.0"

## <a name="additional-resources"></a>其他資源

- <a href="https://docs.botframework.com/en-us/csharp/builder/sdkreference/dc/d2f/class_microsoft_1_1_bot_1_1_connector_1_1_activity.html" target="_blank">Activity 類別</a> \(英文\)
::: moniker-end
