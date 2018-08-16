---
title: 將 LUIS 用於 Language Understanding | Microsoft Docs
description: 了解如何搭配 Bot Builder SDK 將 LUIS 用於自然語言理解。
keywords: Language Understanding, LUIS, intent, recognizer, entities, middleware, 意圖, 辨識器, 實體, 中介軟體
author: ivorb
ms.author: v-demak
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 4/30/17
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 9a04a73e8ac8716ec528586e069aa2d16d90e28f
ms.sourcegitcommit: f95702d27abbd242c902eeb218d55a72df56ce56
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/19/2018
ms.locfileid: "39300594"
---
# <a name="using-luis-for-language-understanding"></a>將 LUIS 用於 Language Understanding

讓您的 Bot 能透過交談和上下文了解使用者的意思，並不是簡單的工作，但這樣的功能可讓您的 Bot 使用起來更有自然交談的感覺。 Language Understanding (稱為 LUIS) 能讓您這麼做，使您的 Bot 可以辨識使用者訊息的意圖，這樣使用者就能使用更自然的語言，且更順利地引導交談流程。 如果您需要 LUIS 與 Bot 之整合方式的詳細背景資訊，請參閱[Bot 的語言理解](./bot-builder-concept-LUIS.md)。 

此主題逐步引導您設定簡單的 Bot，並使用 LUIS 辨識幾個不同的意圖。

## <a name="installing-packages"></a>安裝套件

首先，請確認您有 LUIS 所需的套件。

# <a name="ctabcs"></a>[C#](#tab/cs)

對以下 NuGet 套件的 v4 發行前版本[加入參考](https://docs.microsoft.com/en-us/nuget/tools/package-manager-ui) \(機器翻譯\)：


* `Microsoft.Bot.Builder.Ai.LUIS`

# <a name="javascripttabjs"></a>[JavaScript](#tab/js)

您可以在您的專案中透過 npm，加入對 botbuilder 和 botbuilder-ai 套件的參考：

* `npm install --save botbuilder@preview`
* `npm install --save botbuilder-ai@preview`

---


## <a name="set-up-middleware-to-use-your-luis-app"></a>設定中介軟體以使用您 LUIS 應用程式

首先，設定 _LUIS 應用程式_，它是您在 [www.luis.ai](https://www.luis.ai) \(英文\) 建立的服務。 該 LUIS 應用程式可以被定型成能夠辨識特定意圖。 您可以在 [LUIS 網站](https://www.luis.ai) \(英文\) 上找到如何建立 LUIS 應用程式的詳細資料。

針對此範例，您將只使用能辨識 Help (說明)、Cancel (取消) 和 Weather (天氣) 意圖的 LUIS 示範應用程式；應用程式識別碼已經在範例程式碼中。 您將需要有能夠登入 [www.luis.ai](https://www.luis.ai) \(英文\) 的認知服務金鑰，然後從 [使用者設定] > [撰寫金鑰] 複製金鑰。

> [!NOTE] 
> 若要為此範例中使用的公用 LUIS 應用程式建立您自己的複本，請複製 [FirstSimpleBotExample.json](https://github.com/Microsoft/LUIS-Samples/blob/master/examples/simple-bot-example/FirstSimpleBotExample.json) LUIS 檔案。 然後[匯入 LUIS 應用程式](https://docs.microsoft.com/en-us/azure/cognitive-services/luis/create-new-app#import-new-app)，接著[定型](https://docs.microsoft.com/en-us/azure/cognitive-services/LUIS/luis-how-to-train)並[發佈](https://docs.microsoft.com/en-us/azure/cognitive-services/LUIS/publishapp)它。 使用您新的 LUIS 應用程式的應用程式識別碼來取代範例程式碼中的公用應用程式識別碼。

透過直接將 LUIS 應用程式新增至 Bot 的中介軟體堆疊，來將您的 Bot 設定成針對收到的每個使用者訊息呼叫您的 LUIS 應用程式。 中介軟體將辨識的結果儲存在內容物件中，然後可以由您的 Bot 邏輯存取。

# <a name="ctabcs"></a>[C#](#tab/cs)
從 Echo Bot 範本開始，然後開啟 **Startup.cs**。 

為 `Microsoft.Bot.Builder.Ai.LUIS` 新增 `using` 陳述式

```csharp
using Microsoft.AspNetCore.Builder;
using Microsoft.AspNetCore.Hosting;
using Microsoft.Bot.Builder.BotFramework;
using Microsoft.Bot.Builder.Integration.AspNet.Core;
// add this
using Microsoft.Bot.Builder.Ai.LUIS;
using Microsoft.Extensions.Configuration;
using Microsoft.Extensions.DependencyInjection;
using Microsoft.Extensions.Logging;
using System;
```

更新 `Startup.cs` 檔案中的 `ConfigureServices` 方法，以新增連線到您 LUIS 應用程式的 `LuisRecognizerMiddleware` 物件。 

```csharp
// This method gets called by the runtime. Use this method to add services to the container.
public void ConfigureServices(IServiceCollection services)
{
    services.AddBot<EchoBot>(options =>
    {
        options.CredentialProvider = new ConfigurationCredentialProvider(Configuration);
. 
        options.Middleware.Add(new CatchExceptionMiddleware<Exception>(async (context, exception) =>
        {
            await context.TraceActivity("EchoBot Exception", exception);
            await context.SendActivity("Sorry, it looks like something went wrong!");
        }));

        // The Memory Storage used here is for local bot debugging only. When the bot
        // is restarted, anything stored in memory will be gone. 
        IStorage dataStore = new MemoryStorage();

        options.Middleware.Add(new ConversationState<EchoState>(dataStore));

        // Add LUIS recognizer as middleware
        options.Middleware.Add(
            new LuisRecognizerMiddleware(
                new LuisModel(
                    // This appID is for a public app that's made available for demo purposes
                    "eb0bf5e0-b468-421b-9375-fdfb644c512e",
                    // You can use it by replacing <subscriptionKey> with your Authoring Key
                    // which you can find at https://www.luis.ai under User settings > Authoring Key
                    "<subscriptionKey>",
                    // The location-based URL begins with "https://<region>.api.cognitive.microsoft.com", where region is the region associated with the key you are using. Some examples of regions are `westus`, `westcentralus`, `eastus2`, and `southeastasia`.
                    new Uri("https://westus.api.cognitive.microsoft.com/luis/v2.0/apps/"))));
    });
}
```

<!--TODO: DOES PUBLIC APP WORK WITH KEYS IN DIFFERENT REGIIONS? --> 貼上來自 [https://www.luis.ai](https://www.luis.ai) \(英文\) 的訂用帳戶金鑰，以取代 `<subscriptionKey>`。

> [!NOTE] 
> 如果您是使用自己的 LUIS 應用程式而不是公用的，則您可以從 [https://www.luis.ai](https://www.luis.ai) \(英文\) 取得 LUIS 應用程式的識別碼、訂用帳戶金鑰和 URL。 
>
>您可以透過登入 [LUIS 網站](https://www.luis.ai) \(英文\) 來找到 `LuisModel` 中使用的基底 URL，請移至 [發佈] 索引標籤，然後查看 [資源和金鑰] 下的 [端點] 欄。 基底 URL 是**端點 URL** 的訂用帳戶識別碼和其他參數之前的部分。

# <a name="javascripttabjs"></a>[JavaScript](#tab/js)

首先，要求/匯入 [LuisRecognizer](https://github.com/Microsoft/botbuilder-js/tree/master/doc/botbuilder-ai/classes/botbuilder_ai.luisrecognizer.md) \(英文\) 類別並建立 LUIS 模型的執行個體：

```javascript
const { LuisRecognizer } = require('botbuilder-ai');

const model = new LuisRecognizer({
    // This appID is for a public app that's made available for demo purposes
    // You can use it by providing your LUIS subscription key
     appId: 'eb0bf5e0-b468-421b-9375-fdfb644c512e',
    // replace subscriptionKey with your Authoring Key
    // your key is at https://www.luis.ai under User settings > Authoring Key 
    subscriptionKey: '<your subscription key>',
    // The serviceEndpoint URL begins with "https://<region>.api.cognitive.microsoft.com", where region is the region associated with the key you are using. Some examples of regions are `westus`, `westcentralus`, `eastus2`, and `southeastasia`.
    serviceEndpoint: 'https://westus.api.cognitive.microsoft.com'
});
```

將模型新增至中介軟體堆疊：

```javascript
adapter.use(model);
```


> [!NOTE] 
> 如果您是使用自己的 LUIS 應用程式而不是公用的，則您可以從 [https://www.luis.ai](https://www.luis.ai) \(英文\) 取得 LUIS 應用程式的識別碼、訂用帳戶識別碼和 URL。 

---



LUIS 語言理解現在已經插入到您的 Bot 中。 接下來，讓我們看看如何從儲存在內容物件上的 LUIS 模型取得意圖。


## <a name="get-the-intent-from-the-turn-context"></a>從回合內容取得意圖

之後可以使用每一個交談回合的內容，從您的 Bot 存取 LUIS 的結果。

# <a name="ctabcs"></a>[C#](#tab/cs)

若要讓您的 Bot 只根據 LUIS 應用程式偵測到的意圖傳送及回覆，請將 `OnTurn` 中的程式碼用以下程式碼取代：

```cs
using System.Threading.Tasks;
using Microsoft.Bot;
using Microsoft.Bot.Builder;
using Microsoft.Bot.Builder.Ai.LUIS;
using Microsoft.Bot.Builder.Core.Extensions;
using Microsoft.Bot.Schema;

namespace Bot_Builder_Echo_Bot1
{
    public class EchoBot : IBot
    {
        /// <summary>
        /// Every Conversation turn for our EchoBot will call this method. In here
        /// the bot checks the Activty type to verify it's a message, checks the /// intent from the LUIS recognizer, and sends a reply based on the recognized intent
        /// </summary>
        /// <param name="context">Turn-scoped context containing all the data needed
        /// for processing this conversation turn. </param>        
        public async Task OnTurn(ITurnContext context)
        {
            // This bot is only handling Messages
            if (context.Activity.Type == ActivityTypes.Message)
            {

                var result = context.Services.Get<RecognizerResult>
                    (LuisRecognizerMiddleware.LuisRecognizerResultKey);
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
                        // Cancel the process.
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

# <a name="javascripttabjs"></a>[JavaScript](#tab/js)

```javascript
// Listen for incoming activity 
server.post('/api/messages', (req, res) => {
    // Route received activity to adapter for processing
    adapter.processActivity(req, res, async (context) => {
        if (context.activity.type === 'message') {
            const results = model.get(context);
            const topIntent = LuisRecognizer.topIntent(results);
            switch (topIntent) {

                case 'Cancel':
                    await context.sendActivity("<cancelling the process>")
                    break;
                case 'Help':
                    await context.sendActivity("<here's some help>");
                    break;
                case 'Weather':
                    await context.sendActivity("The weather today is sunny.");
                    break;
                case 'None':                    
                    await context.sendActivity("Sorry, I don't understand.")
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
辨識出的任何實體都會以實體名稱對值的對應傳回，而且可以透過 `results.entities` 存取。 透過在建立 LuisRecognizer 時傳遞 `verbose=true` 設定，可以傳回其他實體中繼資料。 新增的中繼資料可以透過 `results.entities.$instance` 來存取。

---

<!-- TODO: SHOW RUNNING THE FIRST BOT --> 請嘗試在 Bot Framework Emulator 中執行 Bot，並對它說出 "Weather" (天氣)、"Help" (說明) 和 "Cancel" (取消) 等字詞。

![執行 Bot](./media/how-to-luis/run-luis-bot.png)


## <a name="using-a-luis-recognizer-with-conversation-state"></a>搭配交談狀態使用 LUIS 辨識器
<!-- TBD, complete example --> 如果您對使用者的回覆有超過一個回合，您可能會想透過儲存交談狀態來追蹤您在交談中的位置。 您可以使用 LUIS 提供的意圖，以協助您設定交談的狀態資料，例如是否已開始或完成交談主題。

LUIS 辨識器中介軟體會在您 Bot 的每一回合執行，以便您從使用者收到的每個訊息取得意圖。 如果您想要根據意圖來開始多回合交談流程，其中一個方法是略過變更主題的邏輯，直到您完成目前的邏輯。

# <a name="ctabcs"></a>[C#](#tab/cs)

在 EchoState.cs 中，將 EchoState 變更為下面這樣：

```csharp
public class ConversationStateInfo 
{
    public bool WeatherTopicStarted  { get; set; }
    public bool HelpTopicStarted  { get; set; }
    public bool CancelTopicStarted  { get; set; }
}
```

在 Startup.cs 中，將 ConversationState 的初始化變更成使用 `ConversationStateInfo`。
```cs
    options.Middleware.Add(new ConversationState<ConversationStateInfo>(dataStore));
```

在 EchoBot.cs 中，編輯 `OnTurn`：
```cs
public async Task OnTurn(ITurnContext context)
{
    // This bot is only handling Messages
    if (context.Activity.Type == ActivityTypes.Message)
    {
        var text = context.Activity.Text;
        var conversationState = context.GetConversationState<ConversationStateInfo>() ?? new ConversationStateInfo();

        // Here, you can add some other logic based on the topic flags in conversation state
        // For example, if you know that a particular topic was started in a previous turn,
        // you can send the reply for that topic and bypass getting the intent from LUIS 
        if (conversationState.WeatherTopicStarted)
        {
            // Set this flag to false since this reply concludes the topic.
            conversationState.WeatherTopicStarted = false;
            // Assume that they responded to the prompt with a location.
            await context.SendActivity($"The weather in {text} is sunny.");
        }
        else
        {
            var result = context.Services.Get<RecognizerResult>
            (LuisRecognizerMiddleware.LuisRecognizerResultKey);
            var topIntent = result?.GetTopScoringIntent();
            switch ((topIntent != null) ? topIntent.Value.intent : null)
            {
                case null:
                    await context.SendActivity("Failed to get results from LUIS.");
                    break;
                case "None":
                    await context.SendActivity("Sorry, I don't understand.");
                    break;
                case "Weather":
                    conversationState.WeatherTopicStarted = true;
                    await context.SendActivity($"Looks like you want a weather forecast. What city do you want the forecast for?");

                    break;
                case "Help":
                    conversationState.HelpTopicStarted = true;
                    await context.SendActivity("<here's some help>");
                    break;
                case "Cancel":
                    // Cancel the process.
                    conversationState.CancelTopicStarted = true;
                    await context.SendActivity("<cancelling the process>");
                    break;
                default:
                    // Received an intent we didn't expect, so send its name and score.
                    await context.SendActivity($"Intent: {topIntent.Value.intent} ({topIntent.Value.score}).");
                    break;
            }
        }
    }
}
```

# <a name="javascripttabjs"></a>[JavaScript](#tab/js)

新增 `LuisRecognizer` 之後新增交談狀態中介軟體。

```javascript
const model = new LuisRecognizer({
    // This appID is for a public app that's made available for demo purposes
    // You can use it by providing your LUIS subscription key
     appId: 'eb0bf5e0-b468-421b-9375-fdfb644c512e',
    // replace subscriptionKey with your Authoring Key
    // your key is at https://www.luis.ai under User settings > Authoring Key 
    subscriptionKey: '<your subscription>',
    // The serviceEndpoint URL begins with "https://<region>.api.cognitive.microsoft.com", where region is the region associated with the key you are using. Some examples of regions are `westus`, `westcentralus`, `eastus2`, and `southeastasia`.
    serviceEndpoint: 'https://westus.api.cognitive.microsoft.com'
});
adapter.use(model)

// Add conversation state middleware
const conversationState = new ConversationState(new MemoryStorage());
adapter.use(conversationState);
```

然後您可以儲存狀態，以表示哪些主題已經啟動。

```javascript
// Listen for incoming activities 
server.post('/api/messages', (req, res) => {
    // Route received activity to adapter for processing
    adapter.processActivity(req, res, async(context) => {
        if (context.activity.type === 'message') {
            var utterance = context.activity.text;
            
            // Check topic flags in conversation state 
            if (conversationState.weatherTopicStarted) 
            {
                // Assume the user's message is a reply to the bot's prompt for a location
                await context.sendActivity(`The weather in ${utterance} is sunny.`);
                // This conversation flow is now finished. Set flag to false,
                // so that on the next turn the user can ask for another weather forecast.
                conversationState.WeatherTopicStarted = false;
            }
            // To add more steps to the other topics
            // you could check the topic flags here
            else 
            {
                const results = model.get(context);
                const topIntent = LuisRecognizer.topIntent(results);
                switch (topIntent) {
                    case 'None':
                        //Add app logic when there is no result
                        await context.sendActivity("<null case>")
                        break;
                    case 'Cancel':
                        conversationState.cancelTopicStarted = true;
                        await context.sendActivity("<cancelling the process>")
                        break;
                    case 'Help':
                        conversationState.helpTopicStarted = true;
                        await context.sendActivity("<here's some help>");
                        break;
                    case 'Weather':
                        conversationState.weatherTopicStarted = true;
                        await context.sendActivity("Looks like you want a weather forecast. What city do you want the forecast for?");
                        break;
                    default:
                        // Add app logic for the recognition results.
                        await context.sendActivity(`Received this intent: ${topIntent}`);
                }
            }
        }
    });
});
```

---

請嘗試在 Bot Framework Emulator 中 Bot，並留意 "get weather" 現在是兩個回合的交談流程。

![執行 Bot](./media/how-to-luis/run-luis-bot-2step-weather.png)


## <a name="using-luis-with-dialogs"></a>搭配使用 LUIS 和對話

如果您要使用 LUIS 提供的意圖來觸發多回合交談流程，使用對話來封裝此流程會很有用。 此範例 Bot 可搭配偵測用於觸發家庭自動化對話或天氣對話之意圖的 LUIS 應用程式使用。

> [!NOTE] 
> 若要為此範例中使用的公用 LUIS 應用程式建立您自己複本，請複製 [WeatherOrHomeAutomation.json](https://github.com/Microsoft/LUIS-Samples/blob/master/examples/simple-bot-example/WeatherOrHomeAutomation.json) LUIS 檔案。 然後[匯入 LUIS 應用程式](https://docs.microsoft.com/en-us/azure/cognitive-services/luis/create-new-app#import-new-app)，接著[定型](https://docs.microsoft.com/en-us/azure/cognitive-services/LUIS/luis-how-to-train)並[發佈](https://docs.microsoft.com/en-us/azure/cognitive-services/LUIS/publishapp)它。 使用您新的 LUIS 應用程式的應用程式識別碼來取代範例程式碼中的公用應用程式識別碼。

# <a name="ctabcs"></a>[C#](#tab/cs)

首先，修改 Startup.cs 中的 `ConfigureServices`，以為您的 LUIS 應用程式新增中介軟體。 將 `EchoBot` 重新命名為 `LuisDialogBot`。
在此範例中，新增為 `LuisRecognizerMiddleware` 的 LUIS 應用程式會偵測 `homeautomation` 或 `weather` 意圖，以觸發具有那些名稱的對話。 

```csharp
public void ConfigureServices(IServiceCollection services)
{  
    // Rename EchoBot to LuisDialogBot
    services.AddBot<LuisDialogBot>(options =>
    {
        options.CredentialProvider = new ConfigurationCredentialProvider(Configuration); 
        options.Middleware.Add(new CatchExceptionMiddleware<Exception>(async (context, exception) =>
        {
            await context.TraceActivity("EchoBot Exception", exception);
            await context.SendActivity("Sorry, it looks like something went wrong!");
        }));

        // The Memory Storage used here is for local bot debugging only. When the bot
        // is restarted, anything stored in memory will be gone. 
        IStorage dataStore = new MemoryStorage();

        // Use Dictionary<string, object> for the conversation state type
        options.Middleware.Add(new ConversationState<Dictionary<string, object>>(dataStore));

        // Add LUIS recognizer as middleware
        options.Middleware.Add(
            new LuisRecognizerMiddleware(
                new LuisModel(
                    // This appID is for a public app that's made available for demo purposes
                    "428affb6-7650-46ac-9184-68c00a4f1729",
                    // You can use it by replacing <subscriptionKey> with your Authoring Key
                    // which you can find at https://www.luis.ai under User settings > Authoring Key
                    "<subscriptionKey>",
                    // The location-based URL begins with "https://<region>.api.cognitive.microsoft.com", where region is the region associated with the key you are using. Some examples of regions are `westus`, `westcentralus`, `eastus2`, and `southeastasia`.
                    new Uri("https://westus.api.cognitive.microsoft.com/luis/v2.0/apps/"))));
    });
}
```

將 **EchoBot.cs** 重新命名為 **LuisDialogBot.cs**，並將 `EchoBot` 類別重新命名為 `LuisDialogBot`。 然後，在 **LuisDialogBot.cs** 中新增下列 `using` 陳述式。

```csharp
using System;
using System.Collections.Generic;
using System.Threading.Tasks;
using Microsoft.Bot.Builder;
using Microsoft.Bot.Builder.Ai.LUIS;
using Microsoft.Bot.Builder.Core.Extensions;
using Microsoft.Bot.Builder.Dialogs;
using Microsoft.Bot.Builder.Prompts;
using Microsoft.Bot.Schema;
```

在 **LuisDialogBot.cs** 中，將以下程式碼新增到 `LuisDialogBot` 類別。

```csharp
    public class LuisDialogBot : IBot
    {
        private DialogSet _dialogs;

        public LuisDialogBot()
        {
            _dialogs = new DialogSet();

            _dialogs.Add("homeautomation", CreateHomeAutomationWaterfall());
            _dialogs.Add("weather", CreateWeatherWaterfall());
            _dialogs.Add("weather_city", new Builder.Dialogs.TextPrompt());
        }

        // App ID for a separate LUIS app used to tell if the user wants to turn the lights on or off
        // The `homeautomation` dialogs uses results from this app to determine the "on/off" argument. 
        private static LuisModel luisHomeAutomation =
            new LuisModel("76feb726-515b-44c4-acc9-adb216965a58", "SUBSCRIPTION-KEY", new System.Uri("https://westus.api.cognitive.microsoft.com/luis/v2.0/apps/"));

        public async Task OnTurn(ITurnContext context)
        {
            // This bot is only handling Messages
            if (context.Activity.Type == ActivityTypes.Message)
            {

                // Create a dialog context
                var state = ConversationState<Dictionary<string, object>>.Get(context);
                var dc = _dialogs.CreateContext(context, state);

                // Run the next dialog step
                await dc.Continue();
                
                // Check if any dialog has responded on this turn
                if (!context.Responded)
                {

                    var luisResult = context.Services.Get<RecognizerResult>(LuisRecognizerMiddleware.LuisRecognizerResultKey);
                    var topIntent = luisResult?.GetTopScoringIntent();
                    var utterance = context.Activity.Text;
                    var dialogArgs = new Dictionary<string, object>();

                    switch ((topIntent != null) ? topIntent.Value.intent.ToLowerInvariant() : null)
                    {
                        case "homeautomation":
                            // The Homeautomation_TurnOn and Homeautomation_TurnOff dialogs 
                            // use results from a separate LUIS app.
                            // The results determine the "on/off" argument to pass to the dialog. 
                            var recognizerHomeAutomation = new LuisRecognizer(luisHomeAutomation);
                            RecognizerResult recognizerResult = await recognizerHomeAutomation.Recognize(utterance, System.Threading.CancellationToken.None);
                            var topHomeAutoIntent = recognizerResult.GetTopScoringIntent().intent;


                            dialogArgs.Add("IntentFromHomeAuto", "");
                            switch ((topHomeAutoIntent != null) ? topHomeAutoIntent.ToLowerInvariant() : null)
                            {
                                case "homeautomation_turnon":
                                    dialogArgs["Intent_HomeAutomation"] = "on";
                                    await dc.Begin("homeautomation", dialogArgs);
                                    break;
                                case "homeautomation_turnoff":
                                    dialogArgs["Intent_HomeAutomation"] = "off";
                                    await dc.Begin("homeautomation", dialogArgs);
                                    break;
                                case null:
                                    await dc.Begin("homeautomation", null);
                                    break;
                                default:
                                    dialogArgs["Intent_HomeAutomation"] = topHomeAutoIntent;
                                    await dc.Begin("homeautomation", dialogArgs);
                                    break;
                            }
                            break;
                        case "weather":
                            dialogArgs.Add("LuisResult", luisResult);
                            await dc.Begin("weather", dialogArgs);
                            break;
                        case null:
                            await context.SendActivity($"Couldn't get a result from LUIS. You said: {utterance}");
                            break;
                        default:
                            // The intent didn't match any case, so just display the recognition results.
                            await context.SendActivity($"you said: {utterance}");
                            await context.SendActivity($"Recognized intent: {topIntent.Value.intent}.");

                            break;

                    }

                }
            }
        }

    }


```

在 `OnTurn` 之後，於 `LuisDialogBot` 類別內新增實作對話的程式碼。

```csharp
        // The home automation waterfall has one step
        private WaterfallStep[] CreateHomeAutomationWaterfall()
        {
            return new WaterfallStep[] {
                TurnLightsOnOrOff
            };
        }

        // The weather waterfall has two steps
        private WaterfallStep[] CreateWeatherWaterfall()
        {
            return new WaterfallStep[] {
                AskWeatherLocation,
                SendWeatherReport
            };
        }

        /// <summary>
        /// This is the first step and only of the home automation dialog.
        /// </summary>
        /// <param name="dc"></param>
        /// <param name="args">Can be "on", "off", another string for an intent name, or null.
        /// null indicates that no result was received from which to get an intent name.</param>
        /// <param name="next"></param>
        /// <returns></returns>
        private async Task TurnLightsOnOrOff(DialogContext dc, IDictionary<string, object> args, SkipStepFunction next)
        {
            var intentFromHomeAutomation = args["Intent_HomeAutomation"];
            if (args != null)
            {
                switch (intentFromHomeAutomation)
                {
                    case "on":
                    case "off":
                        await dc.Context.SendActivity($"Turning {intentFromHomeAutomation} your lights!");
                        break;
                    default:
                        await dc.Context.SendActivity($"Intent detected by homeautomation was: {intentFromHomeAutomation}, but the home automation system doesn't support that yet.");
                        break;
                }

            }
            else
            {
                await dc.Context.SendActivity($"You said {dc.Context.Activity.AsMessageActivity().Text}. Unable to get a result from which to determine on/off/other operation");
            }

            await dc.End();

        }

        // This is the first step of the weather dialog
        private async Task AskWeatherLocation(DialogContext dc, IDictionary<string, object> args, SkipStepFunction next)
        {
            var dialogState = dc.ActiveDialog.State as IDictionary<string, object>;
            
            if (args["LuisResult"] is RecognizerResult luisResult)
            {
                var location = GetEntity<string>(luisResult, "Weather_Location");
                if (!string.IsNullOrEmpty(location))
                {
                    dialogState.Add("Location", location);
                }
            }

            // Save info back to the dialog instance
            dc.ActiveDialog.State = dialogState;

            if (!dialogState.ContainsKey("Location"))
            {
                await dc.Prompt("weather_city", "What city do you want the weather for?");
            }
            else
            {
                // We've set the location parameter for the weather report,
                // so go to the next step in the waterfall
                await dc.Continue();
            }
        }

        // This the second step of the weather dialog
        private async Task SendWeatherReport(DialogContext dc, IDictionary<string, object> args, SkipStepFunction next)
        {
            var dialogState = dc.ActiveDialog.State as IDictionary<string, object>;
            if (args != null)
            {
                if (!dialogState.ContainsKey("Location"))
                {   // Interpret args as a reply to prompt only if they didn't give location
                    TextResult locationResult = (TextResult)args;
                    dialogState.Add("Location", locationResult.Text);
                }

            }

            // You can add some logic that uses the location 
            // to get the weather from a weather service instead of hard-coding it
            await dc.Context.SendActivity($"The weather forecast for '{dialogState["Location"]}' is sunny and 70 degrees F.");

            dc.ActiveDialog.State = new Dictionary<string, object>(); // clear the dialog state 
            await dc.End();
        }
```

新增此協助程式函式。

```cs
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

# <a name="javascripttabjs"></a>[JavaScript](#tab/js)

如果您還沒安裝對話程式庫，請先安裝它。 

```cmd
npm install --save botbuilder-dialogs
```

首先，建立 LUIS 應用程式，並使用 `adapter.use` 將它新增至您的 Bot：

```javascript
const { BotFrameworkAdapter, ConversationState, MemoryStorage, TurnContext } = require('botbuilder');
const { LuisRecognizer } = require('botbuilder-ai');
const { DialogSet } = require('botbuilder-dialogs');
const restify = require('restify');

// Create server
let server = restify.createServer();
server.listen(process.env.port || process.env.PORT || 3978, function () {
    console.log(`${server.name} listening to ${server.url}`);
});

// Create adapter
const adapter = new BotFrameworkAdapter({
    appId: process.env.MICROSOFT_APP_ID,
    appPassword: process.env.MICROSOFT_APP_PASSWORD
});

// Create LuisRecognizer 
// The LUIS application is public, meaning you can use your own subscription key to test the applications.
const luisRecognizer = new LuisRecognizer({
    appId: '1fefd4a7-ae5b-4e63-99f7-0cf499a1423b',
    subscriptionKey: process.env.LUIS_SUBSCRIPTION_KEY,
    serviceEndpoint: 'https://westus.api.cognitive.microsoft.com/',
    verbose: true
});

// Add the recognizer to your bot
adapter.use(luisRecognizer);

// create conversation state
const conversationState = new ConversationState(new MemoryStorage());
adapter.use(conversationState);

// register some dialogs for usage with the intents detected by the LUIS app
const dialogs = new DialogSet();
```

接下來，加入對話：

```javascript

dialogs.add('HomeAutomation_TurnOn', [
    async (dialogContext) => {
        const state = conversationState.get(dialogContext.context);
        // state.homeAutomationTurnOn counts how many times this dialog was called 
        state.homeAutomationTurnOn = state.homeAutomationTurnOn ? state.homeAutomationTurnOn + 1 : 1;
        await dialogContext.context.sendActivity(`${state.homeAutomationTurnOn}: You reached the "HomeAutomation_TurnOn" dialog.`);

        await dialogContext.end();
    }
]);

dialogs.add('Weather_GetForecast', [
    async (dialogContext) => {
        const state = conversationState.get(dialogContext.context);
        // state.weatherGetForecast counts how many times this dialog was called  
        state.weatherGetForecast = state.weatherGetForecast ? state.weatherGetForecast + 1 : 1;
        await dialogContext.context.sendActivity(`${state.weatherGetForecast}: You reached the "Weather_GetForecast" dialog.`);

        await dialogContext.end();
    }
]);

dialogs.add('None', [
    async (dialogContext) => {
        const state = conversationState.get(dialogContext.context);
        // state.None counts how many times this dialog was called        
        state.None = state.None ? state.None + 1 : 1;
        await dialogContext.context.sendActivity(`${state.None}: You reached the "None" dialog.`);

        await dialogContext.end();
    }
]);
```

在您的 Bot 中，根據辨識出的意圖叫用每個對話：
```javascript
server.post('/api/messages', (req, res) => {
    adapter.processActivity(req, res, async (context) => {
        if (context.activity.type === 'message') {
            const state = conversationState.get(context);
            const dc = dialogs.createContext(context, state);

            // Retrieve the LUIS results from our LUIS application
            const luisResults = luisRecognizer.get(context);

            // Extract the top intent from LUIS and use it to select which dialog to start
            // "NotFound" is the intent name for when no top intent can be found.
            const topIntent = LuisRecognizer.topIntent(luisResults, "NotFound");

            const isMessage = context.activity.type === 'message';
            if (isMessage) {
                switch (topIntent) {
                    case 'homeautomation':                    
                        await dc.begin("HomeAutomation_TurnOn", luisResults);
                        break;
                    case 'weather':                    
                        await dc.begin("Weather_GetForecast", luisResults);
                        break;
                    case 'None':
                        await dc.begin("None");
                        break;
                    case 'NotFound':
                        await context.sendActivity(`Sorry, I didn't get any results from LUIS.`);
                        break;
                    default:
                        // handle intents for which we have no dialog
                        await context.sendActivity(`You reached the ${topIntent} intent.`);
                        break;
                }
            }
            
            if (!context.responded) {
                await dc.continue();
                if (!context.responded && isMessage) {
                    await dc.context.sendActivity(`Hi! I'm the LUIS dialog bot. Say something and LUIS will decide how the message should be routed.`);
                }
            }
        }
    });
});
```

嘗試在 Bot Framework Emulator 中執行 Bot，並對它說 "Turn on the lights" (開燈) 和 "Get weather in Seattle" (西雅圖天氣) 之類的話。

![執行 Bot](./media/how-to-luis/run-luis-bot-js-single-step-dialog.png)

---

## <a name="extract-entities"></a>擷取實體

除了辨識意圖，LUIS 應用程式也可以擷取實體，也就是滿足使用者要求的重要單字。 例如，在天氣對話範例中，LUIS 應用程式可能可以從使用者的訊息中擷取天氣預報的位置。 

對話常見的使用方法是識別使用者訊息中的任何實體，然後針對必要但未找到的實體發出提示。 接著，後續的對話步驟會處理對提示的回應。

# <a name="ctabcs"></a>[C#](#tab/cs)

假設使用者的訊息是 "What's the weather in Seattle?" (西雅圖的天氣如何？)。 針對此範例中的 Bot，[LuisRecognizer](https://docs.microsoft.com/en-us/dotnet/api/microsoft.bot.builder.ai.luis.luisrecognizer) 會提供您包含 [`Entities` 屬性](https://docs.microsoft.com/en-us/dotnet/api/microsoft.bot.builder.core.extensions.recognizerresult#properties-)的 [RecognizerResult](https://docs.microsoft.com/en-us/dotnet/api/microsoft.bot.builder.core.extensions.recognizerresult)，其結構如下：

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

您新增到 `LuisDialogBot` 類別的下列協助程式函式會從 LUIS 的 `RecognizerResult` 取得實體。

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

當在從對話中的多個步驟收集實體等資訊時，將您需要的資訊以對話特定的狀態儲存會很有幫助。 例如，在 `AskWeatherLocation` 對話中，實體是從傳遞至對話的 LUIS 結果中擷取出的。 如果 `Weather_Location` 找到實體，則會將它新增至對話狀態。

```cs

        // This is the first step of the weather dialog
        private async Task AskWeatherLocation(DialogContext dc, IDictionary<string, object> args, SkipStepFunction next)
        {
            var dialogState = dc.ActiveDialog.State as IDictionary<string, object>;
            
            if (args["LuisResult"] is RecognizerResult luisResult)
            {
                var location = GetEntity<string>(luisResult, "Weather_Location");
                if (!string.IsNullOrEmpty(location))
                {
                    dialogState.Add("Location", location);
                }
            }

            // Save info back to the dialog instance
            dc.ActiveDialog.State = dialogState;

            if (!dialogState.ContainsKey("Location"))
            {
                await dc.Prompt("weather_city", "What city do you want the weather for?");
            }
            else
            {
                // We've set the location parameter for the weather report,
                // so go to the next step in the waterfall
                await dc.Continue();
            }
        }
```

在對話的第二個步驟中，您可以從對話執行個體的狀態取得儲存在上一個步驟的資訊 (及使用者對位置提示的回覆)，並使用它傳送天氣預報給使用者。

```cs

// This the second step of the weather dialog
private async Task SendWeatherReport(DialogContext dc, IDictionary<string, object> args, SkipStepFunction next)
{
    var dialogState = dc.ActiveDialog.State as IDictionary<string, object>;
    if (args != null)
    {
        if (!dialogState.ContainsKey("Location"))
        {   // Interpret args as a reply to prompt only if they didn't give location
            TextResult locationResult = (TextResult)args;
            dialogState.Add("Location", locationResult.Text);
        }

    }

    // You can add some logic that uses the location and date
    // to get the weather from a weather service instead of hard-coding it
    await dc.Context.SendActivity($"The weather forecast for '{dialogState["Location"]}' is sunny and 70 degrees F.");
    
    // clear the dialog state 
    dc.ActiveDialog.State = new Dictionary<string, object>(); 
    await dc.End();
}
```

# <a name="javascripttabjs"></a>[JavaScript](#tab/js)

例如，我們假設使用者的訊息是 "What's the weather in Seattle?" (西雅圖的天氣如何？)。 針對此範例中的 Bot，[LuisRecognizer](https://docs.microsoft.com/en-us/javascript/api/botbuilder-ai/luisrecognizer) 會提供您包含 `entities` 屬性的 [RecognizerResult](https://docs.microsoft.com/en-us/javascript/api/botbuilder-core-extensions/recognizerresult)，其結構如下：

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

此 `findEntities` 函式會尋找 LUIS 應用程式辨識出任何的 `Weather_Location` 實體。

<!-- TODO: Turn into a waterfall -->

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

從 `Weather_GetForecast` 對話呼叫 `findEntities`。

```javascript
// Pass the LUIS recognizer result to the args parameter
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
```

實體是從傳遞至 `dc.begin` 的 LUIS 結果中擷取出的。

```javascript
await dc.begin("Weather_GetForecast", luisResults);
```
---

## <a name="additional-resources"></a>其他資源

如需 LUIS 的背景資訊，請參閱 [Language Understanding](./bot-builder-concept-luis.md)。

如需如何建置用於這些範例之 LUIS 應用程式的資訊，請參閱[適用於 Bot Builder 的 LUIS 應用程式](https://aka.ms/luis-bot-examples) \(英文\)。

LUIS 可以結合其他認知服務，讓您的 Bot 更強大。 請參閱[使用分派工具結合 LUIS 應用程式和 QnA 服務](./bot-builder-tutorial-dispatch.md) \(英文\)，以了解如何在您的 Bot 中結合 QnA 和 Language Understanding (LUIS)。

## <a name="next-steps"></a>後續步驟


> [!div class="nextstepaction"]
> [產生強型別 LUIS 結果](./bot-builder-howto-v4-luisgen.md)


