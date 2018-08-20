---
title: 使用對話和使用者屬性儲存狀態 | Microsoft Docs
description: 了解如何使用適用於 .NET 的 Bot 建立器 SDK V4 儲存和擷取資料。
keywords: 對話狀態, 使用者狀態, 狀態中介軟體, 對話流程, 檔案儲存體, Azure 資料表儲存體
author: ivorb
ms.author: v-demak
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 05/03/18
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 16df371b1cabb4b3eb47d1f491a5d45e26627d38
ms.sourcegitcommit: dcbc8ad992a3e242a11ebcdf0ee99714d919a877
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/30/2018
ms.locfileid: "39352847"
---
# <a name="save-state-using-conversation-and-user-properties"></a>使用對話和使用者屬性儲存狀態

[!INCLUDE [pre-release-label](../includes/pre-release-label.md)]

若要讓 Bot 儲存對話與使用者狀態，請先初始化狀態管理員中介軟體，然後使用對話和使用者狀態屬性。
如需該狀態使用方式的詳細資訊，請參閱[狀態和儲存體](./bot-builder-storage-concept.md)。

## <a name="initialize-state-manager-middleware"></a>初始化狀態管理員中介軟體

在 SDK 中，您需要先初始化 Bot 配接器以使用狀態管理員中介軟體，然後才能使用對話或使用者屬性存放區。 「對話狀態」會用於對話屬性，「使用者狀態」則會用於使用者屬性。 (您可以跨多個對話存取使用者狀態屬性)。狀態管理員中介軟體會提供抽象層，讓您使用簡單的索引鍵/值或物件存放區存取屬性，而不受基礎儲存體類型所影響。 狀態管理員會負責將資料寫入至儲存體和管理並行執行，不論基礎儲存體類型是記憶體內部、檔案儲存體還是 Azure 資料表儲存體。


# <a name="ctabcsharp"></a>[C#](#tab/csharp)
若要查看 `ConversationState` 的初始化方式，請參閱 Microsoft.Bot.Samples.EchoBot-AspNetCore 範例中的 `Startup.cs`。

```csharp
services.AddBot<EchoBot>(options =>
{
    options.CredentialProvider = new ConfigurationCredentialProvider(Configuration);

    IStorage dataStore = new MemoryStorage();
    options.Middleware.Add(new ConversationState<EchoState>(dataStore));
});
```

在 `options.Middleware.Add(new ConversationState<EchoState>(dataStore));` 這一行中，`ConversationState` 是會新增至 Bot 作為中介軟體的對話狀態管理員物件。 `EchoState` 類型參數是表示您要如何儲存對話狀態資訊的類型。 Bot 可使用任何類別類型的對話或使用者狀態資料。

`EchoState` 的實作是在 `EchoBot.cs` 中：
```csharp
public class EchoState
{
    public int TurnNumber { get; set; }
}
``` 

# <a name="javascripttabjs"></a>[JavaScript](#tab/js)

定義儲存體提供者，並將它指派給您要使用的狀態管理員。
您可以對 `ConversationState` 和 `UserState` 管理中介軟體使用相同的儲存體提供者。
然後，您會使用 `BotStateSet` 程式庫將它連線至中介軟體層，由這一層為您管理資料保存。

```javascript
const { BotFrameworkAdapter, MemoryStorage, ConversationState, UserState, BotStateSet } = require('botbuilder');
const restify = require('restify');

// Create adapter
const adapter = new BotFrameworkAdapter({ 
    appId: process.env.MICROSOFT_APP_ID, 
    appPassword: process.env.MICROSOFT_APP_PASSWORD 
});

// Add conversation state middleware.
const conversationState = new ConversationState(new MemoryStorage());
adapter.use(conversationState);

// Alternatively, use both conversation and user state middleware.
// const storage = new MemoryStorage;
// const conversationState = new ConverstationState(storage);
// const userState = new UserState(storage);
// adapter.use(new BotStateSet(conversationState, userState));
```    
---

> [!NOTE] 
> 記憶體內部資料儲存體僅供測試之用。 這是會變動的暫存儲存體。 每次 Bot 重新啟動時，就會清除資料。 請參閱本文稍後的[檔案儲存體](#file-storage)和 [Azure 資料表儲存體](#azure-table-storage)，以設定對話狀態和使用者狀態的其他基礎儲存體媒體。 

### <a name="configuring-state-manager-middleware"></a>設定狀態管理員中介軟體

在初始化狀態中介軟體時，選擇性的「狀態設定」參數可讓您變更屬性儲存方式的預設行為。 預設設定值為：

* 將屬性保存超過回合內容的存留期。
* 如果多個 Bot 執行個體寫入至屬性，則允許 Bot 的最後一個執行個體覆寫前一個。

## <a name="use-conversation-and-user-state-properties"></a>使用對話和使用者狀態屬性 
<!-- middleware and message context properties -->

狀態管理員中介軟體設定完成後，您就可以從內容物件取得對話狀態和使用者狀態屬性。
<!-- Changes are written to storage before the `SendActivity()` pipeline completes. -->

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

您可以使用 Bot 建立器 SDK 中的 `Microsoft.Bot.Samples.EchoBot` 範例，來了解其運作方式。 

在 `OnTurn` 處理常式中，`context.GetConversationState` 會取得對話狀態以存取您所定義的資料，您可以使用屬性修改狀態 (在此案例中為遞增 `TurnNumber`)。

```csharp
public async Task OnTurn(ITurnContext context)
{
    // This bot is only handling Messages
    if (context.Activity.Type == ActivityTypes.Message)
    {
        // Get the conversation state from the turn context
        var state = context.GetConversationState<EchoState>();

        // Bump the turn count. 
        state.TurnCount++;

        // Echo back to the user whatever they typed.
        await context.SendActivity($"Turn {state.TurnCount}: You sent '{context.Activity.Text}'");
    }
}
```   

# <a name="javascripttabjs"></a>[JavaScript](#tab/js)

您可以使用 Yeoman 產生器範例所產生的 EchoBot，來了解其運作方式。

此程式碼範例會說明如何將回合計數器儲存至對話狀態。

```javascript
const { BotFrameworkAdapter, MemoryStorage, ConversationState } = require('botbuilder');
const restify = require('restify');

// Create server
let server = restify.createServer();
server.listen(process.env.port || process.env.PORT || 3978, function () {
    console.log(`${server.name} listening to ${server.url}`);
});

// Create adapter
const adapter = new BotFrameworkAdapter({ 
    appId: process.env.MICROSOFT_APP_ID, 
    appPassword: process.env.MICROSOFT_APP_PASSWORD 
});

// Add conversation state middleware
const conversationState = new ConversationState(new MemoryStorage());
adapter.use(conversationState);

// Listen for incoming requests 
server.post('/api/messages', (req, res) => {
    // Route received request to adapter for processing
    adapter.processActivity(req, res, async (context) => {
        if (context.activity.type === 'message') {
            const state = conversationState.get(context);
            const count = state.count === undefined ? state.count = 0 : ++state.count;
            await context.sendActivity(`${count}: You said "${context.activity.text}"`);
        } else {
            await context.sendActivity(`[${context.activity.type} event detected]`);
        }
    });
});
```
---

## <a name="using-conversation-state-to-direct-conversation-flow"></a>使用對話狀態進行對話流程導向

在設計對話流程時，定義狀態旗標來進行對話流程導向是很實用的做法。 旗標可以是簡單的**布林值**類型，也可以是包含目前主題名稱的類型。 旗標可協助您追蹤您在對話中的位置。 例如，**布林值**類型的旗標可以讓您知道您是否位於對話中。 主題名稱屬性則可讓您知道您目前位於哪個對話。

下列範例會使用布林值「have asked name」屬性，來將 Bot 向使用者詢問名稱的時間加上旗標。 Bot 在收到下一個訊息時，就會檢查該屬性。 如果該屬性設定為 `true`，Bot 就知道系統剛剛向使用者詢問其名稱，並將內送訊息解譯為名稱，以儲存為使用者屬性。


# <a name="ctabcsharp"></a>[C#](#tab/csharp)
```csharp
public class ConversationInfo
{
    public bool haveAskedNameFlag { get; set; }
    public bool haveAskedNumberFlag { get; set; }
}

public class UserInfo
{
    public string name { get; set; }
    public string telephoneNumber { get; set; }
    public bool done { get; set; }
}

public async Task OnTurn(ITurnContext context)
{
    // Get state objects. Default objects are created if they don't already exist.
    var convo = ConversationState<ConversationInfo>.Get(context);
    var user = UserState<UserInfo>.Get(context);

    if (context.Activity.Type is ActivityTypes.Message)
    {
        if (string.IsNullOrEmpty(user.name) && !convo.haveAskedNameFlag)
        {
            // Ask for the name.
            await context.SendActivity("Hello. What's your name?");

            // Set flag to show we've asked for the name. We save this out so the
            // context object for the next turn of the conversation can check haveAskedName
            convo.haveAskedNameFlag = true;
        }
        else if (!convo.haveAskedNumberFlag)
        {
            // Save the name.
            var name = context.Activity.AsMessageActivity().Text;
            user.name = name;
            convo.haveAskedNameFlag = false; // Reset flag

            // Ask for the phone number. You might want a flag to track this, too.
            await context.SendActivity($"Hello, {name}. What's your telephone number?");
            convo.haveAskedNumberFlag = true;
        }
        else if (convo.haveAskedNumberFlag)
        {
            // save the telephone number
            var telephonenumber = context.Activity.AsMessageActivity().Text;

            user.telephoneNumber = telephonenumber;
            convo.haveAskedNumberFlag = false; // Reset flag
            await context.SendActivity($"Got it. I'll call you later.");
        }
    }
}
```

若要設定可讓 `UserState<UserInfo>.Get(context)` 傳回的使用者狀態，您可以新增使用者狀態中介軟體。 例如，在 ASP .NET Core EchoBot 的 `Startup.cs` 中，變更 ConfigureServices.cs 內的程式碼：

```csharp
public void ConfigureServices(IServiceCollection services)
{
    services.AddBot<EchoBot>(options =>
    {
        options.CredentialProvider = new ConfigurationCredentialProvider(Configuration);
        
        IStorage dataStore = new MemoryStorage();
        options.Middleware.Add(new ConversationState<ConversationInfo>(dataStore));
        options.Middleware.Add(new UserState<UserInfo>(dataStore));
    });
}
```

# <a name="javascripttabjs"></a>[JavaScript](#tab/js)

**app.js**

```js
const { BotFrameworkAdapter, MemoryStorage, ConversationState, UserState, BotStateSet } = require('botbuilder');
const restify = require('restify');

// Create server
let server = restify.createServer();
server.listen(process.env.port || process.env.PORT || 3978, function () {
    console.log(`${server.name} listening to ${server.url}`);
});

// Create adapter (it's ok for MICROSOFT_APP_ID and MICROSOFT_APP_PASSWORD to be blank for now)  
const adapter = new BotFrameworkAdapter({ 
    appId: process.env.MICROSOFT_APP_ID, 
    appPassword: process.env.MICROSOFT_APP_PASSWORD 
});

// Storage
const storage = new MemoryStorage();
const conversationState = new ConversationState(storage);
const userState  = new UserState(storage);
adapter.use(new BotStateSet(conversationState, userState));

// Listen for incoming requests 
server.post('/api/messages', (req, res) => {
    // Route received request to adapter for processing
    adapter.processActivity(req, res, async (context) => {
        const isMessage = (context.activity.type === 'message');
        const convo = conversationState.get(context);
        const user = userState.get(context);

        if (isMessage) {
            if(!user.name && !convo.haveAskedNameFlag){
                // Ask for the name.
                await context.sendActivity("What is your name?")
                // Set flag to show we've asked for the name. We save this out so the
                // context object for the next turn of the conversation can check haveAskedNameFlag
                convo.haveAskedNameFlag = true;
            } else if(convo.haveAskedNameFlag){
                // Save the name.
                user.name = context.activity.text;
                convo.haveAskedNameFlag = false; // Reset flag

                await context.sendActivity(`Hello, ${user.name}. What's your telephone number?`);
                convo.haveAskedNumberFlag = true; // Set flag
            } else if(convo.haveAskedNumberFlag){
                // save the phone number
                user.telephonenumber = context.activity.text;
                convo.haveAskedNumberFlag = false; // Reset flag
                await context.sendActivity(`Got it. I'll call you later.`);
            }
        }

        // ...
    });
});

```

---

另一種方法是使用「瀑布圖」模型的對話方塊。 對話會追蹤對話方塊狀態，讓您不需要建立旗標來追蹤狀態。 如需詳細資訊，請參閱[使用對話來管理對話方塊](bot-builder-dialog-manage-conversation-flow.md)。

## <a name="file-storage"></a>檔案儲存體

記憶體儲存體提供者會使用記憶體內部儲存體，當 Bot 重新啟動時，系統會處置此儲存體。 其僅適合測試用途。 如果您想要保存資料，但不想要將 Bot 掛到資料庫，則可以使用檔案儲存體提供者。 雖然此提供者也僅供測試之用，但會將狀態資料保存至檔案以供您檢查。 資料會使用 JSON 格式寫到檔案之外。

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

移至 Microsoft.Bot.Samples.EchoBot-AspNetCore 範例中的 `Startup.cs`，並編輯 `ConfigureServices` 方法中的程式碼。
```csharp
// This method gets called by the runtime. Use this method to add services to the container.
public void ConfigureServices(IServiceCollection services)
{
    services.AddBot<EchoBot>(options =>
    {
        options.CredentialProvider = new ConfigurationCredentialProvider(Configuration);

        // Using file storage instead of in-memory storage.
        IStorage dataStore = new FileStorage(System.IO.Path.GetTempPath());
        options.Middleware.Add(new ConversationState<EchoState>(dataStore));
    });
}
``` 

執行程式碼，並讓 echobot 回傳輸入幾次。

然後，移至 `System.IO.Path.GetTempPath()` 所指定的目錄。 您應該會看到名稱開頭為「conversation」的檔案。 將它開啟，並查看 JSON。 檔案包含如下所示內容：
```json
{
  "$type": "Microsoft.Bot.Samples.Echo.EchoState, Microsoft.Bot.Samples.EchoBot",
  "TurnNumber": "3",
  "eTag": "ecfe2a23566b4b52b2fe697cffc59385"
}
```

`$type` 會指定您在 Bot 中用來儲存對話狀態的資料結構類型。 `TurnNumber` 欄位會對應到 `EchoState` 類別中的 `TurnNumber` 屬性。 `eTag` 欄位繼承自 `IStoreItem` 而且是唯一值，每次 Bot 更新對話狀態時便會自動更新此值。  eTag 欄位可讓 Bot 啟用開放式並行存取。

# <a name="javascripttabjs"></a>[JavaScript](#tab/js)

若要使用 `FileStorage`，請更新前面[使用對話和使用者狀態屬性](#use-conversation-and-user-state-properties)一節所述的 echo Bot 範例。 請確定 `storage` 設定為 `FileStorage` 而不是 `MemoryStorage`，而且需要 botbuilder 的 `FileStorage`。 這些是唯一需要的變更。 

```javascript
// Storage
const storage = new FileStorage("c:/temp");
const conversationState = new ConversationState(storage);
const userState  = new UserState(storage);
adapter.use(new BotStateSet(conversationState, userState));
```

`FileStorage` 提供者會採用「路徑」作為參數。 指定路徑可讓您輕鬆在 Bot 中找到具有所保存資訊的檔案。 每個「對話」都會有專為其建立的新檔案。 因此，在「路徑」中，您可能會發現多個開頭為 `conversation!` 的檔案名稱。 您可以依日期排序，以更加輕鬆地尋找最新的對話。 另一方面，您只會找到一個「使用者」狀態的檔案。 檔案名稱的開頭會是 `user!`。 任何時候上述任一物件的狀態有所變更時，狀態管理員就會更新檔案以反映變更的內容。

執行 Bot 並對其傳送一些訊息。 接著，尋找儲存體檔案並開啟。 以下是追蹤回合計數器的 echo Bot，可能會有的 JSON 內容。

```json
{
  "turnNumber": "3",
  "eTag": "322"
}
```
---

## <a name="azure-table-storage"></a>Azure 表格儲存體

您也可以使用 Azure 資料表儲存體作為儲存體媒體。

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

在 Microsoft.Bot.Samples.EchoBot-AspNetCore 範例中，將參考新增至 `Microsoft.Bot.Builder.Azure` NuGet 套件。

然後，移至 `Startup.cs`、新增 `using Microsoft.Bot.Builder.Azure;` 陳述式，並編輯 `ConfigureServices` 方法中的程式碼。
```csharp
services.AddBot<EchoBot>(options =>
{
    options.CredentialProvider = new ConfigurationCredentialProvider(Configuration);
    // The parameters are the connection string and table name.
    // "UseDevelopmentStorage=true" is the connection string to use if you are using the Azure Storage Emulator.
    // Replace it with your own connection string if you're not using the emulator
    options.Middleware.Add(new ConversationState<EchoState>(new AzureTableStorage("UseDevelopmentStorage=true","conversationstatetable")));
    // you could also specify the cloud storage account instead of the connection string
    /* options.Middleware.Add(new ConversationState<EchoState>(
        new AzureTableStorage(WindowsAzure.Storage.CloudStorageAccount.DevelopmentStorageAccount, "conversationstatetable"))); */
    options.EnableProactiveMessages = true;
});
```
`UseDevelopmentStorage=true` 是可與 [Azure 儲存體模擬器][AzureStorageEmulator]搭配使用的連接字串。 如果您不是使用該模擬器，請將它更換為您自己的連接字串。

如果具有 `AzureTableStorage` 建構函式中所指定名稱的資料表不存在，則會建立該資料表。

<!-- 
TODO: step-by-step inspection of the stored table
-->

# <a name="javascripttabjs"></a>[JavaScript](#tab/js)

在 echobot 範例的 `app.js` 中，您可以使用 `AzureTableStorage` 建立 ConversationState。 

```bash
npm install --save botbuilder-azure@preview
```

```javascript
const { BotFrameworkAdapter, FileStorage, MemoryStorage, ConversationState } = require('botbuilder');
const { TableStorage } = require('botbuilder-azure');

// ...

// Create adapter
const adapter = new BotFrameworkAdapter({
    appId: process.env.MICROSOFT_APP_ID,
    appPassword: process.env.MICROSOFT_APP_PASSWORD
});

// Add conversation state middleware
// The parameters are the connection string and table name.
// "UseDevelopmentStorage=true" is the connection string to use if you are using the Azure Storage Emulator.
// Replace it with your own connection string if you're not using the emulator
var azureStorage = new TableStorage({ tableName: "TestAzureTable1", storageAccountOrConnectionString: "UseDevelopmentStorage=true"})

// You can alternatively use your account name and table name
// var azureStorage = new TableStorage({tableName: "TestAzureTable2", storageAccessKey: "V3ah0go4DLkMtQKUPC6EbgFeXnE6GeA+veCwDNFNcdE6rqSVE/EQO/kjfemJaitPwtAkmR9lMKLtcvgPhzuxZg==", storageAccountOrConnectionString: "storageaccount"});

const conversationState = new ConversationState(azureStorage);
adapter.use(conversationState);

```

`UseDevelopmentStorage=true` 是可與 [Azure 儲存體模擬器][AzureStorageEmulator]搭配使用的連接字串。
如果具有 `AzureTableStorage` 建構函式中所指定名稱的資料表不存在，則會建立該資料表。

---

若要查看所儲存的對話狀態資料，請執行範例，然後使用 [Azure 儲存體總管][AzureStorageExplorer]開啟資料表。

![Azure 儲存體總管中的 echobot 對話狀態資料](media/how-to-state/echostate-azure-storage-explorer.png)


**分割區索引鍵**是專為目前對話產生的唯一索引鍵。 如果您重新啟動 Bot 或開始新的對話，新的對話就會有自己的資料列以及自己的分割區索引鍵。 `EchoState` 資料結構會序列化為 JSON，並儲存在 Azure 資料表的 **Json** 資料行。 

```json
{
    "$type":"Microsoft.Bot.Samples.Echo.AspNetCore.EchoState, Microsoft.Bot.Samples.EchoBot-AspNetCore",
    "TurnNumber":2,
    "LastMessage":"second message",
    "eTag":"*"
}
```

BotBuilder SDK 會新增 `$type` 和 `eTag` 欄位。 如需 eTags 的詳細資訊，請參閱[管理開放式並行存取](bot-builder-howto-v4-storage.md#manage-concurrency-using-etags)


## <a name="next-steps"></a>後續步驟

您已了解如何使用狀態來協助您在儲存體中讀取和寫入 Bot 資料，接下來讓我們來了解如何直接在儲存體中讀取和寫入。

> [!div class="nextstepaction"]
> [如何直接寫入到儲存體](bot-builder-howto-v4-storage.md)。

## <a name="additional-resources"></a>其他資源
如需更多關於儲存體的背景資料，請參閱 [Bot 建立器 SDK 中的儲存體](bot-builder-storage-concept.md)

<!-- Links -->
[AzureStorageEmulator]: https://docs.microsoft.com/en-us/azure/storage/common/storage-use-emulator
[AzureStorageExplorer]: https://azure.microsoft.com/en-us/features/storage-explorer/
