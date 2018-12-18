---
title: Bot 的運作方式 | Microsoft Docs
description: 說明 Bot Builder SDK 內的活動和 http 運作方式。
keywords: 對話流程, 回合, bot 對話, 對話方塊, 提示, 瀑布, 對話方塊集
author: johnataylor
ms.author: johtaylo
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 11/15/2018
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: c4d4879f7ad127838de9d2563dee8f8d7320d61e
ms.sourcegitcommit: 91156d0866316eda8d68454a0c4cd74be5060144
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 12/07/2018
ms.locfileid: "53010573"
---
# <a name="how-bots-work"></a>Bot 的運作方式

[!INCLUDE [pre-release-label](../includes/pre-release-label.md)]

Bot 是使用者可運用文字、圖形 (例如卡片或影像) 或語音等對話方式進行互動的應用程式。 使用者與 Bot 之間的每次互動都會產生「活動」。 Bot Service 會在使用者的 Bot 連線應用程式 (例如 Facebook、Skype、Slack 等，我們稱之為「通道」) 與 Bot 之間傳送資訊。 每個通道都可以在其傳送的活動中包含其他資訊。 建立 Bot 之前，請務必了解 Bot 如何使用活動物件來與其使用者通訊。 讓我們先看看當我們執行簡單回應 Bot 時所交換的活動。

![活動圖表](media/bot-builder-activity.png)

這裡說明的兩種活動類型為：「對話更新」和「訊息」。

有對象加入對話時，Bot Framework Service 可以傳送對話更新。 例如，使用 Bot Framework 模擬器開始對話時，您會看到兩個對話更新活動 (一個用於使用者加入對話，另一個用於 Bot 加入對話)。 若要區分這些對話更新活動，請檢查「已新增成員」屬性是否包含 Bot 以外的成員。 

訊息活動會隨附合作對象之間的對話資訊。 在回應 Bot 範例中，訊息活動會隨附簡單的文字，而通道會呈現這段文字。 或者，訊息活動可能隨附要說出的文字、建議的動作或要顯示的卡片。

在此範例中，Bot 建立並傳送了訊息活動，以回應其已接收的輸入訊息活動。 不過，Bot 可以用其他方式來回應所收到的訊息活動；Bot 通常不會在訊息活動中傳送一些歡迎文字來回應對話更新活動。 在[歡迎使用者](bot-builder-welcome-user.md)中可找到詳細資訊。

### <a name="http-details"></a>HTTP 詳細資料

活動會透過 HTTP POST 要求從 Bot Framework Service 送達 Bot。 Bot 會以 200 HTTP 狀態碼回應傳入 POST 要求。 從 Bot 傳送至通道的活動，會以個別的 HTTP POST 傳送至 Bot Framework Service。 接著會以 200 HTTP 狀態碼確認。

通訊協定不會指定進行這些 POST 要求及其確認的順序。 不過，為了配合常見的 HTTP 服務架構，這些要求通常為巢狀，表示輸出 HTTP 要求是從輸入 HTTP 要求範圍內的 Bot 提出。 上面的圖表可說明此模式。 由於有兩個不同的背對背 HTTP 連線，所以必須針對兩者提供安全性模型。

### <a name="defining-a-turn"></a>定義一個回合

在對話中，人們通常一次一個人發言，輪流發言。 Bot 通常會回應使用者輸入。 在 Bot Builder SDK 內，一個「回合」是由使用者對 Bot 的傳入活動，以及 Bot 傳回給使用者作為立即回應的任何活動所組成。 您可以將回合視為與特定活動送達相關聯的處理。

「回合內容」物件會提供活動相關資訊，例如傳送者和接收者、通道、以及處理活動所需的其他資料。 也允許在遍及 Bot 各種階層的回合期間新增資訊。

回合內容是 SDK 中最重要的抽象概念之一。 不只隨附所有中介軟體元件的輸入活動和應用程式邏輯，但也提供中介軟體元件和應用程式邏輯可藉以傳送輸出活動的機制。

## <a name="the-activity-processing-stack"></a>活動處理堆疊

讓我們深入探索先前的圖表，著重於訊息活動的送達。

![活動處理堆疊](media/bot-builder-activity-processing-stack.png)

在上述範例中，Bot 使用了包含相同文字訊息的另一個訊息活動回覆訊息活動。 處理會從送達網路伺服器的 HTTP POST 要求 (以 JSON 承載形式隨附活動資訊) 著手。 在 C# 中，這通常會是 ASP.NET 專案，在 JavaScript Node.js 專案中，這可能是其中一種熱門架構，例如 Express 或 Restify。

「配接器」 (SDK 的整合式元件) 是 SDK 執行階段的核心。 在 HTTP POST 主體中會以 JSON 形式隨附活動。 此 JSON 會還原序列化以建立活動物件，而該物件會透過呼叫「處理活動」方法送交配接器。 接收活動時，配接器會建立「回合內容」並呼叫中介軟體。 「回合內容」這個名稱沿用「回合」這個字，用來描述與活動送達相關聯的所有處理。 回合內容是 SDK 中重要的抽象概念之一，不只隨附所有中介軟體元件的輸入活動和應用程式邏輯，但也提供中介軟體元件和應用程式邏輯用以傳送輸出活動的機制。 回合內容會提供「傳送、更新和刪除活動」回應方法來回應活動。 每個回應方法都會以非同步程序執行。 

[!INCLUDE [alert-await-send-activity](../includes/alert-await-send-activity.md)]


## <a name="middleware"></a>中介軟體
中介軟體就像任何其他傳訊中介軟體，包含一組依序執行的線性元件，可讓每個元件都有機會在活動上運作。 中介軟體管線的最後一個階段是回呼，可在應用程式已向配接器註冊的 Bot 類別上叫用回合處理常式 (在 C# 中為 `OnTurnAsync` 以及在 JS 中為 `onTurn`) 函式。 回合處理常式會以回合內容作為其引數，在回合處理常式函式內執行的應用程式邏輯，通常會處理輸入活動的內容，並且在回應時產生一或多個活動，然後使用回合內容的「傳送活動」函式將這些活動送出。 對回合內容呼叫「傳送活動」會導致在輸出活動上叫用中介軟體元件。 中介軟體元件會在 Bot 的回合處理常式函式前後執行。 此執行原本就是巢狀，因此有時候會比喻為俄羅斯娃娃。 如需有關中介軟體的深入詳細資訊，請參閱[中介軟體主題](~/v4sdk/bot-builder-concept-middleware.md)。

## <a name="bot-structure"></a>Bot 結構
在下列各節中，我們會檢查 Bot 的重要部分。

### <a name="prerequisites"></a>必要條件
- 採用 [C#](https://aka.ms/EchoBotWithStateCSharp) 或 [JS](https://aka.ms/EchoBotWithStateJS) 的一份 **EchoBotWithCounter** 範例。 這裡只會顯示相關的程式碼，但您可以參考範例中的完整原始程式碼。

# <a name="ctabcs"></a>[C#](#tab/cs)

Bot 是一種 [ASP.NET Core](https://docs.microsoft.com/aspnet/core/?view=aspnetcore-2.1) Web 應用程式。 如果您查看 [ASP.NET](https://docs.microsoft.com/aspnet/core/fundamentals/index?view=aspnetcore-2.1&tabs=aspnetcore2x) 基礎，您會在 **Program.cs** 和 **Startup.cs** 等檔案中看到類似的程式碼。 所有 Web 應用程式都需要這些檔案，並不是 Bot 特有的。 

### <a name="bot-logic"></a>Bot 邏輯

主要 Bot 邏輯定義於 `EchoWithCounterBot` 類別，該類別衍生自 `IBot` 介面。 `IBot` 會定義單一方法 `OnTurnAsync`。 您的應用程式必須實作這個方法。 `OnTurnAsync` 具有 turnContext，其可提供傳入活動的相關資訊。 傳入活動會對應至輸入 HTTP 要求。 活動可能是各種類型，因此我們會先查看您的 Bot 是否已收到訊息。 如果這是一則訊息，我們從回合內容取得對話狀態、讓回合計數器遞增，然後將新的回合計數器值保存在對話狀態中。 接著使用 SendActivityAsync 呼叫，將訊息傳回給使用者。 傳出活動會對應至輸出 HTTP 要求。

```cs
public async Task OnTurnAsync(ITurnContext turnContext, CancellationToken cancellationToken = default(CancellationToken))
{
    if (turnContext.Activity.Type == ActivityTypes.Message)
    {
        // Get the conversation state from the turn context.
        var oldState = await _accessors.CounterState.GetAsync(turnContext, () => new CounterState());

        // Bump the turn count for this conversation.
        var newState = new CounterState { TurnCount = oldState.TurnCount + 1 };

        // Set the property using the accessor.
        await _accessors.CounterState.SetAsync(turnContext, newState);

        // Save the new turn count into the conversation state.
        await _accessors.ConversationState.SaveChangesAsync(turnContext);

        // Echo back to the user whatever they typed.
        var responseMessage = $"Turn {newState.TurnCount}: You sent '{turnContext.Activity.Text}'\n";
        await turnContext.SendActivityAsync(responseMessage);
    }
    else
    {
        await turnContext.SendActivityAsync($"{turnContext.Activity.Type} event detected");
    }
}
```

### <a name="set-up-services"></a>設定服務

startup.cs 檔案中的 `ConfigureServices` 方法會從 [.bot](bot-builder-basics.md#the-bot-file) 檔案載入連線服務，攔截在交談回合期間發生的任何錯誤並加以記錄，設定您的認證提供者，以及建立交談狀態物件以在記憶體中儲存交談資料。

```csharp
services.AddBot<EchoWithCounterBot>(options =>
{
    // Creates a logger for the application to use.
    ILogger logger = _loggerFactory.CreateLogger<EchoWithCounterBot>();

    var secretKey = Configuration.GetSection("botFileSecret")?.Value;
    var botFilePath = Configuration.GetSection("botFilePath")?.Value;

    // Loads .bot configuration file and adds a singleton that your Bot can access through dependency injection.
    BotConfiguration botConfig = null;
    try
    {
        botConfig = BotConfiguration.Load(botFilePath ?? @".\BotConfiguration.bot", secretKey);
    }
    catch
    {
        //...
    }

    services.AddSingleton(sp => botConfig);

    // Retrieve current endpoint.
    var environment = _isProduction ? "production" : "development";
    var service = botConfig.Services.Where(s => s.Type == "endpoint" && s.Name == environment).FirstOrDefault();
    if (!(service is EndpointService endpointService))
    {
        throw new InvalidOperationException($"The .bot file does not contain an endpoint with name '{environment}'.");
    }

    options.CredentialProvider = new SimpleCredentialProvider(endpointService.AppId, endpointService.AppPassword);

    // Catches any errors that occur during a conversation turn and logs them.
    options.OnTurnError = async (context, exception) =>
    {
        logger.LogError($"Exception caught : {exception}");
        await context.SendActivityAsync("Sorry, it looks like something went wrong.");
    };

    // The Memory Storage used here is for local bot debugging only. When the bot
    // is restarted, everything stored in memory will be gone.
    IStorage dataStore = new MemoryStorage();

    // ...

    // Create Conversation State object.
    // The Conversation State object is where we persist anything at the conversation-scope.
    var conversationState = new ConversationState(dataStore);

    options.State.Add(conversationState);
});
```

`ConfigureServices` 方法也會建立並註冊 `EchoBotAccessors`，這些存取只定義於 **EchoBotState Accessors.cs** 檔案中，並且會使用 ASP.NET Core 中的相依性插入架構傳入公用 `EchoWithCounterBot` 建構函式。

```csharp
// Accessors created here are passed into the IBot-derived class on every turn.
services.AddSingleton<EchoBotAccessors>(sp =>
{
    var options = sp.GetRequiredService<IOptions<BotFrameworkOptions>>().Value;
    // ...
    var conversationState = options.State.OfType<ConversationState>().FirstOrDefault();
    // ...

    // Create the custom state accessor.
    // State accessors enable other components to read and write individual properties of state.
    var accessors = new EchoBotAccessors(conversationState)
    {
        CounterState = conversationState.CreateProperty<CounterState>(EchoBotAccessors.CounterStateName),
    };

    return accessors;
});
```

`Configure` 方法會指定應用程式使用 Bot Framework 和一些其他檔案，以完成您應用程式的設定。 所有使用 Bot Framework 的 Bot 都需要該組態呼叫。 `ConfigureServices` 和 `Configure` 會在應用程式啟動由執行階段呼叫。

### <a name="manage-state"></a>管理狀態

此檔案包含 Bot 用來維護目前狀態的簡單類別。 其只包含我們用於使計數器遞增的 `int`。

```cs
public class CounterState
{
    public int TurnCount { get; set; } = 0;
}
```

### <a name="accessor-class"></a>存取子類別

`EchoBotAccessors` 類別會建立為 `Startup` 類別中的單一項目，並傳入 IBot 衍生類別中。 在此案例中為 `public class EchoWithCounterBot : IBot`。 Bot 會使用存取子來保存對話資料。 `EchoBotAccessors` 的建構函式會傳入 Startup.cs 檔案中建立的對話物件。

```cs
public class EchoBotAccessors
{
    public EchoBotAccessors(ConversationState conversationState)
    {
        ConversationState = conversationState ?? throw new ArgumentNullException(nameof(conversationState));
    }

    public static string CounterStateName { get; } = $"{nameof(EchoBotAccessors)}.CounterState";

    public IStatePropertyAccessor<CounterState> CounterState { get; set; }

    public ConversationState ConversationState { get; }
}
```

# <a name="javascripttabjs"></a>[JavaScript](#tab/js)

Yeoman 產生器會建立 [Restify](http://restify.com/) Web 應用程式。 如果您查看其文件中的 Restify 快速入門，您會看到類似於所產生 **index.js** 檔案的應用程式。 本節主要說明 **package.json****.env** 、**index.js**、**bot.js** 和 **echobot-with-counter.bot** 檔案。 有些檔案中的程式碼不會複製於此，但是您會在執行 Bot 時看到，還可參考 [Node.js echobot-with-counter](https://aka.ms/js-echobot-with-counter) 範例。

### <a name="packagejson"></a>package.json

**package.json** 會指定 Bot 的相依性及其相關聯的版本。 這是由範本和您的系統所設定。

### <a name="env-file"></a>.env 檔案

**.Env** 檔案指定 Bot 的組態資訊，例如連接埠號碼、應用程式識別碼和密碼，以及其他項目。 如果在生產環境中使用特定技術或使用此 Bot，您必須將您的特定金鑰或 URL 新增至此組態。 不過，針對此 Echo Bot，您不需要立即在此執行任何動作；應用程式識別碼和密碼此時可保持未定義。

若要使用 **.env** 組態檔，範本必須包含額外的套件。  首先，從 npm 取得 `dotenv` 套件：

`npm install dotenv`

### <a name="indexjs"></a>index.js

`index.js` 會設定您的 Bot 和主機服務，該服務會將活動轉送到您的 Bot 邏輯。

#### <a name="required-libraries"></a>必要的程式庫

在 `index.js` 檔案的最上方，您會發現一系列必要的模組或程式庫。 這些模組可讓您存取一組需要納入應用程式中的函式。

```javascript
// Import required packages
const path = require('path');
const restify = require('restify');

// Import required bot services. See https://aka.ms/bot-services to learn more about the different parts of a bot.
const { BotFrameworkAdapter, ConversationState, MemoryStorage } = require('botbuilder');
// Import required bot configuration.
const { BotConfiguration } = require('botframework-config');

const { EchoBot } = require('./bot');

// Read botFilePath and botFileSecret from .env file
// Note: Ensure you have a .env file and include botFilePath and botFileSecret.
const ENV_FILE = path.join(__dirname, '.env');
const env = require('dotenv').config({ path: ENV_FILE });
```

#### <a name="bot-configuration"></a>Bot 組態

下一個部分會從您的 Bot 組態檔載入資訊。

```javascript
// Get the .bot file path
// See https://aka.ms/about-bot-file to learn more about .bot file its use and bot configuration.
const BOT_FILE = path.join(__dirname, (process.env.botFilePath || ''));
let botConfig;
try {
    // Read bot configuration from .bot file.
    botConfig = BotConfiguration.loadSync(BOT_FILE, process.env.botFileSecret);
} catch (err) {
    console.error(`\nError reading bot file. Please ensure you have valid botFilePath and botFileSecret set for your environment.`);
    console.error(`\n - The botFileSecret is available under appsettings for your Azure Bot Service bot.`);
    console.error(`\n - If you are running this bot locally, consider adding a .env file with botFilePath and botFileSecret.`);
    console.error(`\n - See https://aka.ms/about-bot-file to learn more about .bot file its use and bot configuration.\n\n`);
    process.exit();
}

// For local development configuration as defined in .bot file
const DEV_ENVIRONMENT = 'development';

// Define name of the endpoint configuration section from the .bot file
const BOT_CONFIGURATION = (process.env.NODE_ENV || DEV_ENVIRONMENT);

// Get bot endpoint configuration by service name
// Bot configuration as defined in .bot file
const endpointConfig = botConfig.findServiceByNameOrId(BOT_CONFIGURATION);
```

#### <a name="bot-adapter-http-server-and-bot-state"></a>Bot 配接器、HTTP 伺服器和 Bot 狀態

後續部分會設定伺服器和配接器，讓您的 Bot 與使用者通訊及傳送回應。 此伺服器會在 **BotConfiguration.bot** 組態中指定的連接埠上接聽，或退回到 3978 以便使用模擬器連線。 此配接器將作為 Bot 的導體，用於導向傳入和傳出通訊、驗證等等。

我們也會建立狀態物件，其使用 `MemoryStorage` 作為儲存體提供者。 此狀態會定義為 `ConversationState`，這只是表示會保留您的對話狀態。 `ConversationState` 會在記憶體中儲存您感興趣的資訊，在此情況下僅只回合計數器。

```javascript
// Create bot adapter.
// See https://aka.ms/about-bot-adapter to learn more about bot adapter.
const adapter = new BotFrameworkAdapter({
    appId: endpointConfig.appId || process.env.microsoftAppID,
    appPassword: endpointConfig.appPassword || process.env.microsoftAppPassword
});

// Catch-all for any unhandled errors in your bot.
adapter.onTurnError = async (context, error) => {
    // This check writes out errors to console log .vs. app insights.
    console.error(`\n [onTurnError]: ${ error }`);
    // Send a message to the user
    context.sendActivity(`Oops. Something went wrong!`);
    // Clear out state
    await conversationState.clear(context);
    // Save state changes.
    await conversationState.saveChanges(context);
};

// Define a state store for your bot. See https://aka.ms/about-bot-state to learn more about using MemoryStorage.
// A bot requires a state store to persist the dialog and user state between messages.
let conversationState;

// For local development, in-memory storage is used.
// CAUTION: The Memory Storage used here is for local bot debugging only. When the bot
// is restarted, anything stored in memory will be gone.
const memoryStorage = new MemoryStorage();
conversationState = new ConversationState(memoryStorage);

// Create the main dialog.
const bot = new EchoBot(conversationState);

// Create HTTP server
let server = restify.createServer();
server.listen(process.env.port || process.env.PORT || 3978, function() {
    console.log(`\n${ server.name } listening to ${ server.url }`);
    console.log(`\nGet Bot Framework Emulator: https://aka.ms/botframework-emulator`);
    console.log(`\nTo talk to your bot, open echoBot-with-counter.bot file in the Emulator`);
});
```

#### <a name="bot-logic"></a>Bot 邏輯

配接器的 `processActivity` 會將傳入活動傳送至 Bot 邏輯。
`processActivity` 內的第三個參數是一個函式處理常式，系統將會呼叫，以在收到的[活動](#the-activity-processing-stack)經由配接器預先處理，並透過任何中介軟體路由傳送之後，執行 Bot 的邏輯。 回合內容變數 (當作引數傳遞至函式處理常式) 可用來提供傳入活動、傳送者和接收者、通道、對話等的相關資訊。活動處理會路由傳送至 EchoBot 的 `onTurn`。

```javascript
// Listen for incoming requests.
server.post('/api/messages', (req, res) => {
    // Route received request to adapter for processing
    adapter.processActivity(req, res, (context) => {
        // Route to main dialog.
        await bot.onTurn(context);
    });
});
```

### <a name="echobot"></a>EchoBot

所有活動處理都會路由傳送至此類別的 `onTurn` 處理常式。 建立類別時會傳入狀態物件。 建構函式會使用此狀態物件，建立 `this.countProperty` 存取子來保存此 Bot 的回合計數器。

在每個回合上，我們會先查看 Bot 是否已收到訊息。 如果 Bot 並未收到訊息，我們將以所接收的活動類型回應。 接下來，我們會建立狀態變數，以保留您的 Bot 對話資訊。 如果計數變數為 `undefined`，則會將其設定為 1 (這會發生在 Bot 第一次啟動時)，或隨著每則新訊息讓其遞增。 我們會以計數及其所傳送的訊息來回應使用者。 最後，我們會設定計數並將變更儲存到狀態。

```javascript
const { ActivityTypes } = require('botbuilder');

// Turn counter property
const TURN_COUNTER_PROPERTY = 'turnCounterProperty';

class EchoBot {

    constructor(conversationState) {
        // Creates a new state accessor property.
        // See https://aka.ms/about-bot-state-accessors to learn more about the bot state and state accessors
        this.countProperty = conversationState.createProperty(TURN_COUNTER_PROPERTY);
        this.conversationState = conversationState;
    }

    async onTurn(turnContext) {
        // Handle message activity type. User's responses via text or speech or card interactions flow back to the bot as Message activity.
        // Message activities may contain text, speech, interactive cards, and binary or unknown attachments.
        // see https://aka.ms/about-bot-activity-message to learn more about the message and other activity types
        if (turnContext.activity.type === ActivityTypes.Message) {
            // read from state.
            let count = await this.countProperty.get(turnContext);
            count = count === undefined ? 1 : ++count;
            await turnContext.sendActivity(`${ count }: You said "${ turnContext.activity.text }"`);
            // increment and set turn counter.
            await this.countProperty.set(turnContext, count);
        } else {
            // Generic handler for all other activity types.
            await turnContext.sendActivity(`[${ turnContext.activity.type } event detected]`);
        }
        // Save state changes
        await this.conversationState.saveChanges(turnContext);
    }
}

exports.EchoBot = EchoBot;
```

---

## <a name="the-bot-file"></a>Bot 檔案

**.bot** 檔案包含端點、應用程式識別碼和密碼等資訊，並且參考 Bot 所使用的服務。 當您開始從範本建置 Bot 時，系統會為您建立這個檔案，但也可以透過模擬器或其他工具建立自己的檔案。 使用[模擬器](../bot-service-debug-emulator.md)測試 Bot 時，您可以指定所要使用的 .bot 檔案。

```json
{
    "name": "echobot-with-counter",
    "services": [
        {
            "type": "endpoint",
            "name": "development",
            "endpoint": "http://localhost:3978/api/messages",
            "appId": "",
            "appPassword": "",
            "id": "1"
        }
    ],
    "padlock": "",
    "version": "2.0"
}
```

## <a name="additional-resources"></a>其他資源

- 若要了解 bot 中的狀態角色，請參閱[管理狀態](bot-builder-concept-state.md)。
- 若要了解 .bot 檔案在管理資源中所扮演的角色，請參閱[使用 .bot 檔案管理資源](bot-file-basics.md)。
- 若要建立您的第一個 Bot，請參閱其中一個快速入門：[使用 Azure Bot 服務](../bot-service-quickstart.md)、[使用 C# ](../dotnet/bot-builder-dotnet-sdk-quickstart.md) 或[使用 JavaScript](../javascript/bot-builder-javascript-quickstart.md)
