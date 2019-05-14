---
title: 將遙測資料新增至 Bot | Microsoft Docs
description: 了解如何整合 Bot 與新的遙測功能。
keywords: 遙測, appinsights, 監視 bot
author: ivorb
ms.author: v-ivorb
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 02/06/2019
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 414417e3722e2d9063e1d177b534b6caa814c0db
ms.sourcegitcommit: f84b56beecd41debe6baf056e98332f20b646bda
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 05/03/2019
ms.locfileid: "65032439"
---
# <a name="add-telemetry-to-your-bot"></a>將遙測新增至 Bot

[!INCLUDE[applies-to](../includes/applies-to.md)]

在 Bot Framework sdk 4.2 版中，已在產品中新增遙測記錄功能。  這可讓 Bot 應用程式將事件資料傳送至 Application Insights 之類的服務。 第一節會說明這兩種方法，之後則會有更廣泛的遙測功能。

本文件說明如何整合 Bot 與新的遙測功能。

## <a name="basic-telemetry-options"></a>基本遙測選項

### <a name="basic-application-insights"></a>基本的 Application insights
設定 Bot 有兩種方法。  第一種方法假設您要與 Application Insights 整合。

設定檔包含有關 Bot 在執行時所用外部服務的中繼資料。  例如，CosmosDB、Application Insights 和 Language Understanding (LUIS) 服務連線和中繼資料都會儲存在這裡。   

如果您想要「貯存」Application Insights，不需要額外的 Application Insights 專屬設定 (例如遙測初始設定式)，只要在初始化期間傳入組態物件 (通常是 `IConfiguration`) 即可。   這是最簡單的初始化方法並且會設定 Application Insights，以開始追蹤要求、其他服務的外部呼叫，以及跨服務相互關聯事件。

您必須新增 **Microsoft.Bot.Builder.Integration.ApplicationInsights.Core** NuGet 套件。

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

**Startup.cs**
```csharp
public void ConfigureServices(IServiceCollection services)
{
     ...
     // Add Application Insights - pass in the bot configuration
     services.AddBotApplicationInsights(<your IConfiguration variable name - likely "config">);
     ...
}

public void Configure(IApplicationBuilder app, IHostingEnvironment env)
{
     app.UseBotApplicationInsights()
                 ...
                .UseDefaultFiles()
                .UseStaticFiles()
                .UseBotFramework();
                ...
}
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

```JavaScript
const appInsightsClient = new ApplicationInsightsTelemetryClient(<your configuration variable name - likely "config">);
```

---

### <a name="overriding-the-telemetry-client"></a>覆寫遙測用戶端

如果您想要自訂 Application Insights 用戶端，或者想要登入完全不同的服務，則必須以不同的方式設定系統。 透過 nuget 下載套件 `Microsoft.Bot.Builder.ApplicationInsights`，或使用 npm 安裝 `botbuilder-applicationinsights`。 檢測金鑰可於 Azure 入口網站上找到。

**修改 Application Insights 組態**

```csharp

public void ConfigureServices(IServiceCollection services)
{
     ...
     // Create Application Insight Telemetry Client
     // with custom configuration.
     var telemetryClient = TelemetryClient(myCustomConfiguration)
     
     // Add Application Insights
     services.AddBotApplicationInsights(new BotTelemetryClient(telemetryClient), "InstrumentationKey");
```

**使用自訂遙測** 如果您想要將 Bot Framework 所產生的遙測事件記錄到完全不同的系統，請建立自基底介面衍生的新類別並加以設定。  

```csharp
public void ConfigureServices(IServiceCollection services)
{
     ...
     // Create my IBotTelemetryClient-based logger
     var myTelemetryClient = MyTelemetryLogger();
     
     // Add Application Insights
     services.AddSingleton(myTelemetryClient);
     ...
}
```

### <a name="add-custom-logging-to-your-bot"></a>將自訂記錄新增至您的 Bot

一旦 Bot 設定了新的遙測記錄支援，您即可開始將遙測新增至您的 Bot。  `BotTelemetryClient` (採用 C#，`IBotTelemetryClient`) 有數種方法可記錄不同類型的事件。  選擇適當的事件類型，可讓您利用 Application Insights 現有的報告 (如果您使用 Application Insights 的話)。  在一般情況下，通常會使用 `TraceEvent`。  使用 `TraceEvent` 記錄的資料會落在 Kusto 的 `CustomEvent` 資料表中。

如果在 Bot 內使用對話，則每個對話式物件 (包括提示) 都會包含新的 `TelemetryClient` 屬性。  這是可讓您執行記錄的 `BotTelemetryClient`。  這不是只為了方便而已，我們會在本文稍看到如果已設定這個屬性，`WaterfallDialogs` 將會產生事件。

#### <a name="identifiers-and-custom-events"></a>識別碼和自訂事件

將事件記錄到 Application Insights 時，所產生的事件包含您不必填入的預設屬性。  比方說，每個自訂事件 (以 `TraceEvent` API 產生) 都會包含 `user_id` 和 `session_id` 屬性。  此外，也會新增 `activitiId`、`activityType` 和 `channelId`。

>注意：系統不會提供這些值給自訂遙測用戶端。

屬性 |類型 | 詳細資料
--- | --- | ---
`user_id`| `string` | [ChannelID](https://github.com/Microsoft/BotBuilder/blob/master/specs/botframework-activity/botframework-activity.md#channel-id) + [From.Id](https://github.com/Microsoft/BotBuilder/blob/master/specs/botframework-activity/botframework-activity.md#from)
`session_id`| `string`|  [ConversationID](https://github.com/Microsoft/BotBuilder/blob/master/specs/botframework-activity/botframework-activity.md#conversation)
`customDimensions.activityId`| `string` | [Bot 活動識別碼](https://github.com/Microsoft/BotBuilder/blob/master/specs/botframework-activity/botframework-activity.md#id)
`customDimensions.activityType` | `string` | [Bot 活動類型](https://github.com/Microsoft/BotBuilder/blob/master/specs/botframework-activity/botframework-activity.md#channel-id)
`customDimensions.channelId` | `string` |  [Bot 活動通道識別碼](https://github.com/Microsoft/BotBuilder/blob/master/specs/botframework-activity/botframework-activity.md#channel-id)

## <a name="in-depth-telemetry"></a>深入的遙測

4.4 版的 SDK 新增了三個新元件。  所有元件會使用 `IBotTelemetryClient` (在 node.js 中則使用 `BotTelemetryClient` ) 介面進行記錄，並可使用自訂實作來加以覆寫。

- Bot Framework 中介軟體元件 (*TelemetryLoggerMiddleware*)，會在接收、傳送、更新或刪除訊息時留下記錄。 您可以進行覆寫來建立自訂記錄。
- *LuisRecognizer* 類別。  您可以透過兩種方式進行覆寫來建立自訂記錄 - 經由叫用 (新增/取代屬性) 或衍生類別。
- QnAMaker 類別。  您可以透過兩種方式進行覆寫來建立自訂記錄 - 經由叫用 (新增/取代屬性) 或衍生類別。

### <a name="telemetry-middleware"></a>遙測中介軟體

|C#  | JavaScript |
|:-----|:------------|
|**Microsoft.Bot.Builder.TelemetryLoggerMiddleware** | **botbuilder-core** |

#### <a name="out-of-box-usage"></a>現成可用的使用方式

TelemetryLoggerMiddleware 是無須修改即可新增的 Bot Framework 元件，其會執行記錄而能提供會隨附於 Bot Framework SDK 的現成可用報告。 

```csharp
var telemetryClient = sp.GetService<IBotTelemetryClient>();
var telemetryLogger = new TelemetryLoggerMiddleware(telemetryClient, logPersonalInformation: true);
options.Middleware.Add(telemetryLogger);  // Add to the middleware collection
```

#### <a name="adding-properties"></a>新增屬性
如果您決定要新增其他屬性，就可以衍生 TelemetryLoggerMiddleware 類別。  例如，如果您想要將屬性 "MyImportantProperty" 新增至 `BotMessageReceived` 事件。  當使用者對 Bot 傳送訊息時便會記錄 `BotMessageReceived`。  若要新增其他屬性，可透過下列方式來完成：

```csharp
class MyTelemetryMiddleware : TelemetryLoggerMiddleware
{
    ...
    public Task OnReceiveActivityAsync(
                  Activity activity,
                  CancellationToken cancellation)
    {
        // Fill in the "standard" properties for BotMessageReceived
        // and add our own property.
        var properties = FillReceiveEventProperties(activity, 
                    new Dictionary<string, string>
                    { {"MyImportantProperty", "myImportantValue" } } );
                    
        // Use TelemetryClient to log event
        TelemetryClient.TrackEvent(
                        TelemetryLoggerConstants.BotMsgReceiveEvent,
                        properties);
    }
    ...
}
```

而在啟動時，我們會新增新的類別：

```csharp
var telemetryLogger = new TelemetryLuisRecognizer(telemetryClient, logPersonalInformation: true);
options.Middleware.Add(telemetryLogger);  // Add to the middleware collection
```

#### <a name="completely-replacing-properties--additional-events"></a>完全取代屬性/其他事件

如果您決定要完全取代所記錄的屬性，便可以衍生 `TelemetryLoggerMiddleware` 類別 (如上方擴充屬性時)。   同樣地，記錄新事件也會以相同方式來執行。

例如，如果您要完全取代 `BotMessageSend` 屬性並傳送多個事件，下列範例會示範要如何執行：

```csharp
class MyTelemetryMiddleware : TelemetryLoggerMiddleware
{
    ...
    public Task<RecognizerResult> OnLuisRecognizeAsync(
                  Activity activity,
                  string dialogId = null,
                  CancellationToken cancellation)
    {
        // Override properties for BotMsgSendEvent
        var botMsgSendProperties = new Dictionary<string, string>();
        properties.Add("MyImportantProperty", "myImportantValue");
        // Log event
        TelemetryClient.TrackEvent(
                        TelemetryLoggerConstants.BotMsgSendEvent,
                        botMsgSendProperties);
                        
        // Create second event.
        var secondEventProperties = new Dictionary<string, string>();
        secondEventProperties.Add("activityId",
                                   activity.Id);
        secondEventProperties.Add("MyImportantProperty",
                                   "myImportantValue");
        TelemetryClient.TrackEvent(
                        "MySecondEvent",
                        secondEventProperties);
    }
    ...
}
```
注意：若未記錄標準屬性，將會導致隨附於產品的現成可用報告停止運作。

#### <a name="events-logged-from-telemetry-middleware"></a>從遙測中介軟體記錄的事件
[BotMessageSend](#customevent-botmessagesend)
[BotMessageReceived](#customevent-botmessagereceived)
[BotMessageUpdate](#customevent-botmessageupdate)
[BotMessageDelete](#customevent-botmessagedelete)

### <a name="telemetry-support-luis"></a>遙測支援 LUIS 

|C#  | JavaScript |
|:-----|:------------|
| **Microsoft.Bot.Builder.AI.Luis** | **botbuilder-ai** |

#### <a name="out-of-box-usage"></a>現成可用的使用方式
LuisRecognizer 是現有的 Bot Framework 元件，藉由透過 `luisOptions` 傳遞 IBotTelemetryClient 介面即可啟用遙測。  您可以視需要覆寫所記錄的預設屬性，並記錄新的事件。

在建構 `luisOptions` 期間，必須提供 `IBotTelemetryClient` 物件才能讓此功能運作。

```csharp
var luisOptions = new LuisPredictionOptions(
      ...
      telemetryClient,
      false); // Log personal information flag. Defaults to false.

var client = new LuisRecognizer(luisApp, luisOptions);
```

#### <a name="adding-properties"></a>新增屬性
如果您決定要新增其他屬性，就可以衍生 `LuisRecognizer` 類別。  例如，如果您想要將屬性 "MyImportantProperty" 新增至 `LuisResult` 事件。  執行 LUIS 預測呼叫時，會記錄 `LuisResult`。  若要新增其他屬性，可透過下列方式來完成：

```csharp
class MyLuisRecognizer : LuisRecognizer 
{
   ...
   override protected Task OnRecognizerResultAsync(
           RecognizerResult recognizerResult,
           ITurnContext turnContext,
           Dictionary<string, string> properties = null,
           CancellationToken cancellationToken = default(CancellationToken))
   {
       var luisEventProperties = FillLuisEventProperties(result, 
               new Dictionary<string, string>
               { {"MyImportantProperty", "myImportantValue" } } );
        
        TelemetryClient.TrackEvent(
                        LuisTelemetryConstants.LuisResultEvent,
                        luisEventProperties);
        ..
   }    
   ...
}
```

#### <a name="add-properties-per-invocation"></a>經由叫用來新增屬性
有時候您必須在叫用期間新增其他屬性：
```csharp
var additionalProperties = new Dictionary<string, string>
{
   { "dialogId", "myDialogId" },
   { "conversationInfo", "myConversationInfo" },
};

var result = await recognizer.RecognizeAsync(turnContext,
     additionalProperties,
     CancellationToken.None).ConfigureAwait(false);
```

#### <a name="completely-replacing-properties--additional-events"></a>完全取代屬性/其他事件
如果您決定要完全取代所記錄的屬性，便可以衍生 `LuisRecognizer` 類別 (如上方擴充屬性時)。   同樣地，記錄新事件也會以相同方式來執行。

例如，如果您要完全取代 `LuisResult` 屬性並傳送多個事件，下列範例會示範要如何執行：

```csharp
class MyLuisRecognizer : LuisRecognizer
{
    ...
    override protected Task OnRecognizerResultAsync(
             RecognizerResult recognizerResult,
             ITurnContext turnContext,
             Dictionary<string, string> properties = null,
             CancellationToken cancellationToken = default(CancellationToken))
    {
        // Override properties for LuisResult event
        var luisProperties = new Dictionary<string, string>();
        properties.Add("MyImportantProperty", "myImportantValue");
        
        // Log event
        TelemetryClient.TrackEvent(
                        LuisTelemetryConstants.LuisResult,
                        luisProperties);
                        
        // Create second event.
        var secondEventProperties = new Dictionary<string, string>();
        secondEventProperties.Add("MyImportantProperty2",
                                   "myImportantValue2");
        TelemetryClient.TrackEvent(
                        "MySecondEvent",
                        secondEventProperties);
        ...
    }
    ...
}
```
注意：若未記錄標準屬性，將會導致隨附於產品的 Application Insights 現成可用報告停止運作。

#### <a name="events-logged-from-telemetryluisrecognizer"></a>從 TelemetryLuisRecognizer 記錄的事件
[LuisResult](#customevent-luisevent)

### <a name="telemetry-qna-recognizer"></a>遙測 QnA 辨識器

|C#  | JavaScript |
|:-----|:------------|
| **Microsoft.Bot.Builder.AI.QnA** | **botbuilder-ai** |


#### <a name="out-of-box-usage"></a>現成可用的使用方式
QnAMaker 類別是現有的 Bot Framework 元件，會另外新增兩個建構函式參數來啟用記錄功能，以啟用隨附於 Bot Framework SDK 的現成可用報告。 新的 `telemetryClient` 會參考執行記錄的 `IBotTelemetryClient` 介面。  

```csharp
var qna = new QnAMaker(endpoint, options, client, 
                       telemetryClient: telemetryClient,
                       logPersonalInformation: true);
```
#### <a name="adding-properties"></a>新增屬性 
如果您決定要新增其他屬性，方法有兩種 - 在必須於 QnA 呼叫期間新增屬性以便擷取答案時，或從 `QnAMaker` 類別衍生。  

下列範例會示範如何從 `QnAMaker` 類別衍生。  此範例會顯示如何將屬性 "MyImportantProperty" 新增至 `QnAMessage` 事件。  執行 QnA `GetAnswers` 呼叫時，會記錄 `QnAMessage` 事件。  此外，我們還會記錄第二個事件 "MySecondEvent"。

```csharp
class MyQnAMaker : QnAMaker 
{
   ...
   protected override Task OnQnaResultsAsync(
                 QueryResult[] queryResults, 
                 ITurnContext turnContext, 
                 Dictionary<string, string> telemetryProperties = null, 
                 Dictionary<string, double> telemetryMetrics = null, 
                 CancellationToken cancellationToken = default(CancellationToken))
   {
            var eventData = await FillQnAEventAsync(queryResults, turnContext, telemetryProperties, telemetryMetrics, cancellationToken).ConfigureAwait(false);

            // Add my property
            eventData.Properties.Add("MyImportantProperty", "myImportantValue");

            // Log QnaMessage event
            TelemetryClient.TrackEvent(
                            QnATelemetryConstants.QnaMsgEvent,
                            eventData.Properties,
                            eventData.Metrics
                            );

            // Create second event.
            var secondEventProperties = new Dictionary<string, string>();
            secondEventProperties.Add("MyImportantProperty2",
                                       "myImportantValue2");
            TelemetryClient.TrackEvent(
                            "MySecondEvent",
                            secondEventProperties);       }    
    ...
}
```

#### <a name="adding-properties-during-getanswersasync"></a>在 GetAnswersAsync 期間新增屬性
如果您有需要在執行階段新增的屬性，`GetAnswersAsync` 方法會提供可新增至事件的屬性和 (或) 計量。

例如，如果您想要將 `dialogId` 新增至事件，則可以透過如下方式來完成：
```csharp
var telemetryProperties = new Dictionary<string, string>
{
   { "dialogId", myDialogId },
};

var results = await qna.GetAnswersAsync(context, opts, telemetryProperties);
```
`QnaMaker` 類別會提供覆寫屬性的功能，包括 PersonalInfomation 屬性。

#### <a name="completely-replacing-properties--additional-events"></a>完全取代屬性/其他事件
如果您決定要完全取代所記錄的屬性，便可以衍生 `TelemetryQnAMaker` 類別 (如上方擴充屬性時)。   同樣地，記錄新事件也會以相同方式來執行。

例如，如果您要完全取代 `QnAMessage` 屬性，下列範例會示範要如何執行：

```csharp
class MyLuisRecognizer : TelemetryQnAMaker
{
    ...
    protected override Task OnQnaResultsAsync(
         QueryResult[] queryResults, 
         ITurnContext turnContext, 
         Dictionary<string, string> telemetryProperties = null, 
         Dictionary<string, double> telemetryMetrics = null, 
         CancellationToken cancellationToken = default(CancellationToken))
    {
        // Add properties from GetAnswersAsync
        var properties = telemetryProperties ?? new Dictionary<string, string>();
        // GetAnswerAsync properties overrides - don't add if already present.
        properties.TryAdd("MyImportantProperty", "myImportantValue");

        // Log event
        TelemetryClient.TrackEvent(
                           QnATelemetryConstants.QnaMsgEvent,
                            properties);
    }
    ...
}
```
注意：若未記錄標準屬性，將會導致隨附於產品的現成可用報告停止運作。

#### <a name="events-logged-from-telemetryluisrecognizer"></a>從 TelemetryLuisRecognizer 記錄的事件
[QnAMessage](#customevent-qnamessage)


## <a name="waterfalldialog-events"></a>WaterfallDialog 事件

除了產生您自己的事件，SDK 內的 `WaterfallDialog` 物件現在也會產生事件。 下一節說明從 Bot Framework 內產生的事件。 在 `WaterfallDialog` 上設定 `TelemetryClient` 屬性，就會儲存這些事件。

以下範例示範如何修改利用 `WaterfallDialog` 來記錄遙測事件的範例 (BasicBot)。  BasicBot 會利用當 `WaterfallDialog` 置於 `ComponentDialog` (`GreetingDialog`) 內時使用的通用樣式。

```csharp
// IBotTelemetryClient is direct injected into our Bot
public BasicBot(BotServices services, UserState userState, ConversationState conversationState, IBotTelemetryClient telemetryClient)
...

// The IBotTelemetryClient passed to the GreetingDialog
...
Dialogs = new DialogSet(_dialogStateAccessor);
Dialogs.Add(new GreetingDialog(_greetingStateAccessor, telemetryClient));
...

// The IBotTelemetryClient to the WaterfallDialog
...
AddDialog(new WaterfallDialog(ProfileDialog, waterfallSteps) { TelemetryClient = telemetryClient });
...

```

一旦 `WaterfallDialog` 有已設定的 `IBotTelemetryClient`，它就會開始記錄事件。

## <a name="events-generated-by-the-bot-framework-service"></a>Bot Framework Service 所產生的事件

除了 `WaterfallDialog` (可從 Bot 程式碼產生事件)，Bot Framework Channel 服務也會記錄事件。  這可協助您診斷通道或整體 Bot 失敗的問題。

### <a name="customevent-activity"></a>CustomEvent："Activity"
**記錄來源：** 通道服務 在收到訊息時由通道服務記錄。

### <a name="exception-bot-errors"></a>例外狀況：「Bot 錯誤」
**記錄來源：** 通道服務 在 Bot 呼叫傳回非 2XX Http 回應時由通道記錄。

## <a name="all-events-generated"></a>產生的所有事件

### <a name="customevent-waterfallstart"></a>CustomEvent："WaterfallStart" 

當 WaterfallDialog 開始時，就會記錄 `WaterfallStart` 事件。

- `user_id`  ([從遙測初始設定式](https://aka.ms/telemetry-initializer))
- `session_id` ([從遙測初始設定式](https://aka.ms/telemetry-initializer))
- `customDimensions.activityId`  ([從遙測初始設定式](https://aka.ms/telemetry-initializer))
- `customDimensions.activityType`  ([從遙測初始設定式](https://aka.ms/telemetry-initializer))
- `customDimensions.channelId` ([從遙測初始設定式](https://aka.ms/telemetry-initializer))
- `customDimensions.DialogId` (這是傳入您瀑布的 dialogId (字串)。  您可以將此視為「瀑布類型」)
- `customDimensions.InstanceID` (每個對話執行個體特有)

### <a name="customevent-waterfallstep"></a>CustomEvent："WaterfallStep" 

記錄瀑布式對話中的個別步驟。

- `user_id`  ([從遙測初始設定式](https://aka.ms/telemetry-initializer))
- `session_id` ([從遙測初始設定式](https://aka.ms/telemetry-initializer))
- `customDimensions.activityId`  ([從遙測初始設定式](https://aka.ms/telemetry-initializer))
- `customDimensions.activityType`  ([從遙測初始設定式](https://aka.ms/telemetry-initializer))
- `customDimensions.channelId` ([從遙測初始設定式](https://aka.ms/telemetry-initializer))
- `customDimensions.DialogId` (這是傳入您瀑布的 dialogId (字串)。  您可以將此視為「瀑布類型」)
- `customDimensions.StepName` (若為 lambda 則是方法名稱或 `StepXofY`)
- `customDimensions.InstanceID` (每個對話執行個體特有)

### <a name="customevent-waterfalldialogcomplete"></a>CustomEvent："WaterfallDialogComplete"

記錄於瀑布式對話完成時。

- `user_id`  ([從遙測初始設定式](https://aka.ms/telemetry-initializer))
- `session_id` ([從遙測初始設定式](https://aka.ms/telemetry-initializer))
- `customDimensions.activityId`  ([從遙測初始設定式](https://aka.ms/telemetry-initializer))
- `customDimensions.activityType`  ([從遙測初始設定式](https://aka.ms/telemetry-initializer))
- `customDimensions.channelId` ([從遙測初始設定式](https://aka.ms/telemetry-initializer))
- `customDimensions.DialogId` (這是傳入您瀑布的 dialogId (字串)。  您可以將此視為「瀑布類型」)
- `customDimensions.InstanceID` (每個對話執行個體特有)

### <a name="customevent-waterfalldialogcancel"></a>CustomEvent："WaterfallDialogCancel" 

記錄於瀑布式對話取消時。

- `user_id`  ([從遙測初始設定式](https://aka.ms/telemetry-initializer))
- `session_id` ([從遙測初始設定式](https://aka.ms/telemetry-initializer))
- `customDimensions.activityId`  ([從遙測初始設定式](https://aka.ms/telemetry-initializer))
- `customDimensions.activityType`  ([從遙測初始設定式](https://aka.ms/telemetry-initializer))
- `customDimensions.channelId` ([從遙測初始設定式](https://aka.ms/telemetry-initializer))
- `customDimensions.DialogId` (這是傳入您瀑布的 dialogId (字串)。  您可以將此視為「瀑布類型」)
- `customDimensions.StepName` (若為 lambda 則是方法名稱或 `StepXofY`)
- `customDimensions.InstanceID` (每個對話執行個體特有)

### <a name="customevent-botmessagereceived"></a>CustomEvent：BotMessageReceived 
記錄於 Bot 從使用者收到新訊息時。

若未加以覆寫，則會使用 `Microsoft.Bot.Builder.IBotTelemetry.TrackEvent()` 方法從 `Microsoft.Bot.Builder.TelemetryLoggerMiddleware` 記錄此事件。

- 工作階段識別碼  
  - 使用 Application Insights 時，會從 `TelemetryBotIdInitializer` 將此屬性記錄為 Application Insights 中所使用的**工作階段**識別碼 (Temeletry.Context.Session.Id)。  
  - 對應至 Bot Framework 通訊協定所定義的[對話識別碼](https://github.com/Microsoft/BotBuilder/blob/master/specs/botframework-activity/botframework-activity.md#conversation)。
  - 所記錄的屬性名稱是 `session_id`。

- 使用者識別碼
  - 使用 Application Insights 時，會從 `TelemetryBotIdInitializer` 將此屬性記錄為 Application Insights 中所使用的**使用者**識別碼 (Telemetry.Context.User.Id)。  
  - 此屬性的值結合了 Bot Framework 通訊協定所定義的[通道識別碼](https://github.com/Microsoft/BotBuilder/blob/master/specs/botframework-activity/botframework-activity.md#channel-id)和[使用者識別碼](https://github.com/Microsoft/BotBuilder/blob/master/specs/botframework-activity/botframework-activity.md#from) (串連在一起) 屬性。
  - 所記錄的屬性名稱是 `user_id`。

- ActivityID 
  - 使用 Application Insights 時，會從 `TelemetryBotIdInitializer` 將此屬性記錄為事件的屬性。
  - 對應至 Bot Framework 通訊協定所定義的[活動識別碼](https://github.com/Microsoft/BotBuilder/blob/master/specs/botframework-activity/botframework-activity.md#Id)。
  - 屬性名稱是 `activityId`。

- 通道識別碼
  - 使用 Application Insights 時，會從 `TelemetryBotIdInitializer` 將此屬性記錄為事件的屬性。  
  - 對應至 Bot Framework 通訊協定的[通道識別碼](https://github.com/Microsoft/BotBuilder/blob/master/specs/botframework-activity/botframework-activity.md#id)。
  - 所記錄的屬性名稱是 `channelId`。

- ActivityType 
  - 使用 Application Insights 時，會從 `TelemetryBotIdInitializer` 將此屬性記錄為事件的屬性。  
  - 對應至 Bot Framework 通訊協定的[活動類型](https://github.com/Microsoft/BotBuilder/blob/master/specs/botframework-activity/botframework-activity.md#type)。
  - 所記錄的屬性名稱是 `activityType`。

- 文字
  - 會在 `logPersonalInformation` 屬性設定為 `true` 時**選擇性地**記錄。
  - 對應至 Bot Framework 通訊協定的[活動文字](https://github.com/Microsoft/BotBuilder/blob/master/specs/botframework-activity/botframework-activity.md#text)欄位。
  - 所記錄的屬性名稱是 `text`。

- Speak

  - 會在 `logPersonalInformation` 屬性設定為 `true` 時**選擇性地**記錄。
  - 對應至 Bot Framework 通訊協定的[活動語音](https://github.com/Microsoft/BotBuilder/blob/master/specs/botframework-activity/botframework-activity.md#speak)欄位。
  - 所記錄的屬性名稱是 `speak`。

  - 

- FromId
  - 對應至 Bot Framework 通訊協定的[來源識別碼](https://github.com/Microsoft/BotBuilder/blob/master/specs/botframework-activity/botframework-activity.md#from)欄位。
  - 所記錄的屬性名稱是 `fromId`。

- FromName
  - 會在 `logPersonalInformation` 屬性設定為 `true` 時**選擇性地**記錄。
  - 對應至 Bot Framework 通訊協定的[來源名稱](https://github.com/Microsoft/BotBuilder/blob/master/specs/botframework-activity/botframework-activity.md#from)欄位。
  - 所記錄的屬性名稱是 `fromName`。

- RecipientId
  - 對應至 Bot Framework 通訊協定的[來源名稱](https://github.com/Microsoft/BotBuilder/blob/master/specs/botframework-activity/botframework-activity.md#from)欄位。
  - 所記錄的屬性名稱是 `fromName`。

- RecipientName
  - 對應至 Bot Framework 通訊協定的[來源名稱](https://github.com/Microsoft/BotBuilder/blob/master/specs/botframework-activity/botframework-activity.md#from)欄位。
  - 所記錄的屬性名稱是 `fromName`。

- ConversationId
  - 對應至 Bot Framework 通訊協定的[來源名稱](https://github.com/Microsoft/BotBuilder/blob/master/specs/botframework-activity/botframework-activity.md#from)欄位。
  - 所記錄的屬性名稱是 `fromName`。

- ConversationName
  - 對應至 Bot Framework 通訊協定的[來源名稱](https://github.com/Microsoft/BotBuilder/blob/master/specs/botframework-activity/botframework-activity.md#from)欄位。
  - 所記錄的屬性名稱是 `fromName`。

- 地區設定
  - 對應至 Bot Framework 通訊協定的[來源名稱](https://github.com/Microsoft/BotBuilder/blob/master/specs/botframework-activity/botframework-activity.md#from)欄位。
  - 所記錄的屬性名稱是 `fromName`。

### <a name="customevent-botmessagesend"></a>CustomEvent：BotMessageSend 
**記錄來源：** TelemetryLoggerMiddleware 

記錄於 Bot 傳送訊息時。

- UserID  ([從遙測初始設定式](https://aka.ms/telemetry-initializer))
- SessionID ([從遙測初始設定式](https://aka.ms/telemetry-initializer))
- ActivityID  ([從遙測初始設定式](https://aka.ms/telemetry-initializer))
- Channel  ([從遙測初始設定式](https://aka.ms/telemetry-initializer))
- ActivityType  ([從遙測初始設定式](https://aka.ms/telemetry-initializer))
- ReplyToID
- RecipientId
- ConversationName
- 地區設定
- RecipientName (PII 選用)
- 文字 (PII 選用)
- 語音 (PII 選用)


### <a name="customevent-botmessageupdate"></a>CustomEvent：BotMessageUpdate
**記錄來源：** TelemetryLoggerMiddleware 記錄於 Bot 更新訊息時 (罕見案例)
- UserID  ([從遙測初始設定式](https://aka.ms/telemetry-initializer))
- SessionID ([從遙測初始設定式](https://aka.ms/telemetry-initializer))
- ActivityID ([從遙測初始設定式](https://aka.ms/telemetry-initializer))
- Channel  ([從遙測初始設定式](https://aka.ms/telemetry-initializer))
- ActivityType  ([從遙測初始設定式](https://aka.ms/telemetry-initializer))
- RecipientId
- ConversationId
- ConversationName
- 地區設定
- 文字 (PII 選用)


### <a name="customevent-botmessagedelete"></a>CustomEvent：BotMessageDelete
**記錄來源：** TelemetryLoggerMiddleware 記錄於 Bot 刪除訊息時 (罕見案例)
- UserID  ([從遙測初始設定式](https://aka.ms/telemetry-initializer))
- SessionID ([從遙測初始設定式](https://aka.ms/telemetry-initializer))
- ActivityID  ([從遙測初始設定式](https://aka.ms/telemetry-initializer))
- Channel  ([從遙測初始設定式](https://aka.ms/telemetry-initializer))
- ActivityType  ([從遙測初始設定式](https://aka.ms/telemetry-initializer))
- RecipientId
- ConversationId
- ConversationName

### <a name="customevent-luisevent"></a>CustomEvent：LuisEvent
**記錄來源：** LuisRecognizer

記錄 LUIS 服務的結果。

- UserID  ([從遙測初始設定式](https://aka.ms/telemetry-initializer))
- SessionID ([從遙測初始設定式](https://aka.ms/telemetry-initializer))
- ActivityID ([從遙測初始設定式](https://aka.ms/telemetry-initializer))
- 通道 ([從遙測初始設定式](https://aka.ms/telemetry-initializer))
- ActivityType ([從遙測初始設定式](https://aka.ms/telemetry-initializer))
- ApplicationId
- 意圖
- IntentScore
- Intent2 
- IntentScore2 
- FromId
- SentimentLabel
- SentimentScore
- Entities (json 形式)
- 問題 (PII 選用)

## <a name="customevent-qnamessage"></a>CustomEvent：QnAMessage
**記錄來源：** QnAMaker

記錄 QnA Maker 服務的結果。

- UserID ([從遙測初始設定式](https://aka.ms/telemetry-initializer))
- SessionID ([從遙測初始設定式](https://aka.ms/telemetry-initializer))
- ActivityID ([從遙測初始設定式](https://aka.ms/telemetry-initializer))
- 通道 ([從遙測初始設定式](https://aka.ms/telemetry-initializer))
- ActivityType  ([從遙測初始設定式](https://aka.ms/telemetry-initializer))
- 使用者名稱 (PII 選用)
- 問題 (PII 選用)
- MatchedQuestion
- QuestionId
- Answer
- 分數
- ArticleFound

## <a name="querying-the-data"></a>查詢資料
使用 Application Insights 時，所有資料都會相互關聯 (即使是跨服務)。  我們可以查詢成功的要求，以及查看該要求的所有相關事件。  
下列查詢會告訴您最近的要求：
```sql
requests 
| where timestamp > ago(3d) 
| where resultCode == 200
| order by timestamp desc
| project timestamp, operation_Id, appName
| limit 10
```

從第一個查詢中，選取一些 `operation_Id`，然後尋找更多資訊：

```sql
let my_operation_id = "<OPERATION_ID>";
let union_all = () {
    union
    (traces | where operation_Id == my_operation_id),
    (customEvents | where operation_Id == my_operation_id),
    (requests | where operation_Id == my_operation_id),
    (dependencies | where operation_Id  == my_operation_id),
    (exceptions | where operation_Id == my_operation_id)
};
union_all
    | order by timestamp asc
    | project itemType, name, performanceBucket
```

這可提供單一要求的時間順序明細，以及每次呼叫的持續時間貯體。
![範例呼叫](media/performance_query.png)

> 注意："Activity" `customEvent` 事件時間戳記失序，因為這些事件是以非同步方式記錄。

## <a name="create-a-dashboard"></a>建立儀表板

最簡單的測試方法就是使用 [Azure 入口網站的範本部署頁面](https://portal.azure.com/#create/Microsoft.Template)建立儀表板。  
- 按一下 [在編輯器中建置您自己的範本]
- 複製並貼上為了協助您建立儀表板而提供的其中一個 .json 檔案：
  - [系統健康情況儀表板](https://aka.ms/system-health-appinsights)
  - [交談健康情況儀表板](https://aka.ms/conversation-health-appinsights)
- 按一下 [儲存]
- 填入 `Basics`： 
   - 訂用帳戶：<your test subscription>
   - 資源群組：<a test resource group>
   - 位置：<such as West US>
- 填入 `Settings`：
   - 見解元件名稱：<像是 `core672so2hw`>
   - 見解元件資源群組：<像是 `core67`>
   - 儀表板名稱：<像是 `'ConversationHealth'` 或 `SystemHealth`>
- 按一下 `I agree to the terms and conditions stated above`
- 按一下 `Purchase`
- 驗證
   - 按一下 [`Resource Groups`](https://ms.portal.azure.com/#blade/HubsExtension/Resources/resourceType/Microsoft.Resources%2Fsubscriptions%2FresourceGroups)
   - 從上面選取您的資源群組 (像是 `core67`)。
   - 如果您沒有看到新的資源，則查看 [部署] 並了解是否有任何失敗。
   - 以下是您通常看到的失敗：
     
```json
{"code":"DeploymentFailed","message":"At least one resource deployment operation failed. Please list deployment operations for details. Please see https://aka.ms/arm-debug for usage details.","details":[{"code":"BadRequest","message":"{\r\n \"error\": {\r\n \"code\": \"InvalidTemplate\",\r\n \"message\": \"Unable to process template language expressions for resource '/subscriptions/45d8a30e-3363-4e0e-849a-4bb0bbf71a7b/resourceGroups/core67/providers/Microsoft.Portal/dashboards/Bot Analytics Dashboard' at line '34' and column '9'. 'The template parameter 'virtualMachineName' is not found. Please see https://aka.ms/arm-template/#parameters for usage details.'\"\r\n }\r\n}"}]}
```

若要檢視資料，請移至 Azure 入口網站。 按一下左邊的 [儀表板]，然後從下拉式清單中選取您建立的儀表板。

## <a name="additional-resources"></a>其他資源
您可以參考這些實作遙測的範例：
- C#
  - [LUIS 搭配 AppInsights](https://aka.ms/luis-with-appinsights-cs)
  - [QnA 搭配 AppInsights](https://aka.ms/qna-with-appinsights-cs)
- JS
  - [LUIS 搭配 AppInsights](https://aka.ms/luis-with-appinsights-js)
  - [QnA 搭配 AppInsights](https://aka.ms/qna-with-appinsights-js)

