---
title: 將自然語言理解新增至您的 Bot | Microsoft Docs
description: 了解如何搭配 Bot Framework SDK 將 LUIS 用於自然語言理解。
keywords: Language Understanding, LUIS, intent, recognizer, entities, middleware, 意圖, 辨識器, 實體, 中介軟體
author: ivorb
ms.author: v-ivorb
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: cognitive-services
ms.date: 11/28/18
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 1f077cb5efd838f8a91a0f18a9bcc2f64455ceb6
ms.sourcegitcommit: 721bb09f10524b0cb3961d7131966f57501734b8
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 04/12/2019
ms.locfileid: "59541114"
---
# <a name="add-natural-language-understanding-to-your-bot"></a>將自然語言理解新增至您的 Bot

[!INCLUDE [pre-release-label](../includes/pre-release-label.md)]

讓您的 Bot 能透過對話和上下文了解使用者的意思，並不是簡單的工作，但這樣的功能可讓您的 Bot 使用起來更有自然對話的感覺。 Language Understanding (稱為 LUIS) 能讓您這麼做，使您的 Bot 可以辨識使用者訊息的意圖，這樣使用者就能使用更自然的語言，且更順利地引導交談流程。 此主題逐步引導您設定簡單的 Bot，並使用 LUIS 辨識幾個不同的意圖。 
## <a name="prerequisites"></a>必要條件
- [luis.ai](https://www.luis.ai) 帳戶
- [Bot Framework 模擬器](https://github.com/Microsoft/BotFramework-Emulator/blob/master/README.md#download) (英文)
- 本文中的程式碼是以**採用 LUIS 的 NLP** 範例為基礎。 您需要採用 [C# 範例](https://aka.ms/cs-luis-sample)或 [JS 範例](https://aka.ms/js-luis-sample)的範例複本。 
- [Bot 基本概念](bot-builder-basics.md)、[自然語言處理](https://docs.microsoft.com/en-us/azure/cognitive-services/luis/what-is-luis)和 [.bot](bot-file-basics.md) 檔案的知識。

## <a name="create-a-luis-app-in-the-luis-portal"></a>在 LUIS 入口網站中建立 LUIS 應用程式
登入 LUIS 入口網站以建立自己的範例 LUIS 應用程式版本。 您可以在 [我的應用程式] 上建立和管理應用程式。 

1. 選取 [匯入新的應用程式]。 
1. 按一下 [選擇應用程式檔案 (JSON 格式)...] 
1. 選取 `reminders.json` 檔案，該檔案位於範例的 `CognitiveModels` 資料夾中。 在 [選擇性名稱] 中，輸入 **LuisBot**。 此檔案包含三個意圖：Calendar_Add、Calendar_Find 和 None。 我們將使用這些意圖，了解使用者將訊息傳送給 Bot 時的用意。 如果您想要包含實體，請參閱本文結尾的[選讀小節](#optional---extract-entities)。
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

### <a name="configure-your-bot-to-use-your-luis-app"></a>設定 Bot 來使用 LUIS 應用程式
請確定您已為專案安裝 nuget 套件 **Microsoft.Bot.Builder.AI.Luis**。

接下來，我們會在 `BotServices.cs` 中初始化 BotService 類別的新執行個體，這會從 `.bot` 檔案擷取上述資訊。 外部服務是使用 `BotConfiguration` 類別來設定。

```csharp
using Microsoft.Bot.Builder.AI.Luis;
using Microsoft.Bot.Configuration;

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

    // Initialize Bot Connected Services client.
    var connectedServices = new BotServices(botConfig);
    services.AddSingleton(sp => connectedServices);

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

接下來，在 `LuisBot.cs` 檔案中，Bot 會取得此 LUIS 執行個體。

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
// Language Understanding (LUIS) service name as defined in the .bot file.YOUR_LUIS_APP_NAME is "LuisBot" in the JavaScript code.
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

### <a name="get-the-intent-by-calling-luis"></a>藉由呼叫 LUIS 來取得意圖

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

### <a name="test-the-bot"></a>測試 Bot

1. 在您的電腦本機執行範例。 如需指示，請參閱 [C#](https://github.com/Microsoft/BotBuilder-Samples/blob/master/samples/csharp_dotnetcore/12.nlp-with-luis/README.md) 或 [JS](https://github.com/Microsoft/BotBuilder-Samples/blob/master/samples/javascript_nodejs/12.nlp-with-luis/README.md) 範例的讀我檔案。

1. 在模擬器中，輸入如下所示的訊息。 

![測試 nlp 範例輸入](./media/nlp-luis-sample-message.png)

Bot 會以最高評分意圖來回應，在此情況下是 'Calendar_Add' 意圖。 請記得，您在 luis.ai 入口網站中匯入的 `reminders.json` 檔案定義了意圖 'Calendar_Add'、'Calendar_Find' 和 'None'。 

![測試 nlp 範例回應](./media/nlp-luis-sample-response.png) 

預測分數表示 LUIS 對預測結果的信賴程度。 預測分數介於零 (0) 到一 (1) 之間。 高信賴度 LUIS 分數的範例是 0.99。 低信賴度的範例是 0.01。 

## <a name="optional---extract-entities"></a>選用 - 擷取實體

除了辨識使用者意圖，LUIS 應用程式也可以傳回實體。 實體是與意圖相關的重要字組，有時候是履行使用者要求或可讓 Bot 更聰明運作的要件。 

### <a name="why-use-entities"></a>為何使用實體

LUIS 實體可讓 Bot 以智慧方式了解與標準意圖不同的特定事項或事件。 這可讓您向使用者收集額外資訊，進而讓 Bot 更聰明地回應，或可能略過要求使用者提供該資訊的某些問題。 例如，在天氣 Bot 中，LUIS 應用程式可以使用 _Location_ 實體，從使用者的訊息中擷取所要求天氣預報的位置。 這可讓 Bot 略過使用者所在位置的問題，並提供使用者更聰明且更簡潔的交談。

### <a name="prerequisites"></a>必要條件

若要在此範例中使用實體，您必須建立包含實體的 LUIS 應用程式。 請依照上一節[建立 LUIS 應用程式](#create-a-luis-app-in-the-luis-portal)中的步驟，但不要使用 `reminders.json` 檔案，而使用 [reminders-with-entities.json](https://github.com/Microsoft/BotFramework-Samples/tree/master/SDKV4-Samples/dotnet_core/nlp-with-luis) 檔案來建置 LUIS 應用程式。 這個檔案會提供相同意圖加上額外三個實體：約會、會議和排程。 這些實體可協助 LUIS 判斷使用者訊息的意圖。 

### <a name="extract-and-display-entities"></a>擷取並顯示實體
在此範例應用程式中可以新增下列選用程式碼，以在 LUIS 使用實體來協助識別使用者意圖時，擷取並顯示實體資訊。 

# <a name="ctabcs"></a>[C#](#tab/cs)

下列協助程式函式可以新增至 Bot，以從 LUIS 的 `RecognizerResult` 中取得實體。 其會需要使用 `Newtonsoft.Json.Linq` 程式庫，而您必須將該程式庫新增到 **using** 陳述式。 如果在剖析 LUIS 所傳回的 JSON 時找到實體資訊，Newtonsoft _DeserializeObject_ 函式會將此 JSON 轉換成動態物件，並提供實體資訊的存取權。

```cs
using Newtonsoft.Json.Linq;

private string ParseLuisForEntities(RecognizerResult recognizerResult)
{
   var result = string.Empty;

   // recognizerResult.Entities returns type JObject.
   foreach (var entity in recognizerResult.Entities)
   {
       // Parse JObject for a known entity types: Appointment, Meeting, and Schedule.
       var appointFound = JObject.Parse(entity.Value.ToString())["Appointment"];
       var meetingFound = JObject.Parse(entity.Value.ToString())["Meeting"];
       var schedFound = JObject.Parse(entity.Value.ToString())["Schedule"];

       // We will return info on the first entity found.
       if (appointFound != null)
       {
           // use JsonConvert to convert entity.Value to a dynamic object.
           dynamic o = JsonConvert.DeserializeObject<dynamic>(entity.Value.ToString());
           if (o.Appointment[0] != null)
           {
              // Find and return the entity type and score.
              var entType = o.Appointment[0].type;
              var entScore = o.Appointment[0].score;
              result = "Entity: " + entType + ", Score: " + entScore + ".";
              
              return result;
            }
        }

        if (meetingFound != null)
        {
            // use JsonConvert to convert entity.Value to a dynamic object.
            dynamic o = JsonConvert.DeserializeObject<dynamic>(entity.Value.ToString());
            if (o.Meeting[0] != null)
            {
                // Find and return the entity type and score.
                var entType = o.Meeting[0].type;
                var entScore = o.Meeting[0].score;
                result = "Entity: " + entType + ", Score: " + entScore + ".";
                
                return result;
            }
        }

        if (schedFound != null)
        {
            // use JsonConvert to convert entity.Value to a dynamic object.
            dynamic o = JsonConvert.DeserializeObject<dynamic>(entity.Value.ToString());
            if (o.Schedule[0] != null)
            {
                // Find and return the entity type and score.
                var entType = o.Schedule[0].type;
                var entScore = o.Schedule[0].score;
                result = "Entity: " + entType + ", Score: " + entScore + ".";
                
                return result;
            }
        }
    }

    // No entities were found, empty string returned.
    return result;
}
```

此偵測到的實體資訊即可與所識別的使用者意圖一起顯示。 若要顯示這項資訊，請在意圖資訊顯示之後，將下列幾行程式碼新增至範例程式碼的 _OnTurnAsync_ 工作。

```cs
// See if LUIS found and used an entity to determine user intent.
var entityFound = ParseLuisForEntities(recognizerResult);

// Inform the user if LUIS used an entity.
if (entityFound.ToString() != string.Empty)
{
   await turnContext.SendActivityAsync($"==>LUIS Entity Found: {entityFound}\n");
}
else
{
   await turnContext.SendActivityAsync($"==>No LUIS Entities Found.\n");
}
```
# <a name="javascripttabjs"></a>[JavaScript](#tab/js)

您可將下列程式碼新增至 Bot，以從 LUIS 傳回的 `luisRecognizer` 結果中擷取實體資訊。 在 `onTurn` 內，處理程式碼範例檔案 bot.js 會將下列一行新增至常數 _topIntent_ 的宣告之後。 這會擷取任何傳回的實體資訊： 

```javascript
// Since the LuisRecognizer was configured to include the raw results, get returned entity data.
var entityData = results.luisResult.entities;

```

若要向使用者顯示任何傳回的實體資訊，請在 _sendActivity_ 呼叫之後新增下列程式碼行，該呼叫使用於範例程式碼內，以在找到 topIntent 時通知使用者。

```javascript
// See if LUIS found and used an entity to determine user intent.
if (entityData.length > 0)
{
   if ((entityData[0].type == "Appointment") || (entityData[0].type == "Meeting") || (entityData[0].type == "Schedule") )
   {
      // inform user if LUIS used an entity.
      await turnContext.sendActivity(`LUIS Entity Found: Entity: ${entityData[0].entity}, Score: ${entityData[0].score}.`);
   }
}
else{
       await turnContext.sendActivity(`No LUIS Entities Found.`);
}
```

此程式碼會先在傳回的結果內查看 LUIS 是否傳回任何實體資訊，而若是如此，它會顯示第一個偵測到的實體相關資訊。

---

### <a name="test-bot-with-entities"></a>測試包含實體的 Bot

1. 若要測試包含實體的 Bot，請[如上](#test-the-bot)所述在本機執行範例。

1. 在模擬器中，輸入如下所示的訊息。 

![測試 nlp 範例輸入](./media/nlp-luis-sample-message.png)

Bot 現在會以最高評分意圖 'Calendar_Add' 加上 LUIS 用來判斷使用者意圖的 'Meetings' 實體來回應。

![測試 nlp 範例回應](./media/nlp-luis-sample-entity-response.png) 

偵測實體可協助改善 Bot 的整體效能。 例如，偵測 "Meeting" 實體 (如上所示) 可讓應用程式立即呼叫為了在使用者的行事曆上，建立新會議而設計的特製化函式。

## <a name="next-steps"></a>後續步驟

> [!div class="nextstepaction"]
> [使用 QnA Maker 回答問題](./bot-builder-howto-qna.md)
