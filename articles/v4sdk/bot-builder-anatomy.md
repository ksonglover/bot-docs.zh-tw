---
title: Bot 的剖析 | Microsoft Docs
description: 細分 Bot 的各個部分及其運作方式
keywords: ''
author: ivorb
ms.author: ivorb
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 06/25/2018
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 238be22eff746edff30a5446d20991dfaedc945a
ms.sourcegitcommit: 44f100a588ffda19c275b118f4f97029f12d1449
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/26/2018
ms.locfileid: "42928286"
---
# <a name="anatomy-of-a-bot"></a>Bot 的剖析

[!INCLUDE [pre-release-label](../includes/pre-release-label.md)]

在大部分[基本觀念](bot-builder-basics.md)中，Bot 是使用者使用訊息進行通訊的對話式應用程式。 Bot 會遵循其各自語言中 Web 應用程式的基本結構，並使用 Bot Framework SDK 在其上建置。

Bot 是由幾個不同元件所組成，可讓您[傳送訊息](./bot-builder-howto-send-messages.md)以及追蹤[狀態](./bot-builder-storage-concept.md)。 您可以透過[模擬器](../bot-service-debug-emulator.md)、[WebChat](../bot-service-manage-test-webchat.md)，或在生產環境中透過其中一個[通道](bot-concepts.md)來與 Bot 通訊。 

在此，我們將逐步解說基本 Echo Bot，並檢查讓其得以運作的每個組成片段。

## <a name="files-created-for-echo-bot"></a>針對 Echo Bot 建立的檔案

每一種程式設計語言都會為 Echo Bot 建立不同的檔案，在此範例中我們稱之為 **BasicEcho**，而這些檔案可分成三個不同的區段：系統區段、協助程式項目和 Bot 的核心。 下表列出 C# 和 JavaScript 的主要檔案。

| C# | JavaScript |
| --- | --- |
| `Properties > launchSettings.json` <br> `wwwroot > default.htm` <br> `appsettings.json` <br> `BasicEcho.bot` <br> `EchoBot.cs` <br> `EchoState.cs` <br> `Program.cs` <br> `readme.md` <br> `Startup.cs` | `.env` <br> `app.js` <br> `package.json` <br> `README.md` <br> `basicEcho.bot` |

我們將在下面各節中逐步引導您了解這些檔案中的一些主要功能。 其中有些檔案有其自己的內含項目組合，這包含 Bot 特有程式庫和一般程式設計程式庫。 這些內含項目會提供您所需的一組函式，以及您希望應用程式中具有的一些函式。 雖然我們不會詳細說明這些內含項目，但是歡迎您使用 Visual Studio 來查看定義，以及查看它們屬於哪個命名空間。

## <a name="system-section"></a>系統區段

系統區段具有可讓 Bot 函式作為標準應用程式所需的內容。 其中包括組態檔、配接器，以及一些 JSON 檔案。 除了一些可能需要您的特定資訊的組態檔，這些項目都可以 (且通常應該) 單獨留下。

# <a name="ctabcs"></a>[C#](#tab/cs)

Bot 是一種 [ASP.NET Core Web](https://docs.microsoft.com/en-us/aspnet/core/?view=aspnetcore-2.1) 應用程式架構。 如果您查看 [ASP.NET 基礎](https://docs.microsoft.com/en-us/aspnet/core/fundamentals/index?view=aspnetcore-2.1&tabs=aspnetcore2x)，您會在以下討論的 appsettings.json、Program.cs 和 Startup.cs 等檔案中看到類似的程式碼。 所有 Web 應用程式都需要這些檔案，並不是 Bot 特有的。 有些檔案中的程式碼不會複製於此，但是您會在執行 Bot 時看到。

### <a name="appsettingsjson"></a>appsettings.json

這個檔案只包含採用基本 JSON 格式的 Bot 設定資訊 (主要是應用程式識別碼和密碼)。  使用[模擬器](../bot-service-debug-emulator.md)測試時，不需要這些資訊並可留空，但是生產環境中需要這些資訊。

### <a name="programcs"></a>Program.cs

此檔案是 ASP.NET Web 應用程式運作所需的檔案，可指定要執行的 `Startup` 類別。

### <a name="startupcs"></a>Startup.cs

**Startup.cs** 有我們稍後會討論的其他有趣部分，但我們在此會先討論系統區段。

`Startup` 的建構函式會指定應用程式設定和環境變數，以及為您的 Web 應用程式建立組態。

`Configure` 方法會指定應用程式使用 Bot Framework 和一些其他檔案，以完成您應用程式的設定。 所有使用 Bot Framework 的 Bot 都需要該組態呼叫。

```cs
using System;
using Microsoft.AspNetCore.Builder;
using Microsoft.AspNetCore.Hosting;
using Microsoft.Bot.Builder.BotFramework;
using Microsoft.Bot.Builder.Core.Extensions;
using Microsoft.Bot.Builder.Integration.AspNet.Core;
using Microsoft.Bot.Builder.TraceExtensions;
using Microsoft.Extensions.Configuration;
using Microsoft.Extensions.DependencyInjection;

namespace BasicEcho
{
    public class Startup
    {
        // This method gets called by the runtime. Use this method to add services to the container.
        // For more information on how to configure your application, visit https://go.microsoft.com/fwlink/?LinkID=398940
        public Startup(IHostingEnvironment env)
        {
            var builder = new ConfigurationBuilder()
                .SetBasePath(env.ContentRootPath)
                .AddJsonFile("appsettings.json", optional: true, reloadOnChange: true)
                .AddJsonFile($"appsettings.{env.EnvironmentName}.json", optional: true)
                .AddEnvironmentVariables();

            Configuration = builder.Build();
        }

        public IConfiguration Configuration { get; }

        // ...
        // Definition of ConfigureServices covered in the next section
        // ...

        // This method gets called by the runtime. Use this method to configure the HTTP request pipeline.
        public void Configure(IApplicationBuilder app, IHostingEnvironment env)
        {
            if (env.IsDevelopment())
            {
                app.UseDeveloperExceptionPage();
            }

            app.UseDefaultFiles()
                .UseStaticFiles()
                .UseBotFramework();
        }
    }
}

```

### <a name="launchsettingsjson-and-readmemd"></a>launchSettings.json 和 readme.md

**launchSettings.json** 只包含一些 Web 應用程式的設定組態，而 **readme.md** 是關於 Echo Bot 的一行簡單說明。 這些檔案的內容對於了解此 Bot 並不重要。 

# <a name="javascripttabjs"></a>[JavaScript](#tab/js)

系統區段主要包含 **package.json**、**.env** 檔案、**app.js** 及 **README.md**。 有些檔案中的程式碼不會複製於此，但是您會在執行 Bot 時看到。

### <a name="packagejson"></a>package.json

**package.json** 會指定 Bot 的相依性及其相關聯的版本。 這是由範本和您的系統所設定。

### <a name="env-file"></a>.env 檔案

**.Env** 檔案指定 Bot 的組態資訊，例如連接埠號碼、應用程式識別碼和密碼，以及其他項目。 如果在生產環境中使用特定技術或使用此 Bot，您必須將您的特定金鑰或 URL 新增至此組態。 不過，針對此 Echo Bot，您不需要立即在此執行任何動作；應用程式識別碼和密碼此時可保持未定義。 

若要使用 **.env** 組態檔，範本必須包含額外的套件。  首先，從 npm 取得 `dotenv` 套件：

`npm install dotenv`

然後將下列一行新增至您的 Bot 與其他必要的程式庫：

```javascript
const dotenv = require('dotenv');
```

### <a name="appjs"></a>app.js

`app.js` 檔案的頂端有伺服器和配接器，可讓您的 Bot 與使用者通訊及傳送回應。 此伺服器會在 **.env** 組態中指定的連接埠上接聽，或退回到 3978 以便使用模擬器連線。 此配接器將作為 Bot 的導體，用於導向傳入和傳出通訊、驗證等等。 

```javascript
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
``` 

### <a name="readmemd"></a>README.md

**README.md** 提供有關建置 Bot 某些層面的一些有用資訊，例如基本 Bot 結構或何謂「對話方塊」，但是對於了解 Bot 並不重要。

---


## <a name="helper-items"></a>協助程式項目

如同任何程式，擁有協助程式方法和其他定義可以簡化我們的程式碼。 我們的 Bot 有幾個部分屬於該類別，例如 `.bot` 檔案，這些對我們的 Bot 很重要，但不一定屬於核心 Bot 邏輯。

### <a name="bot-file"></a>.bot 檔案

`.bot` 檔案包含端點、應用程式識別碼和密碼等資訊，可讓[通道](bot-concepts.md)連線到您的 Bot。 當您開始從範本建置 Bot 時，系統會為您建立這個檔案，但也可以透過模擬器或其他工具建立自己的檔案。

除非您將 Bot 命名為 BasicEcho 以外的名稱，否則檔案內容應該符合您在此所看到的內容。 您可能需要根據 Bot 使用方式來變更及更新此資訊，但是若要執行此 Echo Bot，則不需要立即進行變更。

```json
{
  "name": "BasicEcho",
  "secretKey": "",
  "services": [
    {
      "appId": "",
      "id": "http://localhost:3978/api/messages",
      "type": "endpoint",
      "appPassword": "",
      "endpoint": "http://localhost:3978/api/messages",
      "name": "BasicEcho"
    }
  ]
}
```

# <a name="ctabcs"></a>[C#](#tab/cs)

### <a name="echostatecs"></a>EchoState.cs

**EchoState.cs** 包含 Bot 用來維護目前狀態的簡單類別。 其只包含我們用於使回合計數器遞增的 `int`。 我們會在下一節中更詳細說明，但您現在只需要了解 `EchoState` 是包含回合計數的類別。

```cs
namespace BasicEcho
{
    /// <summary>
    /// Class for storing conversation state.
    /// </summary>
    public class EchoState
    {
        public int TurnCount { get; set; } = 0;
    }
}
```

### <a name="defaulthtm"></a>default.htm

**default.htm** 是您執行 Bot 時顯示的網頁 (以 `html` 撰寫)。 其會顯示連線到 Bot 及如何與其互動的實用資訊，不過，網頁的內容不會影響 Bot 的行為。 當您執行 Bot 時，您會看到程式碼快顯。

# <a name="javascripttabjs"></a>[JavaScript](#tab/js)

### <a name="required-libraries"></a>必要的程式庫

在 `app.js` 檔案的最上方，您會發現一系列必要的模組或程式庫。 這些模組可讓您存取一組需要納入應用程式中的函式。 

```javascript
const { BotFrameworkAdapter, MemoryStorage, ConversationState } = require('botbuilder');
const restify = require('restify');
```

---

## <a name="core-of-the-bot"></a>Bot 的核心

最後，您真正有興趣的是什麼！ Bot 的核心會定義 Bot 與使用者 (包含中介軟體和 Bot 邏輯) 的互動方式。

# <a name="ctabcs"></a>[C#](#tab/cs)

### <a name="middleware"></a>中介軟體

在 **Startup.cs** 內，有一個稱為 `ConfigureServices` 的方法。 這個方法會設定您的認證提供者並新增中介軟體。

此處新增的[中介軟體](bot-builder-concept-middleware.md)可執行兩件事。 第一件是例外狀況處理，可讓您的 Bot 正常地失敗。 它會以內嵌方式定義為 lambda 運算式，只要將例外狀況列印到終端機，並告知使用者發生問題。

第二個中介軟體會使用之前所定義的 `MemoryStorage` 來維護您的狀態。 此狀態會定義為 `ConversationState`，這只是表示會保留您的對話狀態。 它會使用我們使用回合計數器定義的 `EchoState` 類別，來儲存您感興趣的資訊。

``` cs
// This method gets called by the runtime. Use this method to add services to the container.
public void ConfigureServices(IServiceCollection services)
{
    services.AddBot<EchoBot>(options =>
    {
        options.CredentialProvider = new ConfigurationCredentialProvider(Configuration);

        // The CatchExceptionMiddleware provides a top-level exception handler for your bot.
        // Any exceptions thrown by other Middleware, or by your OnTurn method, will be 
        // caught here. To facilitate debugging, the exception is sent out, via Trace, 
        // to the emulator. Trace activities are NOT displayed to users, so in addition
        // an "Ooops" message is sent.
        options.Middleware.Add(new CatchExceptionMiddleware<Exception>(async (context, exception) =>
        {
            await context.TraceActivity("EchoBot Exception", exception);
            await context.SendActivity("Sorry, it looks like something went wrong!");
        }));

        // The Memory Storage used here is for local bot debugging only. When the bot
        // is restarted, anything stored in memory will be gone. 
        IStorage dataStore = new MemoryStorage();

        // The File data store, shown here, is suitable for bots that run on 
        // a single machine and need durable state across application restarts.

        // IStorage dataStore = new FileStorage(System.IO.Path.GetTempPath());

        // For production bots use the Azure Table Store, Azure Blob, or 
        // Azure CosmosDB storage provides, as seen below. To include any of 
        // the Azure based storage providers, add the Microsoft.Bot.Builder.Azure 
        // Nuget package to your solution. That package is found at:
        //      https://www.nuget.org/packages/Microsoft.Bot.Builder.Azure/

        // IStorage dataStore = new Microsoft.Bot.Builder.Azure.AzureTableStorage("AzureTablesConnectionString", "TableName");
        // IStorage dataStore = new Microsoft.Bot.Builder.Azure.AzureBlobStorage("AzureBlobConnectionString", "containerName");

        options.Middleware.Add(new ConversationState<EchoState>(dataStore));
    });
}
```

### <a name="bot-logic"></a>Bot 邏輯

主要 Bot 邏輯會定義於 Bot 的`OnTurn` 處理常式中。 `OnTurn` 會提供[內容](./bot-builder-concept-activity-processing.md#turn-context)變數作為引數，其提供有關傳入[活動](bot-builder-concept-activity-processing.md)、對話等的資訊。

連入活動可以屬於[各種類型](../bot-service-activities-entities.md#activity-types)，因此我們會先查看您的 Bot 是否已收到訊息。 如果這是一則訊息，我們會建立狀態變數，以保留您目前的 Bot 對話資訊。 我們會接著讓 `int` 遞增，以便在向使用者回應計數及其傳送的訊息之前，持續追蹤回合計數。

```cs
using System.Threading.Tasks;
using Microsoft.Bot;
using Microsoft.Bot.Builder;
using Microsoft.Bot.Builder.Core.Extensions;
using Microsoft.Bot.Schema;

namespace BasicEcho
{
    public class EchoBot : IBot
    {
        /// <summary>
        /// Every Conversation turn for our EchoBot will call this method. In here
        /// the bot checks the Activty type to verify it's a message, bumps the 
        /// turn conversation 'Turn' count, and then echoes the users typing
        /// back to them. 
        /// </summary>
        /// <param name="context">Turn scoped context containing all the data needed
        /// for processing this conversation turn. </param>        
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
    }    
}
```

# <a name="javascripttabjs"></a>[JavaScript](#tab/js)

### <a name="middleware"></a>中介軟體

在 **app.js** 內，我們會在此 Bot 中使用[中介軟體](bot-builder-concept-middleware.md)來維護您的狀態，並從 `botbuilder` 使用在頂端定義為必要模組之一的 `MemoryStorage`。 此狀態會定義為 `ConversationState`，這只是表示會保留您的對話狀態。 `ConversationState` 會在記憶體中儲存您感興趣的資訊，在此情況下僅只回合計數器。

```javascript
// Add conversation state middleware
const conversationState = new ConversationState(new MemoryStorage());
adapter.use(conversationState);
```

### <a name="bot-logic"></a>Bot 邏輯

*processActivity* 內的第三個參數是一個函式處理常式，系統將會呼叫，以在收到的[活動](bot-builder-concept-activity-processing.md)經由配接器預先處理，並透過任何中介軟體路由傳送之後，執行 Bot 的邏輯。 [內容](./bot-builder-concept-activity-processing.md#turn-context)變數 (當作引數傳遞至函式處理常式) 可用來提供連入活動、寄件者和接收者、通道、對話等的相關資訊。

我們會先查看 Bot 是否已收到訊息。 如果 Bot 並未收到訊息，我們將以所接收的活動類型回應。 接下來，我們會建立狀態變數，以保留您的 Bot 對話資訊。 如果未定義計數變數，我們會將它設定為 0 (這會發生在您啟動 Bot 時)，或隨著每則新訊息讓它遞增。 最後，我們會以計數及其所傳送的訊息來回應使用者。

```javascript
// Listen for incoming requests 
server.post('/api/messages', (req, res) => {
    // Route received request to adapter for processing
    adapter.processActivity(req, res, (context) => {
        // This bot is only handling Messages
        if (context.activity.type === 'message') {

            // Get the conversation state
            const state = conversationState.get(context);

            // If state.count is undefined set it to 0, otherwise increment it by 1
            const count = state.count === undefined ? state.count = 0 : ++state.count;

            // Echo back to the user whatever they typed.
            return context.sendActivity(`${count}: You said "${context.activity.text}"`);
        } else {
            // Echo back the type of activity the bot detected if not of type message
            return context.sendActivity(`[${context.activity.type} event detected]`);
        }
    });
});
```

---
