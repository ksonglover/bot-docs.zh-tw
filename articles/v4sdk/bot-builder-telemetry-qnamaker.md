---
title: 將遙測資料新增至 QnA Bot | Microsoft Docs
description: 了解如何將新的遙測功能整合到您已啟用 QnA Maker 的 Bot。
keywords: 遙測, appinsights, Application Insights, 監視器 Bot, QnA Maker
author: WashingtonKayaker
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 07/31/2019
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 27ce4315af7703e23b9a63308abb47300d9a29bd
ms.sourcegitcommit: 08f9dc91152e0d4565368f72f547cdea1885af89
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 11/26/2019
ms.locfileid: "74528558"
---
# <a name="add-telemetry-to-your-qnamaker-bot"></a>將遙測新增至您的 QnAMaker Bot

[!INCLUDE[applies-to](../includes/applies-to.md)]


Bot Framework SDK 4.2 版中已新增了遙測記錄功能。  此功能可讓聊天機器人應用程式將事件資料傳送給 [Application Insights](https://aka.ms/appinsights-overview) 之類的遙測服務。 遙測可讓您深入了解聊天機器人 (方法是顯示最常使用的功能)、偵測不必要的行為，以及讓您了解可用性、效能和使用方式。

Bot Framework SDK 中已新增下列兩個新元件，可在啟用 QnA Maker 的 Bot 中啟用遙測記錄：`TelemetryLoggerMiddleware` 和 `QnAMaker` 類別。 `TelemetryLoggerMiddleware` 是中介軟體元件，可在每次接收、傳送、更新或刪除訊息時記錄資料，而 'QnAMaker' 類別則會提供可延伸遙測功能的自訂記錄。

在本文中，您將了解：

* 在您 Bot 中連結遙測資料所需的程式碼 

* 必須使用哪些程式碼，才能啟用現成 QnA 記錄和使用標準事件屬性的報表。 

* 修改或擴充 SDK 的預設事件屬性，以滿足各種不同的報告需求。


## <a name="prerequisites"></a>必要條件

* [QnA Maker 範例程式碼](https://aka.ms/cs-qna)

* [Microsoft Azure](https://portal.azure.com/) 訂用帳戶

* [Application Insights 金鑰](../bot-service-resources-app-insights-keys.md)

* 熟悉 [QnA Maker](https://qnamaker.ai/) 很有幫助。

* [QnA Maker](https://aka.ms/create-qna-maker) 帳戶。

* 已發佈的 QnA Maker 知識庫。 如果您沒有，請依照[建立知識庫並回應](https://aka.ms/create-publish-query-in-portal)教學課程中的步驟，建立含有問題和解答的 QnA Maker 知識庫。

> [!NOTE]
> 本文將以 [QnA Maker 範例程式碼](https://aka.ms/cs-qna)作為基礎，逐步解說合併遙測所需的步驟。 

## <a name="wiring-up-telemetry-in-your-qna-maker-bot"></a>在 QnA Maker Bot 中連結遙測

我們會從 [QnA Maker 應用程式範例](https://aka.ms/cs-qna)開始，然後新增所需程式碼以將遙測整合到使用 QnA 服務的 Bot。 這會讓 Application Insights 能夠開始追蹤要求。

1. 在 Visual Studio 中開啟 [QnA Maker 應用程式範例](https://aka.ms/cs-qna)

2. 新增 `Microsoft.Bot.Builder.Integration.ApplicationInsights.Core ` NuGet 套件。 如需如何使用 NuGet 的詳細資訊，請參閱[在 Visual Studio 中安裝和管理套件](https://aka.ms/install-manage-packages-vs)：

3. 在 `Startup.cs` 中納入下列陳述式︰
    ```csharp
    using Microsoft.ApplicationInsights.Extensibility;
    using Microsoft.Bot.Builder.ApplicationInsights;
    using Microsoft.Bot.Builder.Integration.ApplicationInsights.Core;
    ```

    > [!NOTE] 
    > 如果您跟著操作，更新了 QnA Maker 程式碼範例，您將會注意到 `Microsoft.Bot.Builder.Integration.AspNet.Core` 的 using 陳述式早已存在於 QnA Maker 範例中。

4. 將下列程式碼新增至 `Startup.cs` 中的 `ConfigureServices()` 方法。 這會透過[相依性插入 (DI)](https://aka.ms/asp.net-core-dependency-interjection) 而讓遙測服務可供 Bot 使用：
    ```csharp
    // This method gets called by the runtime. Use this method to add services to the container.
    public void ConfigureServices(IServiceCollection services)
    {
        ...
        // Create the Bot Framework Adapter with error handling enabled.
        services.AddSingleton<IBotFrameworkHttpAdapter, AdapterWithErrorHandler>();

        // Add Application Insights services into service collection
        services.AddApplicationInsightsTelemetry();

        // Add the standard telemetry client
        services.AddSingleton<IBotTelemetryClient, BotTelemetryClient>();

        // Create the telemetry middleware to track conversation events
        services.AddSingleton<TelemetryLoggerMiddleware>();

        // Add the telemetry initializer middleware
        services.AddSingleton<IMiddleware, TelemetryInitializerMiddleware>();

        // Add telemetry initializer that will set the correlation context for all telemetry items
        services.AddSingleton<ITelemetryInitializer, OperationCorrelationTelemetryInitializer>();

        // Add telemetry initializer that sets the user ID and session ID (in addition to other bot-specific properties, such as activity ID)
        services.AddSingleton<ITelemetryInitializer, TelemetryBotIdInitializer>();
        ...
    }
    ```
    
    > [!NOTE] 
    > 如果您跟著操作，更新了 QnA Maker 程式碼範例，將會注意到 `services.AddSingleton<IBotFrameworkHttpAdapter, AdapterWithErrorHandler>();` 早已存在。

5. 指示配接器使用已新增至 `ConfigureServices()` 方法的中介軟體程式碼。 開啟 `AdapterWithErrorHandler.cs`，並將 `IMiddleware middleware` 新增至建構函式的參數清單。 新增 `Use(middleware);` 陳述式作為建構函式中的最後一行：
    ```csharp
    public AdapterWithErrorHandler(ICredentialProvider credentialProvider, ILogger<BotFrameworkHttpAdapter> logger, IMiddleware middleware, ConversationState conversationState = null)
            : base(credentialProvider)
    {
        ...

        Use(middleware);
    }
    ```

6. 在您的 `appsettings.json` 檔案中新增 Application Insights 檢測金鑰。 `appsettings.json` 檔案包含有關 Bot 在執行時所用外部服務的中繼資料。 例如，CosmosDB、Application Insights 和 QnA Maker 服務連線及中繼資料都會儲存在該處。 對 `appsettings.json` 檔案新增的內容必須採用下列格式：

    ```json
    {
        "MicrosoftAppId": "",
        "MicrosoftAppPassword": "",
        "QnAKnowledgebaseId": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
        "QnAEndpointKey": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
        "QnAEndpointHostName": "https://xxxxxxxx.azurewebsites.net/qnamaker",
        "ApplicationInsights": {
            "InstrumentationKey": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx"
        }
    }
    ```
   > [!Note] 
   > 
   > * 如需如何取得 _Application Insights 檢測金鑰_的詳細資訊，請參閱 [Application Insights 金鑰](../bot-service-resources-app-insights-keys.md)一文。
    >
    > * 您應該已擁有 [QnA Maker 帳戶](https://aka.ms/create-qna-maker)，如有需要，您可以在[這裡](https://aka.ms/bot-framework-emulator-qna-keys)找到關於取得 QnA 知識庫識別碼、端點金鑰和主機名稱值的資訊。
    > 


至此，您已完成使用 Application Insights 來啟用遙測的初步工作。  您可以使用聊天機器人模擬器在本機執行聊天機器人，然後進入 Application Insights 來查看所記錄的內容，例如回應時間、整體應用程式健康情況和一般執行資訊。 

> [!TIP] 
> 如需啟用/停用活動事件和個人資訊記錄的詳細資訊，請參閱[將遙測新增至您的 Bot](bot-builder-telemetry.md#enabling-or-disabling-activity-logging)

接下來，我們會看看需要納入哪些內容才能將遙測功能新增至 QnA Maker 服務。 


## <a name="enabling-telemetry-to-capture-usage-data-from-the-qna-maker-service"></a>讓遙測能夠從 QnA Maker 服務擷取使用方式資料

QnA Maker 服務有內建遙測記錄可供使用，因此您只需進行一些操作就可以開始從 QnA Maker 取得遙測資料。  首先，我們會了解如何將遙測併入 QnA Maker 程式碼中，以啟用內建的遙測記錄，然後我們將了解如何將其他屬性取代或新增到現有的事件資料，以滿足各種不同的報告需求。

### <a name="enabling-default-qna-logging"></a>啟用預設的 QnA 記錄

1. 在 `QnABot.cs` 的 `QnABot` 類別中，建立類型為 `IBotTelemetryClient` 的私用唯讀欄位：

    ```cs
    public class QnABot : ActivityHandler
        {
            private readonly IBotTelemetryClient _telemetryClient;
            ...
   }
    ```

2. 將 `IBotTelemetryClient` 參數新增至 `QnABot.cs` 中的 `QnABot` 類別，並將其值指派給上一個步驟中建立的私用欄位：

    ```cs
    public QnABot(IConfiguration configuration, ILogger<QnABot> logger, IHttpClientFactory httpClientFactory, IBotTelemetryClient telemetryClient)
    {
        ...
        _telemetryClient = telemetryClient;
    }
    ```

3. 在 `QnABot.cs` 中具現化新的 QnAMaker 物件時，您需要 `telemetryClient`  參數：

    ```cs
    var qnaMaker = new QnAMaker(new QnAMakerEndpoint
                {
                    KnowledgeBaseId = _configuration["QnAKnowledgebaseId"],
                    EndpointKey = _configuration["QnAEndpointKey"],
                    Host = _configuration["QnAEndpointHostName"]
                },
                null,
                httpClient,
                _telemetryClient);
    ```

    > [!TIP] 
    > 請確定您在 `_configuration` 項目中使用的屬性名稱符合在 AppSettings.json 檔案中使用的屬性名稱，而您可以藉由選取 https://www.qnamaker.ai/Home/MyServices: 中的 [檢視程式碼]  按鈕來取得這些屬性的值

        
    ![AppSettings](media/AppSettings.json-QnAMaker.png)

#### <a name="viewing-telemetry-data-logged-from-the-qna-maker-default-entries"></a>從 QnA Maker 預設項目中查看記錄的遙測資料
在 Bot 模擬器中執行 Bot 之後，您可以採取下列步驟在 Application Insights 中查看 QnA Maker Bot 的使用結果：

1. 移至 [Azure 入口網站](https://portal.azure.com/)

2. 按一下 [監視器] > [應用程式]  ，瀏覽至您的 Application Insights。

3. 進入 Application Insights 後，按一下瀏覽列上的 [記錄 (分析)]  ，如下所示：

    ![Log Analytics](media/AppInsights-LogView-QnaBot.png)

4. 輸入下列 Kusto 查詢，然後選取 [執行] 

    ```SQL
    customEvents
    | where name == 'QnaMessage'
    | extend answer = tostring(customDimensions.answer)
    | summarize count() by answer
    ```
5. 讓此頁面在瀏覽器中保持開啟狀態，我們會在新增自訂屬性之後返回該頁面。

> [!TIP]
> 如果您不熟悉用來在 Azure 監視器中撰寫記錄查詢的 Kusto 查詢語言，但熟悉 SQL 查詢語言，則您可能會發現[從 SQL 到 Azure 監視器記錄查詢的功能提要](https://aka.ms/azureMonitor-SQL-cheatsheet)很有用。 

### <a name="modifying-or-extending-the-default-event-properties"></a>修改或擴充預設事件屬性
如果您需要的屬性未在 `QnAMaker` 類別中定義，有兩種方式可以處理這種情況，這兩種方式都需要您建立衍生自 `QnAMaker` 類別的自有類別。 下一節＜[新增屬性](#adding-properties)＞中會說明第一種方式，在該節中，您會將屬性加入至現有的 `QnAMessage` 事件。 第二種方法可讓您建立新的事件，讓您在其中新增屬性，如[使用自訂屬性新增事件](#adding-new-events-with-custom-properties)中所述。  

> [!Note]
> `QnAMessage` 事件是 Bot Framework SDK 的一部分，並提供所有已記錄至 Application Insights 的現成事件屬性。



#### <a name="adding-properties"></a>新增屬性 

下列範例會示範從 `QnAMaker` 類別衍生的方式。  此範例會顯示如何將屬性 "MyImportantProperty" 新增至 `QnAMessage` 事件。  `QnAMessage` 事件會在每次執行 QnA [GetAnswers](https://aka.ms/namespace-QnAMaker-GetAnswersAsync) 呼叫時進行記錄。  

在了解如何新增自訂屬性之後，我們將了解如何建立新的自訂事件，並讓屬性與之相關聯，然後我們會使用 Bot Framework Emulator 在本機執行 Bot，並在 Application Insights 中查看使用 Kusto 查詢語言記錄的內容。

1. 在繼承自 `QnAMaker` 類別的 `Microsoft.BotBuilderSamples` 命名空間中，建立名為 `MyQnAMaker` 的新類別，並將其儲存為 `MyQnAMaker.cs`。 若要從 `QnAMaker` 類別繼承，您必須新增 `Microsoft.Bot.Builder.AI.QnA` using 陳述式。 程式碼應如下所示：


    ```cs
    using Microsoft.Bot.Builder.AI.QnA;

    namespace Microsoft.BotBuilderSamples
    {
        public class MyQnAMaker : QnAMaker
        {

        }
    }
    ```
2. 將類別建構函式新增至 `MyQnAMaker`。 請注意，您將需要兩個額外的 using 陳述式來用於 `System.Net.Http` 和 `Microsoft.Bot.Builder` 建構函式參數：

    ```cs
    ...
    using Microsoft.Bot.Builder.AI.QnA;
    using System.Net.Http;
    using Microsoft.Bot.Builder;

    namespace Microsoft.BotBuilderSamples
    {
        public class MyQnAMaker : QnAMaker
        {
            public MyQnAMaker(
                QnAMakerEndpoint endpoint,
                QnAMakerOptions options = null,
                HttpClient httpClient = null,
                IBotTelemetryClient telemetryClient = null,
                bool logPersonalInformation = false)
                : base(endpoint, options, httpClient, telemetryClient, logPersonalInformation)
            {

            }
        } 
    }  
    ```
3. 在建構函式之後，將新的屬性新增至 QnAMessage 事件，並包含 `System.Collections.Generic`、`System.Threading` 和 `System.Threading.Tasks` 陳述式：

    ```cs
    using Microsoft.Bot.Builder.AI.QnA;
    using System.Net.Http;
    using Microsoft.Bot.Builder;
    using System.Collections.Generic;
    using System.Threading;
    using System.Threading.Tasks;
 
    namespace Microsoft.BotBuilderSamples
    {
            public class MyQnAMaker : QnAMaker
            {
            ...

            protected override async Task OnQnaResultsAsync(
                                QueryResult[] queryResults,
                                Microsoft.Bot.Builder.ITurnContext turnContext,
                                Dictionary<string, string> telemetryProperties = null,
                                Dictionary<string, double> telemetryMetrics = null,
                                CancellationToken cancellationToken = default(CancellationToken))
            {
                var eventData = await FillQnAEventAsync(
                                        queryResults,
                                        turnContext,
                                        telemetryProperties,
                                        telemetryMetrics,
                                        cancellationToken)
                                    .ConfigureAwait(false);

                // Add new property
                eventData.Properties.Add("MyImportantProperty", "myImportantValue");

                // Log QnAMessage event
                TelemetryClient.TrackEvent(
                                QnATelemetryConstants.QnaMsgEvent,
                                eventData.Properties,
                                eventData.Metrics
                                );
            }

        } 
    }    
    ```

4. 修改您的 Bot 以使用新的類別 (不是建立 `QnAMaker` 物件)，您將會在 `QnABot.cs` 中建立 `MyQnAMaker` 物件：

    ```cs
    var qnaMaker = new MyQnAMaker(new QnAMakerEndpoint
                {
                    KnowledgeBaseId = _configuration["QnAKnowledgebaseId"],
                    EndpointKey = _configuration["QnAEndpointKey"],
                    Host = _configuration["QnAEndpointHostName"]
                },
                null,
                httpClient,
                _telemetryClient);
    ```

##### <a name="viewing-telemetry-data-logged-from-the-new-property-_myimportantproperty_"></a>從新的 MyImportantProperty  屬性中檢視記錄的遙測資料
在模擬器中執行您的 Bot 之後，就可以透過下列方式在 Application Insights 中檢視結果：

1. 切換回具有 [記錄 (分析)]  檢視的瀏覽器。

2. 輸入下列 Kusto 查詢，然後選取 [執行]  。  這會提供新屬性的執行次數：

    ```SQL
    customEvents
    | where name == 'QnaMessage'
    | extend MyImportantProperty = tostring(customDimensions.MyImportantProperty)
    | summarize count() by MyImportantProperty
    ```

3. 若要顯示詳細資料，而不是計數，請移除最後一行並重新執行查詢：

    ```SQL
    customEvents
    | where name == 'QnaMessage'
    | extend MyImportantProperty = tostring(customDimensions.MyImportantProperty)
    ```
### <a name="adding-new-events-with-custom-properties"></a>使用自訂屬性新增事件
如果您需要將資料記錄到與 `QnaMessage` 不同的事件，您可以使用自己的屬性來建立自己的自訂事件。  若要這樣做，我們會將程式碼新增至 `MyQnAMaker` 類別的結尾，如下所示：

```CS
public class MyQnAMaker : QnAMaker
{
    ...

    // Create second event.
    var secondEventProperties = new Dictionary<string, string>();

    // Create new property for the second event.
    secondEventProperties.Add(
                        "MyImportantProperty2",
                        "myImportantValue2");

    // Log secondEventProperties event
    TelemetryClient.TrackEvent(
                    "MySecondEvent",
                    secondEventProperties);

} 
```                            
## <a name="the-application-insights-dashboard"></a>Application Insights 儀表板

無論何時，只要您在 Azure 中建立 Application Insights 資源，系統就會自動建立新的儀表板，並讓此儀表板與該資源相關聯。  您可以選取 [Application Insights] 刀鋒視窗頂端標示為**應用程式儀表板**的按鈕來查看該儀表板。 

![應用程式儀表板連結](media/Application-Dashboard-Link.png)


或者，若要檢視資料，也可移至 Azure 入口網站。 按一下左邊的 [儀表板]  ，然後從下拉式清單中選取您想要的儀表板。

在其中，您會看到一些關於聊天機器人效能的預設資訊，以及已釘選到儀表板的其他查詢。


## <a name="additional-information"></a>其他資訊

* [將遙測資料新增至 Bot](bot-builder-telemetry.md)

* [什麼是 Application Insights？](https://aka.ms/appinsights-overview)

* [在 Application Insights 中使用搜尋](https://aka.ms/search-in-application-insights)

* [使用 Azure Application Insights 建立自訂 KPI 儀表板](https://aka.ms/custom-kpi-dashboards-application-insights)
