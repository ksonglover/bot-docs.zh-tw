---
title: 將 LUIS 用於 Language Understanding | Microsoft Docs
description: 了解如何搭配 Bot Builder SDK 將 LUIS 用於自然語言理解。
keywords: Language Understanding, LUIS, intent, recognizer, entities, middleware, 意圖, 辨識器, 實體, 中介軟體
author: ivorb
ms.author: v-ivorb
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 09/19/18
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: c4a7cfba6f588c95dbebf4886ffd7e432d99c3ae
ms.sourcegitcommit: 3cb288cf2f09eaede317e1bc8d6255becf1aec61
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/27/2018
ms.locfileid: "47389797"
---
# <a name="using-luis-for-language-understanding"></a>將 LUIS 用於 Language Understanding

[!INCLUDE [pre-release-label](../includes/pre-release-label.md)]

讓您的 Bot 能透過交談和上下文了解使用者的意思，並不是簡單的工作，但這樣的功能可讓您的 Bot 使用起來更有自然交談的感覺。 Language Understanding (稱為 LUIS) 能讓您這麼做，使您的 Bot 可以辨識使用者訊息的意圖，這樣使用者就能使用更自然的語言，且更順利地引導交談流程。 如果您需要 LUIS 與 Bot 之整合方式的詳細背景資訊，請參閱[Bot 的語言理解](./bot-builder-concept-LUIS.md)。 

此主題逐步引導您設定簡單的 Bot，並使用 LUIS 辨識幾個不同的意圖。

## <a name="installing-packages"></a>安裝套件

首先，請確認您有 LUIS 所需的套件。

# <a name="ctabcs"></a>[C#](#tab/cs)

對以下 NuGet 套件的 v4 版本[新增參考](https://docs.microsoft.com/en-us/nuget/tools/package-manager-ui)：


* `Microsoft.Bot.Builder.AI.LUIS`

# <a name="javascripttabjs"></a>[JavaScript](#tab/js)

在您的專案中使用 npm，安裝 botbuilder 和 botbuilder-ai 套件：

* `npm install --save botbuilder`
* `npm install --save botbuilder-ai`

---

## <a name="set-up-your-luis-app"></a>設定 LUIS 應用程式

首先，設定 _LUIS 應用程式_，其是您在 [luis.ai](https://www.luis.ai) 建立的服務。 該 LUIS 應用程式可以被定型成能夠辨識特定意圖。 您可以在 LUIS 網站上找到如何建立 LUIS 應用程式的詳細資料。

針對此範例，您將只使用能辨識 Help (說明)、Cancel (取消) 和 Weather (天氣) 意圖的 LUIS 示範應用程式；應用程式識別碼已經在範例程式碼中。 您將需要有能夠登入 [luis.ai](https://www.luis.ai) 的認知服務金鑰，然後從 [使用者設定] > [撰寫金鑰] 複製金鑰。

> [!NOTE]
> 若要為此範例中使用的公用 LUIS 應用程式建立您自己的複本，請複製 [JSON](https://github.com/Microsoft/LUIS-Samples/blob/master/examples/simple-bot-example/FirstSimpleBotExample.json) LUIS 檔案。 然後[匯入](https://docs.microsoft.com/en-us/azure/cognitive-services/luis/create-new-app#import-new-app) LUIS 應用程式，加以[訊息](https://docs.microsoft.com/en-us/azure/cognitive-services/LUIS/luis-how-to-train)，然後[發佈](https://docs.microsoft.com/en-us/azure/cognitive-services/LUIS/publishapp)。 使用您新的 LUIS 應用程式的應用程式識別碼來取代範例程式碼中的公用應用程式識別碼。


### <a name="configure-your-bot-to-call-your-luis-app"></a>設定 Bot 來呼叫 LUIS 應用程式。

# <a name="ctabcs"></a>[C#](#tab/cs)

雖然您可以每次建立及呼叫 LUIS 應用程式，但最好是將做法編碼以將 LUIS 服務註冊為單一實體，再將其當作參數傳遞至 Bot 的建構函式。 我們會在此示範該方法，因為這方法稍微複雜一點。

從 Echo Bot 範本開始，然後開啟 **Startup.cs**。 

為 `Microsoft.Bot.Builder.AI.LUIS` 新增 `using` 陳述式

```csharp
// add this
using Microsoft.Bot.Builder.AI.LUIS;
```

在狀態初始化之後，於 `ConfigureServices` 的結尾處新增以下程式碼。 這會從 `appsettings.json` 檔案抓取資訊，但是可以從 `.bot` 檔案抓取這些字串，像是本文結尾連結或硬式編碼的範例，以供測試。

單一實體會將新的 `LuisRecognizer` 傳回至建構函式。

```csharp
    // Create and register a LUIS recognizer.
    services.AddSingleton(sp =>
    {
        // Get LUIS information from appsettings.json.
        var section = this.Configuration.GetSection("Luis");
        var luisApp = new LuisApplication(
            applicationId: section["AppId"],
            endpointKey: section["SubscriptionKey"],
            azureRegion: section["Region"]);

        // Specify LUIS options. These may vary for your bot.
        var luisPredictionOptions = new LuisPredictionOptions
        {
            IncludeAllIntents = true,
        };

        return new LuisRecognizer(
            application: luisApp,
            predictionOptions: luisPredictionOptions,
            includeApiResults: true);
    });
```

貼上來自 [luis.ai](https://www.luis.ai) 的訂用帳戶金鑰，以取代 `<subscriptionKey>`。 如果您按一下右上方的帳戶名稱，然後移至 [設定]，最容易找到該金鑰，在此稱為 [撰寫金鑰]。

> [!NOTE]
> 如果您使用自己的 LUIS 應用程式而不是公用的，可以從 [luis.ai](https://www.luis.ai) 取得 LUIS 應用程式的識別碼、訂用帳戶金鑰和 URL。 在您應用程式頁面的 [發佈] 和 [設定] 索引標籤底下，可以找到這些資訊。
>
>您可以透過登入 [luis.ai](https://www.luis.ai) 來找到 `LuisModel` 中使用的基底 URL，請移至 [發佈] 索引標籤，然後查看 [資源和金鑰] 下的 [端點] 欄。 基底 URL 是**端點 URL** 的訂用帳戶識別碼和其他參數之前的部分。

接下來，我們需要將這個 LUIS 執行個體提供給 Bot。 開啟 **EchoBot.cs**，在檔案頂端新增下列程式碼。 類別標題和狀態項目也包含在內，供您參考，但我們不會在此加以說明。

```csharp
public class EchoBot : IBot
{
    /// <summary>
    /// Gets the Echo Bot state.
    /// </summary>
    private IStatePropertyAccessor<EchoState> EchoStateAccessor { get; }

    /// <summary>
    /// Gets the LUIS recognizer.
    /// </summary>
    private LuisRecognizer Recognizer { get; } = null;

    public EchoBot(ConversationState state, LuisRecognizer luis)
    {
        EchoStateAccessor = state.CreateProperty<EchoState>("EchoBot.EchoState");

        // The incoming luis variable is the LUIS Recognizer we added above.
        this.Recognizer = luis ?? throw new ArgumentNullException(nameof(luis));
    }

    /// ...
```

# <a name="javascripttabjs"></a>[JavaScript](#tab/js)

先遵循 JavaScript [快速入門](../javascript/bot-builder-javascript-quickstart.md)中的步驟來建立 Bot。 我們會將 LUIS 資訊硬式編碼到 Bot 中，但是可以從 `.bot` 檔案 (在本文結尾連結的範例中) 提取。

在新的 Bot 中，編輯 **app.js** 以要求 `LuisRecognizer` 類別，並建立 LUIS 模型的執行個體：

```javascript
const { ActivityTypes } = require('botbuilder');
const { LuisRecognizer } = require('botbuilder-ai');

const luisApplication = {
    // This appID is for a public app that's made available for demo purposes
    // You can use it or use your own LUIS "Application ID" at https://www.luis.ai under "App Settings".
     applicationId: 'eb0bf5e0-b468-421b-9375-fdfb644c512e',
    // Replace endpointKey with your "Subscription Key"
    // your key is at https://www.luis.ai under Publish > Resources and Keys, look in the Endpoint column
    // The "subscription-key" is embeded in the Endpoint link. 
    endpointKey: '<your subscription key>',
    // You can find your app's region info embeded in the Endpoint link as well.
    // Some examples of regions are `westus`, `westcentralus`, `eastus2`, and `southeastasia`.
    azureRegion: 'westus'
}

// Create configuration for LuisRecognizer's runtime behavior.
const luisPredictionOptions = {
    includeAllIntents: true,
    log: true,
    staging: false
}

// Create the bot that handles incoming Activities.
const luisBot = new LuisBot(luisApplication, luisPredictionOptions);
```

然後，在 Bot `LuisBot` 的建構函式中，取得應用程式以建立 LuisRecognizer 執行個體。

```javascript
    /**
     * The LuisBot constructor requires one argument (`application`) which is used to create an instance of `LuisRecognizer`.
     * @param {object} luisApplication The basic configuration needed to call LUIS. In this sample the configuration is retrieved from the .bot file.
     * @param {object} luisPredictionOptions (Optional) Contains additional settings for configuring calls to LUIS.
     */
    constructor(application, luisPredictionOptions) {
        this.luisRecognizer = new LuisRecognizer(application, luisPredictionOptions, true);
    }
```

> [!NOTE] 
> 如果您使用自己的 LUIS 應用程式而不是公用的，可以從 [https://www.luis.ai](https://www.luis.ai) 取得 LUIS 應用程式的應用程式識別碼、訂用帳戶金鑰和區域。 在您應用程式頁面的 [發佈] 和 [設定] 索引標籤底下，可以找到這些資訊。

---

現在已針對您的 Bot 設定 LUIS 語言理解。 接下來，讓我們看看如何從 LUIS 取得意圖。

## <a name="get-the-intent-by-calling-luis"></a>藉由呼叫 LUIS 來取得意圖

您的 Bot 可藉由呼叫 LUIS 辨識器來取得 LUIS 的結果。

# <a name="ctabcs"></a>[C#](#tab/cs)

若要讓 Bot 只根據 LUIS 應用程式偵測到的意圖傳送及回覆，請呼叫 `LuisRecognizer` 來取得 `RecognizerResult`。 每當您需要取得 LUIS 意圖時，可以在您的程式碼中完成此動作。

```cs
using System.Threading.Tasks;
using Microsoft.Bot;
using Microsoft.Bot.Builder;
using Microsoft.Bot.Builder.Core.Extensions;
using Microsoft.Bot.Schema;
// add this reference
using Microsoft.Bot.Builder.AI.LUIS;

namespace EchoBot
{
    public class EchoBot : IBot
    {
        /// <summary>
        /// Echo bot turn handler 
        /// </summary>
        /// <param name="context">Turn scoped context containing all the data needed
        /// for processing this conversation turn. </param>        
        public async Task OnTurnAsync(ITurnContext context, System.Threading.CancellationToken token)
        {            
            // This bot is only handling Messages
            if (context.Activity.Type == ActivityTypes.Message)
            {
                // Call LUIS recognizer
                var result = this.Recognizer.RecognizeAsync(context, System.Threading.CancellationToken.None);
                
                var topIntent = result?.GetTopScoringIntent();

                switch ((topIntent != null) ? topIntent.Value.intent : null)
                {
                    case null:
                        await context.SendActivity("Failed to get results from LUIS.");
                        break;
                    case "None":
                        await context.SendActivity("Sorry, I don't understand.");
                        break;
                    case "Help":
                        await context.SendActivity("<here's some help>");
                        break;
                    case "Cancel":
                        // Cancel the process.
                        await context.SendActivity("<cancelling the process>");
                        break;
                    case "Weather":
                        // Report the weather.
                        await context.SendActivity("The weather today is sunny.");
                        break;
                    default:
                        // Received an intent we didn't expect, so send its name and score.
                        await context.SendActivity($"Intent: {topIntent.Value.intent} ({topIntent.Value.score}).");
                        break;
                }
            }
        }
    }    
}

```

在語句中辨識出的任何意圖，都會以意圖名稱對評分的對應傳回，而且可以從 `result.Intents` 存取。 系統提供靜態 `LuisRecognizer.topIntent()` 方法來協助簡化尋找結果集之評分最高的意圖。

辨識出的任何實體都會以實體名稱對值的對應傳回，而且可以利用 `results.entities` 來存取。 透過在建立 LuisRecognizer 時傳遞 `verbose=true` 設定，可以傳回其他實體中繼資料。 新增的中繼資料可以利用 `results.entities.$instance` 來存取。

# <a name="javascripttabjs"></a>[JavaScript](#tab/js)

編輯程式碼以供接聽連入的活動，讓其能呼叫 `LuisRecognizer` 來取得 `RecognizerResult`。

```javascript
const { ActivityTypes } = require('botbuilder');
const { LuisRecognizer } = require('botbuilder-ai');

// Listen for incoming requests 
server.post('/api/messages', (req, res) => {
    // Route received request to adapter for processing
    adapter.processActivity(req, res, async (context) => {
        if (context.activity.type === 'message') {
            // Perform a call to LUIS to retrieve results for the user's message.
            const results = await this.luisRecognizer.recognize(turnContext);

            // Since the LuisRecognizer was configured to include the raw results, get the `topScoringIntent` as specified by LUIS.
            const topIntent = results.luisResult.topScoringIntent;
            
            switch (topIntent) {
                case 'None':
                    await context.sendActivity("Sorry, I don't understand.")
                    break;
                case 'Cancel':
                    await context.sendActivity("<cancelling the process>")
                    break;
                case 'Help':
                    await context.sendActivity("<here's some help>");
                    break;
                case 'Weather':
                    await context.sendActivity("The weather today is sunny.");
                    break;                        
                case 'null':
                    await context.sendActivity("Failed to get results from LUIS.")
                    break;
                default:
                    // Received an intent we didn't expect, so send its name and score.
                    await context.sendActivity(`The top intent was ${topIntent}`);
            }
        }
    });
});
```

在語句中辨識出的任何意圖，都會以意圖名稱對評分的對應傳回，而且可以從 `results.intents` 存取。 系統提供靜態 `LuisRecognizer.topIntent()` 方法來協助簡化尋找結果集之評分最高的意圖。


---

請嘗試在 Bot Framework Emulator 中執行 Bot，並對其說出 "Weather" (天氣)、"Help" (說明) 和 "Cancel" (取消) 等字詞。

![執行 Bot](./media/how-to-luis/run-luis-bot.png)

## <a name="extract-entities"></a>擷取實體

除了辨識意圖，LUIS 應用程式也可以擷取實體，也就是滿足使用者要求的重要單字。 例如，在天氣 Bot 中，LUIS 應用程式可能可以從使用者的訊息中擷取天氣預報的位置。

建構對話的常見方法是識別使用者訊息中的任何實體，然後針對必要但未找到的實體發出提示。 接著，後續的步驟會處理對提示的回應。

# <a name="ctabcs"></a>[C#](#tab/cs)

假設使用者的訊息是 "What's the weather in Seattle?" (西雅圖的天氣如何？)。 `LuisRecognizer` 會將 `RecognizerResult` 提供給您，內含具有以下結構的 `Entities` 屬性：

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

下列協助程式函式可以新增至 Bot，以從 LUIS 的 `RecognizerResult` 中取得實體。 其會需要使用 `Newtonsoft.Json.Linq` 程式庫，而您必須將該程式庫新增到 **using** 陳述式。

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

從對話中的多個步驟蒐集實體等資訊時，在狀態中儲存您需要的資訊會很有幫助。 如果找到實體，則可將其新增至適當的狀態欄位。 在對話中，如果目前的步驟已經完成相關聯的欄位，則可以略過提示輸入該資訊的步驟。

# <a name="javascripttabjs"></a>[JavaScript](#tab/js)

假設使用者的訊息是 "What's the weather in Seattle?" (西雅圖的天氣如何？)。 `LuisRecognizer` 會將 `RecognizerResult` 提供給您，內含具有以下結構的 `entities` 屬性：

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

此 `findEntities` 函式會尋找 LUIS 應用程式辨識出符合傳入 `entityName` 的任何實體。


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

從對話中的多個步驟蒐集實體等資訊時，在狀態中儲存您需要的資訊會很有幫助。 如果找到實體，則可將其新增至適當的狀態欄位。 在對話中，如果目前的步驟已經完成相關聯的欄位，則可以略過提示輸入該資訊的步驟。

## <a name="additional-resources"></a>其他資源

如需使用 LUIS 的範例，請查看 [[C#](https://aka.ms/cs-luis-sample)] 或 [[JavaScript](https://aka.ms/js-luis-sample)] 的專案。

## <a name="next-steps"></a>後續步驟

> [!div class="nextstepaction"]
> [使用分派工具結合 LUIS 和 QnA 服務](./bot-builder-tutorial-dispatch.md)
