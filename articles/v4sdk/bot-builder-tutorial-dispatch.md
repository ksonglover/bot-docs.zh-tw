---
title: 使用分派工具整合多個 LUIS 和 QnA 服務 | Microsoft Docs
description: 了解如何在 Bot 中使用 LUIS 和 QnA Maker。
keywords: Luis, QnA, 分派工具, 多種服務
author: DeniseMak
ms.author: v-demak
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 10/09/2018
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: d1216cf863015e48a18c2f52e7326262c3b271ff
ms.sourcegitcommit: a9270702536eb34dedf5128bf57889c1ffcd7f4c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/11/2018
ms.locfileid: "49088890"
---
# <a name="integrate-multiple-luis-and-qna-services-with-the-dispatch-tool"></a>使用分派工具整合多個 LUIS 和 QnA 服務

[!INCLUDE [pre-release-label](../includes/pre-release-label.md)]

<!--TODO: Add the JS sections back in and update them once the JS sample is working.-->

本教學課程將說明如何使用分派工具產生的 LUIS 模型，將 Bot 與多個 Language Understanding Intelligent Service (LUIS) 應用程式和 QnAMaker 服務進行整合。 這個範例結合下列服務。

| 服務類型 | 名稱 | 說明 |
|------|------|------|
| LUIS 應用程式 | HomeAutomation | 使用相關聯的實體資料辨識住家自動化意圖。|
| LUIS 應用程式 | Weather | 使用位置資料辨識 Weather.GetForecast 和 Weather.GetCondition 意圖。|
| QnAMaker 服務 | 常見問題集  | 提供 Bot 相關簡單問題的解答。 |

本文的程式碼取自 **NLP with Dispatch** 範例 [[C#](https://aka.ms/dispatch-sample-cs)<!-- | [JS](https://aka.ms/dispatch-sample-js)-->]。

如需語言服務概觀，請參閱[語言理解](bot-builder-concept-luis.md)。 請參閱 [LUIS](bot-builder-howto-v4-luis.md) 和 [QnA Maker](bot-builder-howto-qna.md) 的操作說明文章，以取得在 Bot 中實作這些服務的指示。

您可以遵循讀我檔案中的指示，以取得設定及測試 Bot 的範例，或者您也可以往前跳至[程式碼的相關注意事項](#notes-about-the-code)。

## <a name="create-the-services-and-test-the-bot"></a>建立服務及測試 Bot

遵循範例的讀我檔案指示。 您將使用 CLI 工具來建立和發佈這些服務，以及在組態 (**.bot**) 檔中更新其相關資訊。

1. 複製或提取範例存放庫。
1. 安裝 BotBuilder CLI 工具。
1. 手動設定必要的服務。

### <a name="test-your-bot"></a>測試 Bot

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

使用 Visual Studio 或 Visual Studio Code 啟動 Bot。

<!--
# [JavaScript](#tab/javascript)
-->

---

使用 Bot Framework 模擬器連線到 Bot。

以下是我們所納入服務涵蓋的一些輸入：

* QnA Maker
  * `hi`、`good morning`
  * `what are you`、`what do you do`
* LUIS (住家自動化)
  * `turn on bedroom light`
  * `turn off bedroom light`
  * `make some coffee`
* LUIS (天氣)
  * `whats the weather in chennai india`
  * `what's the forecast for bangalore`
  * `show me the forecast for nebraska`

## <a name="notes-about-the-code"></a>關於程式碼的注意事項

### <a name="packages"></a>封裝

此範例會使用下列套件。

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

這些 [NuGet 套件](https://docs.microsoft.com/en-us/nuget/tools/package-manager-ui#updating-a-package)的最新 v4 版本。

* `Microsoft.Bot.Builder`
* `Microsoft.Bot.Builder.AI.Luis`
* `Microsoft.Bot.Builder.AI.QnA`
* `Microsoft.Bot.Builder.Integration.AspNet.Core`
* `Microsoft.Bot.Configuration`

<!--
# [JavaScript](#tab/javascript)

Download the [LUIS Dispatch sample][DispatchBotJs].  Install the required packages, including the `botbuilder-ai` package for LUIS and QnA Maker, using npm:

* `npm install --save botbuilder`
* `npm install --save botbuilder-ai`
-->

---

### <a name="botbuilder-cli-tools"></a>BotBuilder CLI 工具

此範例會使用這些 [BotBuilder CLI 工具](https://aka.ms/botbuilder-tools-readme) (可透過 npm 取得) 來建立、訓練及發佈 LUIS、QnA Maker 和分派服務；並且在您的 Bot 組態 (**.bot**) 檔中記錄這些服務的相關資訊。

* [分派](https://aka.ms/botbuilder-tools-dispatch)
* [LUDown](https://aka.ms/botbuilder-tools-ludown-readme)
* [LUIS](https://aka.ms/botbuilder-tools-luis)
* [MSBot](https://aka.ms/botbuilder-tools-msbot-readme)
* [QnAMaker](https://aka.ms/botbuilder-tools-qnaMaker)

> [!TIP]
> 若要確定您有最新版的 npm 和 CLI 工具，請執行下列命令。
>
> ```shell
> npm i -g npm dispatch ludown luis-apis msbot qnamaker
> ```

在您使用工具來設定服務之後，此範例的 **.bot** 檔案應該如下所示。 (您可以執行 `msbot secret -n` 來加密這個檔案中的敏感性值。)

```json
{
    "name": "NLP-With-Dispatch-Bot",
    "description": "",
    "services": [
        {
            "type": "endpoint",
            "name": "development",
            "id": "http://localhost:3978/api/messages",
            "appId": "",
            "appPassword": "",
            "endpoint": "http://localhost:3978/api/messages"
        },
        {
            "type": "luis",
            "name": "Home Automation",
            "appId": "<your-home-automation-luis-app-id>",
            "version": "0.1",
            "authoringKey": "<your-luis-authoring-key>",
            "subscriptionKey": "<your-cognitive-services-subscription-key>",
            "region": "westus",
            "id": "110"
        },
        {
            "type": "luis",
            "name": "Weather",
            "appId": "<your-weather-luis-app-id>",
            "version": "0.1",
            "authoringKey": "<your-luis-authoring-key>",
            "subscriptionKey": "<your-cognitive-services-subscription-key>",
            "region": "westus",
            "id": "92"
        },
        {
            "type": "qna",
            "name": "Sample QnA",
            "kbId": "<your-qna-knowledge-base-id>",
            "subscriptionKey": "<your-cognitive-services-subscription-key>",
            "endpointKey": "<your-qna-endpoint-key>",
            "hostname": "<your-qna-host-name>",
            "id": "184"
        },
        {
            "type": "dispatch",
            "name": "NLP-With-Dispatch-BotDispatch",
            "appId": "<your-dispatch-app-id>",
            "authoringKey": "<your-luis-authoring-key>",
            "subscriptionKey": "<your-cognitive-services-subscription-key>",
            "version": "Dispatch",
            "region": "westus",
            "serviceIds": [
                "110",
                "92",
                "184"
            ],
            "id": "27"
        }
    ],
    "padlock": "",
    "version": "2.0"
}
```

### <a name="connecting-to-the-services-from-your-bot"></a>從 Bot 連線到服務

若要連線到分派、LUIS 和 QnA Maker 服務，您的 Bot 會從 **.bot** 檔案提取資訊。

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

在 **Startup.cs** 中，`ConfigureServices` 會讀入組態檔，而 `InitBotServices` 會使用該資訊來初始化服務。 每次建立 Bot 時，系統就會使用已註冊的 `BotServices` 物件初始化該 Bot。 以下是這兩種方法的相關部分。

```csharp
public void ConfigureServices(IServiceCollection services)
{
    //...
    var botConfig = BotConfiguration.Load(botFilePath ?? @".\BotConfiguration.bot", secretKey);
    services.AddSingleton(sp => botConfig
        ?? throw new InvalidOperationException($"The .bot config file could not be loaded. ({botConfig})"));

    // Retrieve current endpoint.
    var environment = _isProduction ? "production" : "development";
    var service = botConfig.Services.Where(s => s.Type == "endpoint" && s.Name == environment).FirstOrDefault();
    if (!(service is EndpointService endpointService))
    {
        throw new InvalidOperationException($"The .bot file does not contain an endpoint with name '{environment}'.");
    }

    var connectedServices = InitBotServices(botConfig);

    services.AddSingleton(sp => connectedServices);
    //...
}
```

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
/// <summary>
/// Depending on the intent from Dispatch, routes to the right LUIS model or QnA service.
/// </summary>
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

/// <summary>
/// Dispatches the turn to the request QnAMaker app.
/// </summary>
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

/// <summary>
/// Dispatches the turn to the requested LUIS model.
/// </summary>
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

執行 `dispatch eval` 會產生 **Summary.html** 檔案，其中提供語言模型的預測效能統計資料。

> [!TIP]
> 您可在任何 LUIS 應用程式中執行 `dispatch eval`，而不只是在分派工具建立的 LUIS 應用程式中執行。

### <a name="edit-intents-for-duplicates-and-overlaps"></a>編輯重複和重疊項目的意圖

檢閱在 **Summary.html** 中標記為重複項目的範例表達方式，並移除類似或重疊的範例。 例如，假設在 `Home Automation` LUIS 應用程式中，「開燈」這類要求對應至「TurnOnLights」意圖，但「為什麼無法打開燈？」要求則對應至 「無」意圖，因此系統便會將其傳遞至 QnA Maker。 使用分派工具結合 LUIS 應用程式和 QnA Maker 服務時，您必須執行下列動作：

* 從原始的 `Home Automation` LUIS 應用程式移除「無」意圖，然後在發送器應用程式中，新增該意圖表達方式到「無」意圖。
* 如果您未從原始 LUIS 應用程式中移除「無」意圖，則必須在 Bot 中新增邏輯，以將符合該意圖之訊息傳遞至 QnA maker 服務。

> [!TIP]
> 請檢閱 [Language Understanding 的最佳作法](./bot-builder-concept-luis.md#best-practices-for-language-understanding)，以取得如何改善語言模型效能的秘訣。

## <a name="to-clean-up-resources-from-this-sample"></a>若要清除此範例中的資源

此範例會建立數個應用程式和資源。 您可遵循下列指示進行刪除。

### <a name="luis-resources"></a>LUIS 資源

1. 登入 [luis.ai](https://www.luis.ai) 入口網站。
1. 移至 [我的應用程式] 頁面。
1. 選取此範例所建立的應用程式。
   * `Home Automation`
   * `Weather`
   * `NLP-With-Dispatch-BotDispatch`
1. 按一下 [刪除]，然後按一下 [確定] 進行確認。

### <a name="qna-maker-resources"></a>QnA Maker 資源

1. 登入 [qnamaker.ai](https://www.qnamaker.ai/) 入口網站。
1. 移至 [我的知識庫] 頁面。
1. 按一下 `Sample QnA` 知識庫的刪除按鈕，然後按一下 [刪除] 進行確認。

### <a name="azure-resources"></a>Azure 資源

> [!WARNING]
> 請勿刪除任何其他應用程式或服務所依賴的資源。

1. 登入 [Azure 入口網站](https://portal.azure.com/)。
1. 針對您為此範例建立的認知服務資源，移至其 [概觀] 頁面。
1. 按一下 [刪除]，然後按一下 [是] 進行確認。

## <a name="additional-resources"></a>其他資源

* [語言理解](bot-builder-concept-luis.md)
* [設計知識型 Bot](../bot-service-design-pattern-knowledge-base.md)
* [設定語音預備](../bot-service-manage-speech-priming.md)
* [將 LUIS 用於 Language Understanding](bot-builder-howto-v4-luis.md)
* [使用 QnA Maker 回答問題](bot-builder-howto-qna.md)
