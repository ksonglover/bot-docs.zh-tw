---
title: 將自然語言理解新增至您的 Bot | Microsoft Docs
description: 了解如何搭配 Bot Builder SDK 將 LUIS 用於自然語言理解。
keywords: Language Understanding, LUIS, intent, recognizer, entities, middleware, 意圖, 辨識器, 實體, 中介軟體
author: ivorb
ms.author: v-ivorb
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: cognitive-services
ms.date: 11/16/18
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: faf26b1c4ba87061631f217ee074283759f77c97
ms.sourcegitcommit: 392c581aa2f59cd1798ee2136b6cfee56aa3ee6d
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 11/20/2018
ms.locfileid: "52156697"
---
# <a name="add-natural-language-understanding-to-your-bot"></a>將自然語言理解新增至您的 Bot

[!INCLUDE [pre-release-label](../includes/pre-release-label.md)]

讓您的 Bot 能透過對話和上下文了解使用者的意思，並不是簡單的工作，但這樣的功能可讓您的 Bot 使用起來更有自然對話的感覺。 Language Understanding (稱為 LUIS) 能讓您這麼做，使您的 Bot 可以辨識使用者訊息的意圖，這樣使用者就能使用更自然的語言，且更順利地引導交談流程。 此主題逐步引導您設定簡單的 Bot，並使用 LUIS 辨識幾個不同的意圖。 
## <a name="prerequisites"></a>必要條件
- [luis.ai](https://www.luis.ai) 帳戶
- [Bot Framework 模擬器](https://github.com/Microsoft/BotFramework-Emulator/blob/master/README.md#download) (英文)
- 本文中的程式碼是以**採用 LUIS 的 NLP** 範例為基礎。 您需要採用 [C#](https://aka.ms/cs-luis-sample) 或 [JS](https://aka.ms/js-luis-sample) 的一份範例。 
- [Bot 基本概念](bot-builder-basics.md)、[自然語言處理](https://docs.microsoft.com/en-us/azure/cognitive-services/luis/what-is-luis)和 [.bot](bot-file-basics.md) 檔案的知識。

## <a name="create-a-luis-app-in-the-luis-portal"></a>在 LUIS 入口網站中建立 LUIS 應用程式
登入 LUIS 入口網站以建立自己的範例 LUIS 應用程式版本。 您可以在 [我的應用程式] 上建立和管理應用程式。 

1. 選取 [匯入新的應用程式]。 
1. 按一下 [選擇應用程式檔案 (JSON 格式)...] 
1. 選取 `reminders.json` 檔案，該檔案位於範例的 `CognitiveModels` 資料夾中。 在 [選擇性名稱] 中，輸入 **LuisBot**。 此檔案包含三個意圖：Calendar-Add、Calendar-Find 和 None。 我們將使用這些意圖，了解使用者將訊息傳送給 Bot 時的用意。 
1. [訓練](https://docs.microsoft.com/en-us/azure/cognitive-services/LUIS/luis-how-to-train)應用程式。
1. 將應用程式[發佈](https://docs.microsoft.com/en-us/azure/cognitive-services/LUIS/publishapp)到「生產」環境。

### <a name="obtain-values-to-connect-to-your-luis-app"></a>取得值以連線到您的 LUIS 應用程式

在您的 LUIS 應用程式發佈之後，您即可從 Bot 進行存取。 您必須記錄數個值，以從 Bot 存取您的 LUIS 應用程式。 您可以使用 LUIS 入口網站擷取該資訊。

#### <a name="retrieve-application-information-from-the-luisai-portal"></a>從 LUIS.ai 入口網站擷取應用程式資訊
.bot 檔案可作為將所有服務參考匯合在一起的位置。 您所擷取的資訊會新增至下一節中的 .bot 檔案。 
1. 從 [luis.ai](https://www.luis.ai) 中選取已發佈的 LUIS 應用程式。
1. 開啟已發佈的 LUIS 應用程式後，選取 [管理] 索引標籤。
1. 選取左側的 [應用程式資訊] 索引標籤，將針對 [應用程式識別碼] 顯示的值記錄為 <YOUR_APP_ID>。
1. 選取左側 [金鑰和端點] 索引標籤，將針對 [撰寫金鑰] 所顯示的值記錄為 <YOUR_AUTHORING_KEY>。 請注意，「您的訂用帳戶金鑰」與「您的撰寫金鑰」相同。 
1. 向下捲動到頁面結尾，將針對 [地區] 顯示的值記錄為 <YOUR_REGION>。
1. 將針對 [端點] 顯示的值記錄為 <YOUR_ENDPOINT>。

#### <a name="update-the-bot-file"></a>更新 Bot 檔案
將存取 LUIS 應用程式所需的資訊 (包括應用程式識別碼、撰寫金鑰、訂用帳戶金鑰、端點和區域) 新增至 `nlp-with-luis.bot` 檔案中。 這些是您先前從已發佈的 LUIS 應用程式儲存的值。

```json
{
    "name": "LuisBot",
    "description": "",
    "services": [
        {
            "type": "endpoint",
            "name": "development",
            "endpoint": "http://localhost:3978/api/messages",
            "appId": "",
            "appPassword": "",
            "id": "166"
        },
        {
            "type": "luis",
            "name": "LuisBot",
            "appId": "<luis appid>",
            "version": "0.1",
            "authoringKey": "<luis authoring key>",
            "subscriptionKey": "<luis subscription key>",
            "region": "<luis region>",
            "id": "158"
        }
    ],
    "padlock": "",
    "version": "2.0"
}
```
# <a name="ctabcs"></a>[C#](#tab/cs)

## <a name="configure-your-bot-to-use-your-luis-app"></a>設定 Bot 來使用 LUIS 應用程式

接下來，我們會在 `BotServices.cs` 中初始化 BotService 類別的新執行個體，這會從 `.bot` 檔案擷取上述資訊。 外部服務是使用 `BotConfiguration` 類別來設定。

```csharp
public class BotServices
{
    // Initializes a new instance of the BotServices class
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

    // Gets the set of LUIS Services used. LuisServices is represented as a dictionary.  
    public Dictionary<string, LuisRecognizer> LuisServices { get; } = new Dictionary<string, LuisRecognizer>();
}
```

然後在 `ConfigureServices` 方法內新增下列程式碼，以在 `Startup.cs` 檔案中將 LUIS 應用程式註冊為單一應用程式。

```csharp
public void ConfigureServices(IServiceCollection services)
{
    var secretKey = Configuration.GetSection("botFileSecret")?.Value;
    var botFilePath = Configuration.GetSection("botFilePath")?.Value;

    // Loads .bot configuration file and adds a singleton that your Bot can access through dependency injection.
    var botConfig = BotConfiguration.Load(botFilePath ?? @".\nlp-with-luis.bot", secretKey);
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
        
        // ...
    });
}
```

接下來，在 `Luis.cs` 檔案中，Bot 會取得此 LUIS 執行個體。

```csharp
public class LuisBot : IBot
{
    public static readonly string LuisKey = "LuisBot";
    private const string WelcomeText = "This bot will introduce you to natural language processing with LUIS. Type an utterance to get started";

    // Services configured from the ".bot" file.
    private readonly BotServices _services;

    // Initializes a new instance of the LuisBot class.
    public LuisBot(BotServices services)
    {
        _services = services ?? throw new System.ArgumentNullException(nameof(services));
        if (!_services.LuisServices.ContainsKey(LuisKey))
        {
            throw new System.ArgumentException($"Invalid configuration....");
        }
    }
    // ...
}
```

# <a name="javascripttabjs"></a>[JavaScript](#tab/js)

在我們的範例中，啟動程式碼位於 **index.js** 檔案、Bot 邏輯的程式碼位於 **bot.js** 檔案，而其他組態資訊位於 **nlp-with-luis.bot** 檔案。

在 **bot.js** 檔案中，我們會讀入組態資訊以產生 LUIS 服務及初始化 Bot。
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

<!--
## Extract entities

Besides recognizing intent, a LUIS app can also extract entities, which are important words for fulfilling a user's request. For example, for a weather bot, the LUIS app might be able to extract the location for the weather report from the user's message.

A common way to structure your conversation is to identify any entities in the user's message, and prompt for any of the required entities that are not found. Then, the subsequent steps handle the response to the prompt.


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


When gathering information like entities from multiple steps in a conversation, it can be helpful to save the information you need in your state. If an entity is found, it can be added to the appropriate state field. In your conversation if the current step already has the associated field completed, the step to prompt for that information can be skipped.

/Snip -->

## <a name="test-the-bot"></a>測試 Bot

1. 在您的電腦本機執行範例。 如需指示，請參閱 [C#](https://github.com/Microsoft/BotBuilder-Samples/blob/master/samples/csharp_dotnetcore/12.nlp-with-luis/README.md) 或 [JS](https://github.com/Microsoft/BotBuilder-Samples/blob/master/samples/javascript_nodejs/12.nlp-with-luis/README.md) 範例的讀我檔案。

1. 在模擬器中，輸入如下所示的訊息。 

![測試 nlp 範例](~/media/emulator-v4/nlp-luis-sample-testing.png)

Bot 會以最高評分意圖來回應，在此情況下是 `Calendar-Add` 意圖。 請記得，您在 luis.ai 入口網站中匯入的 `reminders.json` 檔案定義了意圖。

預測分數表示 LUIS 對預測結果的信賴程度。 預測分數介於零 (0) 到一 (1) 之間。 高信賴度 LUIS 分數的範例是 0.99。 低信賴度的範例是 0.01。 

## <a name="next-steps"></a>後續步驟

> [!div class="nextstepaction"]
> [使用 QnA Maker 回答問題](./bot-builder-howto-qna.md)
