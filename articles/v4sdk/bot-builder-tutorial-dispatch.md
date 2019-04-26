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
ms.date: 01/15/2019
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 373470b000b168e6e434ed5ed08b35c18ab09a99
ms.sourcegitcommit: aea57820b8a137047d59491b45320cf268043861
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 04/22/2019
ms.locfileid: "59904961"
---
# <a name="use-multiple-luis-and-qna-models"></a>使用多個 LUIS 和 QnA 模型

[!INCLUDE[applies-to](../includes/applies-to.md)]

在本教學課程中，我們示範有 Bot 針對不同案例支援的多個 LUIS 模型和 QnA maker 服務時，如何使用 Dispatch 服務來路由傳送語句。 在此情況下，我們會針對以住家自動化和天氣資訊為主的交談，設定具有多個 LUIS 模型的分派，加上以 QnA Maker 服務根據常見問題集文字檔案，來回答問題作為輸入。 這個範例結合下列服務。

| Name | 說明 |
|------|------|
| HomeAutomation | 使用相關聯的實體資料，辨識住家自動化意圖的 LUIS 應用程式。|
| Weather | 使用位置資料辨識 `Weather.GetForecast` 和 `Weather.GetCondition` 意圖的 LUIS 應用程式。|
| 常見問題集  | 提供 Bot 相關簡單問題解答的 QnA Maker 知識庫。 |

## <a name="prerequisites"></a>必要條件

- 本文中的程式碼是以**採用 Dispatch 的 NLP** 範例為基礎。 您需要採用 [C#](https://aka.ms/dispatch-sample-cs) 或 [JS](https://aka.ms/dispatch-sample-js) 的一份範例。
- 需具備 [Bot 基本概念](bot-builder-basics.md)、[自然語言處理](bot-builder-howto-v4-luis.md)、[QnA Maker](bot-builder-howto-qna.md) 和 [.bot](bot-file-basics.md) 檔案的知識。
- 用於測試的 [Bot Framework 模擬器](https://github.com/Microsoft/BotFramework-Emulator/blob/master/README.md#download)。

## <a name="create-the-services-and-test-the-bot"></a>建立服務及測試 Bot

您可以依照 [ C#](https://aka.ms/dispatch-sample-readme-cs) 或 [JS](https://aka.ms/dispatch-sample-readme-js) 的**讀我檔案**指示，使用命令列介面呼叫建立此 Bot，或遵循下列步驟以使用 Azure、LUIS 和 QnAMaker 使用者介面手動建立 Bot。

 ### <a name="create-your-bot-using-service-ui"></a>使用服務 UI 建立 Bot
 
若要開始手動建立 Bot，請將下列 4 個檔案 (位於 GitHub [BotFramework-Samples](https://aka.ms/botdispatchgitsamples) 存放庫) 下載到本機資料夾：[home-automation.json](https://aka.ms/dispatch-home-automation-json)、[weather.json](https://aka.ms/dispatch-weather-json)、[nlp-with-dispatchDispatch.json](https://aka.ms/dispatch-dispatch-json)、[QnAMaker.tsv](https://aka.ms/dispatch-qnamaker-tsv) 達成此目的的其中一種方法是開啟上面的 GitHub 存放庫連結，按一下 **BotFramework-Samples**，然後將存放庫「複製或下載」到本機電腦。 請注意，這些檔案位於與必要條件中所提範例不同的存放庫。

### <a name="manually-create-luis-apps"></a>手動建立 LUIS 應用程式

登入 [LUIS Web 入口網站](https://www.luis.ai/)。 在 [我的應用程式] 區段之下，選取 [匯入新的應用程式] 索引標籤。 下列對話方塊隨即顯示：

![匯入 LUIS json 檔案](./media/tutorial-dispatch/import-new-luis-app.png)

選取 [選擇應用程式檔案] 按鈕，然後選取下載的檔案 'home-automation.json'。 將選用名稱欄位保留空白。 選取 [完成] 。

在 LUIS 開啟您的住家自動化應用程式後，請選取 [訓練] 按鈕。 這會使用您剛使用 'home-automation.json' 檔案匯入的語句集合，訓練應用程式。

訓練完成時，請選取 [發佈] 按鈕。 下列對話方塊隨即顯示：

![發佈 LUIS 應用程式](./media/tutorial-dispatch/publish-luis-app.png)

選擇 [生產] 環境，然後選取 [發佈] 按鈕。

開啟已發佈的新 LUIS 應用程式後，選取 [管理] 索引標籤。在 [應用程式資訊] 頁面中，記錄 `Application ID` 和 `Display name` 值。 在 [金鑰和端點] 頁面中，記錄 `Authoring Key` 和 `Region` 值。 這些值稍後會由 'nlp-with-dispatch.bot' 檔案使用。

完成後，針對在本機下載的 'weather.json' 和 'nlp-with-dispatchDispatch.json' 檔案重複相同步驟，以「訓練」和「發佈」您的 LUIS 氣象應用程式和 LUIS 分派應用程式。

### <a name="manually-create-qna-maker-app"></a>手動建立 QnA Maker 應用程式

設定 QnA Maker 知識庫的第一個步驟是先在 Azure 中設定 QnA Maker 服務。 若要這麼做，請遵循[這裡](https://aka.ms/create-qna-maker)的逐步指示。 現在登入 [LUIS Web 入口網站](https://qnamaker.ai)。 向下移至步驟 2

![建立 QnA 步驟 2](./media/tutorial-dispatch/create-qna-step-2.png)

及選取
1. 您的 Azure AD 帳戶。
1. 您的 Azure 訂用帳戶名稱。
1. 針對您的 QnA Maker 服務建立的名稱。 (如果您的 Azure QnA 服務並未一開始就出現在此下拉式清單中，請嘗試重新整理頁面。) 

移至步驟 3

![建立 QnA 步驟 3](./media/tutorial-dispatch/create-qna-step-3.png)

提供您的 QnA Maker 知識庫名稱。 此範例中，我們會使用 'sample-qna' 名稱。

移至步驟 4

![建立 QnA 步驟 4](./media/tutorial-dispatch/create-qna-step-4.png)

選取 [+ 新增檔案] 選項並選取下載的檔案 'QnAMaker.tsv'

有其他選取項目可將「閒聊」個性新增至您的知識庫，但我們的範例不包含此選項。

選取 [儲存並訓練]，在完成時選取 [發佈] 索引標籤並發佈您的應用程式。

發佈 QnA Maker 應用程式後，選取 [設定] 索引標籤，然後捲動至 [部署詳細資料]。 記錄以下來自 _Postman_ 範例 HTTP 要求的值。

```
POST /knowledgebases/<Your_Knowledgebase_Id>/generateAnswer
Host: <Your_Hostname>
Authorization: EndpointKey <Your_Endpoint_Key>
```
這些值稍後會由 'nlp-with-dispatch.bot' 檔案使用。

### <a name="manually-update-your-bot-file"></a>手動更新 .bot 檔案

建立所有服務應用程式後，每個服務應用程式的資訊都必須新增至 'nlp-with-dispatch.bot' 檔案。 在您先前下載的 C# 或 JS 範例檔案中開啟此檔案。 在 "type": "luis" 或 "type": "dispatch" 的每個區段中新增下列值

```
"appId": "<Your_Recorded_App_Id>",
"authoringKey": "<Your_Recorded_Authoring_Key>",
"subscriptionKey": "<Your_Recorded_Authoring_Key>",
"version": "0.1",
"region": "<Your_Recorded_Region>",
```

針對 "type": "qna" 的區段，新增下列值：

```
"type": "qna",
"name": "sample-qna",
"id": "201",
"kbId": "<Your_Recorded_Knowledgebase_Id>",
"subscriptionKey": "<Your_Azure_Subscription_Key>", // Used when creating your QnA service.
"endpointKey": "<Your_Recorded_Endpoint_Key>",
"hostname": "<Your_Recorded_Hostname>"
```

完成所有變更時，儲存這個檔案。

### <a name="test-your-bot"></a>測試 Bot

現在使用模擬器執行範例。 當模擬器開啟時，選取 'nlp-with-dispatch.bot' 檔案。

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
下列程式碼會初始化 Bot 的外部服務參考。 例如，LUIS 和 QnaMaker 服務會在此建立。 這些外部服務會根據您的 `.bot` 檔案內容，使用 `BotConfiguration` 類別 來設定。

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

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

範例程式碼會使用預先定義的命名常數來識別 `.bot` 檔案的各種區段。 如果您已在 _nlp-with-dispatch.bot_ 檔案中從原始範例命名修改任何區段名稱，請務必在 **bot.js**、**homeAutomation.js**、**qna.js** 或 **weather.js** 檔案中找出相關聯的常數宣告，並將該項目變更為修改過的名稱。  

```javascript
// In file bot.js
// this is the LUIS service type entry in the .bot file.
const DISPATCH_CONFIG = 'nlp-with-dispatchDispatch';

// In file homeAutomation.js
// this is the LUIS service type entry in the .bot file.
const LUIS_CONFIGURATION = 'Home Automation';

// In file qna.js
// Name of the QnA Maker service in the .bot file.
const QNA_CONFIGURATION = 'sample-qna';

// In file weather.js
// this is the LUIS service type entry in the .bot file.
const WEATHER_LUIS_CONFIGURATION = 'Weather';
```

在 **bot.js** 中，_nlp-with-dispatch.bot_ 組態檔中包含的資訊用於將分派 Bot 連線至各種服務。 每個建構函式都會根據以上詳述的區段名稱，尋找並使用組態檔的適當區段。

```javascript
class DispatchBot {
    constructor(conversationState, userState, botConfig) {
        //...
        this.homeAutomationDialog = new HomeAutomation(conversationState, userState, botConfig);
        this.weatherDialog = new Weather(botConfig);
        this.qnaDialog = new QnA(botConfig);

        this.conversationState = conversationState;
        this.userState = userState;

        // dispatch recognizer
        const dispatchConfig = botConfig.findServiceByNameOrId(DISPATCH_CONFIG);
        //...
```
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

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

在 **bot.js** `onTurn` 方法中，我們會檢查來自使用者的傳入訊息。 如果收到 _ActivityType.Message_ 類型，則會透過 Bot 的 _dispatchRecognizer_ 送出此訊息。

```javascript
if (turnContext.activity.type === ActivityTypes.Message) {
    // determine which dialog should fulfill this request
    // call the dispatch LUIS model to get results.
    const dispatchResults = await this.dispatchRecognizer.recognize(turnContext);
    const dispatchTopIntent = LuisRecognizer.topIntent(dispatchResults);
    //...
 }
```
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

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

當模型產生結果時，它會指出哪一項服務最適合處理語句。 此 Bot 中的程式碼會將要求路由傳送至對應的服務。

```javascript
switch (dispatchTopIntent) {
   case HOME_AUTOMATION_INTENT:
      await this.homeAutomationDialog.onTurn(turnContext);
      break;
   case WEATHER_INTENT:
      await this.weatherDialog.onTurn(turnContext);
      break;
   case QNA_INTENT:
      await this.qnaDialog.onTurn(turnContext);
      break;
   case NONE_INTENT:
      default:
      // Unknown request
       await turnContext.sendActivity(`I do not understand that.`);
       await turnContext.sendActivity(`I can help with weather forecast, turning devices on and off and answer general questions like 'hi', 'who are you' etc.`);
 }
 ```

 在 `homeAutomation.js` 中
 
 ```javascript
 async onTurn(turnContext) {
    // make call to LUIS recognizer to get home automation intent + entities
    const homeAutoResults = await this.luisRecognizer.recognize(turnContext);
    const topHomeAutoIntent = LuisRecognizer.topIntent(homeAutoResults);
    // depending on intent, call turn on or turn off or return unknown
    switch (topHomeAutoIntent) {
       case HOME_AUTOMATION_INTENT:
          await this.handleDeviceUpdate(homeAutoResults, turnContext);
          break;
       case NONE_INTENT:
       default:
         await turnContext.sendActivity(`HomeAutomation dialog cannot fulfill this request.`);
    }
}
```

在 `weather.js` 中

```javascript
async onTurn(turnContext) {
   // Call weather LUIS model.
   const weatherResults = await this.luisRecognizer.recognize(turnContext);
   const topWeatherIntent = LuisRecognizer.topIntent(weatherResults);
   // Get location entity if available.
   const locationEntity = (LOCATION_ENTITY in weatherResults.entities) ? weatherResults.entities[LOCATION_ENTITY][0] : undefined;
   const locationPatternAnyEntity = (LOCATION_PATTERNANY_ENTITY in weatherResults.entities) ? weatherResults.entities[LOCATION_PATTERNANY_ENTITY][0] : undefined;
   // Depending on intent, call "Turn On" or "Turn Off" or return unknown.
   switch (topWeatherIntent) {
      case GET_CONDITION_INTENT:
         await turnContext.sendActivity(`You asked for current weather condition in Location = ` + (locationEntity || locationPatternAnyEntity));
         break;
      case GET_FORECAST_INTENT:
         await turnContext.sendActivity(`You asked for weather forecast in Location = ` + (locationEntity || locationPatternAnyEntity));
         break;
      case NONE_INTENT:
      default:
         wait turnContext.sendActivity(`Weather dialog cannot fulfill this request.`);
   }
}
```

在 `qna.js` 中

```javascript
async onTurn(turnContext) {
   // Call QnA Maker and get results.
   const qnaResult = await this.qnaRecognizer.generateAnswer(turnContext.activity.text, QNA_TOP_N, QNA_CONFIDENCE_THRESHOLD);
   if (!qnaResult || qnaResult.length === 0 || !qnaResult[0].answer) {
       await turnContext.sendActivity(`No answer found in QnA Maker KB.`);
       return;
    }
    // respond with qna result
    await turnContext.sendActivity(qnaResult[0].answer);
}
```
---

## <a name="edit-intents-to-improve-performance"></a>編輯意圖以改善效能

當 Bot 正在執行時，移除類似或重疊的語句有可能改善 Bot 的效能。 例如，假設在 `Home Automation` LUIS 應用程式中，「開燈」這類要求對應至「TurnOnLights」意圖，但「為什麼無法打開燈？」要求則對應至 「無」意圖，因此系統便會將其傳遞至 QnA Maker。 使用分派工具結合 LUIS 應用程式和 QnA Maker 服務時，您必須執行下列動作：

* 從原始的 `Home Automation` LUIS 應用程式移除 "None" 意圖，然後將該意圖中的語句新增至發送器應用程式中的 "None" 意圖。
* 如果您未從原始 LUIS 應用程式中移除 "None" 意圖，則必須在 Bot 中新增邏輯，以將符合 "None" 意圖的訊息傳遞至 QnA maker 服務。

上述兩個動作都會減少 Bot 以「找不到答案」訊息回應使用者的次數。 

## <a name="additional-resources"></a>其他資源

**更新或建立新的 LUIS 模型：** 此範例是以預先設定的 LUIS 模型為基礎。 在[這裡](https://aka.ms/create-luis-model#updating-your-cognitive-models
)可以找到其他資訊，協助您更新此模型，或建立新的 LUIS 模型。

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