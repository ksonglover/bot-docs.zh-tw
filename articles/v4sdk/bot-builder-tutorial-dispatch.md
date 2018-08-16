---
title: 使用 LUIS 和 QnA Maker 的分派工具 | Microsoft Docs
description: 了解如何在 Bot 中使用 LUIS 和 QnA Maker。
keywords: Luis, QnA, 分派工具, 多種服務
author: DeniseMak
ms.author: v-demak
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 04/25/2018
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: d3c9355a0e87d31029b92614dc182f3d7010c736
ms.sourcegitcommit: 9a38d76afb0e82fdccc1f36f9b1a65042671e538
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/04/2018
ms.locfileid: "39514938"
---
## <a name="integrate-multiple-luis-apps-and-qna-services-with-the-dispatch-tool"></a>使用分派工具整合多個 LUIS 應用程式和 QnA 服務

[!INCLUDE [pre-release-label](~/includes/pre-release-label.md)]

本教學課程將說明如何使用分派工具產生的 LUIS 模型，將 Bot 與多個 Language Understanding Intelligent Service (LUIS) 應用程式和 QnAMaker 服務進行整合。 

假設您已開發下列服務，且想要建立能整合這所有服務的 Bot。

| 服務類型 | Name | 說明 |
|------|------|------|
| LUIS 應用程式 | HomeAutomation | 辨識 HomeAutomation.TurnOn、HomeAutomation.TurnOff 和 HomeAutomation.None 意圖。|
| LUIS 應用程式 | Weather | 辨識 Weather.GetForecast 和 Weather.GetCondition 意圖。|
| QnAMaker 服務 | 常見問題集  | 提供有關居家自動化照明系統問題的解答 |

我們先建立應用程式和服務，再將其整合在一起。

> [!NOTE]
> 三個應用程式全都必須在同一個 Azure 位置內建立，分派工具才能順利存取。 以下範例分派程式碼使用的範例位置是美國西部。

## <a name="create-the-luis-apps"></a>建立 LUIS 應用程式

若要建立 HomeAutomation 和 Weather LUIS 應用程式，最快的方式是下載 [homeautomation.json][HomeAutomationJSON] 和 [weather.json][WeatherJSON] 檔案。 然後移至 [LUIS 網站](https://www.luis.ai/home)並登入。 依序按一下 [我的應用程式]  >  [匯入新應用程式]，然後選擇 homeautomation.json 檔案。 將新應用程式命名為 `homeautomation`。 依序按一下 [我的應用程式]  >  [匯入新應用程式]，然後選擇 weather.json 檔案。 將這另一個新應用程式命名為 `weather`。

## <a name="create-the-qna-cognitive-service-in-azure"></a>在 Azure 中建立 QnA 認知服務

QnA Maker 服務涵蓋兩個部分：Azure 中的認知服務，以及您使用認知服務發佈問與答組合的知識庫。

若要在 Azure 中建立認知服務，請登入 Azure 入口網站 (網址： https://portal.azure.com)，然後執行下列步驟：

1. 按一下 [所有服務]。
1. 在 `Cognitive` 上搜尋並選取 [認知服務]。
1. 按一下 [新增] 。
1. 在 `QnA` 上搜尋並選取 [QnA Maker]。
1. 在 [QnA Maker] 刀鋒視窗上，按一下 [建立]。
1. 填寫資訊並建立 QnA Maker 服務。

    <!-- TODO: Add screenshot.-->

    * 輸入服務的名稱。 在本教學課程中，我們將使用「`SmartLightQnA`」。
    * 選取要使用的訂用帳戶。
    * 選取要使用的管理定價層；我們將使用 F0 (free) 層。
    * 建立要使用的資源群組或選取新群組。 在本教學課程中，我們會建立新的 `SmartLightQnA` 資源群組。
    * 選取搜尋定價層；我們將使用 B (basic) 層。
    * 選取搜尋位置；我們將使用「`West US`」。
    * 輸入要使用的應用程式名稱；我們會保留預設的 `SmartLightQnA`。
    * 選取網站位置；我們會使用 `West US`。
    * 保留可讓應用程式深入解析的預設值。
    * 選取應用程式深入解析位置；我們會使用 `West US 2`。
    * 按一下 [建立] 以建立 QnA Maker 服務。
    * Azure 隨即建立並開始部署服務。

1. 服務部署完成後，請檢視通知並按一下 [移至資源]，瀏覽至服務的刀鋒視窗。
1. 按一下 [金鑰] 以抓取金鑰。

    * 複製服務的名稱和您的第一個金鑰。 稍後的步驟中會需要這些資料。
    * 您可取得兩組金鑰，如此一次可產生其中一組，而不用中斷服務。

## <a name="create-and-publish-the-qna-maker-knowledge-base"></a>建立和發佈 QnA Maker 知識庫

移至 [QnA Maker 網站](https://qnamaker.ai)並登入。 選取 [建立知識庫]，然後建立名稱為「常見問題集」的新知識庫。 按一下 [選取檔案] 按鈕，並上傳[範例 TSV 檔案][FAQ_TSV]。 按一下 [建立]，等服務建立完成後再按一下 [發佈]。

## <a name="use-the-dispatch-tool-to-create-the-dispatcher-luis-app"></a>使用分派工具建立發送器 LUIS 應用程式

接下來，我們要建立 LUIS 應用程式，以結合運用每個我們建立的服務。

在 Node.js 命令提示字元中執行此命令，以安裝[分派工具][DispatchTool]。

```
npm install -g botdispatch
```

執行下列命令，以初始化名稱為 `CombineWeatherAndLights` 的分派工具。 以 [LUIS 撰寫金鑰](https://docs.microsoft.com/en-us/azure/cognitive-services/LUIS/luis-concept-keys) 替代 `"YOUR-LUIS-AUTHORING-KEY"`。

```
dispatch init -name CombineWeatherAndLights -luisAuthoringKey "YOUR-LUIS-AUTHORING-KEY" -luisAuthoringRegion westus
```

針對您建立的每個 LUIS 應用程式，取得 LUIS 應用程式識別碼。 這些識別碼可在 [LUIS 網站](https://www.luis.ai/home)上，從每個應用程式的 [我的應用程式] 下方取得；依序按一下應用程式名稱和 [設定]，即可查看應用程式識別碼。 

然後針對您建立的每個 LUIS 應用程式執行 `dispatch add` 命令。

```
dispatch add -type luis -id "HOMEAUTOMATION-APP-ID" -name homeautomation -version 0.1 -key "YOUR-LUIS-AUTHORING-KEY"
dispatch add -type luis -id "WEATHER-APP-ID" -name weather -version 0.1 -key "YOUR-LUIS-AUTHORING-KEY"

```

針對您建立的 QnA Maker 服務執行 `dispatch add` 命令。 `-key` 參數應為從 Azure 入口網站取得的金鑰，即您於完成「[在 Azure 中建立 QnA 認知服務](./bot-builder-tutorial-dispatch.md#create-the-qna-cognitive-service-in-azure)」步驟時所儲存的金鑰。

```
dispatch add -type qna -id "QNA-KB-ID" -name faq -key "YOUR-QNA-SUBSCRIPTION-KEY"
```

執行 `dispatch create`：

```
dispatch create
```

這會建立名稱為 **CombineWeatherAndLights** 的發送器 LUIS 應用程式。 您可在 [https://www.luis.ai/home](https://www.luis.ai/home) 中查看新的應用程式。 

![LUIS.ai 中的 LUIS 應用程式。](media/tutorial-dispatch/dispatch-app-in-luis.png)

按一下新的應用程式。 在 [意圖] 下方，您可看到其具有 `l_homeautomation`、`l_weather` 和 `q_faq` 意圖。

![LUIS.ai 中的發送器意圖](media/tutorial-dispatch/dispatch-intents-in-luis.png)

按一下 [訓練] 按鈕可訓練 LUIS 應用程式，使用 [發佈] 索引標籤則可[發佈](https://docs.microsoft.com/en-us/azure/cognitive-services/LUIS/publishapp)。 按一下 [設定]，以複製要在 Bot 中使用之新應用程式的識別碼。

## <a name="create-the-bot"></a>建立 Bot

現在您可將發送器應用程式的意圖連結至 Bot，而其可路由訊息至原始的 LUIS 應用程式和 QnAMaker 服務。

您可從 Bot Builder SDK 中隨附的範例開始著手。

# <a name="ctabcsaddref"></a>[C#](#tab/csaddref)

從 [LUIS 分派範例][DispatchBotCS]中的程式碼開始。 在 Visual Studio 中，將[ Nuget 套件更新](https://docs.microsoft.com/en-us/nuget/tools/package-manager-ui#updating-a-package)至下列最新的發行前版本：

* `Microsoft.Bot.Builder.Integration.AspNet.Core`
* `Microsoft.Bot.Builder.Ai.QnA` (QnA Maker 的必要項目)
* `Microsoft.Bot.Builder.Ai.Luis` (LUIS 的必要項目)

# <a name="javascripttabjsaddref"></a>[JavaScript](#tab/jsaddref)

下載 [LUIS 分派範例][DispatchBotJs]。  使用 npm 安裝必要套件，包括適用於 LUIS 和 QnA Maker 的 `botbuilder-ai` 套件：

* `npm install --save botbuilder@preview`
* `npm install --save botbuilder-ai@preview`

---


設定範例以使用發送器應用程式。

# <a name="ctabcsbotconfig"></a>[C#](#tab/csbotconfig)

在 [LUIS 分派範例][DispatchBotCS]的 **appsettings.json** 中，編輯下列欄位。

| Name | 說明 |
|------|------|
| `Luis-SubscriptionKey` |  LUIS 訂用帳戶金鑰。 可能是端點金鑰或撰寫金鑰，詳情請參閱[此處](https://docs.microsoft.com/en-us/azure/cognitive-services/luis/luis-concept-keys)。 | 
| `Luis-ModelId-Dispatcher` | 分派工具產生之 LUIS 應用程式的應用程式識別碼。 | 
| `Luis-ModelId-HomeAutomation` | 您從 homeautomation.json 建立之應用程式的應用程式識別碼  | 
| `Luis-ModelId-Weather` | 您從 weather.json 建立之應用程式的應用程式識別碼 | 
| `QnAMaker-Endpoint-Url` | 此應設為 https://westus.api.cognitive.microsoft.com/qnamaker/v2.0 以用於 Preview QnA Maker 服務。 <br/>將此設為 https://YOUR-QNA-SERVICE-NAME.azurewebsites.net/qnamaker 以用於新的 (GA) QnA Maker 服務。|
| `QnAMaker-SubscriptionKey` | QnA Maker 訂用帳戶金鑰。 | 
| `QnAMaker-KnowledgeBaseId` | 您在 [QnAMaker 入口網站](https://qnamaker.ai)建立之知識庫的識別碼。| 



在 **Startup.cs** 中，查看 `ConfigureServices` 方法。 其包含的程式碼可使用您剛產生之 LUIS 應用程式的應用程式識別碼初始化 `LuisRecognizerMiddleware`。

```csharp
// This method gets called by the runtime. Use this method to add services to the container.
public void ConfigureServices(IServiceCollection services)
{
    services.AddSingleton(this.Configuration);
    services.AddBot<LuisDispatchBot>(options =>
    {
        options.CredentialProvider = new ConfigurationCredentialProvider(Configuration);

        var (luisModelId, luisSubscriptionKey, luisUri) = GetLuisConfiguration(this.Configuration, "Dispatcher");

        var luisModel = new LuisModel(luisModelId, luisSubscriptionKey, luisUri);

        // If you want to get all the intents and scores that LUIS recognized, set Verbose = true in luisOptions
        var luisOptions = new LuisRequest { Verbose = true };

        var middleware = options.Middleware;
        middleware.Add(new LuisRecognizerMiddleware(luisModel, luisOptions: luisOptions));
    });
}
```
`GetLuisConfiguration` 和 `GetQnAMakerConfiguration` 方法可從 appsettings.json 取得您的 LUIS 和 QnA 設定。
```csharp
public static (string modelId, string subscriptionId, Uri uri) GetLuisConfiguration(IConfiguration configuration, string serviceName)
{
    var modelId = configuration.GetSection($"Luis-ModelId-{serviceName}")?.Value;
    var subscriptionId = configuration.GetSection("Luis-SubscriptionKey")?.Value;
    var uri = new Uri(configuration.GetSection("Luis-Url")?.Value);
    return (modelId, subscriptionId, uri);
}

public static (string knowledgeBaseId, string subscriptionKey, string uri) GetQnAMakerConfiguration(IConfiguration configuration)
{
    var knowledgeBaseId = configuration.GetSection("QnAMaker-KnowledgeBaseId")?.Value;
    var subscriptionKey = configuration.GetSection("QnAMaker-SubscriptionKey")?.Value;
    var uri = configuration.GetSection("QnAMaker-Endpoint-Url")?.Value;
    return (knowledgeBaseId, subscriptionKey, uri);
}
```

### <a name="dispatch-the-message"></a>分派訊息

查看 **LuisDispatchBot.cs**，Bot 會從中將訊息分派至 LUIS 應用程式或 QnA Maker 以用於子元件。 

在 `DispatchToTopIntent` 中，如果發送器應用程式偵測到 `l_homeautomation` 或 `l_weather` 意圖，則會呼叫 `DispatchToTopIntent` 方法，以建立 `LuisRecognizer` 來呼叫原始的 `homeautomation` 和 `weather` 應用程式。 如果 Bot 偵測到 `q_faq` 意圖，或用於後援情況的 `none` 意圖，則會呼叫查詢 QnAMaker 的方法。

> [!NOTE] 
> 如果意圖名稱 `l_homeautomation`、`l_weather` 或 `q_faq` 與您使用分派工具建立的 LUIS 應用程式不符，您可編輯這些名稱，以與 [LUIS 入口網站](https://www.luis.ai)看到小寫版本意圖名稱相符。

```csharp
private async Task DispatchToTopIntent(ITurnContext context, (string intent, double score)? topIntent)
{
    switch (topIntent.Value.intent.ToLowerInvariant())
    {
        case "l_homeautomation":
            await DispatchToLuisModel(context, this.luisModelHomeAutomation, "home automation");
            break;
        case "l_weather":
            await DispatchToLuisModel(context, this.luisModelWeather, "weather");
            break;
        case "none":
        // You can provide logic here to handle the known None intent (none of the above).
        // In this example we fall through to the QnA intent.
        case "q_faq":
            await DispatchToQnAMaker(context, this.qnaEndpoint, "FAQ");
            break;
        default:
            // The intent didn't match any case, so just display the recognition results.
            await context.SendActivity($"Dispatch intent: {topIntent.Value.intent} ({topIntent.Value.score}).");

            break;
    }
}
```

`DispatchToQnAMaker` 方法會將使用者訊息傳送至 QnA Maker 服務。 請確定您已在 [QnA Maker 入口網站](https://qnamaker.ai)中發佈該服務再執行 Bot。

```csharp
private static async Task DispatchToQnAMaker(ITurnContext context, QnAMakerEndpoint qnaOptions, string appName)
{
    QnAMaker qnaMaker = new QnAMaker(qnaOptions);
    if (!string.IsNullOrEmpty(context.Activity.Text))
    {
        var results = await qnaMaker.GetAnswers(context.Activity.Text.Trim()).ConfigureAwait(false);
        if (results.Any())
        {
            await context.SendActivity(results.First().Answer);
        }
        else
        {
            await context.SendActivity($"Couldn't find an answer in the {appName}.");
        }
    }
}
```

`DispatchToLuisModel` 方法會將使用者訊息傳送至原始的 `homeautomation` 和 `weather` LUIS 應用程式。 請確定您已在 [LUIS 入口網站](https://www.luis.ai)中發佈該服務再執行 Bot。

```csharp
private static async Task DispatchToLuisModel(ITurnContext context, LuisModel luisModel, string appName)
{
    await context.SendActivity($"Sending your request to the {appName} system ...");
    var (intents, entities) = await RecognizeAsync(luisModel, context.Activity.Text);

    await context.SendActivity($"Intents detected by the {appName} app:\n\n{string.Join("\n\n", intents)}");

    if (entities.Count() > 0)
    {
        await context.SendActivity($"The following entities were found in the message:\n\n{string.Join("\n\n", entities)}");
    }
    
    // Here, you can add code for calling the hypothetical home automation or weather service, 
    // passing in the appName and any intent or entity information that you need 
}
```

`RecognizeAsync` 方法會呼叫 `LuisRecognizer` 以從 LUIS 應用程式取得結果。

```cs
private static async Task<(IEnumerable<string> intents, IEnumerable<string> entities)> RecognizeAsync(LuisModel luisModel, string text)
{
    var luisRecognizer = new LuisRecognizer(luisModel);
    var recognizerResult = await luisRecognizer.Recognize(text, System.Threading.CancellationToken.None);

    // list the intents
    var intents = new List<string>();
    foreach (var intent in recognizerResult.Intents)
    {
        intents.Add($"'{intent.Key}', score {intent.Value}");
    }

    // list the entities
    var entities = new List<string>();
    foreach (var entity in recognizerResult.Entities)
    {
        if (!entity.Key.ToString().Equals("$instance"))
        {
            entities.Add($"{entity.Key}: {entity.Value.First}");
        }
    }

    return (intents, entities);
}
```

# <a name="javascripttabjsbotconfig"></a>[JavaScript](#tab/jsbotconfig)

從[分派 Bot 範例][DispatchBotJs]中的程式碼開始。 開啟 **app.js**，並選擇性使用您建立之 LUIS 應用程式的識別碼取代 `appId` 欄位中的值。 如果您將 `appId` 欄位保留原樣，則可使用建立做為示範的公用 LUIS 應用程式。

```javascript
// Create LuisRecognizers and QnAMaker
// The LUIS applications are public, meaning you can use your own subscription key to test the applications.
// For QnAMaker, users are required to create their own knowledge base.
// The exported LUIS applications and QnAMaker knowledge base can be found with the sample bot.

// The corresponding LUIS application JSON is `dispatchSample.json`
const dispatcher = new LuisRecognizer({
    appId: '0b18ab4f-5c3d-4724-8b0b-191015b48ea9',
    subscriptionKey: process.env.LUIS_SUBSCRIPTION_KEY,
    serviceEndpoint: 'https://westus.api.cognitive.microsoft.com/',
    verbose: true
});

// The corresponding LUIS application JSON is `homeautomation.json`
const homeAutomation = new LuisRecognizer({
    appId: 'c6d161a5-e3e5-4982-8726-3ecec9b4ed8d',
    subscriptionKey: process.env.LUIS_SUBSCRIPTION_KEY,
    serviceEndpoint: 'https://westus.api.cognitive.microsoft.com/',
    verbose: true
});

// The corresponding LUIS application JSON is `weather.json`
const weather = new LuisRecognizer({
    appId: '9d0c9e9d-ce04-4257-a08a-a612955f2fb5',
    subscriptionKey: process.env.LUIS_SUBSCRIPTION_KEY,
    serviceEndpoint: 'https://westus.api.cognitive.microsoft.com/',
    verbose: true
});
```

在下列程式碼中，使用 QnAMaker 的金鑰和知識庫識別碼取代 `endpointKey` 和 `knowledgeBaseID`。 針對 Preview QnA Maker 服務，`host` 應設為 `https://westus.api.cognitive.microsoft.com/qnamaker/v2.0`，而對於全新的 (GA) QnA Maker 服務，則應設為 `https://YOUR-QNA-SERVICE-NAME.azurewebsites.net/qnamaker`。

```javascript
const faq = new QnAMaker(
    {
        knowledgeBaseId: '',
        endpointKey: '',
        host: ''
    },
    {
        answerBeforeNext: true
    }
);

```

**app.js** 中的其餘程式碼將啟動適當的對話方塊，以處理 `l_homeautomation`、`l_weather` 和 `None` 意圖。 若為 `q_faq` 意圖，則可使用來自 QnAMaker 服務的答案回覆。

> [!NOTE] 
> 如果意圖名稱 `l_homeautomation`、`l_weather` 或 `q_faq` 與您使用分派工具建立的 LUIS 應用程式不符，您可編輯這些名稱，以與 [LUIS 入口網站](https://www.luis.ai)看到意圖名稱相符。

```javascript
// create conversation state
const conversationState = new ConversationState(new MemoryStorage());
adapter.use(conversationState);

// register some dialogs for usage with the LUIS apps that are being dispatched to
const dialogs = new DialogSet();

function findEntities(entityName, entityResults) {
    let entities = []
    if (entityName in entityResults) {
        entityResults[entityName].forEach((entity, idx) => {
            entities.push(entity);
        });
    }
    return entities.length > 0 ? entities : undefined;
}

dialogs.add('HomeAutomation_TurnOff', [
    async (dialogContext, args) => {
        const devices = findEntities('HomeAutomation_Device', args.entities);
        const operations = findEntities('HomeAutomation_Operation', args.entities);

        const state = conversationState.get(dialogContext.context);
        state.homeAutomationTurnOff = state.homeAutomationTurnOff ? state.homeAutomationTurnOff + 1 : 1;
        await dialogContext.context.sendActivity(`${state.homeAutomationTurnOff}: You reached the "HomeAutomation.TurnOff" dialog.`);
        if (devices) {
            await dialogContext.context.sendActivity(`Found these "HomeAutomation_Device" entities:\n${devices.join(', ')}`);
        }
        if (operations) {
            await dialogContext.context.sendActivity(`Found these "HomeAutomation_Operation" entities:\n${operations.join(', ')}`);
        }
        await dialogContext.end();
    }
]);

dialogs.add('HomeAutomation_TurnOn', [
    async (dialogContext, args) => {
        const devices = findEntities('HomeAutomation_Device', args.entities);
        const operations = findEntities('HomeAutomation_Operation', args.entities);

        const state = conversationState.get(dialogContext.context);
        state.homeAutomationTurnOn = state.homeAutomationTurnOn ? state.homeAutomationTurnOn + 1 : 1;
        await dialogContext.context.sendActivity(`${state.homeAutomationTurnOn}: You reached the "HomeAutomation.TurnOn" dialog.`);
        if (devices) {
            await dialogContext.context.sendActivity(`Found these "HomeAutomation_Device" entities:\n${devices.join(', ')}`);
        }
        if (operations) {
            await dialogContext.context.sendActivity(`Found these "HomeAutomation_Operation" entities:\n${operations.join(', ')}`);
        }
        await dialogContext.end();
    }
]);

dialogs.add('Weather_GetForecast', [
    async (dialogContext, args) => {
        const locations = findEntities('Weather_Location', args.entities);

        const state = conversationState.get(dialogContext.context);
        state.weatherGetForecast = state.weatherGetForecast ? state.weatherGetForecast + 1 : 1;
        await dialogContext.context.sendActivity(`${state.weatherGetForecast}: You reached the "Weather.GetForecast" dialog.`);
        if (locations) {
            await dialogContext.context.sendActivity(`Found these "Weather_Location" entities:\n${locations.join(', ')}`);
        }
        await dialogContext.end();
    }
]);

dialogs.add('Weather_GetCondition', [
    async (dialogContext, args) => {
        const locations = findEntities('Weather_Location', args.entities);

        const state = conversationState.get(dialogContext.context);
        state.weatherGetCondition = state.weatherGetCondition ? state.weatherGetCondition + 1 : 1;
        await dialogContext.context.sendActivity(`${state.weatherGetCondition}: You reached the "Weather.GetCondition" dialog.`);
        if (locations) {
            await dialogContext.context.sendActivity(`Found these "Weather_Location" entities:\n${locations.join(', ')}`);
        }
        await dialogContext.end();
    }
]);

dialogs.add('None', [
    async (dialogContext) => {
        const state = conversationState.get(dialogContext.context);
        state.noneIntent = state.noneIntent ? state.noneIntent + 1 : 1;
        await dialogContext.context.sendActivity(`${state.noneIntent}: You reached the "None" dialog.`);
        await dialogContext.end();
    }
]);

adapter.use(dispatcher);

// Listen for incoming Activities
server.post('/api/messages', (req, res) => {
    adapter.processActivity(req, res, async (context) => {
        if (context.activity.type === 'message') {
            const state = conversationState.get(context);
            const dc = dialogs.createContext(context, state);

            // Retrieve the LUIS results from our dispatcher LUIS application
            const luisResults = dispatcher.get(context);

            // Extract the top intent from LUIS and use it to select which LUIS application to dispatch to
            const topIntent = LuisRecognizer.topIntent(luisResults);

            const isMessage = context.activity.type === 'message';
            if (isMessage) {
                switch (topIntent) {
                    case 'l_homeautomation':
                        const homeAutoResults = await homeAutomation.recognize(context);
                        const topHomeAutoIntent = LuisRecognizer.topIntent(homeAutoResults);
                        await dc.begin(topHomeAutoIntent, homeAutoResults);
                        break;
                    case 'l_weather':
                        const weatherResults = await weather.recognize(context);
                        const topWeatherIntent = LuisRecognizer.topIntent(weatherResults);
                        await dc.begin(topWeatherIntent, weatherResults);
                        break;
                    case 'q_faq':
                        await faq.answer(context);
                        break;
                    default:
                        await dc.begin('None');
                }
            }

            if (!context.responded) {
                await dc.continue();
                if (!context.responded && isMessage) {
                    await dc.context.sendActivity(`Hi! I'm the LUIS dispatch bot. Say something and LUIS will decide how the message should be routed.`);
                }
            }
        }
    });
});

```

---
## <a name="run-the-bot"></a>執行 Bot

使用 [Bot Framework 模擬器](../bot-service-debug-emulator.md)測試 Bot。 向 Bot 傳送「開燈」這類訊息，以將訊息分派至住家自動化應用程式，然後再傳送「取得西雅圖天氣」以分派給天氣 LUIS 應用程式。

> [!NOTE] 
> 執行 Bot 之前，請確保已在 [LUIS 入口網站](https://www.luis.ai)中發佈所有您建立的 LUIS 應用程式，並檢查是否已在 [QnA Maker 入口網站](https://qnamaker.ai)中發佈 QnA Maker 服務。

![將訊息傳送到分派 Bot](media/tutorial-dispatch/run-dispatch-bot.png)

## <a name="evaluate-the-dispatchers-performance"></a>評估發送器的效能

雖然有時在 LUIS 應用程式和 QnA maker 服務中都會提供使用者訊息做為範例，但分派工具產生的結合 LUIS 應用程式並不會針對所有輸入值妥善執行。 請使用 `eval` 選項檢查應用程式效能。 

```
dispatch eval
```

執行 `dispatch eval` 會產生 **Summary.html** 檔案，其中將提供新語言模型的效能統計資料。

> [!TIP] 
> 您可在任何 LUIS 應用程式中實際執行 `dispatch eval`，而不只是在分派工具建立的 LUIS 應用程式中執行。

### <a name="edit-intents-for-duplicates-and-overlaps"></a>編輯重複和重疊項目的意圖

檢閱在 **Summary.html** 中標記為重複項目的範例表達方式，並移除類似或重疊的範例。 例如，假設在用於住家自動化的 `homeautomation` LUIS 應用程式中，「開燈」這類要求對應至「TurnOnLights」意圖，但「為什麼無法打開燈？」要求則對應至 「無」意圖，因此系統便會將其傳遞至 QnA maker。 使用分派工具結合 LUIS 應用程式和 QnA maker 服務時，您必須執行下列動作： 

* 從原始的 `homeautomation` LUIS 應用程式移除「無」意圖，然後在發送器應用程式中，新增該意圖表達方式到「無」意圖。
* 如果您未從原始 LUIS 應用程式中移除「無」意圖，則必須在 Bot 中新增邏輯，以將符合該意圖之訊息傳遞至 QnA maker 服務。

> [!TIP] 
> 請檢閱 [Language Understanding 的最佳作法](./bot-builder-concept-luis.md#best-practices-for-language-understanding)，以取得如何改善語言模型效能的秘訣。

## <a name="additional-resources"></a>其他資源

* [將 LUIS 用於交談流程][luis-v4-how-to]

[luis-v4-how-to]: bot-builder-howto-v4-luis.md
<!-- links -->

[HomeAutomationJSON]: https://aka.ms/dispatch-luis1
[WeatherJSON]: https://aka.ms/dispatch-luis2
[DispatchJSON]: https://aka.ms/dispatch-luis
[FAQ_TSV]: https://aka.ms/dispatch-qna-tsv

[DispatchTool]: https://github.com/Microsoft/botbuilder-tools/tree/master/Dispatch
[DispatchBotCS]: https://aka.ms/dispatch-sample-cs
[DispatchBotJs]: https://aka.ms/dispatch-sample-js



