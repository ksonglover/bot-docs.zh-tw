---
title: 保存使用者資料 | Microsoft Docs
description: 了解如何將使用者狀態資料儲存在 Bot 建立器 SDK 中的儲存體。
keywords: 保存使用者資料, 儲存體, 交談資料
author: v-ducvo
ms.author: v-ducvo
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 4/23/2018
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 28377a1e611464012df28d3edf78d1cf01351345
ms.sourcegitcommit: 44f100a588ffda19c275b118f4f97029f12d1449
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/26/2018
ms.locfileid: "42928366"
---
# <a name="persist-user-data"></a>保存使用者資料

[!INCLUDE [pre-release-label](../includes/pre-release-label.md)]

當 Bot 要求使用者輸入時，您可能會想要以某些形式的儲存體保存某些資訊。 Bot 建立器 SDK 可讓您使用「記憶體中的儲存體」、「檔案儲存體」或資料庫儲存體 (例如 CosmosDB 或 SQL) 來儲存使用者輸入，其中本機儲存體類型主要用於測試或原型設計，而後面的儲存體類型最適合用於生產 Bot。

本教學課程說明如何定義您的儲存體物件，以及如何將使用者輸入儲存到該儲存體物件，以便保存。 

> [!NOTE]
> 不論您選擇使用的儲存體類型為何，用於連結和保存資料的程序都相同。 本教學課程使用`FileStorage` 作為保存資料的儲存體。
> 如需狀態和其他儲存體類型的詳細資訊，請參閱[管理交談和使用者狀態](bot-builder-howto-v4-state.md)。

## <a name="prequisite"></a>必要條件 

本教學課程是以[訂位](bot-builder-tutorial-waterfall.md)教學課程為基礎。 在該教學課程中，您可以建置一個 Bot，其要求使用者輸入有關餐廳訂位的三項資訊。 但是，不會保存使用者的輸入。 本教學課程會將資料儲存體持續性新增至該 Bot。

## <a name="add-storage-to-middleware-layer"></a>將儲存體新增至中介軟體層

Bot 建立器 V4 SDK 會透過狀態管理員中介軟體處理狀態和儲存體。 此中介軟體會提供抽象層，讓您使用簡單的索引鍵/值存放區存取屬性，而不受基礎儲存體類型所影響。 狀態管理員會負責將資料寫入至儲存體和管理並行執行，不論基礎儲存體類型是記憶體內部、檔案儲存體還是 Azure 資料表儲存體。 在本教學課程中，我們將使用 `FileStorage` 來保存使用者輸入。

`FileStorage` 提供者隨附 `bot-builder` 套件。 您一開始使用的範例會使用 `MemoryStorage` 提供者。 該儲存體類型會使用 Bot 的暫時性「內部記憶體」，而該記憶體會在 Bot 重新啟動時加以處置。 另一方面，`FileStorage` 程式庫的運作方式類似於資料庫。 也就是說，此程式庫會將儲存體資訊寫入至本機電腦上的檔案。 您可以指定要將此儲存體檔案放在放置，以便稍後加以檢查。

若要使用 `FileStorage`，尋找 Bot 中 `conversationState` 物件定義所在的陳述式，並加以更新以建立 `new botbuilder.FileStorage("c:/temp")`。 此外，您可以定義此儲存體檔案應寫出的位置。 如此一來，您就可以輕鬆地找到它，以檢查已保存的內容。

# <a name="ctabcstab"></a>[C#](#tab/cstab)
```cs
var storage = new FileStorage("c:/temp");

// These two classes are simply Dictionaries to store state
options.Middleware.Add(new ConversationState<MyBot.convoState>(storage));
options.Middleware.Add(new UserState<MyBot.userState>(storage));
```

您可以從中選擇 Bot 建立器 SDK 所提供的三個不同範圍狀態物件。

| State | 影響範圍 | 說明 |
| ---- | ---- | ---- |
| `dc.ActiveDialog.State` | 對話方塊 | 瀑布式對話方塊可用的狀態 |
| `ConversationState` | 對話 | 目前對話可用的狀態。 |
| `UserState` | user | 多重對話可用的狀態。 |

# <a name="javascripttabjstab"></a>[JavaScript](#tab/jstab)

**app.js**
```javascript
// Storage
const storage = new FileStorage("c:/temp");
const convoState = new ConversationState(storage);
const userState  = new UserState(storage);
adapter.use(new BotStateSet(convoState, userState));
```

`BotStateSet` 可以同時管理 `ConversationState` 和 `UserState`。 在儲存使用者資料時，您可以選擇它。 您可以從中選擇 Bot 建立器 SDK 所提供的三個不同範圍狀態物件。

| State | 影響範圍 | 說明 |
| ---- | ---- | ---- |
| `dc.activeDialog.state` | 對話方塊 | 瀑布式對話方塊可用的狀態 |
| `ConversationState` | 對話 | 目前對話可用的狀態。 |
| `UserState` | user | 多重對話可用的狀態。 |

---
 

## <a name="persist-state"></a>保存狀態

視您要儲存的內容及其需要保存的時間長度而定，您可以對 Bot 寫入至上述其中一個狀態位置。  

# <a name="ctabcstab"></a>[C#](#tab/cstab)

「狀態管理員中介軟體」會在每次結束時為您處理狀態寫入。 所以，您只需要在您的 Bot 中將資料指派給您所選的狀態物件。 在此範例中，我們使用 `dc.ActiveDialog.State` 來追蹤使用者對於預訂的輸入。 如此一來，您可以將使用者輸入儲存在對話方塊的暫存狀態物件範圍，而不需將它儲存在全域變數中。 只要對話方塊正在進行中，此物件就存在；如果您想要將該對話方塊保存更久，則必須將它移轉至其中一個其他狀態物件。 在此情況下，我們會將預訂 `msg` 指派給瀑布式對話方塊最後一個步驟中的對話狀態。

```cs
dialogs.Add("reserveTable", new WaterfallStep[]
{
    async (dc, args, next) =>
    {
        // Prompt for the guest's name.
        await dc.Context.SendActivity("Welcome to the reservation service.");

        dc.ActiveDialog.State = new Dictionary<string, object>();

        await dc.Prompt("dateTimePrompt", "Please provide a reservation date and time.");
    },
    async(dc, args, next) =>
    {
        var dateTimeResult = ((DateTimeResult)args).Resolution.First();

        dc.ActiveDialog.State["date"] = Convert.ToDateTime(dateTimeResult.Value);
        
        // Ask for next info
        await dc.Prompt("partySizePrompt", "How many people are in your party?");

    },
    async(dc, args, next) =>
    {
        dc.ActiveDialog.State["partySize"] = (int)args["Value"];

        // Ask for next info
        await dc.Prompt("textPrompt", "Who's name will this be under?");
    },
    async(dc, args, next) =>
    {
        dc.ActiveDialog.State["name"] = args["Text"];
        string msg = "Reservation confirmed. Reservation details - " +
        $"\nDate/Time: {dc.ActiveDialog.State["date"].ToString()} " +
        $"\nParty size: {dc.ActiveDialog.State["partySize"].ToString()} " +
        $"\nReservation name: {dc.ActiveDialog.State["name"]}";

        var convo = ConversationState<convoState>.Get(dc.Context);

        // In production, you may want to store something more helpful
        convo[$"{dc.ActiveDialog.State["name"]} reservation"] = msg;

        await dc.Context.SendActivity(msg);
        await dc.End();
    }
});
```

# <a name="javascripttabjstab"></a>[JavaScript](#tab/jstab)

「狀態管理員中介軟體」會在每次結束時為您處理將狀態寫入至檔案。 所以，您只需要在您的 Bot 中將資料指派給您所選的狀態物件。 在此範例中，我們使用 `dc.activeDialog.state` 來追蹤 `reservervationInfo` 物件中的使用者輸入。 如此一來，您可以將使用者輸入儲存在對話方塊的暫存狀態物件範圍，而不需將它儲存在全域變數中。 因為只要對話方塊正在進行中，此物件就存在；如果您想要保存該對話方塊，則必須將它移轉至其中一個其他狀態物件。 在此情況下，我們會將 `reservationInfo` 指派給瀑布式對話方塊最後一個步驟中的 `convo` 狀態。

```javascript
// Reserve a table:
// Help the user to reserve a table

dialogs.add('reserveTable', [
    async function(dc, args, next){
        await dc.context.sendActivity("Welcome to the reservation service.");

        dc.activeDialog.state.reservationInfo = {}; // Clears any previous data
        await dc.prompt('dateTimePrompt', "Please provide a reservation date and time.");
    },
    async function(dc, result){
        dc.activeDialog.state.reservationInfo.dateTime = result[0].value;

        // Ask for next info
        await dc.prompt('partySizePrompt', "How many people are in your party?");
    },
    async function(dc, result){
        dc.activeDialog.state.reservationInfo.partySize = result;

        // Ask for next info
        await dc.prompt('textPrompt', "Who's name will this be under?");
    },
    async function(dc, result){
        dc.activeDialog.state.reservationInfo.reserveName = result;
        
        // Persist data
        var convo = conversationState.get(dc.context);; // conversationState.get(dc.context);
        convo.reservationInfo = dc.activeDialog.state.reservationInfo;

        // Confirm reservation
        var msg = `Reservation confirmed. Reservation details: 
            <br/>Date/Time: ${dc.activeDialog.state.reservationInfo.dateTime} 
            <br/>Party size: ${dc.activeDialog.state.reservationInfo.partySize} 
            <br/>Reservation name: ${dc.activeDialog.state.reservationInfo.reserveName}`;
            
        await dc.context.sendActivity(msg);
        await dc.end();
    }
]);
```

---

現在，您已準備好將此項目連結到 Bot 邏輯。

## <a name="start-the-dialog"></a>開始對話

您不需要在此進行任何程式碼變更。 只要執行 Bot 並傳送訊息，即可開始 `reserveTable` 交談。

## <a name="check-file-storage-content"></a>檢查檔案儲存體內容

在您執行 Bot 並已經過 `reserveTable` 對話之後，請在您指定的位置尋找儲存至檔案的資訊 (例如："C:/temp")。 檔案名稱前面會加上 "conversation!" 或 "user!"。 這有助於依照日期將檔案排序，您就可以更輕鬆地找到最新的檔案。

## <a name="next-steps"></a>後續步驟

既然您已經知道如何儲存使用者輸入，就讓我們看看您可使用提示程式庫，要求使用者提供哪種類型的輸入。

> [!div class="nextstepaction"]
> [提示使用者提供輸入](~/v4sdk/bot-builder-prompts.md)
