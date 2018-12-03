---
title: 使用多個 LUIS 和 QnA 模型 | Microsoft Docs
description: 了解如何在 Bot 中使用 LUIS 和 QnA Maker。
keywords: Luis, QnA, 分派工具, 多種服務, 路由意圖
author: DeniseMak
ms.author: v-demak
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 11/26/2018
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 9b0ddf5cf8af61048ba78f10824c9573da82fc08
ms.sourcegitcommit: a722f960cd0a8513d46062439eb04de3a0275346
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 11/27/2018
ms.locfileid: "52336268"
---
# <a name="use-multiple-luis-and-qna-models"></a>使用多個 LUIS 和 QnA 模型

[!INCLUDE [pre-release-label](../includes/pre-release-label.md)]

在本教學課程中，我們示範有 Bot 針對不同案例支援的多個 LUIS 模型和 QnA maker 服務時，如何使用 Dispatch 服務來路由傳送語句。 在此情況下，我們會針對以住家自動化和天氣資訊為主的交談，設定具有多個 LUIS 模型的分派，加上以 QnA Maker 服務根據常見問題集文字檔案來回答問題作為輸入。 這個範例結合下列服務。

| 名稱 | 說明 |
|------|------|
| HomeAutomation | 使用相關聯的實體資料，辨識住家自動化意圖的 LUIS 應用程式。|
| Weather | 使用位置資料辨識 `Weather.GetForecast` 和 `Weather.GetCondition` 意圖的 LUIS 應用程式。|
| 常見問題集  | 提供 Bot 相關簡單問題解答的 QnA Maker 知識庫。 |

## <a name="prerequisites"></a>必要條件

- 本文中的程式碼是以**採用 Dispatch 的 NLP** 範例為基礎。 您需要採用 [C#](https://aka.ms/dispatch-sample-cs) 或 [JS](https://aka.ms/dispatch-sample-js) 的一份範例。
- 需具備 [Bot 基本概念](bot-builder-basics.md)、[自然語言處理](bot-builder-howto-v4-luis.md)、[QnA Maker](bot-builder-howto-qna.md) 和 [.bot](bot-file-basics.md) 檔案的知識。
- 用於測試的 [Bot Framework 模擬器](https://github.com/Microsoft/BotFramework-Emulator/blob/master/README.md#download)。

## <a name="create-the-services-and-test-the-bot"></a>建立服務及測試 Bot

遵循 [C#](https://github.com/Microsoft/BotBuilder-Samples/blob/master/samples/csharp_dotnetcore/14.nlp-with-dispatch/README.md) 或 [JS](https://github.com/Microsoft/BotBuilder-Samples/blob/master/samples/javascript_nodejs/14.nlp-with-dispatch/README.md) 的 **README** 指示，以使用模擬器建置和執行範例。 

以下是我們已納入服務所涵蓋的一些問題和命令，可供您參考：

* QnA Maker
  * `hi`、`good morning`
  * `what are you`、`what do you do`
* LUIS (住家自動化)
  * `turn on bedroom light`
  * `turn off bedroom light`
  * `make some coffee`
* LUIS (天氣)
  * `whats the weather in redmond washington`
  * `what's the forecast for london`
  * `show me the forecast for nebraska`

### <a name="connecting-to-the-services-from-your-bot"></a>從 Bot 連線到服務

若要連線到分派、LUIS 和 QnA Maker 服務，您的 Bot 會從 **.bot** 檔案提取資訊。

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

在 **Startup.cs** 中，`ConfigureServices` 會讀入組態檔，而 `InitBotServices` 會使用該資訊來初始化服務。 每次建立 Bot 時，系統就會使用已註冊的 `BotServices` 物件初始化該 Bot。 以下是這兩種方法的相關部分。

```csharp
public void ConfigureServices(IServiceCollection services)
{
    //...
    var botConfig = BotConfiguration.Load(botFilePath ?? @".\nlp-with-dispatch.bot", secretKey);
    services.AddSingleton(sp => botConfig
        ?? throw new InvalidOperationException($"The .bot config file could not be loaded. ({botConfig})"));

    // ...
    
    var connectedServices = InitBotServices(botConfig);
    services.AddSingleton(sp => connectedServices);
    
    services.AddBot<NlpDispatchBot>(options =>
    {
          
          // The Memory Storage used here is for local bot debugging only. 
          // When the bot is restarted, everything stored in memory will be gone.

          Storage dataStore = new MemoryStorage();

          // ...

          // Create Conversation State object.
          // The Conversation State object is where we persist anything at the conversation-scope.

          var conversationState = new ConversationState(dataStore);
          options.State.Add(conversationState);
     });
}

```
下列程式碼會初始化 Bot 的外部服務參考。 例如，LUIS 和 QnaMaker 服務會在此建立。 這些外部服務會根據您的 ".bot" 檔案內容，使用 `BotConfiguration` 類別 來設定。

```csharp
private static BotServices InitBotServices(BotConfiguration config)
{
    var qnaServices = new Dictionary<string, QnAMaker>();
    var luisServices = new Dictionary<string, LuisRecognizer>();

    foreach (var service in config.Services)
    {
        switch (service.Type)
        {
            case ServiceTypes.Luis:
                {
                    // ...
                    var app = new LuisApplication(luis.AppId, luis.AuthoringKey, luis.GetEndpoint());
                    var recognizer = new LuisRecognizer(app);
                    luisServices.Add(luis.Name, recognizer);
                    break;
                }

            case ServiceTypes.Dispatch:
                // ...
                var dispatchApp = new LuisApplication(dispatch.AppId, dispatch.AuthoringKey, dispatch.GetEndpoint());

                // Since the Dispatch tool generates a LUIS model, we use the LuisRecognizer to resolve the
                // dispatching of the incoming utterance.
                var dispatchARecognizer = new LuisRecognizer(dispatchApp);
                luisServices.Add(dispatch.Name, dispatchARecognizer);
                break;

            case ServiceTypes.QnA:
                {
                    // ...
                    var qnaEndpoint = new QnAMakerEndpoint()
                    {
                        KnowledgeBaseId = qna.KbId,
                        EndpointKey = qna.EndpointKey,
                        Host = qna.Hostname,
                    };

                    var qnaMaker = new QnAMaker(qnaEndpoint);
                    qnaServices.Add(qna.Name, qnaMaker);
                    break;
                }
        }
    }

    return new BotServices(qnaServices, luisServices);
}
```

<!--
# [JavaScript](#tab/javascript)

```javascript
```
-->

---

### <a name="calling-the-services-from-your-bot"></a>從 Bot 呼叫服務

Bot 邏輯會針對合併的分派模型，檢查使用者輸入。

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

在 **NlpDispatchBot.cs** 檔案中，Bot 的建構函式會取得我們在啟動時註冊的 `BotServices` 物件。

```csharp
private readonly BotServices _services;

public NlpDispatchBot(BotServices services)
{
    _services = services ?? throw new System.ArgumentNullException(nameof(services));

    //...
}
```

在 Bot 的 `OnTurnAsync` 方法中，我們會根據分派模型檢查來自使用者的傳入訊息。

```csharp
// Get the intent recognition result
var recognizerResult = await _services.LuisServices[DispatchKey].RecognizeAsync(context, cancellationToken);
var topIntent = recognizerResult?.GetTopScoringIntent();

if (topIntent == null)
{
    await context.SendActivityAsync("Unable to get the top intent.");
}
else
{
    await DispatchToTopIntentAsync(context, topIntent, cancellationToken);
}
```

<!--
# [JavaScript](#tab/javascript)

```javascript
```
-->

---

### <a name="working-with-the-recognition-results"></a>使用辨識結果

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

當模型產生結果時，它會指出哪一項服務最適合處理語句。 此 Bot 中的程式碼會將要求路由傳送至對應的服務，然後概述來自所呼叫服務的回應。

```csharp
// Depending on the intent from Dispatch, routes to the right LUIS model or QnA service.
private async Task DispatchToTopIntentAsync(
    ITurnContext context,
    (string intent, double score)? topIntent,
    CancellationToken cancellationToken = default(CancellationToken))
{
    const string homeAutomationDispatchKey = "l_Home_Automation";
    const string weatherDispatchKey = "l_Weather";
    const string noneDispatchKey = "None";
    const string qnaDispatchKey = "q_sample-qna";

    switch (topIntent.Value.intent)
    {
        case homeAutomationDispatchKey:
            await DispatchToLuisModelAsync(context, HomeAutomationLuisKey);

            // Here, you can add code for calling the hypothetical home automation service, passing in any entity
            // information that you need.
            break;
        case weatherDispatchKey:
            await DispatchToLuisModelAsync(context, WeatherLuisKey);

            // Here, you can add code for calling the hypothetical weather service,
            // passing in any entity information that you need
            break;
        case noneDispatchKey:
            // You can provide logic here to handle the known None intent (none of the above).
            // In this example we fall through to the QnA intent.
        case qnaDispatchKey:
            await DispatchToQnAMakerAsync(context, QnAMakerKey);
            break;

        default:
            // The intent didn't match any case, so just display the recognition results.
            await context.SendActivityAsync($"Dispatch intent: {topIntent.Value.intent} ({topIntent.Value.score}).");
            break;
    }
}

// Dispatches the turn to the request QnAMaker app.
private async Task DispatchToQnAMakerAsync(
    ITurnContext context,
    string appName,
    CancellationToken cancellationToken = default(CancellationToken))
{
    if (!string.IsNullOrEmpty(context.Activity.Text))
    {
        var results = await _services.QnAServices[appName].GetAnswersAsync(context).ConfigureAwait(false);
        if (results.Any())
        {
            await context.SendActivityAsync(results.First().Answer, cancellationToken: cancellationToken);
        }
        else
        {
            await context.SendActivityAsync($"Couldn't find an answer in the {appName}.");
        }
    }
}


// Dispatches the turn to the requested LUIS model.
private async Task DispatchToLuisModelAsync(
    ITurnContext context,
    string appName,
    CancellationToken cancellationToken = default(CancellationToken))
{
    await context.SendActivityAsync($"Sending your request to the {appName} system ...");
    var result = await _services.LuisServices[appName].RecognizeAsync(context, cancellationToken);

    await context.SendActivityAsync($"Intents detected by the {appName} app:\n\n{string.Join("\n\n", result.Intents)}");

    if (result.Entities.Count > 0)
    {
        await context.SendActivityAsync($"The following entities were found in the message:\n\n{string.Join("\n\n", result.Entities)}");
    }
}
```

<!--
# [JavaScript](#tab/javascript)

```javascript
```
-->

---

## <a name="evaluate-the-dispatchers-performance"></a>評估發送器的效能

雖然有時在 LUIS 應用程式和 QnA maker 服務中都會提供使用者訊息做為範例，但分派工具產生的結合 LUIS 應用程式並不會針對所有輸入值妥善執行。 您可以使用 `eval` 選項來檢查應用程式的效能。

```shell
dispatch eval
```

執行 `dispatch eval` 會產生 **Summary.html** 檔案，其中提供語言模型的預測效能統計資料。 您可在任何 LUIS 應用程式中執行 `dispatch eval`，而不只是在分派工具建立的 LUIS 應用程式中執行。

### <a name="edit-intents-for-duplicates-and-overlaps"></a>編輯重複和重疊項目的意圖

檢閱在 **Summary.html** 中標記為重複項目的範例表達方式，並移除類似或重疊的範例。 例如，假設在 `Home Automation` LUIS 應用程式中，「開燈」這類要求對應至「TurnOnLights」意圖，但「為什麼無法打開燈？」要求則對應至 「無」意圖，因此系統便會將其傳遞至 QnA Maker。 使用分派工具結合 LUIS 應用程式和 QnA Maker 服務時，您必須執行下列動作：

* 從原始的 `Home Automation` LUIS 應用程式移除「無」意圖，然後在發送器應用程式中，新增該意圖表達方式到「無」意圖。
* 如果您未從原始 LUIS 應用程式中移除「無」意圖，則必須在 Bot 中新增邏輯，以將符合該意圖之訊息傳遞至 QnA maker 服務。


## <a name="additional-resources"></a>其他資源 

**刪除資源：** 此範例會建立一些應用程式和資源，您可以使用下面所列的步驟將其刪除，但不得刪除「任何其他應用程式或服務」所依賴的資源。 

_LUIS 資源_
1. 登入 [luis.ai](https://www.luis.ai) 入口網站。
1. 移至 [我的應用程式] 頁面。
1. 選取此範例所建立的應用程式。
   * `Home Automation`
   * `Weather`
   * `NLP-With-Dispatch-BotDispatch`
1. 按一下 [刪除]，然後按一下 [確定] 進行確認。

_QnA Maker 資源_
1. 登入 [qnamaker.ai](https://www.qnamaker.ai/) 入口網站。
1. 移至 [我的知識庫] 頁面。
1. 按一下 `Sample QnA` 知識庫的刪除按鈕，然後按一下 [刪除] 進行確認。

**最佳做法：** 若要改善此範例中使用的服務，請參閱 [LUIS](https://docs.microsoft.com/en-us/azure/cognitive-services/luis/luis-concept-best-practices) 及 [QnA Maker](https://docs.microsoft.com/en-us/azure/cognitive-services/qnamaker/concepts/best-practices) 的最佳做法。
