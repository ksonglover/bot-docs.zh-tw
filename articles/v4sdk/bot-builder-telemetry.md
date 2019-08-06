---
title: 將遙測資料新增至 Bot | Microsoft Docs
description: 了解如何整合 Bot 與新的遙測功能。
keywords: 遙測, appinsights, 監視 bot
author: WashingtonKayaker
ms.author: v-ivorb
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 07/17/2019
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: bd2de7055baf6a37323ad49ccd206c11e8829d9d
ms.sourcegitcommit: 3574fa4e79edf2a0c179d8b4a71939d7b5ffe2cf
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/27/2019
ms.locfileid: "68591043"
---
# <a name="add-telemetry-to-your-bot"></a>將遙測新增至 Bot

[!INCLUDE[applies-to](../includes/applies-to.md)]


Bot Framework SDK 4.2 版中已新增了遙測記錄功能。  此功能可讓聊天機器人應用程式將事件資料傳送給 [Application Insights](https://aka.ms/appinsights-overview) 之類的遙測服務。 遙測可讓您深入了解聊天機器人 (方法是顯示最常使用的功能)、偵測不必要的行為，以及讓您了解可用性、效能和使用方式。


在本文中，您會了解如何使用 Application Insights 將遙測實作到聊天機器人：

* 所需的程式碼，以便在聊天機器人中設定遙測，並連線至 Application Insights

* 在聊天機器人的[對話方塊](bot-builder-concept-dialog.md)中啟用遙測

* 讓遙測能夠從其他服務 (如 [LUIS](bot-builder-howto-v4-luis.md) 和 [QnA Maker](bot-builder-howto-qna.md)) 擷取使用方式資料。

* 在 Application Insights 中視覺呈現遙測資料

## <a name="prerequisites"></a>必要條件

* [CoreBot 程式碼範例](https://aka.ms/cs-core-sample)

* [Application Insights 程式碼範例](https://aka.ms/csharp-corebot-app-insights-sample)

* [Microsoft Azure](https://portal.azure.com/) 訂用帳戶

* [Application Insights 金鑰](../bot-service-resources-app-insights-keys.md)

* 熟悉 [Application Insights](https://aka.ms/appinsights-overview)

> [!NOTE]
> [Application Insights 程式碼範例](https://aka.ms/csharp-corebot-app-insights-sample)是以 [CoreBot 程式碼範例](https://aka.ms/cs-core-sample)為基礎來建置的。 本文會逐步引導您修改 CoreBot 程式碼範例以併入遙測。 如果您在 Visual Studio 中跟著操作，等到結束時就會擁有 Application Insights 程式碼範例。

## <a name="wiring-up-telemetry-in-your-bot"></a>在聊天機器人中設定遙測

我們會從 [CoreBot 應用程式範例](https://aka.ms/cs-core-sample)開始，然後新增所需程式碼以將遙測整合到任何聊天機器人。 這會讓 Application Insights 能夠開始追蹤要求。

1. 在 Visual Studio 中開啟 [CoreBot 應用程式範例](https://aka.ms/cs-core-sample)

2. 新增以下 NuGet 套件。 如需如何使用 NuGet 的詳細資訊，請參閱[在 Visual Studio 中安裝和管理套件](https://aka.ms/install-manage-packages-vs)：
    * `Microsoft.ApplicationInsights`
    * `Microsoft.Bot.Builder.ApplicationInsights`
    * `Microsoft.Bot.Builder.Integration.ApplicationInsights.Core`

3. 在 `Startup.cs` 中納入下列陳述式︰
    ```csharp
    using Microsoft.ApplicationInsights.Extensibility;
    using Microsoft.Bot.Builder.ApplicationInsights;
    using Microsoft.Bot.Builder.Integration.ApplicationInsights.Core;
    using Microsoft.Bot.Builder.Integration.AspNet.Core;
    ```

    注意：如果您跟著操作，更新了 CoreBot 程式碼範例，您將會注意到 `Microsoft.Bot.Builder.Integration.AspNet.Core` 的 using 陳述式早已存在於 CoreBot 範例中。

4. 在 `Startup.cs` 的 `ConfigureServices()` 方法中納入下列程式碼。 這會透過[相依性插入 (DI)](https://aka.ms/asp.net-core-dependency-interjection) 而讓遙測服務可供聊天機器人使用：
    ```csharp
    // This method gets called by the runtime. Use this method to add services to the container.
    public void ConfigureServices(IServiceCollection services)
    {
        ...

        // Create the Bot Framework Adapter with error handling enabled.
        services.AddSingleton<IBotFrameworkHttpAdapter, AdapterWithErrorHandler>();

        // Add Application Insights services into service collection
        services.AddApplicationInsightsTelemetry();

        // Create the telemetry client.
        services.AddSingleton<IBotTelemetryClient, BotTelemetryClient>();

        // Add ASP middleware to store the http body mapped with bot activity key in the httpcontext.items. This will be picked by the TelemetryBotIdInitializer
        services.AddTransient<TelemetrySaveBodyASPMiddleware>();

        // Add telemetry initializer that will set the correlation context for all telemetry items.
        services.AddSingleton<ITelemetryInitializer, OperationCorrelationTelemetryInitializer>();

        // Add telemetry initializer that sets the user ID and session ID (in addition to other bot-specific properties such as activity ID)
        services.AddSingleton<ITelemetryInitializer, TelemetryBotIdInitializer>();

        // Create the telemetry middleware to track conversation events
        services.AddSingleton<IMiddleware, TelemetryLoggerMiddleware>();

        ...
    }
    ```
    
    注意：如果您跟著操作，更新了 CoreBot 程式碼範例，您將會注意到 `services.AddSingleton<IBotFrameworkHttpAdapter, AdapterWithErrorHandler>();` 早已存在 

5. 在 `Startup.cs` 的 `Configure()` 方法中新增 `UseBotApplicationInsights()` 方法呼叫。 這可讓聊天機器人在 HTTP 內容中儲存所需的聊天機器人特有屬性，以供遙測初始設定式在追蹤事件時擷取：

    ```csharp
    // This method gets called by the runtime. Use this method to configure the HTTP request pipeline.
    public void Configure(IApplicationBuilder app, IHostingEnvironment env)
    {
        ...

        app.UseBotApplicationInsights();
    }
    ```
6. 指示配接器使用已新增至 `ConfigureServices()` 方法的中介軟體程式碼。 方法是在 `AdapterWithErrorHandler.cs` 中搭配使用建構函式參數清單中的參數 IMiddleware middleware 以及建構函式中的 `Use(middleware);` 陳述式，如下所示：
    ```csharp
    public AdapterWithErrorHandler(ICredentialProvider credentialProvider, ILogger<BotFrameworkHttpAdapter> logger, IMiddleware middleware, ConversationState conversationState = null)
            : base(credentialProvider)
    {
        ...

        Use(middleware);
    }
    ```
7. 在 `appsettings.json` 檔案中新增 Application Insights 檢測金鑰。`appsettings.json` 檔案包含有關聊天機器人在執行時所用外部服務的中繼資料。 例如，CosmosDB、Application Insights 和 Language Understanding (LUIS) 服務連線和中繼資料都會儲存在該處。 對 `appsettings.json` 檔案新增的內容必須採用下列格式：

    ```json
    {
        "MicrosoftAppId": "",
        "MicrosoftAppPassword": "",
        "ApplicationInsights": {
            "InstrumentationKey": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx"
        }
    }
    ```
    注意：如需如何取得 _Application Insights 檢測金鑰_的詳細資訊，請參閱 [Application Insights 金鑰](../bot-service-resources-app-insights-keys.md)一文。

至此，您已完成使用 Application Insights 來啟用遙測的初步工作。  您可以使用聊天機器人模擬器在本機執行聊天機器人，然後進入 Application Insights 來查看所記錄的內容，例如回應時間、整體應用程式健康情況和一般執行資訊。 

接下來，我們會看看需要納入哪些內容才能將遙測功能新增至對話方塊中。 此功能可讓您取得額外資訊，例如會執行的對話方塊以及每個對話的相關統計資料。

## <a name="enabling-telemetry-in-your-bots-dialogs"></a>在聊天機器人的對話方塊中啟用遙測

若要取得有關對話的內建遙測資訊，您必須將遙測用戶端新增至每個對話方塊。 請遵循下列步驟來更新 CoreBot 範例：

1.  在 `MainDialog.cs` 中，您必須將新的 TelemetryClient 欄位新增至 `MainDialog` 類別，然後更新建構函式參數清單以納入 `IBotTelemetryClient` 參數，再將該參數傳遞給 `AddDialog()` 方法的每個呼叫。


    * 將參數 `IBotTelemetryClient telemetryClient` 新增至 MainDialog 類別的建構函式，然後將其指派給 `TelemetryClient` 欄位：

        ```csharp
        public MainDialog(IConfiguration configuration, ILogger<MainDialog> logger, IBotTelemetryClient telemetryClient)
            : base(nameof(MainDialog))
        {
            TelemetryClient = telemetryClient;

            ...

        }
        ```

    * 使用 `AddDialog` 方法新增每個對話方塊時，請設定 `TelemetryClient` 值：

        ```cs
        public MainDialog(IConfiguration configuration, ILogger<MainDialog> logger, IBotTelemetryClient telemetryClient)
            : base(nameof(MainDialog))
        {
            ...

            AddDialog(new TextPrompt(nameof(TextPrompt))
            {
                TelemetryClient = telemetryClient,
            });

            ...
        }
        ```

    * 瀑布式對話方塊會產生與其他對話方塊無關的事件。 在瀑布式步驟清單之後設定 `TelemetryClient` 屬性：

        ```csharp
        // The IBotTelemetryClient to the WaterfallDialog
        AddDialog(new WaterfallDialog(
            nameof(WaterfallDialog),
            new WaterfallStep[]
        {
            IntroStepAsync,
            ActStepAsync,
            FinalStepAsync,
        })
        {
            TelemetryClient = telemetryClient,
        });

        ```

2. 在 `DialogExtensions.cs` 中，您必須在 `Run()` 方法中設定 `dialogSet` 物件的 `TelemetryClient` 屬性：


    ```csharp
    public static async Task Run(this Dialog dialog, ITurnContext turnContext, IStatePropertyAccessor<DialogState> accessor, CancellationToken cancellationToken = default(CancellationToken))
    {
        ...

        dialogSet.TelemetryClient = dialog.TelemetryClient;

        ...
        
    }

    ```

3. 在 `BookingDialog.cs` 中，使用和更新 `MainDialog.cs` 時所用的相同程式來啟用此檔案中所新增四個對話方塊的遙測。 別忘了將 `TelemetryClient` 欄位新增至 BookingDialog 類別，並將 `IBotTelemetryClient telemetryClient` 參數新增至 BookingDialog 建構函式。


> [!TIP] 
> 如果您跟著操作並更新 CoreBot 程式碼範例，請在遇到任何問題時參閱 [Application Insights程式碼範例](https://aka.ms/csharp-corebot-app-insights-sample)。

這就是將遙測新增至聊天機器人對話方塊的全部過程了，至此，如果您執行了聊天機器人，應該就會看到系統將資料記錄到 Application Insights 中，但如果您有任何整合技術 (例如 LUIS 和 QnA Maker)，您還必須將 `TelemetryClient` 新增至該程式碼。


## <a name="enabling-telemetry-to-capture-usage-data-from-other-services-like-luis-and-qna-maker"></a>讓遙測能夠從其他服務 (如 LUIS 和 QnA Maker) 擷取使用方式資料

接下來，我們會在 LUIS 服務中實作遙測功能。 LUIS 服務有內建遙測記錄可供使用，因此您只需進行一些操作就可以開始從 LUIS 取得遙測資料。  

在此範例中，我們只需要提供遙測用戶端，方法就如同我們對對話方塊所做的一樣。 

1. `LuisHelper.cs` 中的 `ExecuteLuisQuery()` 方法需要 `IBotTelemetryClient telemetryClient`  參數：

    ```cs
    public static async Task<BookingDetails> ExecuteLuisQuery(IBotTelemetryClient telemetryClient, IConfiguration configuration, ILogger logger, ITurnContext turnContext, CancellationToken cancellationToken)
    ```

2. `LuisPredictionOptions` 類別可讓您提供選擇性參數供 LUIS 預測要求使用。  若要啟用遙測，則在 `LuisHelper.cs` 中建立 `luisPredictionOptions` 物件時，您必須設定 `TelemetryClient` 參數：

    ```cs
    var luisPredictionOptions = new LuisPredictionOptions()
    {
        TelemetryClient = telemetryClient,
    };

    var recognizer = new LuisRecognizer(luisApplication, luisPredictionOptions);
    ```

3. 最後一個步驟是將遙測用戶端傳遞給 `MainDialog.cs` 的 `ActStepAsync()` 方法中的 `ExecuteLuisQuery` 呼叫：

    ```cs
    await LuisHelper.ExecuteLuisQuery(TelemetryClient, Configuration, Logger, stepContext.Context, cancellationToken)
    ```

這樣就行了，您應該會擁有功能性聊天機器人，可將遙測資料記錄到 Application Insights。 您可以使用 [Microsoft Bot Framework Emulator](https://aka.ms/bot-framework-emulator-readme) 在本機執行聊天機器人。 聊天機器人的行為應該不會有任何變更，但會開始將資訊記錄到 Application Insights 中。 請傳送多則訊息來與聊天機器人互動，在下一節中，我們會在 Application Insights 中檢閱遙測結果。

如需聊天機器人的測試和偵測相關資訊，請參閱下列文章：

 * [對 Bot 進行偵錯](../bot-service-debug-bot.md)
 * [測試和偵錯指導方針](bot-builder-testing-debugging.md)
 * [使用模擬器進行偵錯](../bot-service-debug-emulator.md)


## <a name="visualizing-your-telemetry-data-in-application-insights"></a>在 Application Insights 中視覺呈現遙測資料
Application Insights 可監視聊天機器人應用程式的可用性、效能及使用方式 (不論應用程式是裝載在雲端還是內部部署環境)。 它會利用 Azure 監視器中強大的資料分析平台，為您提供應用程式作業的深入解析以及診斷錯誤，毋需等待使用者回報錯誤。 有幾種方式可以查看 Application Insights 所收集的遙測資料，其中兩種主要方式是透過查詢和儀表板。 

### <a name="querying-your-telemetry-data-in-application-insights-using-kusto-queries"></a>使用 Kusto 查詢在 Application Insights 中查詢遙測資料
請以本節作為起點，以了解如何在 Application Insights 中使用記錄查詢。 本節內容會示範兩個實用查詢，並提供其他有額外資訊的文件連結。

查詢資料

1. 移至 [Azure 入口網站](https://portal.azure.com)
2. 瀏覽至 Application Insights。 若要這麼做，最簡單的方法是按一下 [監視] > [應用程式]  ，即可在其中找到。 
3. 進入 Application Insights 後，您可以按一下瀏覽列上的 [記錄 (分析)]  。

    ![記錄 (分析)](media/AppInsights-LogView.png)

4. 這會開啟 [查詢] 視窗。  輸入下列查詢，然後選取 [執行]  ：

    ```sql
    customEvents
    | where name=="WaterfallStart"
    | extend DialogId = customDimensions['DialogId']
    | extend InstanceId = tostring(customDimensions['InstanceId'])
    | join kind=leftouter (customEvents | where name=="WaterfallComplete" | extend InstanceId = tostring(customDimensions['InstanceId'])) on InstanceId    
    | summarize starts=countif(name=='WaterfallStart'), completes=countif(name1=='WaterfallComplete') by bin(timestamp, 1d), tostring(DialogId)
    | project Percentage=max_of(0.0, completes * 1.0 / starts), timestamp, tostring(DialogId) 
    | render timechart
    ```
5. 這會傳回已執行完成的瀑布式對話方塊百分比。

    ![記錄 (分析)](media/AppInsights-Query-PercentCompleteDialog.png)


> [!TIP]
> 您可以選取 [記錄 (分析)]  刀鋒視窗右上方的按鈕，來將任何查詢釘選到 Application Insights 儀表板。 只要選取要作為釘選目的地的儀表板，下一次造訪該儀表板時就可以使用該查詢。


## <a name="the-application-insights-dashboard"></a>Application Insights 儀表板

無論何時，只要您在 Azure 中建立 Application Insights 資源，系統就會自動建立新的儀表板，並讓此儀表板與該資源相關聯。  您可以選取 [Application Insights] 刀鋒視窗頂端標示為**應用程式儀表板**的按鈕來查看該儀表板。 

![應用程式儀表板連結](media/Application-Dashboard-Link.png)


或者，若要檢視資料，也可移至 Azure 入口網站。 按一下左邊的 [儀表板]  ，然後從下拉式清單中選取您想要的儀表板。

在其中，您會看到一些關於聊天機器人效能的預設資訊，以及您已釘選到儀表板的其他查詢。



## <a name="additional-information"></a>其他資訊

* [什麼是 Application Insights？](https://aka.ms/appinsights-overview)

* [在 Application Insights 中使用搜尋](https://aka.ms/search-in-application-insights)

* [使用 Azure Application Insights 建立自訂 KPI 儀表板](https://aka.ms/custom-kpi-dashboards-application-insights)


<!--
The easiest way to test is by creating a dashboard using [Azure portal's template deployment page](https://portal.azure.com/#create/Microsoft.Template).
- Click ["Build your own template in the editor"]
- Copy and paste either one of these .json file that is provided to help you create the dashboard:
  - [System Health Dashboard](https://aka.ms/system-health-appinsights)
  - [Conversation Health Dashboard](https://aka.ms/conversation-health-appinsights)
- Click "Save"
- Populate `Basics`: 
   - Subscription: <your test subscription>
   - Resource group: <a test resource group>
   - Location: <such as West US>
- Populate `Settings`:
   - Insights Component Name: <like `core672so2hw`>
   - Insights Component Resource Group: <like `core67`>
   - Dashboard Name:  <like `'ConversationHealth'` or `SystemHealth`>
- Click `I agree to the terms and conditions stated above`
- Click `Purchase`
- Validate
   - Click on [`Resource Groups`](https://ms.portal.azure.com/#blade/HubsExtension/Resources/resourceType/Microsoft.Resources%2Fsubscriptions%2FresourceGroups)
   - Select your Resource Group from above (like `core67`).
   - If you don't see a new Resource, then look at "Deployments" and see if any have failed.
   - Here's what you typically see for failures:
     
```json
{"code":"DeploymentFailed","message":"At least one resource deployment operation failed. Please list deployment operations for details. Please see https://aka.ms/arm-debug for usage details.","details":[{"code":"BadRequest","message":"{\r\n \"error\": {\r\n \"code\": \"InvalidTemplate\",\r\n \"message\": \"Unable to process template language expressions for resource '/subscriptions/45d8a30e-3363-4e0e-849a-4bb0bbf71a7b/resourceGroups/core67/providers/Microsoft.Portal/dashboards/Bot Analytics Dashboard' at line '34' and column '9'. 'The template parameter 'virtualMachineName' is not found. Please see https://aka.ms/arm-template/#parameters for usage details.'\"\r\n }\r\n}"}]}
```
-->









<!--
## Additional information

### Customize your telemetry client

If you want to customize your telemetry to log into a separate service, you have to configure the system differently. If using Application Insights to do so, download the package `Microsoft.Bot.Builder.ApplicationInsights` via NuGet, or use npm to install `botbuilder-applicationinsights`. Details on getting the Application Insights keys can be found [here](../bot-service-resources-app-insights-keys.md). Otherwise, include what is necessary for logging to that service, then follow the section below.

**Use Custom Telemetry**
If you want to log telemetry events generated by the Bot Framework into a completely separate system, create a new class derived from the base interface `IBotTelemetryClient` and configure. Then, when adding your telemetry client as above, just inject your custom client. 

```csharp
public void ConfigureServices(IServiceCollection services)
{
    ...
    // Add the telemetry client.
    services.AddSingleton<IBotTelemetryClient, CustomTelemetryClient>();
    ...
}
```

### Add custom logging to your bot

Once the Bot has the new telemetry logging support configured, you can begin adding telemetry to your bot.  The `BotTelemetryClient`(in C#, `IBotTelemetryClient`) has several methods to log distinct types of events.  Choosing the appropriate type of event enables you to take advantage of Application Insights existing reports (if you are using Application Insights).  For general scenarios `TraceEvent` is typically used.  The data logged using `TraceEvent` lands in the `CustomEvent` table in Kusto.

If using a Dialog within your Bot, every Dialog-based object (including Prompts) will contain a new `TelemetryClient` property.  This is the `BotTelemetryClient` that enables you to perform logging.  This is not just a convenience, we'll see later in this article if this property is set, `WaterfallDialogs` will generate events.

### Details of telemetry options

There are three main components available for your bot to log telemetry, and each component has customization available for logging your own events, which are discussed in this section. 

- A  [Bot Framework Middleware component](#telemetry-middleware) (*TelemetryLoggerMiddleware*) that will log when messages are received, sent, updated or deleted. You can override for custom logging.
- [*LuisRecognizer* class.](#telemetry-support-luis)  You can override for custom logging in two ways - per invocation (add/replace properties) or derived classes.
- [*QnAMaker*  class.](#telemetry-qnamaker)  You can override for custom logging in two ways - per invocation (add/replace properties) or derived classes.

All components log using the `IBotTelemetryClient`  (or `BotTelemetryClient` in node.js) interface which can be overridden with a custom implementation.


#### Telemetry Middleware

|C#  | JavaScript |
|:-----|:------------|
|**Microsoft.Bot.Builder.TelemetryLoggerMiddleware** | **botbuilder-core** |

##### Out of box usage

The TelemetryLoggerMiddleware is a Bot Framework component that can be added without modification, and it will perform logging that enables out of the box reports that ship with the Bot Framework SDK. 

```csharp
// Create the telemetry middleware to track conversation events
services.AddSingleton<IMiddleware, TelemetryLoggerMiddleware>();

// Create the Bot Framework Adapter with error handling enabled.
services.AddSingleton<IBotFrameworkHttpAdapter, AdapterWithErrorHandler>();
```

And in our adapter, we would specify the use of middleware:

```csharp
public AdapterWithErrorHandler(ICredentialProvider credentialProvider, ILogger<BotFrameworkHttpAdapter> logger, IMiddleware middleware, ConversationState conversationState = null)
           : base(credentialProvider)
{
    ...
    Use(middleware);
    ...
}
```

##### Adding properties
If you decide to add additional properties, the TelemetryLoggerMiddleware class can be derived.  For example, if you would like to add the property "MyImportantProperty" to the `BotMessageReceived` event.  `BotMessageReceived` is logged when the user sends a message to the bot.  Adding the additional property can be accomplished in the following way:

```csharp
class MyTelemetryMiddleware : TelemetryLoggerMiddleware
{
    ...
    protected override Task OnReceiveActivityAsync(
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

In Startup, we would add the new class:

```csharp
// Create the telemetry middleware to track conversation events
services.AddSingleton<IMiddleware, MyTelemetryMiddleware>();
```

##### Completely replacing properties / Additional event(s)

If you decide to completely replace properties being logged, the `TelemetryLoggerMiddleware` class can be derived (like above when extending properties).   Similarly, logging new events is performed in the same way.

For example, if you would like to completely replace the`BotMessageSend` properties and send multiple events, the following demonstrates how this could be performed:

```csharp
class MyTelemetryMiddleware : TelemetryLoggerMiddleware
{
    ...
    protected override Task OnReceiveActivityAsync(
                  Activity activity,
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
Note: When the standard properties are not logged, it will cause the out of box reports shipped with the product to stop working.

##### Events Logged from Telemetry Middleware
[BotMessageSend](bot-builder-telemetry-reference.md#customevent-botmessagesend)
[BotMessageReceived](bot-builder-telemetry-reference.md#customevent-botmessagereceived)
[BotMessageUpdate](bot-builder-telemetry-reference.md#customevent-botmessageupdate)
[BotMessageDelete](bot-builder-telemetry-reference.md#customevent-botmessagedelete)

#### Telemetry support LUIS 

|C#  | JavaScript |
|:-----|:------------|
| **Microsoft.Bot.Builder.AI.Luis** | **botbuilder-ai** |

##### Out of box usage
The LuisRecognizer is an existing Bot Framework component, and telemetry can be enabled by passing a IBotTelemetryClient interface through `luisOptions`.  You can override the default properties being logged and log new events as required.

During construction of `luisOptions`, an `IBotTelemetryClient` object must be provided for this to work.

```csharp
var luisPredictionOptions = new LuisPredictionOptions()
{
    TelemetryClient = telemetryClient
};
var recognizer = new LuisRecognizer(luisApplication, luisPredictionOptions);
```

##### Adding properties
If you decide to add additional properties, the `LuisRecognizer` class can be derived.  For example, if you would like to add the property "MyImportantProperty" to the `LuisResult` event.  `LuisResult` is logged when a LUIS prediction call is performed.  Adding the additional property can be accomplished in the following way:

```csharp
class MyLuisRecognizer : LuisRecognizer 
{
   ...
   protected override Task OnRecognizerResultAsync(
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

##### Add properties per invocation
Sometimes it's necessary to add additional properties during the invocation:
```csharp
var additionalProperties = new Dictionary<string, string>
{
   { "dialogId", "myDialogId" },
   { "conversationInfo", "myConversationInfo" },
};

var result = await recognizer.RecognizeAsync(turnContext,
     additionalProperties).ConfigureAwait(false);
```

##### Completely replacing properties / Additional event(s)
If you decide to completely replace properties being logged, the `LuisRecognizer` class can be derived (like above when extending properties).   Similarly, logging new events is performed in the same way.

For example, if you would like to completely replace the`LuisResult` properties and send multiple events, the following demonstrates how this could be performed:

```csharp
class MyLuisRecognizer : LuisRecognizer
{
    ...
    protected override Task OnRecognizerResultAsync(
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
Note: When the standard properties are not logged, it will cause the Application Insights out of box reports shipped with the product to stop working.

##### Events Logged from TelemetryLuisRecognizer
[LuisResult](bot-builder-telemetry-reference.md#customevent-luisevent)

### Telemetry QnAMaker

|C#  | JavaScript |
|:-----|:------------|
| **Microsoft.Bot.Builder.AI.QnA** | **botbuilder-ai** |


##### Out of box usage
The QnAMaker class is an existing Bot Framework component that adds two additional constructor parameters which enable logging that enable out of the box reports that ship with the Bot Framework SDK. The new `telemetryClient` references a `IBotTelemetryClient` interface which performs the logging.  

```csharp
var qna = new QnAMaker(endpoint, options, client, 
                       telemetryClient: telemetryClient,
                       logPersonalInformation: true);
```
##### Adding properties 
If you decide to add additional properties, there are two methods of doing this - when properties need to be added during the QnA call to retrieve answers or deriving from the `QnAMaker` class.  

The following demonstrates deriving from the `QnAMaker` class.  The example shows adding the property "MyImportantProperty" to the `QnAMessage` event.  The`QnAMessage` event is logged when a QnA `GetAnswers`call is performed.  In addition, we log a second event "MySecondEvent".

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

##### Adding properties during GetAnswersAsync
If you have properties that need to be added during runtime, the `GetAnswersAsync` method can provide properties and/or metrics to add to the event.

For example, if you want to add a `dialogId` to the event, it can be done like the following:
```csharp
var telemetryProperties = new Dictionary<string, string>
{
   { "dialogId", myDialogId },
};

var results = await qna.GetAnswersAsync(context, opts, telemetryProperties);
```
The `QnaMaker` class provides the capability of overriding properties, including PersonalInfomation properties.

##### Completely replacing properties / Additional event(s)
If you decide to completely replace properties being logged, the `TelemetryQnAMaker` class can be derived (like above when extending properties).   Similarly, logging new events is performed in the same way.

For example, if you would like to completely replace the`QnAMessage` properties, the following demonstrates how this could be performed:

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
Note: When the standard properties are not logged, it will cause the out of box reports shipped with the product to stop working.

##### Events Logged from TelemetryLuisRecognizer
[QnAMessage](bot-builder-telemetry-reference.md#customevent-qnamessage)

### All other events

A full list of events logged for your bot's telemetry can be found on the [telemetry reference page](bot-builder-telemetry-reference.md).

#### Identifiers and Custom Events

When logging events into Application Insights, the events generated contain default properties that you won't have to fill.  For example, `user_id` and `session_id`properties are contained in each Custom Event (generated with the `TraceEvent` API).  In addition, `activitiId`, `activityType` and `channelId` are also added.

> [!NOTE]
> Custom telemetry clients will not be provided these values.

Property |Type | Details
--- | --- | ---
`user_id`| `string` | [ChannelID](https://github.com/Microsoft/BotBuilder/blob/master/specs/botframework-activity/botframework-activity.md#channel-id) + [From.Id](https://github.com/Microsoft/BotBuilder/blob/master/specs/botframework-activity/botframework-activity.md#from)
`session_id`| `string`|  [ConversationID](https://github.com/Microsoft/BotBuilder/blob/master/specs/botframework-activity/botframework-activity.md#conversation)
`customDimensions.activityId`| `string` | [The bot activity ID](https://github.com/Microsoft/BotBuilder/blob/master/specs/botframework-activity/botframework-activity.md#id)
`customDimensions.activityType` | `string` | [The bot activity type](https://github.com/Microsoft/BotBuilder/blob/master/specs/botframework-activity/botframework-activity.md#channel-id)
`customDimensions.channelId` | `string` |  [The bot activity channel ID](https://github.com/Microsoft/BotBuilder/blob/master/specs/botframework-activity/botframework-activity.md#channel-id)
-->
