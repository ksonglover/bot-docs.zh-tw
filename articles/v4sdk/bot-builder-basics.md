---
title: Bot 的運作方式 | Microsoft Docs
description: 說明 Bot Framework SDK 內的活動和 http 運作方式。
keywords: 對話流程, 回合, bot 對話, 對話方塊, 提示, 瀑布, 對話方塊集
author: johnataylor
ms.author: johtaylo
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 05/23/2019
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 94a3459760c8f0f14886a068d082dafeb9530b19
ms.sourcegitcommit: ea64a56acfabc6a9c1576ebf9f17ac81e7e2a6b7
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 05/24/2019
ms.locfileid: "66215547"
---
# <a name="how-bots-work"></a>Bot 的運作方式

[!INCLUDE[applies-to](../includes/applies-to.md)]

Bot 是使用者可運用文字、圖形 (例如卡片或影像) 或語音等對話方式進行互動的應用程式。 使用者與 Bot 之間的每次互動都會產生「活動」  。 Bot Framework Service 是 Azure Bot Service 的元件，會在使用者的 Bot 連線應用程式 (例如 Facebook、Skype、Slack 等，我們稱之為「通道」  ) 與 Bot 之間傳送資訊。 每個通道都可以在其傳送的活動中包含其他資訊。 建立 Bot 之前，請務必了解 Bot 如何使用活動物件來與其使用者通訊。 讓我們先看看當我們執行簡單回應 Bot 時所交換的活動。 

![活動圖表](media/bot-builder-activity.png)

這裡說明的兩種活動類型為：「對話更新」  和「訊息」  。

有對象加入對話時，Bot Framework Service 可以傳送對話更新。 例如，使用 Bot Framework 模擬器開始對話時，您會看到兩個對話更新活動 (一個用於使用者加入對話，另一個用於 Bot 加入對話)。 若要區分這些對話更新活動，請檢查「已新增成員」  屬性是否包含 Bot 以外的成員。 

訊息活動會隨附合作對象之間的對話資訊。 在回應 Bot 範例中，訊息活動會隨附簡單的文字，而通道會呈現這段文字。 或者，訊息活動可能隨附要說出的文字、建議的動作或要顯示的卡片。

在此範例中，Bot 建立並傳送了訊息活動，以回應其已接收的輸入訊息活動。 不過，Bot 可以用其他方式來回應所收到的訊息活動；Bot 通常不會在訊息活動中傳送一些歡迎文字來回應對話更新活動。 在[歡迎使用者](bot-builder-welcome-user.md)中可找到詳細資訊。

### <a name="http-details"></a>HTTP 詳細資料

活動會透過 HTTP POST 要求從 Bot Framework Service 送達 Bot。 Bot 會以 200 HTTP 狀態碼回應傳入 POST 要求。 從 Bot 傳送至通道的活動，會以個別的 HTTP POST 傳送至 Bot Framework Service。 接著會以 200 HTTP 狀態碼確認。

通訊協定不會指定進行這些 POST 要求及其確認的順序。 不過，為了配合常見的 HTTP 服務架構，這些要求通常為巢狀，表示輸出 HTTP 要求是從輸入 HTTP 要求範圍內的 Bot 提出。 上面的圖表可說明此模式。 由於有兩個不同的背對背 HTTP 連線，所以必須針對兩者提供安全性模型。

### <a name="defining-a-turn"></a>定義一個回合

在對話中，人們通常一次一個人發言，輪流發言。 Bot 通常會回應使用者輸入。 在 Bot Framework SDK 內，一個「回合」  是由使用者對 Bot 的傳入活動，以及 Bot 傳回給使用者作為立即回應的任何活動所組成。 您可以將回合視為與特定活動送達相關聯的處理。

「回合內容」  物件會提供活動相關資訊，例如傳送者和接收者、通道、以及處理活動所需的其他資料。 也允許在遍及 Bot 各種階層的回合期間新增資訊。

回合內容是 SDK 中最重要的抽象概念之一。 不只隨附所有中介軟體元件的輸入活動和應用程式邏輯，但也提供中介軟體元件和應用程式邏輯可藉以傳送輸出活動的機制。

## <a name="the-activity-processing-stack"></a>活動處理堆疊

讓我們深入探索先前的圖表，著重於訊息活動的送達。

![活動處理堆疊](media/bot-builder-activity-processing-stack.png)

在上述範例中，Bot 使用了包含相同文字訊息的另一個訊息活動回覆訊息活動。 處理會從送達網路伺服器的 HTTP POST 要求 (以 JSON 承載形式隨附活動資訊) 著手。 在 C# 中，這通常會是 ASP.NET 專案，在 JavaScript Node.js 專案中，這可能是其中一種熱門架構，例如 Express 或 Restify。

「配接器」  (SDK 的整合式元件) 是 SDK 執行階段的核心。 在 HTTP POST 主體中會以 JSON 形式隨附活動。 此 JSON 會還原序列化以建立活動物件，而該物件會透過呼叫「處理活動」  方法送交配接器。 接收活動時，配接器會建立「回合內容」  並呼叫中介軟體。 

如上所述，回合內容會提供 Bot 用以傳送輸出活動的機制，其最常用於回應入活動。 為了達成此目的，回合內容會提供_傳送、更新和刪除活動_回應方法。 每個回應方法都會以非同步程序執行。 

[!INCLUDE [alert-await-send-activity](../includes/alert-await-send-activity.md)]

## <a name="activity-handlers"></a>活動處理常式

當 Bot 收到活動時，其會將活動傳遞至其*活動處理常式*。 實際上，有一個叫做*回合處理常式*的基底處理常式。 所有活動都會透過該處路由傳送。 然後，該回合處理常式會針對其所收到的活動類型，呼叫個別的活動處理常式。

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

比方說，如果 Bot 收到訊息活動，則回合處理常式會看見該傳入活動並將它傳送給 `OnMessageActivityAsync` 活動處理常式。 

建置您的 Bot 時，用於處理和回應訊息的 Bot 邏輯會進入此 `OnMessageActivityAsync` 處理常式。 同樣地，您用於處理加入交談成員的邏輯會進入 `OnMembersAddedAsync` 處理常式，每當成員加入交談時就會呼叫該處理常式。

若要針對這些處理常式實作您的邏輯，您將會覆寫 Bot 中的這些方法，如下面 [Bot 邏輯](#bot-logic)一節所示。 上述每個處理常式都沒有基底實作，所以只要新增您想要的覆寫邏輯即可。

在某些情況中，您會想要覆寫基底回合處理常式，例如在回合結束時[儲存狀態](bot-builder-concept-state.md)。 若要這麼做，請務必先呼叫 `await base.OnTurnAsync(turnContext, cancellationToken);`，確保在其他程式碼之前執行 `OnTurnAsync` 的基底實作。 此外，該基底實作會負責呼叫其餘的活動處理常式，例如 `OnMessageActivityAsync`。

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

比方說，如果 Bot 收到訊息活動，則回合處理常式會看見該傳入活動並將其傳送給 `onMessage` 活動處理常式。

建置您的 Bot 時，用於處理和回應訊息的 Bot 邏輯會進入此 `onMessage` 處理常式。 同樣地，您用於處理加入交談成員的邏輯會進入 `onMembersAdded` 處理常式，每當成員加入交談時就會呼叫該處理常式。

若要針對這些處理常式實作您的邏輯，您將會覆寫 Bot 中的這些方法，如下面 [Bot 邏輯](#bot-logic)一節所示。 針對每個處理常式，定義您的 Bot 邏輯，然後**務必在結束時呼叫 `next()`** 。 呼叫 `next()`，確定下一個處理常式已執行。

您會想要覆寫基底回合處理常式的情況並不常見，嘗試這麼做時，務必小心。 對於您想在回合結束時進行的事情 (例如 [儲存狀態](bot-builder-concept-state.md))，有個叫做 `onDialog` 的特殊處理常式。 `onDialog` 處理常式在結束時 (其餘的處理常式執行之後) 執行，並不會繫結至特定活動類型。 對於上述所有的處理常式，務必呼叫 `next()` 以確保包裝程序的其餘部分。

---

## <a name="middleware"></a>中介軟體

中介軟體就像任何其他傳訊中介軟體，包含一組依序執行的線性元件，可讓每個元件都有機會在活動上運作。 中介軟體管線的最後一個階段是在應用程式已向配接器的*程序活動*方法註冊的 Bot 類別上，回呼回合處理常式。 回合處理常式通常是採用 C# 的 `OnTurnAsync` 和採用 JavaScript 的 `onTurn`。

回合處理常式會以回合內容作為其引數，在回合處理常式函式內執行的應用程式邏輯，通常會處理輸入活動的內容，並且在回應時產生一或多個活動，然後使用回合內容的「傳送活動」  函式將這些活動送出。 對回合內容呼叫「傳送活動」  會導致在輸出活動上叫用中介軟體元件。 中介軟體元件會在 Bot 的回合處理常式函式前後執行。 此執行原本就是巢狀，因此有時候會比喻為俄羅斯娃娃。 如需有關中介軟體的深入詳細資訊，請參閱[中介軟體主題](~/v4sdk/bot-builder-concept-middleware.md)。

## <a name="bot-structure"></a>Bot 結構

在下列各節中，我們會檢查 EchoBot 的_主要片段_，而您可使用針對 [**CSharp**](../dotnet/bot-builder-dotnet-sdk-quickstart.md) 或 [**JavaScript**](../javascript/bot-builder-javascript-quickstart.md) 提供的範本輕鬆地建立。

<!--Need to add section calling out the controller in code, and explaining it further-->

Bot 是一種 Web 應用程式，而我們會針對每種語言提供範本。

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

VSIX 範本可產生 [ASP.NET MVC Core](https://dotnet.microsoft.com/apps/aspnet/mvc) Web 應用程式。 如果您查看 [ASP.NET](https://docs.microsoft.com/aspnet/core/fundamentals/index?view=aspnetcore-2.1&tabs=aspnetcore2x) 基礎，您會在 **Program.cs** 和 **Startup.cs** 等檔案中看到類似的程式碼。 所有 Web 應用程式都需要這些檔案，並不是 Bot 特有的。

### <a name="appsettingsjson-file"></a>appsettings.json 檔案

**appsettings.json** 檔案指定 Bot 的組態資訊，例如應用程式識別碼和密碼，以及其他項目。 如果在生產環境中使用特定技術或使用此 Bot，您必須將您的特定金鑰或 URL 新增至此組態。 不過，針對此 Echo Bot，您不需要立即在此執行任何動作；應用程式識別碼和密碼此時可保持未定義。

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

<!-- TODO: Update this aka link to point to samples/javascript_nodejs/02.echobot (instead of samples/javascript_nodejs/02.a.echobot) once work-in-progress is merged into master. -->

Yeoman 產生器會建立 [Restify](http://restify.com/) Web 應用程式。 如果您查看其文件中的 Restify 快速入門，您會看到類似於所產生 **index.js** 檔案的應用程式。 我們會說明範本所產生的一些主要檔案。 有些檔案中的程式碼不會複製，但是您會在執行 Bot 時看到，還可參考 [Node.js echobot](https://aka.ms/js-echobot-sample) 範例。

### <a name="packagejson"></a>package.json

**package.json** 會指定 Bot 的相依性及其相關聯的版本。 這是由範本和您的系統所設定。

### <a name="env-file"></a>.env 檔案

**.Env** 檔案指定 Bot 的組態資訊，例如連接埠號碼、應用程式識別碼和密碼，以及其他項目。 如果在生產環境中使用特定技術或使用此 Bot，您必須將您的特定金鑰或 URL 新增至此組態。 不過，針對此 Echo Bot，您不需要立即在此執行任何動作；應用程式識別碼和密碼此時可保持未定義。

若要使用 **.env** 組態檔，範本必須包含額外的套件。  首先，從 npm 取得 `dotenv` 套件：

`npm install dotenv`

---

### <a name="bot-logic"></a>Bot 邏輯

Bot 邏輯會處理來自從一或多個通道的傳入活動，並產生傳出活動來回應。

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

主要 Bot 邏輯會定義於 Bot 程式碼，在此名為 `Bots/EchoBot.cs`。 `EchoBot` 衍生自 `AcitivityHandler`，而後者衍生自 `IBot` 介面。 `ActivityHandler` 會針對不同類型的活動定義各種處理常式，例如以下定義的兩項：`OnMessageActivityAsync` 和 `OnMembersAddedAsync`。 這些方法會受到保護，但可加以覆寫，因為是衍生自 `ActivityHandler`。

`ActivityHandler` 中定義的處理常式如下：

| Event | 處理常式 | 說明 |
| :-- | :-- | :-- |
| 收到的任何活動類型 | `OnTurnAsync` | 根據所收到的活動型別，呼叫其中一個其他處理常式。 |
| 收到的訊息活動 | `OnMessageActivityAsync` | 覆寫此項目以處理 `Message` 活動。 |
| 收到的交談更新活動 | `OnConversationUpdateActivityAsync` | 在 `ConversationUpdate` 活動上，如有 Bot 以外的成員加入或離開交談，則會呼叫處理常式。 |
| 已加入交談的非 Bot 成員 | `OnMembersAddedAsync` | 覆寫此項目以處理要加入交談的成員。 |
| 已離開交談的非 Bot 成員 | `OnMembersRemovedAsync` | 覆寫此項目以處理要離開交談的成員。 |
| 收到的事件活動 | `OnEventActivityAsync` | 在 `Event` 活動上，呼叫事件類型特有的處理常式。 |
| 收到的權杖回應事件活動 | `OnTokenResponseEventAsync` | 覆寫此項目以處理權杖回應事件。 |
| 收到的非權杖回應事件活動 | `OnEventAsync` | 覆寫此項目以處理其他類型的事件。 |
| 收到的其他活動類型 | `OnUnrecognizedActivityTypeAsync` | 覆寫此項目以處理任何未處理的活動類型。 |

這些不同的處理常式都有 `turnContext`，其可提供連入活動 (對應至輸入 HTTP 要求) 的相關資訊。 活動可以是各種類型，所以每個處理常式都會在其回合內容參數中提供強型別活動；在大部分情況下，一律處理通常最常見的 `OnMessageActivityAsync`。

如同舊版 4.x 的這個架構，也有實作公用方法 `OnTurnAsync` 的選項。 目前，這個方法的基底實作會處理錯誤檢查，然後根據連入活動的類型，呼叫每個特定處理常式 (如我們在此範例中定義的兩個處理常式)。 在大部分的情況下，您可不理會該方法並使用個別的處理常式，但如果您的情況需要 `OnTurnAsync` 的自訂實作，這仍是一個選項。

> [!IMPORTANT]
> 如果您覆寫 `OnTurnAsync` 方法，您必須呼叫 `base.OnTurnAsync` 來取得基底實作，以呼叫所有其他 `On<activity>Async` 處理常式或自行呼叫這些處理常式。 否則不會呼叫這些處理常式，且不會執行該程式碼。

在此範例中，我們歡迎新使用者使用 `SendActivityAsync` 呼叫來回應使用者所傳送的訊息。 輸出活動會對應至輸出 HTTP POST 要求。

```cs
public class MyBot : ActivityHandler
{
    protected override async Task OnMessageActivityAsync(ITurnContext<IMessageActivity> turnContext, CancellationToken cancellationToken)
    {
        await turnContext.SendActivityAsync(MessageFactory.Text($"Echo: {turnContext.Activity.Text}"), cancellationToken);
    }

    protected override async Task OnMembersAddedAsync(IList<ChannelAccount> membersAdded, ITurnContext<IConversationUpdateActivity> turnContext, CancellationToken cancellationToken)
    {
        foreach (var member in membersAdded)
        {
            await turnContext.SendActivityAsync(MessageFactory.Text($"welcome {member.Name}"), cancellationToken);
        }
    }
}
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

主要 Bot 邏輯定義於 Bot 程式碼，在此名為 `bots\echoBot.js`。 `EchoBot` 衍生自 `AcitivityHandler`。 `ActivityHandler` 會針對不同類型的活動定義各種處理常式，而您可以提供額外的邏輯 (例如此處的 `onMessage` 和 `onConversationUpdate`) 以修改您 Bot 的行為。

`ActivityHandler` 中定義的處理常式如下：

| Event | 處理常式 | 說明 |
| :-- | :-- | :-- |
| 收到的任何活動類型 | `onTurn` | 根據所收到的活動型別，呼叫其中一個其他處理常式。 |
| 收到的訊息活動 | `onMessage` | 為此項目提供函式以處理 `Message` 活動。 |
| 收到的交談更新活動 | `onConversationUpdate` | 在 `ConversationUpdate` 活動上，如有 Bot 以外的成員加入或離開交談，則會呼叫處理常式。 |
| 已加入交談的非 Bot 成員 | `onMembersAdded` | 為此項目提供函式以處理要加入交談的成員。 |
| 已離開交談的非 Bot 成員 | `onMembersRemoved` | 為此項目提供函式以處理要離開交談的成員。 |
| 收到的事件活動 | `onEvent` | 在 `Event` 活動上，呼叫事件類型特有的處理常式。 |
| 收到的權杖回應事件活動 | `onTokenResponseEvent` | 為此項目提供函式以處理權杖回應事件。 |
| 收到的其他活動類型 | `onUnrecognizedActivityType` | 為此項目提供函式以處理任何未處理的活動類型。 |
| 活動處理常式已完成 | `onDialog` | 為此項目提供函式，以在其餘的活動處理常式完成之後，處理任何應在回合結束時進行的處理。 |

在每個回合上，我們會先查看 Bot 是否已收到訊息。 當我們收到來自使用者的訊息時，我們會回應他們所傳送的訊息。

```javascript
const { ActivityHandler } = require('botbuilder');

class MyBot extends ActivityHandler {
    constructor() {
        super();
        // See https://aka.ms/about-bot-activity-message to learn more about the message and other activity types.
        this.onMessage(async (context, next) => {
            await context.sendActivity(`You said '${ context.activity.text }'`);
            // By calling next() you ensure that the next BotHandler is run.
            await next();
        });
        this.onConversationUpdate(async (context, next) => {
            await context.sendActivity('[conversationUpdate event detected]');
            // By calling next() you ensure that the next BotHandler is run.
            await next();
        });
    }
}

module.exports.MyBot = MyBot;
```

---

### <a name="access-the-bot-from-your-app"></a>從您的應用程式存取 Bot

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

#### <a name="set-up-services"></a>設定服務

`Startup.cs` 檔案中的 `ConfigureServices` 方法會載入連線服務，以及從 `appsettings.json` 或 Azure Key Vault (如果有的話) 載入其金鑰、連接狀態等等。 在此，我們會在服務上新增 MVC 及設定相容性版本，然後設定可透過對 Bot 控制器插入相依性取得的配接器和 Bot。

<!-- want to explain the singleton vs transient here?-->

```csharp
// This method gets called by the runtime. Use this method to add services to the container.
public void ConfigureServices(IServiceCollection services)
{
    services.AddMvc().SetCompatibilityVersion(CompatibilityVersion.Version_2_1);

    // Create the credential provider to be used with the Bot Framework Adapter.
    services.AddSingleton<ICredentialProvider, ConfigurationCredentialProvider>();

    // Create the Bot Framework Adapter.
    services.AddSingleton<IBotFrameworkHttpAdapter, BotFrameworkHttpAdapter>();

    // Create the bot as a transient. In this case the ASP Controller is expecting an IBot.
    services.AddTransient<IBot, EchoBot>();
}
```

`Configure` 方法會指定應用程式使用 MVC 和一些其他檔案，以完成您應用程式的設定。 使用 Bot Framework 的所有 Bot 都需要該組態呼叫，不過，當您建置 Bot 時會已經定義於範例或 VSIX 範本中。 `ConfigureServices` 和 `Configure` 會在應用程式啟動由執行階段呼叫。

#### <a name="bot-controller"></a>Bot 控制器

控制站 (遵循標準 MVC 結構) 可讓您決定訊息和 HTTP POST 要求的路由。 對於我們的 Bot，我們會將傳入要求傳遞至配接器的*程序非同步活動*方法，如上面[活動處理堆疊](#the-activity-processing-stack)一節所述。 在該呼叫中，我們會指定 Bot 和任何其他可能需要的授權資訊。

此控制器會實作 `ControllerBase`、保留我們在 `Startup.cs` 中設定的配接器和 Bot (在此可透過插入相依性取得)，並在收到傳入 HTTP POST 時將所需的資訊傳遞到 Bot。

在此，您會看到由路由和控制器屬性所繼續處理的類別。 這些屬性可協助架構適當地路由傳送訊息，並知道要使用哪一個控制器。 如果您變更路由屬性的值，則會變更模擬器或其他通道用於存取您 Bot 的端點。

```cs
// This ASP Controller is created to handle a request. Dependency Injection will provide the Adapter and IBot
// implementation at runtime. Multiple different IBot implementations running at different endpoints can be
// achieved by specifying a more specific type for the bot constructor argument.
[Route("api/messages")]
[ApiController]
public class BotController : ControllerBase
{
    private readonly IBotFrameworkHttpAdapter Adapter;
    private readonly IBot Bot;

    public BotController(IBotFrameworkHttpAdapter adapter, IBot bot)
    {
        Adapter = adapter;
        Bot = bot;
    }

    [HttpPost]
    public async Task PostAsync()
    {
        // Delegate the processing of the HTTP POST to the adapter.
        // The adapter will invoke the bot.
        await Adapter.ProcessAsync(Request, Response, Bot);
    }
}
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

#### <a name="indexjs"></a>index.js

`index.js` 會設定您的 Bot 和主機服務，該服務會將活動轉送到您的 Bot 邏輯。

#### <a name="required-libraries"></a>必要的程式庫

在 `index.js` 檔案的最上方，您會發現一系列必要的模組或程式庫。 這些模組可讓您存取一組需要納入應用程式中的函式。

```javascript
const dotenv = require('dotenv');
const path = require('path');
const restify = require('restify');

// Import required bot services.
// See https://aka.ms/bot-services to learn more about the different parts of a bot.
const { BotFrameworkAdapter } = require('botbuilder');

// This bot's main dialog.
const { MyBot } = require('./bot');

// Import required bot configuration.
const ENV_FILE = path.join(__dirname, '.env');
dotenv.config({ path: ENV_FILE });
```

#### <a name="set-up-services"></a>設定服務

後續部分會設定伺服器和配接器，讓您的 Bot 與使用者通訊及傳送回應。 伺服器會在組態檔中指定的連接埠上接聽，或退回到 3978  以便與模擬器連線。 此配接器將作為 Bot 的導體，用於導向傳入和傳出通訊、驗證等等。

```javascript
// Create HTTP server
const server = restify.createServer();
server.listen(process.env.port || process.env.PORT || 3978, () => {
    console.log(`\n${ server.name } listening to ${ server.url }`);
    console.log(`\nGet Bot Framework Emulator: https://aka.ms/botframework-emulator`);
    console.log(`\nTo talk to your bot, open the emulator select "Open Bot"`);
});

// Create adapter.
// See https://aka.ms/about-bot-adapter to learn more about how bots work.
const adapter = new BotFrameworkAdapter({
    appId: process.env.MicrosoftAppId,
    appPassword: process.env.MicrosoftAppPassword
});

// Catch-all for errors.
adapter.onTurnError = async (context, error) => {
    // This check writes out errors to console log .vs. app insights.
    console.error(`\n [onTurnError]: ${ error }`);
    // Send a message to the user
    await context.sendActivity(`Oops. Something went wrong!`);
};

// Create the main dialog.
const myBot = new MyBot();
```

#### <a name="forwarding-requests-to-the-bot-logic"></a>將要求轉送至 Bot 邏輯

配接器的 `processActivity` 會將傳入活動傳送至 Bot 邏輯。
`processActivity` 內的第三個參數是一個函式處理常式，系統將會呼叫，以在收到的[活動](#the-activity-processing-stack)經由配接器預先處理，並透過任何中介軟體路由傳送之後，執行 Bot 的邏輯。 回合內容變數 (當作引數傳遞至函式處理常式) 可用來提供傳入活動、傳送者和接收者、通道、對話等的相關資訊。活動處理會路由傳送至 Bot 的 `run` 方法。 `run` 定義於 `ActivityHandler` 中，其會執行一些錯誤檢查，然後根據所接收活動的類型，呼叫 Bot 的事件處理常式。

```javascript
// Listen for incoming requests.
server.post('/api/messages', (req, res) => {
    adapter.processActivity(req, res, async (context) => {
        // Route to main dialog.
        await myBot.run(context);
    });
});
```

---

## <a name="manage-bot-resources"></a>管理 Bot 資源

Bot 資源 (例如連線服務的應用程式識別碼、密碼、金鑰或祕密) 必須受到適當管理。 如需如何執行這項操作的詳細資訊，請參閱[管理 Bot 資源](bot-file-basics.md)。

## <a name="additional-resources"></a>其他資源

- 若要了解 bot 中的狀態角色，請參閱[管理狀態](bot-builder-concept-state.md)。
