---
title: 直接寫入儲存體 | Microsoft Docs
description: 了解如何使用適用於 .NET 的 Bot Builder SDK V4 直接寫入儲存體。
keywords: 儲存體, 讀取及寫入, 記憶體儲存體, eTag
author: DeniseMak
ms.author: v-demak
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 05/2/18
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 76f8976aefe3d4fefcffc46e691dbd0b35e41ec7
ms.sourcegitcommit: 1abc32353c20acd103e0383121db21b705e5eec3
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/21/2018
ms.locfileid: "42756742"
---
# <a name="write-directly-to-storage"></a>直接寫入儲存體

<!--
 Note for V4: You can write directly to storage without using the state manager. Therefore, this topic isn't called "managing state". State is in a separate topic.
 ## Manage state data by writing directly to storage
-->
[!INCLUDE [pre-release-label](../includes/pre-release-label.md)]

您可以透過直接讀取及寫入儲存體物件的方式，將資料直接寫入儲存體，無需使用內容物件或中介軟體。

此程式碼範例示範如何讀取儲存體資料及將資料寫入儲存體。 在此範例中，儲存體是一個檔案，但是您可以輕鬆地變更程式碼來初始化儲存體物件，改為使用 Azure 表格儲存體的記憶體儲存體。 

# <a name="ctabcsharpechorproperty"></a>[C#](#tab/csharpechorproperty)
我們會定義物件，並使用 `IStorage.Write` 和 `IStorage.Read` 來儲存及擷取狀態。 

下列範例會將使用者的每個訊息新增到清單。 包含清單的資料結構會儲存到您提供給 `FileStorage` 建構函式之目錄內的檔案。

從 BotBuilder V4 SDK 中的 EchoBot Visual Studio 範本開始。
編輯 `EchoBot.cs` 檔案。 新增下列 using 陳述式並取代 `EchoBot` 類別定義。

```csharp
using System.Collections.Generic;
using System.Linq;
```

```csharp
// In the constructor initialize file storage
public class EchoBot : IBot
{
    private readonly FileStorage _myStorage;

    public EchoBot()
    {
        _myStorage = new FileStorage(System.IO.Path.GetTempPath());
    }

    // Add a class for storing a log of utterances (text of messages) as a list
    public class UtteranceLog : IStoreItem
    {
        // A list of things that users have said to the bot
        public List<string> UtteranceList { get; private set; } = new List<string>();

        // The number of conversational turns that have occurred        
        public int TurnNumber { get; set; } = 0;

        public string eTag { get; set; } = "*";
    }

    // Replace the OnTurn in EchoBot.cs with the following:
    public async Task OnTurn(ITurnContext context)
    {
        var activityType = context.Activity.Type;

        await context.SendActivity($"Activity type: {context.Activity.Type}.");

        if (activityType == ActivityTypes.Message)
        {
            // *********** begin (create or add to log of messages)
            var utterance = context.Activity.Text;
            bool restartList = false;

            if (utterance.Equals("restart"))
            {
                restartList = true;
            }

            // Attempt to read the existing property bag
            UtteranceLog logItems = null;
            try
            {
                logItems = _myStorage.Read<UtteranceLog>("UtteranceLog").Result?.FirstOrDefault().Value;
            }
            catch (System.Exception ex)
            {
                await context.SendActivity(ex.Message);
            }

            // If the property bag wasn't found, create a new one
            if (logItems is null)
            {
                try
                {
                    // add the current utterance to a new object.
                    logItems = new UtteranceLog();
                    logItems.UtteranceList.Add(utterance);

                    await context.SendActivity($"The list is now: {string.Join(", ",logItems.UtteranceList)}");

                    var changes = new KeyValuePair<string, object>[]
                    {
                        new KeyValuePair<string, object>("UtteranceLog", logItems)
                    };
                    await _myStorage.Write(changes);
                }
                catch (System.Exception ex)
                {
                    await context.SendActivity(ex.Message);
                }
            }
            // logItems.ContainsKey("UtteranceLog") == true, we were able to read a log from storage
            else
            {
                // Modify its property
                if (restartList)
                {
                    logItems.UtteranceList.Clear();
                }
                else
                {
                    logItems.UtteranceList.Add(utterance);
                    logItems.TurnNumber++;
                }

                await context.SendActivity($"The list is now: {string.Join(", ", logItems.UtteranceList)}");

                var changes = new KeyValuePair<string, object>[]
                {
                        new KeyValuePair<string, object>("UtteranceLog", logItems)
                };
                await _myStorage.Write(changes);
            }
        }

        return;
    }
}
```
# <a name="javascripttabjsechoproperty"></a>[JavaScript](#tab/jsechoproperty)

下列範例會將使用者的每個訊息新增到清單。 包含清單的資料結構會儲存到您提供給 `FileStorage` 建構函式之目錄內的檔案。

將下列程式碼貼到 `app.js` 中。
``` javascript
const { BotFrameworkAdapter, FileStorage, MemoryStorage, ConversationState, BotStateSet } = require('botbuilder');
const restify = require('restify');

// Create server.
let server = restify.createServer();
server.listen(process.env.port || process.env.PORT || 3978, function () {
    console.log(`${server.name} listening to ${server.url}`);
});

// Create adapter.
const adapter = new BotFrameworkAdapter({
    appId: process.env.MICROSOFT_APP_ID,
    appPassword: process.env.MICROSOFT_APP_PASSWORD
});

// Add storage.
var storage = new FileStorage("C:/temp");
const conversationState = new ConversationState(storage);
adapter.use(new BotStateSet(conversationState));

// Listen for incoming activities.
server.post('/api/messages', (req, res) => {
    // Route received activity to adapter for processing.
    adapter.processActivity(req, res, async (context) => {
        if (context.activity.type === 'message') {
            const state = conversationState.get(context);
            const count = state.count === undefined ? state.count = 0 : ++state.count;

            let utterance = context.activity.text;
            let storeItems = await storage.read(["UtteranceLog"])
            try {
                // check result
                var utteranceLog = storeItems["UtteranceLog"];
                
                if (typeof (utteranceLog) != 'undefined') {
                    // log exists so we can write to it
                    storeItems["UtteranceLog"].UtteranceList.push(utterance);
                    
                    await storage.write(storeItems)
                    try {
                        context.sendActivity('Successful write.');
                    } catch (err) {
                        context.sendActivity(`Srite failed of UtteranceLog: ${err}`);
                    }

                } else {
                    context.sendActivity(`need to create new utterance log`);
                    storeItems["UtteranceLog"] = { UtteranceList: [`${utterance}`], "eTag": "*" }
                    await storage.write(storeItems)
                    try {
                        context.sendActivity('Successful write.');
                    } catch (err) {
                        context.sendActivity(`Write failed: ${err}`);
                    }
                }
            } catch (err) {
                context.sendActivity(`Read rejected. ${err}`);
            };

            await context.sendActivity(`${count}: You said "${context.activity.text}"`);
        } else {
            await context.sendActivity(`[${context.activity.type} event detected]`);
        }
    });
});
```
---

## <a name="manage-concurrency-using-etags"></a>使用 eTag 管理並行

在上一個範例中，您將 `eTag` 屬性設定為 `*`。 存放區物件的 `eTag` (實體標記) 成員是用來管理並行。 `eTag` 指出如果另一個 Bot 執行個體變更了 Bot 正在寫入之儲存體中的物件時應該採取什麼動作。 

<!-- define optimistic concurrency -->

### <a name="last-write-wins---allow-overwrites"></a>最後寫入者優先 - 允許覆寫

將 eTag 設定為 `*` 以允許其他 Bot 執行個體覆寫先前寫入的資料。 星號 (`*`) 的 `eTag` 屬性值指出最後寫入者優先。 建立新的資料存放區時，您可以將屬性的 `eTag` 設定為 `*`，以指出您之前沒有儲存過正在寫入的資料或您想要最後寫入的資料覆寫之前已儲存的任何屬性。 如果您的 Bot 沒有並行的問題，則將正在寫入的任何資料的 `eTag` 屬性設定為 `*` 將會允許覆寫。

這顯示在下列程式碼範例中。

# <a name="ctabcsetagoverwrite"></a>[C#](#tab/csetagoverwrite)
使用我們稍早定義的 `UtteranceLog` 類別。
```csharp
// Add the current utterance to a new object and save it to storage.
logItems = new UtteranceLog();
logItems.UtteranceList.Add(utterance);
logItems.eTag = "*";

var changes = new KeyValuePair<string, object>[]
{
    new KeyValuePair<string, object>("UtteranceLog", logItems)
};
await _myStorage.Write(changes);

```
# <a name="javascripttabjstagoverwrite"></a>[JavaScript](#tab/jstagoverwrite)
將新的語句新增到記錄檔並允許覆寫。

```javascript
storeItems["UtteranceLog"] = { UtteranceList: [`${utterance}`], "eTag": "*" }
await storage.write(storeItems)
```
---

### <a name="maintain-concurrency-and-prevent-overwrites"></a>維護並行和防止覆寫
如果您想要防止並行存取屬性，則需要為 `eTag` 使用 `*` 以外的值，以避免覆寫另一個 Bot 執行個體所做的變更。 當 Bot 嘗試儲存狀態資料且 `eTag` 與儲存體中的不同時，Bot 會收到含有訊息 `etag conflict key=` 的錯誤回應。 <!-- To control concurrency of data that is stored using `IStorage`, the BotBuilder SDK checks the entity tag (ETag) for `Storage.Write()` requests. -->

根據預設，每次 Bot 寫入到儲存體物件時，存放區會檢查它的 `eTag` 屬性是否相同，然後在每次寫入之後將該屬性更新為新的唯一值。 如果寫入的 `eTag` 屬性與儲存體中的 `eTag` 不相符，這表示另一個 Bot 或執行緒已變更資料。 

例如，假設您想要 Bot 編輯已儲存的附註，但不想覆寫另一個 Bot 執行個體所作的變更。 如果另一個 Bot 執行個體已進行編輯，則您想要 使用者編輯最新更新的版本。

# <a name="ctabcsetag"></a>[C#](#tab/csetag)
首先，建立會實作 `IStoreItem` 的類別。
```csharp
public class Note : IStoreItem
{
    public string Name { get; set; }
    public string Contents { get; set; }
    public string eTag { get; set; }
}
```
接下來，建立儲存體物件來建立初始附註，並將物件新增到存放區。
```csharp
// create a note for the first time, with a non-null, non-* eTag.
var note = new Note { Name = "Shopping List", Contents = "eggs", eTag = "x" };

var changes = new KeyValuePair<string, object>[]
{
    new KeyValuePair<string, object>("Note", note)
};
await NoteStore.Write(changes);
```
然後，稍後存取並更新附註，將它的 `eTag` 保持為從存放區讀取的原狀。
```csharp
var note = NoteStore.Read<Note>("Note").Result.FirstOrDefault().Value;

if (note != null)
{
    note.Contents += ", bread";
    var changes = new KeyValuePair<string, object>[]
    {
        new KeyValuePair<string, object>("Note1", note)
    };
    await NoteStore.Write(changes);
}
```
如果存放區中的附註在您寫入變更之前已經更新，對 `Write` 的呼叫將會擲回例外狀況。

# <a name="javascripttabjsetag"></a>[JavaScript](#tab/jsetag)

首先，建立 `myNote` 物件。
```javascript
var myNote = {
    name: "Shopping List",
    contents: "eggs",
    eTag: "*"
}
```
接下來，初始化 `changes` 物件並在其中新增*附註*，然後將它寫入儲存體。

```javascript
// Write a note
var changes = {};
changes["Note"] = myNote;
await storage.write(changes);
```
然後，稍後存取並更新附註，將它的 `eTag` 保持為從存放區讀取的原狀。
```javascript
// Read in a note
var note = await storage.read(["Note"]);
try {
    // Someone has updated the note. Need to update our local instance to match
    if(myNote.eTag != note.eTag){
        myNote = note;
    }

    // Add any updates to the note and write back out
    myNote.contents += ", bread";   // Add to the note
    changes["Note"] = note;
    await storage.write(changes); // Write the changes back to storage
    try {
        context.sendActivity('Successful write.');
    } catch (err) {
        context.sendActivity(`Write failed: ${err}`);
    }
}
catch (err) {
    context.sendActivity(`Unable to read the Note: ${err}`);
}
```
如果存放區中的附註在您寫入變更之前已經更新，對 `Write` 的呼叫將會擲回例外狀況。

---

若要維護並行，一律從儲存體讀取屬性，然後修改所讀取的屬性，如此即可維護 `eTag`。 如果您從存放區讀取使用者資料，回應將會包含 eTag 屬性。 如果您變更資料並將更新的資料寫入存放區，則您的要求應該包含 eTag 屬性且它必須指定與您之前讀取相同的值。 不過，寫入其 `eTag` 設定為 `*` 的物件，將允許寫入可覆寫任何其他變更。

<!-- If the ETag specified in your `Storage.Write()` request matches the current value in the store, the server will save the data and specify a new eTag value in the body of the response, that indicates that the data has been updated. If the ETag specified in your Storage.Write() request does not match the current value in the store, the bot responds with an error indicating an eTag conflict, to indicate that the user's data in the store has changed since you last saved or retrieved it. -->

<!-- TODO: new snippet -->

<!-- jf: I think this section can be cut entirely.

## Save a conversation reference using storage

You can use IStorage to save BotBuilder SDK objects as well as user-defined data. This code snippet uses `Storage.Write()` to save a ConversationReference object for use in sending a proactive message later.

# [C#](#tab/csharpwriteconvref)
```csharp
            var reference = context.ConversationReference;
            var userId = reference.User.Id;

            // save the ConversationReference to a global variable of this class
            conversationReference = reference;

            StoreItems storeItems = new StoreItems();
            StoreItem conversationReferenceToStore = new StoreItem();
            // set the eTag to "*" to indicate you're overwriting previous data
            conversationReferenceToStore.eTag = "*";
            conversationReferenceToStore.Add("ref", reference);
            storeItems[$"ConversationReference/{userId}"] = conversationReferenceToStore;

            await storage.Write(storeItems).ConfigureAwait(false);

            SubscribeUser(userId);

            await context.SendActivity("Thank You! We will message you shortly.");
        }

public void SubscribeUser(string userId)
{
    CreateContextForUserAsync(userId, async (ITurnContext context) =>
    {
        await context.SendActivity("You've been notified.");
        await Task.Delay(2000);
    });
                
}
```
# [JavaScript](#tab/jswriteconvref)
```javascript
const { MemoryStorage } = require('botbuilder');

const storage = new MemoryStorage();

// Listen for incoming activity 
server.post('/api/messages', (req, res) => {
    // Route received activity to adapter for processing
    adapter.processActivity(req, res, async (context) => {
        if (context.activity.type === 'message') {
            const utterances = (context.activity.text || '').trim().toLowerCase()
            if (utterances === 'subscribe') {
                const reference = context.activity;
                const userId = reference.id;
                const changes = {};
                changes['reference/' + userId] = reference;
                await storage.write(changes)
                await subscribeUser(userId)
                await context.sendActivity(`Thank You! We will message you shortly.`);
               
            } else{
                await context.sendActivity("Say 'subscribe'");
            }
    
        }
    });
});
```
---

To read the saved object from storage, call `Storage.Read()`.

# [C#](#tab/csharpread)
```csharp


        private async void CreateContextForUserAsync(String userId,Func<ITurnContext, Task> onReady)
        {
            var referenceKey = $"ConversationReference/{userId}";
            
            ConversationReference localRef = null;
            StoreItems conversationReferenceFromStorage = await storage.Read(referenceKey);
            foreach (var item in conversationReferenceFromStorage)
            {
                StoreItem value = item.Value as StoreItem;
                localRef = value["ref"];
            }
            
            await bot.CreateContext(this.conversationReference, onReady);

        }
```
# [JavaScript](#tab/jsread)
```javascript
async function subscribeUser(userId) {
    setTimeout(() => {
        createContextForUser(userId, (context) => {
            context.sendActivity("You have been notified");
        })
    }, 2000);
}

async function createContextForUser(userId, callback) {
    const referenceKey = 'reference/' + userId;
    var rows = await storage.read([referenceKey])
    var reference = await rows[referenceKey]
    await callback(adapter.createContext(reference))
          
}
```
---

-->

## <a name="next-steps"></a>後續步驟

既然您已經知道如何直接從儲存體讀取和寫入，現在我們來看看如何使用狀態管理員來為您執行這些動作。

> [!div class="nextstepaction"]
> [使用交談和使用者屬性儲存狀態](bot-builder-howto-v4-state.md)

## <a name="additional-resources"></a>其他資源

- [概念：Bot Builder SDK 中的儲存體](bot-builder-storage-concept.md)


