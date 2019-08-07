---
title: 直接寫入儲存體 | Microsoft Docs
description: 了解如何使用適用於 .NET 的 Bot Framework SDK 直接寫入儲存體。
keywords: 儲存體, 讀取及寫入, 記憶體儲存體, eTag
author: DeniseMak
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 05/23/2019
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 1213e7d0c03fd6faa3768fac85a59bf4c7b5b28a
ms.sourcegitcommit: a1eaa44f182a7210197bd793250907df00e9edab
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/03/2019
ms.locfileid: "68756976"
---
# <a name="write-directly-to-storage"></a>直接寫入儲存體

[!INCLUDE[applies-to](../includes/applies-to.md)]

您可以將資料直接讀取及寫入至儲存體物件，無需使用中介軟體或內容物件。 這種方式適用於 Bot 使用者用來保留交談的資料，或來自 Bot 交談流程以外來源的資料。 在此資料儲存體模型中，資料會直接從儲存體讀取，而非使用狀態管理員。 本文中的程式碼範例示範如何使用 [記憶體儲存體]  、[Cosmos DB]  、[Blob 儲存體]  和 [Azure Blob文字記錄儲存區]  ，將資料讀取和寫入至儲存體。 

## <a name="prerequisites"></a>必要條件
- 如果您沒有 Azure 訂用帳戶，請在開始前建立[免費帳戶](https://azure.microsoft.com/free/)。
- 熟悉本文：本機上針對 [dotnet](https://aka.ms/bot-framework-www-c-sharp-quickstart) 或 [nodeJS](https://aka.ms/bot-framework-www-node-js-quickstart) 建立 Bot。
- 適用於 [C#](https://aka.ms/bot-vsix)或 [nodeJS](https://nodejs.org) 和 [yeoman](http://yeoman.io) 的 Bot Framework SDK v4 範本。

## <a name="about-this-sample"></a>關於此範例
本文中的範例程式碼是以基本 Echo Bot 的結構開頭，然後藉由新增額外的程式碼 (下面提供) 來擴充該 Bot 的功能。 此擴充程式碼會建立一份清單，以保留其收到的使用者輸入。 在每個回合中，系統會將完整的使用者輸入清單回應給使用者。 在該回合結束時，包含此輸入清單的資料結構會接著儲存到儲存體。 此範例程式碼中新增其他功能時，系統會探索各種類型的儲存體。

## <a name="memory-storage"></a>記憶體儲存體

Bot Framework SDK 可讓您使用「記憶體內部儲存體」來儲存使用者輸入。 記憶體儲存體僅供測試用途，而不適用於生產環境。 生產 Bot 最適合使用永續性儲存體類型 (例如資料庫儲存體)。 務必先將儲存體設定為 Cosmos DB 或 Blob 儲存體，再發佈您的 Bot。

#### <a name="build-a-basic-bot"></a>建置基本 Bot

本主題的其餘部分是以 Echo Bot 為基礎。 依照建置 [C# EchoBot](https://aka.ms/bot-framework-www-c-sharp-quickstart) 或 [JS EchoBot](https://aka.ms/bot-framework-www-node-js-quickstart) 的快速入門指示，可以在本機建置 Echo Bot 範例程式碼。

### <a name="ctabcsharp"></a>[C#](#tab/csharp)

**EchoBot.cs**
```csharp
using System;
using System.Threading.Tasks;
using Microsoft.Bot.Builder;
using Microsoft.Bot.Schema;
using System.Collections.Generic;
using System.Linq;
using System.Threading;

// Represents a bot saves and echoes back user input.
public class EchoBot : ActivityHandler
{
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
     
   // Echo back user input.
   protected override async Task OnMessageActivityAsync(ITurnContext<IMessageActivity> turnContext, CancellationToken cancellationToken)
   {
      // preserve user input.
      var utterance = turnContext.Activity.Text;  
      // make empty local logitems list.
      UtteranceLog logItems = null;
          
      // see if there are previous messages saved in storage.
      try
      {
         string[] utteranceList = { "UtteranceLog" };
         logItems = _myStorage.ReadAsync<UtteranceLog>(utteranceList).Result?.FirstOrDefault().Value;
      }
      catch
      {
         // Inform the user an error occured.
         await turnContext.SendActivityAsync("Sorry, something went wrong reading your stored messages!");
      }
         
      // If no stored messages were found, create and store a new entry.
      if (logItems is null)
      {
         // add the current utterance to a new object.
         logItems = new UtteranceLog();
         logItems.UtteranceList.Add(utterance);
         // set initial turn counter to 1.
         logItems.TurnNumber++;

         // Show user new user message.
         await turnContext.SendActivityAsync($"{logItems.TurnNumber}: The list is now: {string.Join(", ", logItems.UtteranceList)}");

         // Create Dictionary object to hold received user messages.
         var changes = new Dictionary<string, object>();
         {
            changes.Add("UtteranceLog", logItems);
         }
         try
         {
            // Save the user message to your Storage.
            await _myStorage.WriteAsync(changes, cancellationToken);
         }
         catch
         {
            // Inform the user an error occured.
            await turnContext.SendActivityAsync("Sorry, something went wrong storing your message!");
         }
      }
      // Else, our Storage already contained saved user messages, add new one to the list.
      else
      {
         // add new message to list of messages to display.
         logItems.UtteranceList.Add(utterance);
         // increment turn counter.
         logItems.TurnNumber++;
         
         // show user new list of saved messages.
         await turnContext.SendActivityAsync($"{logItems.TurnNumber}: The list is now: {string.Join(", ", logItems.UtteranceList)}");
         
         // Create Dictionary object to hold new list of messages.
         var changes = new Dictionary<string, object>();
         {
            changes.Add("UtteranceLog", logItems);
         };
         
         try
         {
            // Save new list to your Storage.
            await _myStorage.WriteAsync(changes,cancellationToken);
         }
         catch
         {
            // Inform the user an error occured.
            await turnContext.SendActivityAsync("Sorry, something went wrong storing your message!");
         }
      }
      ...  // OnMessageActivityAsync( )
   }
}

```

### <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

若要使用 .env 組態檔，您的 Bot 必須包含額外的套件。 如果尚未安裝，請從 npm 取得 dotnet 套件：

```powershell
npm install --save dotenv
```

**bot.js**
```javascript
const { ActivityHandler, MemoryStorage } = require('botbuilder');
const restify = require('restify');

// Add memory storage.
var storage = new MemoryStorage();

// Process incoming requests - adds storage for messages.
class MyBot extends ActivityHandler {
    constructor() {
        super();
        // See https://aka.ms/about-bot-activity-message to learn more about the message and other activity types.
        this.onMessage(async turnContext => { console.log('this gets called (message)'); 
        await turnContext.sendActivity(`You said '${ turnContext.activity.text }'`); 
        // Save updated utterance inputs.
        await logMessageText(storage, turnContext);
    });
        this.onConversationUpdate(async turnContext => { console.log('this gets called (conversation update)'); 
        await turnContext.sendActivity('[conversationUpdate event detected]'); });
    }
}

// This function stores new user messages. Creates new utterance log if none exists.
async function logMessageText(storage, turnContext) {
    let utterance = turnContext.activity.text;
    // debugger;
    try {
        // Read from the storage.
        let storeItems = await storage.read(["UtteranceLogJS"])
        // Check the result.
        var UtteranceLogJS = storeItems["UtteranceLogJS"];
        if (typeof (UtteranceLogJS) != 'undefined') {
            // The log exists so we can write to it.
            storeItems["UtteranceLogJS"].turnNumber++;
            storeItems["UtteranceLogJS"].UtteranceList.push(utterance);
            // Gather info for user message.
            var storedString = storeItems.UtteranceLogJS.UtteranceList.toString();
            var numStored = storeItems.UtteranceLogJS.turnNumber;

            try {
                await storage.write(storeItems)
                turnContext.sendActivity(`${numStored}: The list is now: ${storedString}`);
            } catch (err) {
                turnContext.sendActivity(`Write failed of UtteranceLogJS: ${err}`);
            }
        }
        else{
            turnContext.sendActivity(`Creating and saving new utterance log`);
            var turnNumber = 1;
            storeItems["UtteranceLogJS"] = { UtteranceList: [`${utterance}`], "eTag": "*", turnNumber }
            // Gather info for user message.
            var storedString = storeItems.UtteranceLogJS.UtteranceList.toString();
            var numStored = storeItems.UtteranceLogJS.turnNumber;

            try {
                await storage.write(storeItems)
                turnContext.sendActivity(`${numStored}: The list is now: ${storedString}`);
            } catch (err) {
                turnContext.sendActivity(`Write failed: ${err}`);
            }
        }
    }
    catch (err){
        turnContext.sendActivity(`Read rejected. ${err}`);
    }
}

module.exports.MyBot = MyBot;

```
---

### <a name="start-your-bot"></a>啟動 Bot
在本機執行您的 Bot。

### <a name="start-the-emulator-and-connect-your-bot"></a>啟動模擬器並且連線至您的 Bot
- 安裝 Bot Framework [Emulator](https://aka.ms/bot-framework-emulator-readme) 接下來，啟動模擬器，然後在模擬器中連線至您的 Bot：

1. 按一下模擬器 [歡迎使用] 索引標籤中的 [建立新的 Bot 設定]  連結。 
2. 以您啟動 Bot 時顯示在網頁上的資訊，填妥欄位以連線到 Bot。

### <a name="interact-with-your-bot"></a>與您的 Bot 互動
將訊息傳送給 Bot。 Bot 會列出所收到的訊息。

![模擬器測試儲存體 Bot](./media/emulator-direct-storage-test.png)

## <a name="using-cosmos-db"></a>使用 Cosmos DB
既然您已使用記憶體儲存體，我們會將程式碼更新為使用 Azure Cosmos DB。 Cosmos DB 是 Microsoft 全球發行的多模型資料庫。 Azure Cosmos DB 可讓您有彈性且獨立地跨任意數目的 Azure 地理區域調整輸送量和儲存體。 它利用完整的服務等級協定 (SLA) 提供了輸送量、延遲、可用性和一致性的保證。 

### <a name="set-up"></a>設定
若要在 Bot 中使用 Cosmos DB，您需要先設定一些事項，再進入程式碼。 如需建立 Cosmos DB 資料庫和應用程式的深入說明，請存取 [Cosmos DB dotnet](https://aka.ms/Bot-framework-create-dotnet-cosmosdb) 或[Cosmos DB nodejs](https://aka.ms/Bot-framework-create-nodejs-cosmosdb) 的文件。

### <a name="create-your-database-account"></a>建立資料庫帳戶

1. 在新的瀏覽器視窗中，登入 [Azure 入口網站](http://portal.azure.com)。

![建立 Cosmos DB 資料庫](./media/create-cosmosdb-database.png)

2. 按一下 [建立資源] > [資料庫] > [Azure Cosmos DB]  。

![Cosmos DB 新增帳戶頁面](./media/cosmosdb-new-account-page.png)

3. 在 [新增帳戶]  頁面上，提供 [訂用帳戶]  、[資源群組]  資訊。 為您的 [帳戶名稱]  欄位建立唯一的名稱 - 這最終會成為您的資料存取 URL 名稱的一部分。 針對 [API]  ，選取 [Core(SQL)]  ，並提供附近的 [位置]  來改善資料存取時間。
4. 然後按一下 [檢閱 + 建立]  。
5. 驗證成功後，按一下 [建立]  。

建立帳戶需要幾分鐘的時間。 等候入口網站顯示 恭喜! 已建立您的 Azure Cosmos DB 帳戶 頁面。

### <a name="add-a-collection"></a>新增集合

![新增 Cosmos DB 集合](./media/add_database_collection.png)

1. 按一下 [設定] > [新增集合]  。 [新增集合]  區域會顯示在最右邊，您可能需要向右捲動才能看到它。 由於 Cosmos DB 的近期更新，請務必新增單一分割區索引鍵： _/id_。此索引鍵可避免跨分割區查詢錯誤。

![Cosmos DB](./media/cosmos-db-sql-database.png)

2. 新資料庫集合 "bot-cosmos-sql-db" 的集合識別碼為 "bot-storage"。 我們會在下面的編碼範例中使用這些值。

![Cosmos DB 金鑰](./media/comos-db-keys.png)

3. 您資料庫設定的 [金鑰]  索引標籤中會提供端點 URI 和金鑰。 本文稍後設定您的程式碼時，需要使用這些值。 

### <a name="add-configuration-information"></a>新增組態資訊
我們要新增 Cosmos DB 儲存體的組態資料簡短又簡單，當您的 Bot 變得複雜時，您可以使用相同方法新增其他組態設定。 此範例使用上述範例中的 Cosmos DB 資料庫和集合名稱。

### <a name="ctabcsharp"></a>[C#](#tab/csharp)

**EchoBot.cs**
```csharp
public class EchoBot : ActivityHandler
{
   private const string CosmosServiceEndpoint = "<your-cosmos-db-URI>";
   private const string CosmosDBKey = "<your-cosmos-db-account-key>";
   private const string CosmosDBDatabaseName = "bot-cosmos-sql-db";
   private const string CosmosDBCollectionName = "bot-storage";
   ...
   
}
```

### <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

將下列資訊新增至 `.env` 檔案。

**.env**
```javascript
DB_SERVICE_ENDPOINT="<your-Cosmos-db-URI>"
AUTH_KEY="<your-cosmos-db-account-key>"
DATABASE="<bot-cosmos-sql-db>"
COLLECTION="<bot-storage>"
```
---

#### <a name="installing-packages"></a>安裝套件
確定您有 Cosmos DB 所需的套件

### <a name="ctabcsharp"></a>[C#](#tab/csharp)

```powershell
Install-Package Microsoft.Bot.Builder.Azure
```

### <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

您可以透過 npm，在專案中新增對 botbuilder-azure 的參考。
>**注意** - 這個 npm 套件依賴您開發電腦上現有的 Python 安裝。 如果先前尚未安裝 Python，您可以在此為電腦尋找安裝資源：[Python.org](https://www.python.org/downloads/)

```powershell
npm install --save botbuilder-azure 
```

如果尚未安裝，請從 npm 取得 dotnet 套件以存取 `.env` 檔案設定。

```powershell
npm install --save dotenv
```
---

### <a name="implementation"></a>實作 

### <a name="ctabcsharp"></a>[C#](#tab/csharp)

下列範例程式碼會使用與上面提供的[記憶體儲存體](#memory-storage)範例相同的 Bot 程式碼執行。
下列程式碼片段示範如何實作 '_myStorage_' 的 Cosmos DB 儲存體，以取代本機記憶體儲存體。 記憶體儲存體已遭到註解排除，並以對 Cosmos DB 的參考取代。

**EchoBot.cs**
```csharp

using System;
...
using Microsoft.Bot.Builder.Azure;
...
public class EchoBot : ActivityHandler
{
   // Create local Memory Storage - commented out.
   // private static readonly MemoryStorage _myStorage = new MemoryStorage();

   // Replaces Memory Storage with reference to Cosmos DB.
   private static readonly CosmosDbStorage _myStorage = new CosmosDbStorage(new CosmosDbStorageOptions
   {
      AuthKey = CosmosDBKey,
      CollectionId = CosmosDBCollectionName,
      CosmosDBEndpoint = new Uri(CosmosServiceEndpoint),
      DatabaseId = CosmosDBDatabaseName,
   });
   
   ...
}

```

### <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

下列範例程式碼類似於[記憶體儲存體](#memory-storage)，但有些許變更。

需要來自 `botbuilder-azure` 的 `CosmosDbStorage`，並將 dotenv 設定為讀取 `.env` 檔案。

**bot.js**

```javascript
const { CosmosDbStorage } = require("botbuilder-azure");
```
將記憶體儲存體註解化，並以對 Cosmos DB 的參考加以取代。

**bot.js**
```javascript
// initialized to access values in .env file.
var dotenv = require('dotenv');
dotenv.load();

// Create local Memory Storage - commented out.
// var storage = new MemoryStorage();

// Create access to CosmosDb Storage - this replaces local Memory Storage.
var storage = new CosmosDbStorage({
    serviceEndpoint: process.env.DB_SERVICE_ENDPOINT, 
    authKey: process.env.AUTH_KEY, 
    databaseId: process.env.DATABASE,
    collectionId: process.env.COLLECTION
})

```
---

## <a name="start-your-bot"></a>啟動 Bot
在本機執行您的 Bot。

## <a name="test-your-bot-with-bot-framework-emulator"></a>使用 Bot Framework Emulator 測試您的 Bot
現在啟動 Bot Framework Emulator 並連線到 Bot：

1. 按一下模擬器 [歡迎使用] 索引標籤中的 [建立新的 Bot 設定]  連結。 
2. 以您啟動 Bot 時顯示在網頁上的資訊，填妥欄位以連線到 Bot。

## <a name="interact-with-your-bot"></a>與您的 Bot 互動
傳送訊息給 Bot，Bot 就會列出所收到的訊息。
![模擬器執行中](./media/emulator-direct-storage-test.png)


### <a name="view-your-data"></a>檢視資料
在您執行 Bot 並儲存資訊之後，我們可以在 Azure 入口網站的 [資料總管]  索引標籤下檢視所儲存的資料。 

![資料總管範例](./media/data_explorer.PNG)


## <a name="using-blob-storage"></a>使用 Blob 儲存體 
Azure Blob 儲存體是 Microsoft 針對雲端推出的物件儲存體解決方案。 Blob 儲存體已針對儲存大量非結構化物件資料 (例如文字或二進位資料) 最佳化。

### <a name="create-your-blob-storage-account"></a>建立 Blob 儲存體帳戶
若要在 Bot 中使用 Blob 儲存體，您需要先設定一些事項，再進入程式碼。
1. 在新的瀏覽器視窗中，登入 [Azure 入口網站](http://portal.azure.com)。

![建立 Blob 儲存體](./media/create-blob-storage.png)

2. 按一下 [建立資源] > [儲存體] > [儲存體帳戶 - Blob、檔案、資料表、佇列]  。

![Blob 儲存體新增帳戶頁面](./media/blob-storage-new-account.png)

3. 在 [新增帳戶]  頁面中，輸入儲存體帳戶的 [名稱]  ，針對 [帳戶種類]  選取 [Blob 儲存體]  ，提供 [位置]  、[資源群組]  和 [訂用帳戶]  資訊。  
4. 然後按一下 [檢閱 + 建立]  。
5. 驗證成功後，按一下 [建立]  。

### <a name="create-blob-storage-container"></a>建立 Blob 儲存體容器
建立 Blob 儲存體帳戶之後，開啟此帳戶，做法如下： 
1. 選取資源。
2. 現在使用儲存體總管 (預覽)「開啟」

![建立 Blob 儲存體容器](./media/create-blob-container.png)

3. 以滑鼠右鍵按一下 [Blob 容器]，選取 [建立 Blob 容器]  。
4. 新增名稱。 您將使用此名稱作為 "your-blob-storage-container-name" 的值，以供存取您的 Blob 儲存體帳戶。

#### <a name="add-configuration-information"></a>新增組態資訊
尋找您為 Bot 設定 Blob 儲存體所需的 Blob 儲存體金鑰，如上所示：
1. 在 Azure 入口網站中，開啟 Blob 儲存體帳戶，然後選取 [設定] > [存取金鑰]  。

![尋找 Blob 儲存體金鑰](./media/find-blob-storage-keys.png)

我們將使用 key1 _連接字串_作為 "your-blob-storage-account-string" 值，以供存取您的 Blob 儲存體帳戶。

#### <a name="installing-packages"></a>安裝套件
如果先前未安裝使用 Cosmos DB，請安裝下列套件。

### <a name="ctabcsharp"></a>[C#](#tab/csharp)

```powershell
Install-Package Microsoft.Bot.Builder.Azure
```

### <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

透過 npm 在專案中新增對 botbuilder-azure 的參考。
>**注意** - 這個 npm 套件依賴您開發電腦上現有的 Python 安裝。 如果先前尚未安裝 Python，您可以在此為電腦尋找安裝資源：[Python.org](https://www.python.org/downloads/)

```powershell
npm install --save botbuilder-azure 
```

如果尚未安裝，請從 npm 取得 dotnet 套件以存取 `.env` 檔案設定。

```powershell
npm install --save dotenv
```
---

### <a name="implementation"></a>實作 

### <a name="ctabcsharp"></a>[C#](#tab/csharp)

**EchoBot.cs**
```csharp
using Microsoft.Bot.Builder.Azure;
```
更新可將 "myStorage"  指向現有 Blob 儲存體帳戶的程式碼行。

**EchoBot.cs**
```csharp
private static readonly AzureBlobStorage _myStorage = new AzureBlobStorage("<your-blob-storage-account-string>", "<your-blob-storage-container-name>");
```

### <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

將下列資訊新增至 `.env` 檔案。

**.env**
```javascript
BLOB_NAME="<your-blob-storage-container-name>"
BLOB_STRING="<your-blob-storage-account-string>"
```

更新 `bot.js` 檔案，如下所示。 需要來自 `botbuilder-azure` 的 `BlobStorage`

**bot.js**
```javascript
const { BlobStorage } = require("botbuilder-azure");
```

如果您並未新增程式碼來載入 `.env` 檔案，以實作 Cosmos DB 儲存體，請在此新增程式碼。

```javascript
// initialized to access values in .env file.
var dotenv = require('dotenv');
dotenv.load();
```
註解排除先前的儲存體定義並新增下列內容，即可立即更新您的程式碼，將 "_storage_" 指向您現有的 Blob 儲存體帳戶。

**bot.js**
```javascript
var storage = new BlobStorage({
    containerName: process.env.BLOB_NAME,
    storageAccountOrConnectionString: process.env.BLOB_STRING
});
```
---

將儲存體設定為指向 Blob 儲存體帳戶後，Bot 程式碼現在會儲存和擷取 Blob 儲存體中的資料。

## <a name="start-your-bot"></a>啟動 Bot
在本機執行您的 Bot。

## <a name="start-the-emulator-and-connect-your-bot"></a>啟動模擬器並且連線至您的 Bot
接下來，請啟動模擬器，然後在模擬器中連線至您的 Bot：

1. 按一下模擬器 [歡迎使用] 索引標籤中的 [建立新的 Bot 設定]  連結。 
2. 以您啟動 Bot 時顯示在網頁上的資訊，填妥欄位以連線到 Bot。

## <a name="interact-with-your-bot"></a>與您的 Bot 互動
傳送訊息給 Bot，Bot 就會列出所收到的訊息。

![模擬器測試儲存體 Bot](./media/emulator-direct-storage-test.png)

### <a name="view-your-data"></a>檢視資料
在您執行 Bot 並儲存您的資訊之後，我們可以在 Azure 入口網站的 [儲存體總管]  索引標籤下檢視。

## <a name="blob-transcript-storage"></a>Blob 文字記錄儲存體
Azure Blob 文字記錄儲存體會提供一個特製化儲存體選項，讓您輕鬆地以記錄的文字記錄形式儲存和擷取使用者對話。 Azure Blob 文字記錄儲存體特別適合用於自動擷取使用者輸入，以便在對 Bot 效能進行偵錯時檢查。

### <a name="set-up"></a>設定
Azure Blob 文字記錄儲存體可以使用遵循上面「建立 Blob 儲存體帳戶」  和「新增組態資訊」  小節中詳述的步驟所建立的相同 Blob 儲存體帳戶。 我們現在會新增容器來保留我們的文字記錄

![建立文字記錄容器](./media/create-blob-transcript-container.png)

1. 開啟 Azure Blob 儲存體帳戶。
1. 按一下 [儲存體總管]  。
1. 以滑鼠右鍵按一下 [Blob 容器]  ，然後選取 [建立 Blob 容器]  。
1. 輸入文字記錄容器的名稱，然後選取 [確定]  。 (我們輸入了 mybottranscripts)

### <a name="implementation"></a>實作 
下列程式碼會將文字記錄儲存體指標 `_myTranscripts` 連到新的 Azure Blob 文字記錄儲存體帳戶。 若要使用新的容器名稱 <your-blob-transcript-container-name> 建立此連結，請在 Blob 儲存體內建立新容器來保存文字記錄檔。

**echoBot.cs**
```csharp
using Microsoft.Bot.Builder.Azure;

public class EchoBot : ActivityHandler
{
   ...
   
   private readonly AzureBlobTranscriptStore _myTranscripts = new AzureBlobTranscriptStore("<your-blob-transcript-storage-account-string>", "<your-blob-transcript-container-name>");
   
   ...
}

```

### <a name="store-user-conversations-in-azure-blob-transcripts"></a>在 Azure blob 文字記錄中儲存使用者對話
在 Blob 容器可用於儲存文字記錄之後，您可以開始保留使用者與 Bot 的對話。 這些交談稍後可當作偵錯工具，查看使用者與 Bot 的互動方式。 每個模擬器 [重新開始交談]  會啟始建立新的文字記錄交談清單。 下列程式碼會在預存文字記錄檔內保留使用者交談輸入。
- 使用 `LogActivityAsync` 可儲存目前的文字記錄。
- 使用 `ListTranscriptsAsync` 可擷取已儲存的文字記錄。
在此範例程式碼中，每個預存文字記錄的識別碼會儲存到名為 "storedTranscripts" 的清單中。 這份清單稍後用來管理我們保留的預存 Blob 文字記錄數目。

**echoBot.cs**
```csharp

protected override async Task OnMessageActivityAsync(ITurnContext<IMessageActivity> turnContext, CancellationToken cancellationToken)
{
    await _myTranscripts.LogActivityAsync(turnContext.Activity);

    List<string> storedTranscripts = new List<string>();
    PagedResult<Microsoft.Bot.Builder.TranscriptInfo> pagedResult = null;
    var pageSize = 0;
    do
    {
       pagedResult = await _myTranscripts.ListTranscriptsAsync("emulator", pagedResult?.ContinuationToken);
       pageSize = pagedResult.Items.Count();

       // transcript item contains ChannelId, Created, Id.
       // save the channelIds found by "ListTranscriptsAsync" to a local list.
       foreach (var item in pagedResult.Items)
       {
          storedTranscripts.Add(item.Id);
       }
    } while (pagedResult.ContinuationToken != null);
    
    ...
}

```

### <a name="manage-stored-blob-transcripts"></a>管理預存 Blob 文字記錄
雖然預存文字記錄可以作為偵錯工具，但經過一段時間，預存文字記錄可能成長到大於您可保留的數目。 以下包含的其他程式碼會使用 `DeleteTranscriptAsync`，從 Blob 文字記錄存放區中移除所有文字記錄 (但最後三個擷取的文字記錄項目除外)。

**echoBot.cs**
```csharp

protected override async Task OnMessageActivityAsync(ITurnContext<IMessageActivity> turnContext, CancellationToken cancellationToken)
{
    await _myTranscripts.LogActivityAsync(turnContext.Activity);

    List<string> storedTranscripts = new List<string>();
    PagedResult<Microsoft.Bot.Builder.TranscriptInfo> pagedResult = null;
    var pageSize = 0;
    do
    {
       pagedResult = await _myTranscripts.ListTranscriptsAsync("emulator", pagedResult?.ContinuationToken);
       pageSize = pagedResult.Items.Count();

       // transcript item contains ChannelId, Created, Id.
       // save the channelIds found by "ListTranscriptsAsync" to a local list.
       foreach (var item in pagedResult.Items)
       {
          storedTranscripts.Add(item.Id);
       }
    } while (pagedResult.ContinuationToken != null);
    
    // Manage the size of your transcript storage.
    for (int i = 0; i < pageSize; i++)
    {
       // Remove older stored transcripts, save just the last three.
       if (i < pageSize - 3)
       {
          string thisTranscriptId = storedTranscripts[i];
          try
          {
             await _myTranscripts.DeleteTranscriptAsync("emulator", thisTranscriptId);
           }
           catch (System.Exception ex)
           {
              await turnContext.SendActivityAsync("Debug Out: DeleteTranscriptAsync had a problem!");
              await turnContext.SendActivityAsync("exception: " + ex.Message);
           }
       }
    }
    ...
}

```

以下連結提供關於 [Azure Blob 文字記錄儲存體](https://docs.microsoft.com/dotnet/api/microsoft.bot.builder.azure.azureblobtranscriptstore)的詳細資訊 

## <a name="additional-information"></a>其他資訊

### <a name="manage-concurrency-using-etags"></a>使用 eTag 管理並行
在 Bot 程式碼範例中，我們會將每個 `IStoreItem` 的 `eTag` 屬性設定為 `*`。 存放區物件的 `eTag` (實體標記) 成員在 Cosmos DB 中用來管理並行作業。 如果另一個 Bot 執行個體變更了 Bot 所寫入的同一個儲存體中的物件，`eTag` 會告知資料庫該採取什麼動作。 

<!-- define optimistic concurrency -->

#### <a name="last-write-wins---allow-overwrites"></a>最後寫入者優先 - 允許覆寫
星號 (`*`) 的 `eTag` 屬性值指出最後寫入者優先。 建立新的資料存放區時，您可以將屬性的 `eTag` 設定為 `*`，以指出您之前沒有儲存過正在寫入的資料或您想要最後寫入的資料覆寫之前已儲存的任何屬性。 如果您的 Bot 沒有並行的問題，針對正在寫入的任何資料，將其 `eTag` 屬性設定為 `*`，以允許覆寫。

#### <a name="maintain-concurrency-and-prevent-overwrites"></a>維護並行和防止覆寫
將資料儲存到 Cosmos DB 時，如果您想要防止並行存取屬性，以及避免覆寫另一個 Bot 執行個體所做的變更，請對 `eTag` 使用 `*` 以外的值。 當 Bot 嘗試儲存狀態資料且 `eTag` 與儲存體中的 `eTag` 值不同時，Bot 會收到含有 `etag conflict key=` 訊息的錯誤回應。 <!-- To control concurrency of data that is stored using `IStorage`, the BotBuilder SDK checks the entity tag (ETag) for `Storage.Write()` requests. -->

根據預設，每次 Bot 寫入到儲存體物件時，Cosmos DB 存放區會檢查其 `eTag` 屬性是否相同，然後在每次寫入之後將該屬性更新為新的唯一值。 如果寫入的 `eTag` 屬性與儲存體中的 `eTag` 不相符，這表示另一個 Bot 或執行緒已變更資料。 

例如，假設您想要 Bot 編輯已儲存的附註，但不想覆寫另一個 Bot 執行個體所作的變更。 如果另一個 Bot 執行個體已進行編輯，則您想要 使用者編輯最新更新的版本。

### <a name="ctabcsharp"></a>[C#](#tab/csharp)

首先，建立會實作 `IStoreItem` 的類別。

**EchoBot.cs**
```csharp
public class Note : IStoreItem
{
    public string Name { get; set; }
    public string Contents { get; set; }
    public string ETag { get; set; }
}
```

接下來，建立儲存體物件來建立初始附註，並將物件新增到存放區。

**EchoBot.cs**
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

**EchoBot.cs**
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


### <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

將協助程式函式新增至您的 Bot 結尾，可將範例附註寫入資料存放區。
首先建立 `myNoteData` 物件。

**bot.js**
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

在 `createSampleNote` 協助程式函式內初始化 `changes` 物件，並在其中新增「附註」  ，然後將其寫入儲存體。

**bot.js**
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
     var list = changes["Note"].contents;
     await context.sendActivity(`Successful created a note: ${list}`);
} catch (err) {
     await context.sendActivity(`Could not create note: ${err}`);
}
```

新增下列呼叫，即可從您 Bot 的邏輯存取協助程式函式：

**bot.js**
```javascript
// Save a note with etag.
await createSampleNote(storage, turnContext);
```

建立後，為了稍後擷取及更新附註，我們會建立另一個可以在使用者輸入「更新附註」時存取的協助程式函式。

**bot.js**
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
             var list = note["Note"].contents;
             await context.sendActivity(`Successfully updated note: ${list}`);
        } catch (err) {            
             console.log(`Write failed: ${err}`);
        }
    }
    catch (err) {
        await context.sendActivity(`Unable to read the Note: ${err}`);
    }
}
```

新增下列呼叫，即可從您 Bot 的邏輯內存取此協助程式函式：

**bot.js**
```javascript
// Update a note with etag.
await updateSampleNote(storage, turnContext);
```

如果在您嘗試寫回變更之前，另一個使用者已更新存放區中的附註，`eTag` 值不再相符且對 `write` 的呼叫將會擲回例外狀況。

---

若要維護並行，一律從儲存體讀取屬性，然後修改所讀取的屬性，如此即可維護 `eTag`。 如果您從存放區讀取使用者資料，回應將會包含 eTag 屬性。 如果您變更資料並將更新的資料寫入存放區，則您的要求應該包含 eTag 屬性且它必須指定與您之前讀取相同的值。 不過，寫入其 `eTag` 設定為 `*` 的物件，將允許寫入可覆寫任何其他變更。

## <a name="next-steps"></a>後續步驟
既然您已經知道如何直接從儲存體讀取和寫入，現在我們來看看如何使用狀態管理員來為您執行這些動作。

> [!div class="nextstepaction"]
> [使用交談和使用者屬性儲存狀態](bot-builder-howto-v4-state.md)

