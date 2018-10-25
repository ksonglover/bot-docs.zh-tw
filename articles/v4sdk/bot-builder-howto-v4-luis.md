---
title: 將 LUIS 用於 Language Understanding | Microsoft Docs
description: 了解如何搭配 Bot Builder SDK 將 LUIS 用於自然語言理解。
keywords: Language Understanding, LUIS, intent, recognizer, entities, middleware, 意圖, 辨識器, 實體, 中介軟體
author: ivorb
ms.author: v-ivorb
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: cognitive-services
ms.date: 10/12/18
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 395db5e1913b840340e5887cf09e6c59f90742a4
ms.sourcegitcommit: b78fe3d8dd604c4f7233740658a229e85b8535dd
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/24/2018
ms.locfileid: "49997665"
---
# <a name="using-luis-for-language-understanding"></a>將 LUIS 用於 Language Understanding

[!INCLUDE [pre-release-label](../includes/pre-release-label.md)]

讓您的 Bot 能透過對話和上下文了解使用者的意思，並不是簡單的工作，但這樣的功能可讓您的 Bot 使用起來更有自然對話的感覺。 Language Understanding (稱為 LUIS) 能讓您這麼做，使您的 Bot 可以辨識使用者訊息的意圖，這樣使用者就能使用更自然的語言，且更順利地引導交談流程。 如果您需要 LUIS 與 Bot 之整合方式的詳細背景資訊，請參閱[Bot 的語言理解](./bot-builder-concept-LUIS.md)。

## <a name="prerequisites"></a>必要條件
此主題逐步引導您設定簡單的 Bot，並使用 LUIS 辨識幾個不同的意圖。 本文中的程式碼是以 NLP 為基礎，包含 [C#](https://aka.ms/cs-luis-sample) 和 [JavaScript](https://aka.ms/js-luis-sample) 的 LUIS 範例。

## <a name="create-a-luis-app-in-the-luis-portal"></a>在 LUIS 入口網站中建立 LUIS 應用程式

首先，在 [luis.ai](https://www.luis.ai) 登入帳戶，並依照[這裡](https://docs.microsoft.com/en-us/azure/cognitive-services/luis/luis-how-to-start-new-app)顯示的指示在 LUIS 入口網站中建立 LUIS 應用程式。 如果您想要建立本文中所用範例 LUIS 應用程式的自有版本，請在 LUIS 入口網站中[匯入](https://docs.microsoft.com/en-us/azure/cognitive-services/luis/create-new-app#import-new-app) 此 `LUIS.Reminders.json` 檔案 ([C#](https://github.com/Microsoft/BotBuilder-Samples/blob/v4/samples/csharp_dotnetcore/12.nlp-with-luis/CognitiveModels/LUIS-Reminders.json) | [JS](https://github.com/Microsoft/BotBuilder-Samples/blob/master/samples/javascript_nodejs/12.nlp-with-luis/cognitiveModels/reminders.json)) 以建置 LUIS 應用程式，然後加以[訓練](https://docs.microsoft.com/en-us/azure/cognitive-services/LUIS/luis-how-to-train)及[發佈](https://docs.microsoft.com/en-us/azure/cognitive-services/LUIS/publishapp)。

### <a name="obtain-values-to-connect-to-your-luis-app"></a>取得值以連線到您的 LUIS 應用程式

在您的 LUIS 應用程式發佈之後，您即可從 Bot 進行存取。 您必須記錄數個值，以從 Bot 存取您的 LUIS 應用程式。 您可以使用 LUIS 入口網站或 CLI 工具擷取該資訊。

#### <a name="using-luis-portal"></a>使用 LUIS 入口網站
- 從 [luis.ai](https://www.luis.ai) 中選取已發佈的 LUIS 應用程式。
- 開啟已發佈的 LUIS 應用程式後，選取 [管理] 索引標籤。
- 選取左側的 [應用程式資訊] 索引標籤，將針對 [應用程式識別碼] 顯示的值記錄為 <YOUR_APP_ID>。
- 選取左側 [金鑰和端點] 索引標籤，將針對 [撰寫金鑰] 所顯示的值記錄為 <YOUR_AUTHORING_KEY>。 請注意 <YOUR_SUBSCRIPTION_KEY> 與 <YOUR_AUTHORING_KEY> 相同。 向下捲動到頁面結尾，將針對 [地區] 顯示的值記錄為 <YOUR_REGION>，並記將針對 [端點] 顯示的值記錄為 <YOUR_ENDPOINT>。

#### <a name="using-cli-tools"></a>使用 CLI 工具

您可以使用 [luis](https://aka.ms/botbuilder-tools-luis) 和 [msbot](https://aka.ms/botbuilder-tools-msbot-readme) BotBuilder CLI 工具來取得您的 LUIS 應用程式相關中繼資料，並將其新增至 **.bot** 檔案。

1. 開啟終端機或命令提示字元，然後瀏覽至您 Bot 專案的根目錄。
2. 請確定已安裝 `luis` 和 `msbot` 工具。

    ```shell
    npm install luis msbot
    ```

3. 執行 `luis init` 以建立 LUIS 資源檔 (**.luisrc**)。 出現提示時，請提供您的 LUIS 撰寫金鑰和您的區域。 此時您不需要輸入應用程式識別碼。
4. 執行下列命令以下載您的中繼資料，並將其新增至 Bot 的組態檔。
    如果您已加密組態檔，則必須提供祕密金鑰以更新檔案。

    ```shell
    luis get application --appId <your-app-id> --msbot | msbot connect luis --stdin [--secret <YOUR-SECRET>]
    ```

## <a name="configure-your-bot-to-use-your-luis-app"></a>設定 Bot 來使用 LUIS 應用程式

初始化 Bot 時，會首度新增 LUIS 應用程式的參考。 然後，我們可以在 Bot 邏輯內呼叫。

### <a name="prerequisite"></a>必要條件

開始撰寫程式碼之前，請先確定您有 LUIS 應用程式所需的套件。

# <a name="ctabcs"></a>[C#](#tab/cs)

將下列 [NuGet 套件](https://docs.microsoft.com/en-us/nuget/tools/package-manager-ui)新增到您的 Bot。

* `Microsoft.Bot.Builder.AI.Luis`

# <a name="javascripttabjs"></a>[JavaScript](#tab/js)

LUIS 功能位於 `botbuilder-ai` 套件中。 可以透過 npm 將此套件新增到您的專案：

```shell
npm install --save botbuilder-ai
```

---

# <a name="ctabcs"></a>[C#](#tab/cs)

下載並開啟這裡找到的 [NLP LUIS 範例程式碼](https://aka.ms/cs-luis-sample)。 我們會視需要修改程式碼。 

首先，將存取 LUIS 應用程式所需的資訊 (包括應用程式識別碼、撰寫金鑰、訂用帳戶金鑰、端點和區域) 新增至 `BotConfiguration.bot` 檔案中。 這些是您先前從已發佈的 LUIS 應用程式儲存的值。

```csharp
{
  "name": "LuisBot",
  "services": [
    {
      "type": "endpoint",
      "name": "development",
      "endpoint": "http://localhost:3978/api/messages",
      "appId": "",
      "appPassword": "",
      "id": "1"
    },
    {
      "type": "luis",
      "name": "LuisBot",
      "AppId": "<YOUR_APP_ID>",
      "SubscriptionKey": "<YOUR_SUBSCRIPTION_KEY>",
      "AuthoringKey": "<YOUR_AUTHORING_KEY>",
      "GetEndpoint": "<YOUR_ENDPOINT>",
      "Region": "<YOUR_REGION>"
    }
  ],
  "padlock": "",
  "version": "2.0"
}
```

接下來，我們會初始化 BotService 類別 `BotServices.cs` 的新執行個體，這會從 `.bot` 檔案擷取上述資訊。 將下列程式碼新增至 `BotServices.cs` 檔案。

```csharp
public class BotServices
{
    /// Initializes a new instance of the BotServices class
    public BotServices(BotConfiguration botConfiguration)
    {
        foreach (var service in botConfiguration.Services)
        {
            switch (service.Type)
            {
                case ServiceTypes.Luis:
                {
                    var luis = (LuisService)service;
                    if (luis == null)
                    {
                        throw new InvalidOperationException("The LUIS service is not configured correctly in your '.bot' file.");
                    }

                    var app = new LuisApplication(luis.AppId, luis.AuthoringKey, luis.GetEndpoint());
                    var recognizer = new LuisRecognizer(app);
                    this.LuisServices.Add(luis.Name, recognizer);
                    break;
                    }
                }
            }
        }

    /// Gets the set of LUIS Services used.
    /// LuisServices is represented as a dictionary.  
    public Dictionary<string, LuisRecognizer> LuisServices { get; } = new Dictionary<string, LuisRecognizer>();
}
```

然後在 `ConfigureServices` 內新增下列程式碼，以在 `Startup.cs` 檔案中將 LUIS 應用程式註冊為單一應用程式。

```csharp
public void ConfigureServices(IServiceCollection services)
{
    var secretKey = Configuration.GetSection("botFileSecret")?.Value;
    var botFilePath = Configuration.GetSection("botFilePath")?.Value;

    // Loads .bot configuration file and adds a singleton that your Bot can access through dependency injection.
    var botConfig = BotConfiguration.Load(botFilePath ?? @".\BotConfiguration.bot", secretKey);
    services.AddSingleton(sp => botConfig ?? throw new InvalidOperationException($"The .bot config file could not be loaded. ({botConfig})"));

    // Initialize Bot Connected Services clients.
    var connectedServices = new BotServices(botConfig);
    services.AddSingleton(sp => connectedServices);
    services.AddSingleton(sp => botConfig);

    services.AddBot<LuisBot>(options =>
    {
        // Retrieve current endpoint.
        var environment = _isProduction ? "production" : "development";
        var service = botConfig.Services.Where(s => s.Type == "endpoint" && s.Name == environment).FirstOrDefault();
        if (!(service is EndpointService endpointService))
        {
            throw new InvalidOperationException($"The .bot file does not contain an endpoint with name '{environment}'.");
        }

        options.CredentialProvider = new SimpleCredentialProvider(endpointService.AppId, endpointService.AppPassword);

        // Creates a logger for the application to use.
        ILogger logger = _loggerFactory.CreateLogger<LuisBot>();

        // Catches any errors that occur during a conversation turn and logs them.
        options.OnTurnError = async (context, exception) =>
        {
            logger.LogError($"Exception caught : {exception}");
            await context.SendActivityAsync("Sorry, it looks like something went wrong.");
        };
        /// ...
    });
}
```

接下來，我們需要將這個 LUIS 執行個體提供給 Bot。 開啟 `LuisBot.cs` 檔案，並且在檔案開頭處新增下列程式碼。

```csharp
public class LuisBot : IBot
{
    public static readonly string LuisKey = "LuisBot";
    private const string WelcomeText = "This bot will introduce you to natural language processing with LUIS. Type an utterance to get started";

    /// Services configured from the ".bot" file.
    private readonly BotServices _services;

    /// Initializes a new instance of the LuisBot class.
    public LuisBot(BotServices services)
    {
        _services = services ?? throw new System.ArgumentNullException(nameof(services));
        if (!_services.LuisServices.ContainsKey(LuisKey))
        {
            throw new System.ArgumentException($"Invalid configuration. Please check your '.bot' file for a LUIS service named '{LuisKey}'.");
        }
    }
    /// ...
}
```

# <a name="javascripttabjs"></a>[JavaScript](#tab/js)

在我們的範例中，啟動程式碼位於 **index.js** 檔案、Bot 邏輯的程式碼位於 **bot.js** 檔案，而其他組態資訊位於 **nlp-with-luis.bot** 檔案。

遵循指示來建立 LUIS 應用程式以及更新 **.bot** 檔案之後，**nlp-with-luis.bot** 檔案應該包含 LUIS 應用程式的服務項目。

```json
{
    "name": "YOUR_LUIS_APP_NAME",
    "description": "",
    "services": [
        {
            "type": "endpoint",
            "name": "development",
            "endpoint": "http://localhost:3978/api/messages",
            "appId": "",
            "appPassword": "",
            "id": "35"
        },
        {
            "type": "luis",
            "name": "YOUR_LUIS_APP_NAME",
            "appId": "<YOUR_APP_ID>",
            "version": "0.1",
            "authoringKey": "<YOUR_AUTHORING_KEY>",
            "subscriptionKey": "<YOUR_SUBSCRIPTION_KEY>>",
            "region": "<YOUR_REGION>",
            "id": "83"
        }
    ],
    "padlock": "",
    "version": "2.0"
}
```

在 **index.js** 檔案中，我們會讀入組態資訊以產生 LUIS 服務及初始化 Bot。
將 `LUIS_CONFIGURATION` 的值更新為您的 LUIS 應用程式名稱，因為其會出現在您的組態檔中。

```javascript
// Language Understanding (LUIS) service name as defined in the .bot file.YOUR_LUIS_APP_NAME is "LuisBot" in the C# code.
const LUIS_CONFIGURATION = '<YOUR_LUIS_APP_NAME>';

// Get endpoint and LUIS configurations by service name.
const endpointConfig = botConfig.findServiceByNameOrId(BOT_CONFIGURATION);
const luisConfig = botConfig.findServiceByNameOrId(LUIS_CONFIGURATION);

// Map the contents to the required format for `LuisRecognizer`.
const luisApplication = {
    applicationId: luisConfig.appId,
    endpointKey: luisConfig.subscriptionKey || luisConfig.authoringKey,
    azureRegion: luisConfig.region
};

// Create configuration for LuisRecognizer's runtime behavior.
const luisPredictionOptions = {
    includeAllIntents: true,
    log: true,
    staging: false
};

// Create adapter...

// Create the LuisBot.
let bot;
try {
    bot = new LuisBot(luisApplication, luisPredictionOptions);
} catch (err) {
    console.error(`[botInitializationError]: ${ err }`);
    process.exit();
}
```

然後我們會建立 HTTP 伺服器並接聽傳入要求，這會對我們的 Bot 邏輯產生呼叫。

```javascript
// Create HTTP server.
let server = restify.createServer();
server.listen(process.env.port || process.env.PORT || 3978, function() {
    console.log(`\n${ server.name } listening to ${ server.url }.`);
    console.log(`\nGet Bot Framework Emulator: https://aka.ms/botframework-emulator.`);
    console.log(`\nTo talk to your bot, open nlp-with-luis.bot file in the emulator.`);
});

// Listen for incoming requests.
server.post('/api/messages', (req, res) => {
    adapter.processActivity(req, res, async(turnContext) => {
        await bot.onTurn(turnContext);
    });
});
```

---

現在已針對您的 Bot 設定 LUIS。 接下來，讓我們看看如何從 LUIS 取得意圖。

## <a name="get-the-intent-by-calling-luis"></a>藉由呼叫 LUIS 來取得意圖

您的 Bot 可藉由呼叫 LUIS 辨識器來取得 LUIS 的結果。

# <a name="ctabcs"></a>[C#](#tab/cs)

若要讓 Bot 只根據 LUIS 應用程式偵測到的意圖傳送及回覆，請呼叫 `LuisRecognizer` 來取得 `RecognizerResult`。 每當您需要取得 LUIS 意圖時，可以在您的程式碼中完成此動作。

```cs
public async Task OnTurnAsync(ITurnContext turnContext, CancellationToken cancellationToken = default(CancellationToken))

{
    if (turnContext.Activity.Type == ActivityTypes.Message)
    {
        // Check LUIS model
        var recognizerResult = await _services.LuisServices[LuisKey].RecognizeAsync(turnContext, cancellationToken);
        var topIntent = recognizerResult?.GetTopScoringIntent();
        if (topIntent != null && topIntent.HasValue && topIntent.Value.intent != "None")
        {
            await turnContext.SendActivityAsync($"==>LUIS Top Scoring Intent: {topIntent.Value.intent}, Score: {topIntent.Value.score}\n");
        }
        else
        {
            var msg = @"No LUIS intents were found.
                        This sample is about identifying two user intents:
                        'Calendar.Add'
                        'Calendar.Find'
                        Try typing 'Add Event' or 'Show me tomorrow'.";
            await turnContext.SendActivityAsync(msg);
        }
        }
        else if (turnContext.Activity.Type == ActivityTypes.ConversationUpdate)
        {
            // Send a welcome message to the user and tell them what actions they may perform to use this bot
            await SendWelcomeMessageAsync(turnContext, cancellationToken);
        }
        else
        {
            await turnContext.SendActivityAsync($"{turnContext.Activity.Type} event detected", cancellationToken: cancellationToken);
        }
}
```

在語句中辨識出的任何意圖，都會以意圖名稱對評分的對應傳回，而且可以從 `recognizerResult.Intents` 存取。 系統提供靜態 `recognizerResult?.GetTopScoringIntent()` 方法來協助簡化尋找結果集之評分最高的意圖。

辨識出的任何實體都會以實體名稱對值的對應傳回，而且可以利用 `recognizerResults.entities` 來存取。 透過在建立 LuisRecognizer 時傳遞 `verbose=true` 設定，可以傳回其他實體中繼資料。 新增的中繼資料可以利用 `recognizerResults.entities.$instance` 來存取。

# <a name="javascripttabjs"></a>[JavaScript](#tab/js)

在 **bot.js** 檔案中，我們會將使用者的輸入傳遞至 LUIS 辨識器的 `recognize` 方法，以從 LUIS 應用程式得到解答。

```javascript
const { ActivityTypes } = require('botbuilder');
const { LuisRecognizer } = require('botbuilder-ai');

/**
 * A simple bot that responds to utterances with answers from the Language Understanding (LUIS) service.
 * If an answer is not found for an utterance, the bot responds with help.
 */
class LuisBot {
    /**
     * The LuisBot constructor requires one argument (`application`) which is used to create an instance of `LuisRecognizer`.
     * @param {LuisApplication} luisApplication The basic configuration needed to call LUIS. In this sample the configuration is retrieved from the .bot file.
     * @param {LuisPredictionOptions} luisPredictionOptions (Optional) Contains additional settings for configuring calls to LUIS.
     */
    constructor(application, luisPredictionOptions, includeApiResults) {
        this.luisRecognizer = new LuisRecognizer(application, luisPredictionOptions, true);
    }

    /**
     * Every conversation turn calls this method.
     * @param {TurnContext} turnContext Contains all the data needed for processing the conversation turn.
     */
    async onTurn(turnContext) {
        // By checking the incoming Activity type, the bot only calls LUIS in appropriate cases.
        if (turnContext.activity.type === ActivityTypes.Message) {
            // Perform a call to LUIS to retrieve results for the user's message.
            const results = await this.luisRecognizer.recognize(turnContext);

            // Since the LuisRecognizer was configured to include the raw results, get the `topScoringIntent` as specified by LUIS.
            const topIntent = results.luisResult.topScoringIntent;

            if (topIntent.intent !== 'None') {
                await turnContext.sendActivity(`LUIS Top Scoring Intent: ${ topIntent.intent }, Score: ${ topIntent.score }`);
            } else {
                // If the top scoring intent was "None" tell the user no valid intents were found and provide help.
                await turnContext.sendActivity(`No LUIS intents were found.
                                                \nThis sample is about identifying two user intents:
                                                \n - 'Calendar.Add'
                                                \n - 'Calendar.Find'
                                                \nTry typing 'Add Event' or 'Show me tomorrow'.`);
            }
        } else if (turnContext.activity.type === ActivityTypes.ConversationUpdate &&
            turnContext.activity.recipient.id !== turnContext.activity.membersAdded[0].id) {
            // If the Activity is a ConversationUpdate, send a greeting message to the user.
            await turnContext.sendActivity('Welcome to the NLP with LUIS sample! Send me a message and I will try to predict your intent.');
        } else if (turnContext.activity.type !== ActivityTypes.ConversationUpdate) {
            // Respond to all other Activity types.
            await turnContext.sendActivity(`[${ turnContext.activity.type }]-type activity detected.`);
        }
    }
}

module.exports.LuisBot = LuisBot;
```

LUIS 辨識器會傳回語句與可用意圖的相符程度相關資訊。 結果物件的 `luisResult.intents` 屬性包含已評分的意圖陣列。 結果物件的 `luisResult.topScoringIntent` 屬性包含排名最前面的評分意圖及其分數。

---

## <a name="extract-entities"></a>擷取實體

除了辨識意圖，LUIS 應用程式也可以擷取實體，也就是滿足使用者要求的重要單字。 例如，在天氣 Bot 中，LUIS 應用程式可能可以從使用者的訊息中擷取天氣預報的位置。

建構對話的常見方法是識別使用者訊息中的任何實體，然後針對必要但未找到的實體發出提示。 接著，後續的步驟會處理對提示的回應。

<!--Snip
# [C#](#tab/cs)

Let's say the message from the user was "What's the weather in Seattle"? The [LuisRecognizer](https://docs.microsoft.com/en-us/dotnet/api/microsoft.bot.builder.ai.luis.luisrecognizer) gives you a [RecognizerResult](https://docs.microsoft.com/en-us/dotnet/api/microsoft.bot.builder.core.extensions.recognizerresult) with an [`Entities` property](https://docs.microsoft.com/en-us/dotnet/api/microsoft.bot.builder.core.extensions.recognizerresult#properties-) that has this structure:

```json
{
"$instance": {
    "Weather_Location": [
        {
            "startIndex": 22,
            "endIndex": 29,
            "text": "seattle",
            "score": 0.8073087
        }
    ]
},
"Weather_Location": [
        "seattle"
    ]
}
```

The following helper function can be added to your bot to get entities out of the `RecognizerResult` from LUIS. It will require the use of the `Newtonsoft.Json.Linq` library, which you'll have to add to your **using** statements.

```cs
// Get entities from LUIS result
private T GetEntity<T>(RecognizerResult luisResult, string entityKey)
{
    var data = luisResult.Entities as IDictionary<string, JToken>;
    if (data.TryGetValue(entityKey, out JToken value))
    {
        return value.First.Value<T>();
    }
    return default(T);
}
```

When gathering information like entities from multiple steps in a conversation, it can be helpful to save the information you need in your state. If an entity is found, it can be added to the appropriate state field. In your conversation if the current step already has the associated field completed, the step to prompt for that information can be skipped.

# [JavaScript](#tab/js)

Let's say the message from the user was "What's the weather in Seattle"? The [LuisRecognizer](https://docs.microsoft.com/en-us/javascript/api/botbuilder-ai/luisrecognizer) gives you a [RecognizerResult](https://docs.microsoft.com/en-us/javascript/api/botbuilder-core-extensions/recognizerresult) with an `entities` property that has this structure:

```json
{
"$instance": {
    "Weather_Location": [
        {
            "startIndex": 22,
            "endIndex": 29,
            "text": "seattle",
            "score": 0.8073087
        }
    ]
},
"Weather_Location": [
        "seattle"
    ]
}
```

This `findEntities` function looks for any entities recognized by the LUIS app that match the incoming `entityName`.

```javascript
// Helper function for finding a specified entity
// entityResults are the results from LuisRecognizer.get(context)
function findEntities(entityName, entityResults) {
    let entities = []
    if (entityName in entityResults) {
        entityResults[entityName].forEach(entity => {
            entities.push(entity);
        });
    }
    return entities.length > 0 ? entities : undefined;
}
```
/Snip-->

從對話中的多個步驟蒐集實體等資訊時，在狀態中儲存您需要的資訊會很有幫助。 如果找到實體，則可將其新增至適當的狀態欄位。 在對話中，如果目前的步驟已經完成相關聯的欄位，則可以略過提示輸入該資訊的步驟。

## <a name="additional-resources"></a>其他資源

如需使用 LUIS 的範例，請查看 [[C#](https://aka.ms/cs-luis-sample)] 或 [[JavaScript](https://aka.ms/js-luis-sample)] 的專案。

## <a name="next-steps"></a>後續步驟

> [!div class="nextstepaction"]
> [使用分派工具結合 LUIS 和 QnA 服務](./bot-builder-tutorial-dispatch.md)
