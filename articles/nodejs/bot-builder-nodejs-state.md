---
title: 管理狀態資料 | Microsoft Docs
description: 了解如何使用適用於 Node.js 的 Bot Builder SDK 來儲存及擷取狀態資料。
author: DucVo
ms.author: v-ducvo
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: 1001f1aa2fe76127073551e98548fc20ef9e1bd7
ms.sourcegitcommit: f576981342fb3361216675815714e24281e20ddf
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/18/2018
ms.locfileid: "39299667"
---
# <a name="manage-state-data"></a>管理狀態資料
> [!div class="op_single_selector"]
> - [.NET](../dotnet/bot-builder-dotnet-state.md)
> - [Node.js](../nodejs/bot-builder-nodejs-state.md)

[!INCLUDE [State concept overview](../includes/snippet-dotnet-concept-state.md)]

## <a name="in-memory-data-storage"></a>記憶體內部的資料儲存體

記憶體內部的資料儲存體僅供測試之用。 這是可變更的臨時儲存體。 每次 Bot 重新啟動時，就會清除資料。 若要將記憶體內部的儲存體運用於測試用途，您必須執行兩個動作。 首先，建立記憶體內部儲存體的新執行個體：

```javascript
var inMemoryStorage = new builder.MemoryBotStorage();
```

然後，在建立 **UniversalBot** 時，將它設定為 Bot：

```javascript
var inMemoryStorage = new builder.MemoryBotStorage();
var bot = new builder.UniversalBot(connector, [..waterfall steps..])
                    .set('storage', inMemoryStorage); // Register in-memory storage 
```

您可以使用這個方法來設定自己的自訂資料儲存體，或使用任一個 *Azure 擴充功能*。

## <a name="manage-custom-data-storage"></a>管理自訂資料儲存體

在生產環境中基於效能和安全性理由，您可能要實作自己的資料儲存體，或考量實作下列其中一個資料儲存體選項：

1. [使用 Cosmos DB 管理狀態資料](bot-builder-nodejs-state-azure-cosmosdb.md)

2. [使用表格儲存體管理狀態資料](bot-builder-nodejs-state-azure-table-storage.md)

利用這其中任一個 [Azure 擴充功能](https://www.npmjs.com/package/botbuilder-azure)選項，透過適用於 Node.js 的 Bot Framework SDK 設定和保存資料的機制，會與記憶體內部的資料儲存體保持一致。

## <a name="storage-containers"></a>儲存體容器

在適用於 Node.js 的 Bot Builder SDK中，`session` 物件會公開下列用來儲存狀態資料的屬性。

| 屬性 | 目標範圍 | 說明 |
| ---- | ---- | ---- |
| [`userData`][userDataURL] | 使用者 | 包含儲存於指定通道上以供使用者使用的資料。 此資料將跨多個交談進行保存。 |
| [`privateConversationData`][privateConversationDataURL] | 交談 | 包含儲存於指定通道上特定交談內容中以供使用者使用的資料。 此資料是目前使用者私用的，並且僅會針對目前的交談進行保存。 當交談結束時或明確呼叫 `endConversation` 時，即會清除這個屬性。 |
| [`conversationData`][conversationDataURL] | 交談 | 包含儲存於指定通道上特定交談內容中的資料。 此資料會與參與交談的所有使用者共用，並且僅會針對目前的交談進行保存。 當交談結束時或明確呼叫 `endConversation` 時，即會清除這個屬性。 |
| [`dialogData`][dialogDataURL] | 對話 | 包含僅針對目前對話儲存的資料。 每個對話都會針對這個屬性維護自己的複本。 從對話堆疊中移除對話時，即會清除這個屬性。 |

這四個屬性會對應到可用來儲存資料的四個資料儲存體容器。 您用來儲存資料的屬性將取決於您所儲存之資料的適當範圍、您所儲存之資料的本質，以及您想要保存資料的時間長度。 例如，如果您需要儲存使用者資料，以便跨多個交談使用，請考慮使用 `userData` 屬性。 如果您需要將區域變數值暫時儲存於對話範圍內，請考慮使用 `dialogData` 屬性。 如果您需要暫時儲存必須能夠跨多個對話存取的資料，請考慮使用 `conversationData` 屬性。

## <a name="data-persistence"></a>資料持續性

根據預設，會將使用 `userData`、`privateConversationData` 和 `conversationData` 屬性儲存的資料設定為在交談結束之後保存。 如果您不想將資料保存於 `userData` 容器中，請將 `persistUserData` 旗標設為 **false**。 如果您不想將資料保存於 `conversationData` 容器中，請將 `persistConversationData` 旗標設為 **false**。 

```javascript
// Do not persist userData
bot.set(`persistUserData`, false);

// Do not persist conversationData
bot.set(`persistConversationData`, false);
```

> [!NOTE]
> 您無法停用 `privateConversationData` 容器的資料持續性，它一定是持續性的。

## <a name="set-data"></a>設定資料

您可以藉由將簡單的 JavaScript 物件直接儲存至儲存體容器來儲存它們。 對於像是 `Date` 的複雜物件，請考慮將它轉換成 `string`。 這是因為狀態資料會序列化並儲存為 JSON。 下列程式碼範例示範如何儲存基本資料、陣列、物件對應，以及複雜的 `Date` 物件。 

**儲存基本資料**

```javascript
session.userData.userName = "Kumar Sarma";
session.userData.userAge = 37;
session.userData.hasChildren = true;
```

**儲存陣列**

```javascript
session.userData.profile = ["Kumar Sarma", "37", "true"];
```

**儲存物件對應**

```javascript
session.userData.about = {
    "Profile": {
        "Name": "Kumar Sarma",
        "Age": 37,
        "hasChildren": true
    },
    "Job": {
        "Company": "Contoso",
        "StartDate": "June 8th, 2010",
        "Title": "Developer"
    }
}
```
**儲存日期與時間** 

對於複雜的 JavaScript 物件，請先將它轉換為字串，然後再儲存到儲存體容器。 

```javascript 
var startDate = builder.EntityRecognizer.resolveTime([results.response]); 

// Date as string: "2017-08-23T05:00:00.000Z" 
session.userdata.start = startDate.toISOString(); 
``` 

### <a name="saving-data"></a>儲存資料

在每個儲存體容器中建立的資料將保留於記憶體中，直到容器儲存為止。 適用於 Node.js 的 Bot Builder SDK 會分批將資料傳送至 `ChatConnector` 服務，以便在有要傳送的訊息時加以儲存。 若要儲存已存在於儲存體容器中的資料，而不傳送任何訊息，您可以手動呼叫 [`save`](https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.session.html#save) 方法。 如果您未呼叫 `save` 方法，存在於儲存體容器中的資料將會保存以作為批次處理的一部分。

```javascript
session.userData.favoriteColor = "Red";
session.userData.about.job.Title = "Senior Developer"; 
session.save();
```

## <a name="get-data"></a>取得資料

若要存取儲存於特定儲存體容器中的資料，只需參考對應屬性。 下列程式碼範例示範如何存取先前已儲存為基本資料、陣列、物件對應及複雜 Date 物件的資料。

**存取基本資料**

```javascript
var userName = session.userData.userName;
var userAge = session.userData.userAge;
var hasChildren = session.userData.hasChildren;
```

**存取陣列**

```javascript
var userProfile = session.userData.userProfile;

session.send("User Profile:");
for(int i = 0; i < userProfile.length, i++){
    session.send(userProfile[i]);
}
```

**存取物件對應**

```javascript
var about = session.userData.about;

session.send("User %s works at %s.", about.Profile.Name, about.Job.Company);
```

**存取 Date 物件** 

以字串形式擷取日期資料，然後將它轉換成 JavaScript 的 Date 物件。 

```javascript 
// startDate as a JavaScript Date object. 
var startDate = new Date(session.userdata.start); 
``` 

## <a name="delete-data"></a>刪除資料

根據預設，從對話堆疊中移除對話時，即會清除儲存於 `dialogData` 容器中的資料。 同樣地，在呼叫 `endConversation` 方法時，會清除儲存於 `conversationData` 容器和 `privateConversationData` 容器中的資料。 不過，若要刪除儲存於 `userData` 容器中的資料，您必須明確地清除它。

若要明確地清除儲存於任何儲存體容器中的資料，只需重設容器即可，如下列程式碼範例所示。 

```javascript
// Clears data stored in container.
session.userData = {}; 
session.privateConversationData = {};
session.conversationData = {};
session.dialogData = {};
```

絕對不要將資料容器設定為 `null`，或從 `session` 物件移除它，因為這樣做將在您下次嘗試存取容器時造成錯誤。 此外，建議您在手動清除記憶體中的容器之後，手動呼叫 `session.save();`，以清除先前已保存的任何對應資料。

## <a name="next-steps"></a>後續步驟

現在，您已經了解如何管理使用者狀態資料，讓我們看看如何使用該資料，更有效地管理交談流程。

> [!div class="nextstepaction"]
> [管理交談流程](bot-builder-nodejs-dialog-manage-conversation-flow.md)

## <a name="additional-resources"></a>其他資源
- [提示使用者輸入](bot-builder-nodejs-dialog-prompt.md)

[userDataURL]: https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.session.html#userdata
[conversationDataURL]: https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.session.html#conversationdata
[privateConversationDataURL]: https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.session.html#privateconversationdata
[dialogDataURL]: https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.session.html#dialogdata

[ChatConnector]: https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.chatconnector.html
