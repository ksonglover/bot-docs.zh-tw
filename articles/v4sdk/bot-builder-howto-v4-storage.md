---
title: 直接寫入儲存體 | Microsoft Docs
description: 了解如何使用適用於 .NET 的 Bot Builder SDK 直接寫入儲存體。
keywords: 儲存體, 讀取及寫入, 記憶體儲存體, eTag
author: DeniseMak
ms.author: v-demak
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 09/14/18
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 9e63390d157c75e1079654549831cf4936c62710
ms.sourcegitcommit: b78fe3d8dd604c4f7233740658a229e85b8535dd
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/24/2018
ms.locfileid: "49997985"
---
# <a name="write-directly-to-storage"></a>直接寫入儲存體

[!INCLUDE [pre-release-label](../includes/pre-release-label.md)]

您可以將資料直接讀取及寫入至儲存體物件，無需使用中介軟體或內容物件。 這種方式適用於來自 Bot 對話流程以外來源的 Bot 使用者。 例如，假設您的 Bot 允許使用者要求取得氣象報告，且 Bot 會透過外部資料庫讀取氣象報告，以擷取特定日期的報告內容。 氣象資料庫的內容不需依賴使用者的資訊或對話內容，因此您可以直接在儲存體中讀取，不需使用狀態管理員。 本文中的程式碼範例示範如何使用 [記憶體儲存體]、[Cosmos DB]、[Blob 儲存體] 和 [Azure Blob文字記錄儲存區]，將資料讀取和寫入至儲存體。 

## <a name="prerequisites"></a>必要條件
- 如果您沒有 Azure 訂用帳戶，請在開始前建立[免費帳戶](https://azure.microsoft.com/en-us/free/)。
- 安裝 Bot Framework [模擬器](https://github.com/Microsoft/BotFramework-Emulator/releases)

## <a name="memory-storage"></a>記憶體儲存體

我們會先建立 Bot，將資料讀取及寫入至記憶體儲存體。 記憶體儲存體僅供測試用途，而不適用於生產環境。 務必先將儲存體設定為 Cosmos DB 或 Blob 儲存體，再發佈您的 Bot。

#### <a name="build-a-basic-bot"></a>建置基本 Bot

本主題的其餘部分是以 Echo Bot 為基礎。 您可以使用 [C#](../dotnet/bot-builder-dotnet-sdk-quickstart.md) 或 [JS](../javascript/bot-builder-javascript-quickstart.md) 建立一個 Bot。 您可以使用 Bot Framework 模擬器連線至 Bot、與其對話，以及測試您的 Bot。 下列範例會將使用者的每個訊息新增到清單。 包含此清單的資料結構會儲存到您的儲存體。

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

```csharp
using System;
using System.Threading.Tasks;
using Microsoft.Bot.Builder;
using Microsoft.Bot.Builder.TraceExtensions;
using Microsoft.Bot.Builder.Azure;
using Microsoft.Bot.Schema;
using System.Collections.Generic;
using System.Linq;
using System.Threading;

// Create local Memory Storage.
private static readonly MemoryStorage _myStorage = new MemoryStorage();

// Create cancellation token (used by Async Write operation).
public CancellationToken cancellationToken { get; private set; }

// Class for storing a log of utterances (text of messages) as a list.
public class UtteranceLog : IStoreItem
{
     // A list of things that users have said to the bot
     public List<string> UtteranceList { get; } = new List<string>();

     // The number of conversational turns that have occurred        
     public int TurnNumber { get; set; } = 0;

     // Create concurrency control where this is used.
     public string ETag { get; set; } = "*";
}

// Every Conversation turn for our Bot calls this method.
public async Task OnTurnAsync(ITurnContext context)
{
     
     var activityType = context.Activity.Type;
     // See if activity type for this turn is a message from the user.
     if (activityType == ActivityTypes.Message)
     {
         var utterance = context.Activity.Text;
         UtteranceLog logItems = null;
          
         // see if there are previous messages saved in sstorage.
         string[] utteranceList = { "UtteranceLog" };
         logItems = _myStorage.ReadAsync<UtteranceLog>(utteranceList).Result?.FirstOrDefault().Value;

         // If no stored messages were found, create and store a new entry.
         if (logItems is null)
         {
             logItems = new UtteranceLog();
         }
         
         // add new message to list of messages to display.
         logItems.UtteranceList.Add(utterance);
         // increment turn counter.
         logItems.TurnNumber++;
         
         // show user new list of saved messages.
         await context.SendActivityAsync($"The list is now: {string.Join(", ", logItems.UtteranceList)}");
         
         // Create Dictionary object to hold new list of messages.
         {
             changes.Add("UtteranceLog", logItems);
          };
          
          // Save new list to your Storage.
          await _myStorage.WriteAsync(changes,cancellationToken);
     }
     return;
}

```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

```javascript
const { BotFrameworkAdapter, ConversationState, BotStateSet, MemoryStorage } = require('botbuilder');
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

// Add memory storage.
var storage = new MemoryStorage();

const conversationState = new ConversationState(storage);
adapter.use(conversationState);

// Listen for incoming activity .
server.post('/api/messages', (req, res) => {
    // Route received activity to adapter for processing.
    adapter.processActivity(req, res, async (context) => {
        if (context.activity.type === 'message') {
            const state = conversationState.get(context);
            const count = state.count === undefined ? state.count = 0 : ++state.count;

            await logMessageText(storage, context);

            await context.sendActivity(`${count}: You said "${context.activity.text}"`);
        } else {
            await context.sendActivity(`[${context.activity.type} event detected]`);
        }
    });
});

async function logMessageText(storage, context) {
    let utterance = context.activity.text;
    try {
        // Read from the storage.
        let storeItems = await storage.read(["UtteranceLog"])
        // Check the result.
        var utteranceLog = storeItems["UtteranceLog"];

        if (typeof (utteranceLog) != 'undefined') {
            // The log exists so we can write to it.
            storeItems["UtteranceLog"].UtteranceList.push(utterance);

            try {
                await storage.write(storeItems)
                context.sendActivity('Successful write to utterance log.');
            } catch (err) {
                context.sendActivity(`Write failed of UtteranceLog: ${err}`);
            }

         } else {
            context.sendActivity(`need to create new utterance log`);
            storeItems["UtteranceLog"] = { UtteranceList: [`${utterance}`], "eTag": "*" }

            try {
                await storage.write(storeItems)
                context.sendActivity('Successful write to log.');
            } catch (err) {
                context.sendActivity(`Write failed: ${err}`);
            }
        }
    } catch (err) {
        context.sendActivity(`Read rejected. ${err}`);
    };
}
```

---

### <a name="start-your-bot"></a>啟動 Bot
在本機執行您的 Bot。

### <a name="start-the-emulator-and-connect-your-bot"></a>啟動模擬器並且連線至您的 Bot
接下來，請啟動模擬器，然後在模擬器中連線至您的 Bot：

1. 按一下模擬器 [歡迎] 索引標籤中的 [開啟 Bot] 連結。 
2. 在您建立專案的目錄中選取 .bot 檔案。

### <a name="interact-with-your-bot"></a>與您的 Bot 互動
傳送訊息給 Bot，Bot 就會列出所收到的訊息。
![模擬器執行中](../media/emulator-v4/emulator-running.png)

 
## <a name="using-cosmos-db"></a>使用 Cosmos DB
既然您已使用記憶體儲存體，我們會將程式碼更新為使用 Azure Cosmos DB。 Cosmos DB 是 Microsoft 全球發行的多模型資料庫。 Azure Cosmos DB 可讓您有彈性且獨立地跨任意數目的 Azure 地理區域調整輸送量和儲存體。 它利用完整的服務等級協定 (SLA) 提供了輸送量、延遲、可用性和一致性的保證。 

### <a name="set-up"></a>設定
若要在 Bot 中使用 Cosmos DB，您需要先設定一些事項，再進入程式碼。

#### <a name="create-your-database-account"></a>建立資料庫帳戶
1. 在新的瀏覽器視窗中，登入 [Azure 入口網站](http://portal.azure.com)。
2. 按一下 [建立資源] > [資料庫] > [Azure Cosmos DB]。
3. 在 [新增帳戶] 頁面的 [識別碼] 欄位中，提供唯一的名稱。 針對 [API]，選取 [SQL]，然後提供 [訂用帳戶]、[位置] 和 [資源群組] 資訊。
4. 接著，按一下 [建立]。

建立帳戶需要幾分鐘的時間。 等候入口網站顯示 恭喜! 已建立您的 Azure Cosmos DB 帳戶 頁面。

##### <a name="add-a-collection"></a>新增集合
1. 按一下 [設定] > [新增集合]。 [新增集合] 區域會顯示在最右邊，您可能需要向右捲動才能看到它。

![新增 Cosmos DB 集合](./media/add_database_collection.png)

2. 新資料庫集合 "bot-cosmos-sql-db" 的集合識別碼為 "bot-storage"。 我們會在下面的編碼範例中使用這些值。

![Cosmos DB](./media/cosmos-db-sql-database.png)

3. 您資料庫設定的 [金鑰] 索引標籤中會提供端點 URI 和金鑰。 本文稍後設定您的程式碼時，需要使用這些值。 

![Cosmos DB 金鑰](./media/comos-db-keys.png)

#### <a name="add-configuration-information"></a>新增組態資訊
我們要新增 Cosmos DB 儲存體的組態資料簡短又簡單，當您的 Bot 變得複雜時，您可以使用相同方法新增其他組態設定。 此範例使用上述範例中的 Cosmos DB 資料庫和集合名稱。

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

```csharp
private const string CosmosServiceEndpoint = "<your-cosmos-db-URI>";
private const string CosmosDBKey = "<your-cosmos-db-account-key>";
private const string CosmosDBDatabaseName = "bot-cosmos-sql-db";
private const string CosmosDBCollectionName = "bot-storage";
```


# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

將下列資訊新增至 `.env` 檔案。 
 
```javascript
ACTUAL_SERVICE_ENDPOINT=<your database URI>
ACTUAL_AUTH_KEY=<your database key>
DATABASE=Tasks
COLLECTION=Items
```
---

#### <a name="installing-packages"></a>安裝套件
確定您有 Cosmos DB 所需的套件

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

```powershell
Install-Package Microsoft.Bot.Builder.Azure
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

您可以透過 npm，在專案中新增對 botbuilder-azure 的參考：-->


```powershell
npm install --save botbuilder-azure 
```

若要使用 .env 組態檔，Bot 必須包含額外的套件。 首先，從 npm 取得 dotnet 套件：

```powershell
npm install --save dotenv
```

---

### <a name="implementation"></a>實作 

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

下列範例程式碼會使用與上面提供的[記憶體儲存體](#memory-storage)範例相同的 Bot 程式碼執行。
下列程式碼片段示範如何實作 '_myStorage_' 的 Cosmos DB 儲存體，以取代本機記憶體儲存體。 

```csharp
using Microsoft.Bot.Builder.Azure;

// Create access to Cosmos DB storage.
// Replaces Memory Storage with reference to Cosmos DB.
private static readonly CosmosDbStorage _myStorage = new CosmosDbStorage(new CosmosDbStorageOptions
{
   AuthKey = CosmosDBKey,
   CollectionId = CosmosDBCollectionName,
   CosmosDBEndpoint = new Uri(CosmosServiceEndpoint),
   DatabaseId = CosmosDBDatabaseName,
});
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

下列範例程式碼類似於[記憶體儲存體](#memory-storage)，但有些許變更。

需要來自 botbuilder-azure 的 `CosmosDbStorage`，並將 dotenv 設定為讀取 `.env` 檔案。

**app.js**
```javascript
const { CosmosDbStorage } = require("botbuilder-azure");
require('dotenv').config()
```

以對 Cosmos DB 的參考取代記憶體儲存體。

```javascript
//Add CosmosDB 
const storage = new CosmosDbStorage({
    serviceEndpoint: process.env.ACTUAL_SERVICE_ENDPOINT, 
    authKey: process.env.ACTUAL_AUTH_KEY, 
    databaseId: process.env.DATABASE,
    collectionId: process.env.COLLECTION
})

const conversationState = new ConversationState(storage);
adapter.use(conversationState);
```

---

## <a name="start-your-bot"></a>啟動 Bot
在本機執行您的 Bot。

## <a name="start-the-emulator-and-connect-your-bot"></a>啟動模擬器並且連線至您的 Bot
接下來，請啟動模擬器，然後在模擬器中連線至您的 Bot：

1. 按一下模擬器 [歡迎] 索引標籤中的 [開啟 Bot] 連結。 
2. 在您建立專案的目錄中選取 .bot 檔案。

## <a name="interact-with-your-bot"></a>與您的 Bot 互動
傳送訊息給 Bot，Bot 就會列出所收到的訊息。
![模擬器執行中](../media/emulator-v4/emulator-running.png)


### <a name="view-your-data"></a>檢視資料
在您執行 Bot 並儲存您的資訊之後，我們可以在 Azure 入口網站的 [資料總管] 索引標籤下檢視。 

![資料總管範例](./media/data_explorer.PNG)

### <a name="manage-concurrency-using-etags"></a>使用 eTag 管理並行
在 Bot 程式碼範例中，我們會將每個 `IStoreItem` 的 `eTag` 屬性設定為 `*`。 存放區物件的 `eTag` (實體標記) 成員在 Cosmos DB 中用來管理並行作業。 如果另一個 Bot 執行個體變更了 Bot 所寫入的同一個儲存體中的物件，`eTag` 會告知資料庫該採取什麼動作。 

<!-- define optimistic concurrency -->

#### <a name="last-write-wins---allow-overwrites"></a>最後寫入者優先 - 允許覆寫
星號 (`*`) 的 `eTag` 屬性值指出最後寫入者優先。 建立新的資料存放區時，您可以將屬性的 `eTag` 設定為 `*`，以指出您之前沒有儲存過正在寫入的資料或您想要最後寫入的資料覆寫之前已儲存的任何屬性。 如果您的 Bot 沒有並行的問題，針對正在寫入的任何資料，將其 `eTag` 屬性設定為 `*`，以允許覆寫。

#### <a name="maintain-concurrency-and-prevent-overwrites"></a>維護並行和防止覆寫
將資料儲存到 Cosmos DB 時，如果您想要防止並行存取屬性，以及避免覆寫另一個 Bot 執行個體所做的變更，請對 `eTag` 使用 `*` 以外的值。 當 Bot 嘗試儲存狀態資料且 `eTag` 與儲存體中的 `eTag` 值不同時，Bot 會收到含有 `etag conflict key=` 訊息的錯誤回應。 <!-- To control concurrency of data that is stored using `IStorage`, the BotBuilder SDK checks the entity tag (ETag) for `Storage.Write()` requests. -->

根據預設，每次 Bot 寫入到儲存體物件時，Cosmos DB 存放區會檢查其 `eTag` 屬性是否相同，然後在每次寫入之後將該屬性更新為新的唯一值。 如果寫入的 `eTag` 屬性與儲存體中的 `eTag` 不相符，這表示另一個 Bot 或執行緒已變更資料。 

例如，假設您想要 Bot 編輯已儲存的附註，但不想覆寫另一個 Bot 執行個體所作的變更。 如果另一個 Bot 執行個體已進行編輯，則您想要 使用者編輯最新更新的版本。

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

首先，建立會實作 `IStoreItem` 的類別。

```csharp
public class Note : IStoreItem
{
    public string Name { get; set; }
    public string Contents { get; set; }
    public string ETag { get; set; }
}
```

接下來，建立儲存體物件來建立初始附註，並將物件新增到存放區。

```csharp
// create a note for the first time, with a non-null, non-* ETag.
var note = new Note { Name = "Shopping List", Contents = "eggs", ETag = "x" };

var changes = Dictionary<string, object>();
{
    changes.Add("Note", note);
};
await NoteStore.WriteAsync(changes, cancellationToken);
```

然後，稍後存取並更新附註，將它的 `eTag` 保持為從存放區讀取的原狀。

```csharp
var note = NoteStore.ReadAsync<Note>("Note").Result?.FirstOrDefault().Value;

if (note != null)
{
    note.Contents += ", bread";
    var changes = new Dictionary<string, object>();
    {
         changes.Add("Note1", note);
    };
    await NoteStore.WriteAsync(changes, cancellationToken);
}
```

如果存放區中的附註在您寫入變更之前已經更新，對 `Write` 的呼叫將會擲回例外狀況。


# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

將協助程式函式新增至您的 Bot 結尾，可將範例附註寫入資料存放區。
首先建立 `myNoteData` 物件。

```javascript
// Helper function for writing a sample note to a data store
async function createSampleNote(storage, context) {
    var myNoteData = {
        name: "Shopping List",
        contents: "eggs",
        // If any Note file is already stored, the eTag field
        // must be set to "*" in order to allow writing without first reading the stored eTag
        // otherwise you'll likely get an exception indicating an eTag conflict. 
        eTag: "*"
    }
}
```

在 `createSampleNote` 協助程式函式內初始化 `changes` 物件，並在其中新增「附註」，然後將其寫入儲存體。

```javascript
// Write the note data to the "Note" key
var changes = {};
changes["Note"] = myNoteData;
// Creates a file named Note, if it doesn't already exist.
// specifying eTag= "*" will overwrite any existing contents.
// The act of writing to the file automatically updates the eTag property
// The first time you write to Note, the eTag is changed from *, and file contents will become:
//    {"name":"Shopping List","contents":"eggs","eTag":"1"}
try {
    await storage.write(changes);
    await context.sendActivity('Successful created a note.');
} catch (err) {
    await context.sendActivity(`Could not create note: ${err}`);
}
```

接著為了稍後存取及更新附註，我們會建立另一個可以在使用者輸入「更新附註」時存取的協助程式函式。

```javascript
async function updateSampleNote(storage, context) {
    try {
        // Read in a note
        var note = await storage.read(["Note"]);
        console.log(`note.eTag=${note["Note"].eTag}\n note=${JSON.stringify(note)}`);
        // update the note that we just read
        note["Note"].contents += ", bread";
        console.log(`Updated note=${JSON.stringify(note)}`);

        try {
            await storage.write(note); // Write the changes back to storage
            await context.sendActivity('Successfully updated to note.');
        } catch (err) {            console.log(`Write failed: ${err}`);
        }
    }
    catch (err) {
        await context.sendActivity(`Unable to read the Note: ${err}`);
    }
}
```

如果存放區中的附註在您寫入變更之前已經更新，對 `write` 的呼叫將會擲回例外狀況。

---

若要維護並行，一律從儲存體讀取屬性，然後修改所讀取的屬性，如此即可維護 `eTag`。 如果您從存放區讀取使用者資料，回應將會包含 eTag 屬性。 如果您變更資料並將更新的資料寫入存放區，則您的要求應該包含 eTag 屬性且它必須指定與您之前讀取相同的值。 不過，寫入其 `eTag` 設定為 `*` 的物件，將允許寫入可覆寫任何其他變更。

## <a name="using-blob-storage"></a>使用 Blob 儲存體 
Azure Blob 儲存體是 Microsoft 針對雲端推出的物件儲存體解決方案。 Blob 儲存體已針對儲存大量非結構化物件資料 (例如文字或二進位資料) 最佳化。

### <a name="create-your-blob-storage-account"></a>建立 Blob 儲存體帳戶
若要在 Bot 中使用 Blob 儲存體，您需要先設定一些事項，再進入程式碼。
1. 在新的瀏覽器視窗中，登入 [Azure 入口網站](http://portal.azure.com)。
2. 按一下 [建立資源] > [儲存體] > [儲存體帳戶 - Blob、檔案、資料表、佇列]。
3. 在 [新增帳戶] 頁面中，輸入儲存體帳戶的 [名稱]，針對 [帳戶種類] 選取 [Blob 儲存體]，提供 [位置]、[資源群組] 和 [訂用帳戶] 資訊。  
4. 接著，按一下 [建立]。

![建立 Blob 儲存體](./media/create-blob-storage.png)

#### <a name="add-configuration-information"></a>新增組態資訊

尋找您為 Bot 設定 Blob 儲存體所需的 Blob 儲存體金鑰，如上所示：
1. 在 Azure 入口網站中，開啟 Blob 儲存體帳戶，然後選取 [設定] > [存取金鑰]。

![尋找 Blob 儲存體金鑰](./media/find-blob-storage-keys.png)

我們現在會使用兩個金鑰，為我們的程式碼提供 Blob 儲存體帳戶的存取權。

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

```csharp
using Microsoft.Bot.Builder.Azure;
```
更新可將 "myStorage" 指向現有 Blob 儲存體帳戶的程式碼行。

```csharp


private static readonly AzureBlobStorage _myStorage = new AzureBlobStorage("<your-blob-storage-account-string>", "<your-blob-storage-container-name>");
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)
```javascript
const mystorage = new BlobStorage({
   <youy_containerName>,
   <your_storageAccountOrConnectionString>,
   <your_storageAccessKey>
})
```
---

將 "myStorage" 設定為指向 Blob 儲存體帳戶後，Bot 程式碼現在會儲存和擷取 Blob 儲存體中的資料。

## <a name="start-your-bot"></a>啟動 Bot
在本機執行您的 Bot。

## <a name="start-the-emulator-and-connect-your-bot"></a>啟動模擬器並且連線至您的 Bot
接下來，請啟動模擬器，然後在模擬器中連線至您的 Bot：

1. 按一下模擬器 [歡迎] 索引標籤中的 [開啟 Bot] 連結。 
2. 在您建立專案的目錄中選取 .bot 檔案。

## <a name="interact-with-your-bot"></a>與您的 Bot 互動
傳送訊息給 Bot，Bot 就會列出所收到的訊息。
![模擬器執行中](../media/emulator-v4/emulator-running.png)

### <a name="view-your-data"></a>檢視資料
在您執行 Bot 並儲存您的資訊之後，我們可以在 Azure 入口網站的 [儲存體總管] 索引標籤下檢視。

## <a name="blob-transcript-storage"></a>Blob 文字記錄儲存體
Azure Blob 文字記錄儲存體會提供一個特製化儲存體選項，讓您輕鬆地以記錄的文字記錄形式儲存和擷取使用者對話。 Azure Blob 文字記錄儲存體特別適合用於自動擷取使用者輸入，以便在對 Bot 效能進行偵錯時檢查。

### <a name="set-up"></a>設定
Azure Blob 文字記錄儲存體可以使用遵循上面「建立 Blob 儲存體帳戶」和「新增組態資訊」小節中詳述的步驟所建立的相同 Blob 儲存體帳戶。 在本文中，我們新增了新的 Blob 容器 "mybottranscripts"。 

### <a name="implementation"></a>實作 
下列程式碼會將文字記錄儲存體指標 "transcriptStore" 連到新的 Azure Blob 文字記錄儲存體帳戶。 要儲存此處所示使用者對話的原始程式碼是以[對話歷程記錄](https://aka.ms/bot-history-sample-code)範例為基礎。 

```csharp
// In Startup.cs
using Microsoft.Bot.Builder.Azure;

// Enable the conversation transcript middleware.
blobStore = new AzureBlobTranscriptStore(blobStorageConfig.ConnectionString, storageContainer);
var transcriptMiddleware = new TranscriptLoggerMiddleware(blobStore);
options.Middleware.Add(transcriptMiddleware);

// In ConversationHistoryBot.cs
using Microsoft.Bot.Builder;
using Microsoft.Bot.Builder.Azure;
using Microsoft.Bot.Connector;
using Microsoft.Bot.Schema;

private readonly AzureBlobTranscriptStore _transcriptStore;

/// <param name="transcriptStore">Injected via ASP.NET dependency injection.</param>
public ConversationHistoryBot(AzureBlobTranscriptStore transcriptStore)
{
    _transcriptStore = transcriptStore ?? throw new ArgumentNullException(nameof(transcriptStore));
}

```

### <a name="store-user-conversations-in-azure-blob-transcripts"></a>在 Azure blob 文字記錄中儲存使用者對話
在 Blob 容器可用於儲存文字記錄之後，您可以開始保留使用者與 Bot 的對話。 這些對話稍後可當作偵錯工具，查看使用者與 Bot 的互動方式。 當 ctivity.text 收到輸入訊息 _!history_ 時，下列程式碼會保留使用者對話輸入。


```csharp
/// <summary>
/// Every Conversation turn for our EchoBot will call this method. 
/// </summary>
/// <param name="turnContext">A <see cref="ITurnContext"/> containing all the data needed
/// for processing this conversation turn. </param>        
public async Task OnTurnAsync(ITurnContext turnContext, CancellationToken cancellationToken = default(CancellationToken))
{
    var activity = turnContext.Activity;
    if (activity.Type == ActivityTypes.Message)
    {
        if (activity.Text == "!history")
        {
           // Download the activities from the Transcript (blob store) when a request to upload history arrives.
           var connectorClient = turnContext.TurnState.Get<ConnectorClient>(typeof(IConnectorClient).FullName);
           // Get all the message type activities from the Transcript.
           string continuationToken = null;
           var count = 0;
           do
           {
               var pagedTranscript = await _transcriptStore.GetTranscriptActivitiesAsync(activity.ChannelId, activity.Conversation.Id);
               var activities = pagedTranscript.Items
                  .Where(a => a.Type == ActivityTypes.Message)
                  .Select(ia => (Activity)ia)
                  .ToList();
               
               var transcript = new Transcript(activities);

               await connectorClient.Conversations.SendConversationHistoryAsync(activity.Conversation.Id, transcript, cancellationToken: cancellationToken);

               continuationToken = pagedTranscript.ContinuationToken;
           }
           while (continuationToken != null);
```

### <a name="find-all-stored-transcripts-for-your-channel"></a>針對您的通道尋找所有預存的文字記錄
若要查看您已儲存哪些資料，您可使用下列程式碼，以程式設計方式尋找您已儲存的所有文字記錄的 "ConversationIDs"。 使用 Bot 模擬器來測試您的程式碼時，選取 [從頭開始] 可以開始新的文字記錄，其具有新的 "ConversationID"。

```csharp
List<string> storedTranscripts = new List<string>();
PagedResult<Transcript> pagedResult = null;
var pageSize = 0;
do
{
    pagedResult = await _transcriptStore.ListTranscriptsAsync("emulator", pagedResult?.ContinuationToken);
    
    // transcript item contains ChannelId, Created, Id.
    // save the converasationIds (Id) found by "ListTranscriptsAsync" to a local list.
    foreach (var item in pagedResult.Items)
    {
         // Make sure we store an unescaped conversationId string.
         var strConversationId = item.Id;
         storedTranscripts.Add(Uri.UnescapeDataString(strConversationId));
    }
} while (pagedResult.ContinuationToken != null);
```

### <a name="retrieve-user-conversations-from-azure-blob-transcript-storage"></a>從 Azure blob 文字記錄儲存體擷取使用者對話
將 Bot 互動文字記錄儲存到 Azure Blob 文字記錄存放區之後，您可以使用 AzureBlobTranscriptStorage 方法 "GetTranscriptActivities"，以程式設計方式擷取這些文字記錄進行測試或偵錯。 下列程式碼片段可擷取在前 24 小時內，從每個預存的文字記錄接收，並儲存的所有使用者輸入文字記錄。


```csharp
var numTranscripts = storedTranscripts.Count();
for (int i = 0; i < numTranscripts; i++)
{
    PagedResult<IActivity> pagedActivities = null;
    do
    {
        string thisConversationId = storedTranscripts[i];
        // Find all inputs in the last 24 hours.
        DateTime yesterday = DateTime.Now.AddDays(-1);
        // Retrieve iActivities for this transcript.
        pagedActivities = await _myTranscripts.GetTranscriptActivitiesAsync("emulator", thisConversationId, pagedActivities?.ContinuationToken, yesterday);
        foreach (var item in pagedActivities.Items)
        {
            // View as message and find value for key "text" :
            var thisMessage = item.AsMessageActivity();
            var userInput = thisMessage.Text;
         }
    } while (pagedActivities.ContinuationToken != null);
}
```

### <a name="remove-stored-transcripts-from-azure-blob-transcript-storage"></a>從 Azure blob 文字記錄儲存體中移除預存的文字記錄
在您完成使用使用者輸入資料來測試或偵錯 Bot 時，您可以程式設計方式從 Azure Blob 文字記錄存放區中移除預存的文字記錄。 下列程式碼片段可從 Bot 文字記錄存放區中移除所有預存的文字記錄。


```csharp
for (int i = 0; i < numTranscripts; i++)
{
   // Remove all stored transcripts except the last one found.
   if (i > 0)
   {
       string thisConversationId = storedTranscripts[i];    
       await _transcriptStore.DeleteTranscriptAsync("emulator", thisConversationId);
    }
}
```


以下連結提供關於 [Azure Blob 文字記錄儲存體](https://docs.microsoft.com/en-us/dotnet/api/microsoft.bot.builder.azure.azureblobtranscriptstore)的詳細資訊  

## <a name="next-steps"></a>後續步驟
既然您已經知道如何直接從儲存體讀取和寫入，現在我們來看看如何使用狀態管理員來為您執行這些動作。

> [!div class="nextstepaction"]
> [使用交談和使用者屬性儲存狀態](bot-builder-howto-v4-state.md)

