---
title: 使用 Cosmos DB 建立中介軟體 | Microsoft Docs
description: 了解如何建立記錄到 Cosmos DB 的自訂中介軟體。
keywords: 中介軟體, cosmos db, 資料庫, azure, onTurn
author: ivorb
ms.author: v-ivorb
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 03/21/2018
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 29edff2db0e513a37b8ca8ac0f97e0647282567d
ms.sourcegitcommit: f576981342fb3361216675815714e24281e20ddf
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/18/2018
ms.locfileid: "39299351"
---
# <a name="create-middleware-that-logs-in-cosmos-db"></a>建立在 Cosmos DB 中記錄的中介軟體

雖然 SDK 提供實用的中介軟體，仍然有些情況下，您需要自行實作中介軟體，以達到所需的目標。

在此範例中，我們將建立中介軟體層，以便連線到 Cosmos DB 記錄所有收到的訊息和我們傳回的回覆。 您在這裡看到的程式碼，會提供作為完整原始程式碼，與我們的[範例](../dotnet/bot-builder-dotnet-samples.md)一起提供。

## <a name="set-up"></a>設定

我們需要先設定一些事，再進入程式碼。

### <a name="create-your-database"></a>建立資料庫

首先，我們需要有地方來儲存我們的資料庫。  這可以透過 Azure 帳戶完成，或是基於測試目的，您可以使用 [Cosmos DB 模擬器](https://docs.microsoft.com/en-us/azure/cosmos-db/local-emulator)。 無論您選擇何種方法，我們需要資料庫的端點 URI 和索引鍵。

範例，以及此處提供的程式碼，會使用模擬器。 端點和索引鍵是針對 Cosmos DB 測試帳戶所提供，這些是模擬器文件所提供的常見值，無法用於生產環境。

如果您想要使用實際的 Azure Cosmos DB，請遵循 [Cosmos DB 入門](https://docs.microsoft.com/en-us/azure/cosmos-db/sql-api-get-started)主題的步驟 1。 完成之後，您的端點 URI 和索引鍵都會提供於資料庫設定的 [索引鍵] 索引標籤。 底下的設定檔將會需要這些值。

### <a name="build-a-basic-bot"></a>建置基本 Bot

本主題的其餘部分以基本 Bot 為基礎，例如概觀所建置的 HelloBot。 您可以使用 [Bot Framework 模擬器](https://github.com/Microsoft/BotFramework-Emulator)連線至 Bot、與其對話，以及測試您的 Bot。

除了使用 Bot Framework 的標準 Bot 所需的參考，您也必須新增下列 NuGet 套件的參考：

* Microsoft.Azure.Documentdb.Core
* System.Configuration.ConfigurationManager

這些程式庫分別用來與資料庫通訊，以及啟用設定檔的使用。

### <a name="add-configuration-file"></a>新增組態檔

使用設定檔是良好做法的原因有好幾個，因此我們將在此使用設定檔。 我們的設定資料簡短又簡單；不過，當您的 Bot 變複雜時，可以新增其他組態設定。

# <a name="ctabcs"></a>[C#](#tab/cs)
1. 將檔案新增至名為 `app.config` 的專案。
2. 在其中儲存下列資訊。 已針對本機模擬器定義端點和索引鍵，但可以為您的 Cosmos DB 執行個體進行更新。
    ```
    <?xml version="1.0" encoding="utf-8"?>
    <configuration>
        <appSettings>
            <add key="EmulatorDbUrl" value="https://localhost:8081"/>
            <add key="EmulatorDbKey" value="C2y6yDjf5/R+ob0N8A7Cgv30VRDJIWEHLM+4QDU5DE2nQ9nDuVTqobD4b8mGGyPMbIZnqyMsEcaGQy67XIw/Jw=="/>
        </appSettings>
    </configuration>
    ```
# <a name="javascripttabjs"></a>[JavaScript](#tab/js)
1. 將檔案新增至名為 `config.js` 的專案。
2. 在其中儲存下列資訊。 已針對本機模擬器定義端點和索引鍵，但可以為您的 Cosmos DB 執行個體進行更新。
    ```javascript
        var config = {}

        config.EmulatorServiceEndpoint = "https://localhost:8081";
        config.EmulatorAuthKey = "C2y6yDjf5/R+ob0N8A7Cgv30VRDJIWE+4QDU5DE2nQ9nDuVTqobD4b8mGGyPMbIZnqyMsEcaGQy67XIw/Jw==";
        config.database = {
            "id": "Tasks"
        };
        config.collection = {
            "id": "Items"
        };

        module.exports = config;
    ```

---

如果您使用實際的 Cosmos DB，您可以將上述兩個索引鍵的值取代為 Cosmos DB 設定中的值。 或者，您可以再新增兩個索引鍵到您的設定，如此便可在測試用的模擬器與實際 Cosmos DB 之間切換，例如：

# <a name="ctabcs"></a>[C#](#tab/cs)
```
    <add key="ActualDbUrl" value="<your database URI>"/>
    <add key="ActualDbKey" value="<your database key>"/>
```
 如果您新增兩個額外的索引鍵，並想要使用實際資料庫，請務必在下列建構函式中指定正確的索引鍵，而非 `EmulatorDbUrl` 和 `EmulatorDbKey`。

# <a name="javascripttabjs"></a>[JavaScript](#tab/js)
```
    config.ActualServiceEndpoint = "your database URI;
    config.ActualAuthKey = "your database key";
```

如果您新增兩個額外的索引鍵，並想要使用實際資料庫，請務必在下列建構函式中指定正確的索引鍵，而非 `EmulatorServiceEndpoint` 和 `EmulatorAuthKey`。

---



## <a name="creating-your-middleware"></a>建立中介軟體

# <a name="ctabcs"></a>[C#](#tab/cs)
在我們開始撰寫實際的中介軟體邏輯之前，請在 Bot 專案中為中介軟體建立新類別。 有了該類別之後，請將命名空間變更為我們用於其餘 Bot 的命名空間 `Microsoft.Bot.Samples`。 在這裡，您會看到我們已新增一些新程式庫的參考：

* `Newtonsoft.Json;`
* `Microsoft.Azure.Documents;`
* `Microsoft.Azure.Documents.Client;`
* `Microsoft.Azure.Documents.Linq;`

各種 `Microsoft.Azure.Documents.*` 是針對與資料庫通訊的不同部分，而 `Newtonsoft.Json` 可幫助我們剖析來回傳送至資料庫的 JSON。

最後，每個中介軟體都會實作 `IMiddleware`，需要我們定義適當的處理常式，中介軟體才能正常運作。

```cs
using System;
using System.Collections.Generic;
using System.Threading.Tasks;
using System.Configuration;
using Newtonsoft.Json;
using Microsoft.Bot.Builder;
using Microsoft.Bot.Schema;
using Microsoft.Azure.Documents;
using Microsoft.Azure.Documents.Client;
using Microsoft.Azure.Documents.Linq;

namespace Microsoft.Bot.Samples
{
    public class CosmosMiddleware : IMiddleware
    {
        ...
```

# <a name="javascripttabjs"></a>[JavaScript](#tab/js)
在我們開始撰寫實際的中介軟體邏輯之前，我們需要建立自訂中介軟體，它將以 `onTurn()` 開始。 

```javascript
adapter.use({onTurn: async (context, next) =>{
    // Middleware logic here...
}})
    
```

---

#### <a name="defining-local-variables"></a>定義本機變數

# <a name="ctabcs"></a>[C#](#tab/cs)
接下來，我們需要本機變數以便操作資料庫，也需要一個類別來儲存我們想要記錄的資訊。 該資訊類別稱為 `Log`，它會定義與每個成員建立關聯的 JSON 屬性將如何命名。 我們之後會回到該主題。

```cs
    private string Endpoint;
    private string Key;
    private static readonly string Database = "BotData";
    private static readonly string Collection = "BotCollection";
    public DocumentClient docClient;

    public class Log
    {
        [JsonProperty("Time")]
        public string Time;

        [JsonProperty("MessageReceived")]
        public string Message;

        [JsonProperty("ReplySent")]
        public string Reply;
    }
```

# <a name="javascripttabjs"></a>[JavaScript](#tab/js)
接下來，我們需要一個本機變數來儲存我們想要記錄的資訊。 在中介軟體中，我們必須存取 conversationState，它以 Cosmos DB 作為存放裝置提供者。 變數 `info` 將是我們讀取和寫入的狀態物件。 

```javascript
// Add conversation state middleware
const conversationState = new ConversationState(storage);
adapter.use(conversationState);

adapter.use({onTurn: async (context, next) =>{

    const info = conversationState.get(context);
    // More middleware logic 
}})
```

--- 


#### <a name="initialize-database-connection"></a>初始化資料庫連線

# <a name="ctabcs"></a>[C#](#tab/cs)
此中介軟體的建構函式，會負責從以上定義之設定檔提取出所需的資訊，也負責建立 `DocumentClient`，讓我們從資料庫讀取和寫入。 我們也會確定資料庫和集合已經設定。

新 `DocumentClient` 的適當使用，會要求處置它，所以我們解構函式會執行此作業。

建立資料庫時，需要嘗試進行第一個資料庫的基本讀取，然後是該資料庫內的集合。 如果我們遇到 `NotFound` 例外狀況，我們會加以攔截，並建立資料庫或集合，而不是將它透過堆疊往上擲回。 在大部分情況下，這些已經建立好，但在這個範例中，我們不必擔心第一次執行之前還得建立該資料庫。 在範例程式碼中，資料庫和集合分成兩個函式以求簡化，但它們在下列程式碼片段中已經合併。

如需 Cosmos DB 的詳細資訊，請參閱其[網站](https://docs.microsoft.com/en-us/azure/cosmos-db/)。 針對本主題的其餘部分，我們將簡單帶過 Cosmos DB 本身的細節，並專注於您的 Bot 使用它時的必要條件。

`Key`、`Endpoint` 和 `docClient` 的定義已包含在上述程式碼片段，並且只是為了清楚起見而複製到這裡。

```cs
    private string Endpoint;
    private string Key;
    // ...
    public DocumentClient docClient;
    // ...

    public CosmosMiddleware()
    {
        Endpoint = ConfigurationManager.AppSettings["EmulatorDbUrl"];
        Key = ConfigurationManager.AppSettings["EmulatorDbKey"];
        docClient = new DocumentClient(new Uri(Endpoint), Key);
        CreateDatabaseAndCollection().ConfigureAwait(false);
    }

    ~CosmosMiddleware()
    {
        docClient.Dispose();
    }

    private async Task CreateDatabaseAndCollection()
    {
        try
        {
            await docClient.ReadDatabaseAsync(UriFactory.CreateDatabaseUri(Database));
        }
        catch (DocumentClientException e)
        {
            if (e.StatusCode == System.Net.HttpStatusCode.NotFound)
            {
                await docClient.CreateDatabaseAsync(new Database { Id = Database });
            }
            else
            {
                throw;
            }
        }

        try
        {
            await docClient.ReadDocumentCollectionAsync(UriFactory.CreateDocumentCollectionUri
                (Database, Collection));
        }
        catch (DocumentClientException e)
        {
            if (e.StatusCode == System.Net.HttpStatusCode.NotFound)
            {
                await docClient.CreateDocumentCollectionAsync(
                    UriFactory.CreateDatabaseUri(Database),
                    new DocumentCollection { Id = Collection });
            }
            else
            {
                throw;
            }
        }
    }
```

# <a name="javascripttabjs"></a>[JavaScript](#tab/js)
建立資料庫時，需要嘗試進行第一個資料庫的基本讀取，然後是該資料庫內的集合。 如果我們遇到 `undefined`，我們會加以攔截，並建立一個稱為 `log` 的物件，它將會設定為空陣列。 在大部分情況下，`log` 已經建立好，但在這個範例中，我們不必擔心第一次執行之前還得建立該資料庫。 在範例程式碼中，我們檢查看看是否未定義 `info.log`。 如果是，便將它設定為空陣列。

```javascript
adapter.use({onTurn: async (context, next) =>{

    const info = conversationState.get(context);

    if(info.log == undefined){
        info.log = [];
    }

    // More bot logic below
}})
```
---

#### <a name="read-from-database-logic"></a>從資料庫邏輯讀取

最後一個協助程式函式，會從資料庫讀取，並傳回指定數目的最新記錄。 值得注意的是，有比我們此處所用更好的資料庫做法可以擷取資料，特別是當您的資料存放區規模大上許多時。

# <a name="ctabcs"></a>[C#](#tab/cs)
```cs
    public async Task<string> ReadFromDatabase(int numberOfRecords)
    {
        var documents = docClient.CreateDocumentQuery<Log>(
                UriFactory.CreateDocumentCollectionUri(Database, Collection))
                .AsDocumentQuery();
        List<Log> messages = new List<Log>();
        while (documents.HasMoreResults)
        {
            messages.AddRange(await documents.ExecuteNextAsync<Log>());
        }

        // Create a sublist of messages containing the number of requested records.
        List<Log> messageSublist = messages.GetRange(messages.Count - numberOfRecords, numberOfRecords);

        string history = "";

        // Send the last 3 messages.
        foreach (Log logEntry in messageSublist)
        {
            history += ("Message was: " + logEntry.Message + " Reply was: " + logEntry.Reply + "  ");
        }

        return history;
    }
```

# <a name="javascripttabjs"></a>[JavaScript](#tab/js)

下列程式碼仍在 `onTurn()` 函式內，且如果使用者輸入字串 "history" 時便會呼叫它。

```javascript
adapter.use({onTurn: async (context, next) =>{

    const utterance = (context.activity.text || '').trim().toLowerCase();
    const isMessage = context.activity.type === 'message';
    const info = conversationState.get(context);

    if(info.log == undefined){
        info.log = [];
    }

    if(isMessage && utterance == 'history'){
        // Loop through the info.log array
        var logHistory = "";
        for(var i = 0; i < info.log.length; i++){
            logHistory += info.log[i] + " ";
        }
        await context.sendActivity(logHistory);
        // Short circuit the middleware
        return
    }
    //...
}})

```

---

#### <a name="define-onturn"></a>定義 OnTurn()

然後標準的中介軟體方法 `OnTurn()` 會處理其餘的工作。 我們只想在目前活動是訊息的時候才記錄，在呼叫 `next()` 之前和之後都會檢查這點。

我們檢查的第一件事，就是這是否為我們的特殊案例訊息，亦即使用者要求最新的歷程記錄。 如果是，便呼叫讀取我們的資料庫，以取得最新的記錄，並傳送至對話。 由於那樣可完全處理目前的活動，所以我們縮短了管線，而不是繼續執行。

為了取得 Bot 傳送的回應，每當收到訊息時我們便建立處理常式來抓取那些回應，這可以在我們提供給 `OnSendActivity()` 的 Lambda 看到。 它會建置字串，以收集透過此內容物件之 `SendActivity()` 傳送的所有訊息。

一旦執行作業 `next()` 返回管道，便將我們的記錄資料組合，並寫出至資料庫。 

在一些訊息傳送到此 Bot 之後查看 Cosmos DB 資料總管，您應該會在個別記錄中看到資料。 在檢查這些記錄的其中一個時，您應該會看到前三個項目是我們記錄資料的三個值，名稱分別為各自 `JsonProperty` 的指定字串。

# <a name="ctabcs"></a>[C#](#tab/cs)
```cs
    public async Task OnTurn
        (ITurnContext context, MiddlewareSet.NextDelegate next)
    {
        string botReply = "";

        if (context.Activity.Type == ActivityTypes.Message)
        {
            if (context.Activity.Text == "history")
            {
                // Read last 3 responses from the database, and short circuit future execution.
                await context.SendActivity(await ReadFromDatabase(3));
                return;
            }

            // Create a send activity handler to grab all response activities 
            // from the activity list.
            context.OnSendActivity(async (activityContext, activityList, activityNext) =>
            {
                foreach (Activity activity in activityList)
                {
                    botReply += (activity.Text + " ");
                }
                await activityNext();
            });
        }

        // Pass execution on to the next layer in the pipeline.
        await next();

        // Save logs for each conversational exchange only.
        if (context.Activity.Type == ActivityTypes.Message)
        {
            // Build a log object to write to the database.
            var logData = new Log
            {
                Time = DateTime.Now.ToString(),
                Message = context.Activity.Text,
                Reply = botReply
            };

            // Write our log to the database.
            try
            {
                var document = await docClient.CreateDocumentAsync(UriFactory.
                    CreateDocumentCollectionUri(Database, Collection), logData);
            }
            catch (Exception ex)
            {
                // More logic for what to do on a failed write can be added here
                throw ex;
            }
        }
    }
```

# <a name="javascripttabjs"></a>[JavaScript](#tab/js)

以我們上述的 `onTurn` 函式為基礎建置。

```javascript
adapter.use({onTurn: async (context, next) =>{

    const utterance = (context.activity.text || '').trim().toLowerCase();
    const isMessage = context.activity.type === 'message';
    const info = conversationState.get(context);

    if(info.log == undefined){
        info.log = [];
    }

    if(isMessage && utterance == 'history'){
        // Loop through info.log 
        var logHistory = "";
        for(var i = 0; i < info.log.length; i++){
            logHistory += info.log[i] + " ";
        }
        await context.sendActivity(logHistory);
        // Short circuit the middleware
        return
    } else if(isMessage){
        // Store the users response
        info.log.push(`Message was: ${context.activity.text}`); 

        await context.onSendActivities(async (handlerContext, activities, handlerNext) => 
        {
            // Store the bot's reply
            info.log.push(`Reply was: ${activities[0].text}`);
            
            await handlerNext(); 
        });
        await next();
    }
}})

```

---

## <a name="sample-output"></a>範例輸出

在完整的範例中，Bot 會使用提示程式庫，要求使用者的名稱及最愛的號碼。 提示程式庫的詳細資訊位於[提示使用者](~/v4sdk/bot-builder-prompts.md)主題。

以下是我們範例 Bot 執行上述程式碼的範例對話。

![Cosmos 中介軟體範例對話](./media/cosmos-middleware-conversation.png)

## <a name="additional-resources"></a>其他資源

* [Cosmos DB](https://docs.microsoft.com/en-us/azure/cosmos-db/)
* [Cosmos DB 模擬器](https://docs.microsoft.com/en-us/azure/cosmos-db/local-emulator)

