---
title: 使用 QnA Maker 回答問題 | Microsoft Docs
description: 了解如何在 Bot 中使用 QnA Maker。
keywords: 問題和答案, QnA, 常見問題集, QnA Maker
author: ivorb
ms.author: v-ivorb
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: cognitive-services
ms.date: 02/27/2019
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 494af4cc7936c8b191280d3f3f6cd73e7bc7d364
ms.sourcegitcommit: cf3786c6e092adec5409d852849927dc1428e8a2
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 03/01/2019
ms.locfileid: "57224896"
---
# <a name="use-qna-maker-to-answer-questions"></a>使用 QnA Maker 回答問題

[!INCLUDE [pre-release-label](../includes/pre-release-label.md)]

您可以使用 QnA Maker 服務，對 Bot 新增問題和解答支援。 在建立您自己的 QnA Maker 服務時，其中一項基本需求是在服務中植入問題和解答。 許多時候，問題和解答早已存在於常見問題集或其他文件等內容中。 其他時候，您則會想要以更自然的對話方式自訂問題的解答。

在本主題中，我們將建立知識庫並且在 Bot 中使用。

## <a name="prerequisites"></a>必要條件
- [QnA Maker](https://www.qnamaker.ai/) 帳戶
- 本文中的程式碼是以 **QnA Maker** 範例為基礎。 您需要一份 [C# 範例](https://aka.ms/cs-qna)或 [Javascript 範例](https://aka.ms/js-qna-sample)。
- [Bot Framework 模擬器](https://github.com/Microsoft/BotFramework-Emulator/blob/master/README.md#download) (英文)
- [Bot 基本概念](bot-builder-basics.md)、[QnA Maker](https://docs.microsoft.com/en-us/azure/cognitive-services/qnamaker/overview/overview) 及 [.bot](bot-file-basics.md) 檔案的知識。

## <a name="create-a-qna-maker-service-and-publish-a-knowledge-base"></a>建立 QnA Maker 服務及發佈知識庫
1. 首先，您需要建立 [QnA Maker 服務](https://docs.microsoft.com/en-us/azure/cognitive-services/qnamaker/how-to/set-up-qnamaker-service-azure)。
1. 接下來，您將使用 `smartLightFAQ.tsv` 檔案 (位於專案的 CognitiveModels 資料夾中) 來建立知識庫。 用於建立、定型及發佈 QnA Maker[知識庫](https://docs.microsoft.com/en-us/azure/cognitive-services/qnamaker/quickstarts/create-publish-knowledge-base)的步驟會列於 QnA Maker 文件中。 當您遵循這些步驟時，請將您的 KB 命名為 `qna`，並使用 `smartLightFAQ.tsv` 檔案來填入 KB。
> 注意： 本文也可用來存取您自己使用者所開發的 QnA Maker 知識庫。

## <a name="obtain-values-to-connect-your-bot-to-the-knowledge-base"></a>取得值，將您的 Bot 連線到知識庫
1. 在 [QnA Maker](https://www.qnamaker.ai/) 網站上，選取您的知識庫。
1. 開啟您的知識庫，然後選取 [設定]。 記錄針對「服務名稱」顯示的值。 使用 QnA Maker 入口網站介面時，此值很適合用於尋找您感興趣的知識庫。 但不會用於將 Bot 應用程式連線到此知識庫。 
1. 向下捲動以尋找 [部署詳細資料] 及記錄下列值：
   - POST /knowledgebases/<Your_Knowledge_Base_Id>/getAnswers
   - 主機：<Your_Hostname>/qnamaker
   - 授權：EndpointKey <Your_Endpoint_Key>
   
這三個值會為您的應用程式提供透過 Azure QnA 服務連線到 QnA Maker 知識庫所需的資訊。  

## <a name="update-the-bot-file"></a>更新 .bot 檔案
首先，將存取您的知識庫所需的資訊 (包括主機名稱、端點金鑰和知識庫識別碼 (kbId)) 新增至 `qnamaker.bot`。 這些是您在 QnA Maker 中從您知識庫的 [設定] 儲存的值。 
> 注意： 如果您要將 QnA Maker 知識庫的存取權新增到現有 Bot 應用程式中，請務必將 "type": "qna" 區段 (如下所示) 新增到 .bot 檔案中。 此區段內的 "name" 值會提供從您的應用程式存取此資訊所需的金鑰。

```json
{
  "name": "qnamaker",
  "services": [
    {
      "type": "endpoint",
      "name": "development",
      "endpoint": "http://localhost:3978/api/messages",
      "appId": "",
      "appPassword": "",
      "id": "25"    
    },
    {
      "type": "qna",
      "name": "QnABot",
      "kbId": "<Your_Knowledge_Base_Id>",
      "subscriptionKey": "",
      "endpointKey": "<Your_Endpoint_Key>",
      "hostname": "<Your_Hostname>",
      "id": "117"
    }
  ],
  "padlock": "",
   "version": "2.0"
}
```

# <a name="ctabcs"></a>[C#](#tab/cs)

請確定您已為專案安裝 **Microsoft.Bot.Builder.AI.QnA** NuGet 套件。

接下來，我們會在 **BotServices.cs** 中初始化 `BotServices` 類別的新執行個體，這會從 .bot 檔案擷取上述資訊。 外部服務是使用 BotConfiguration 類別來設定。

```csharp
using Microsoft.Bot.Builder.AI.QnA;
using Microsoft.Bot.Configuration;
```

```csharp
public BotServices(BotConfiguration botConfiguration)
{
    foreach (var service in botConfiguration.Services)
    {
        switch (service.Type)
        {
            case ServiceTypes.QnA:
                {
                    // Create a QnA Maker that is initialized and suitable for passing
                    // into the IBot-derived class (QnABot).
                    var qna = service as QnAMakerService;

                    // ...

                    var qnaEndpoint = new QnAMakerEndpoint()
                    {
                        KnowledgeBaseId = qna.KbId,
                        EndpointKey = qna.EndpointKey,
                        Host = qna.Hostname,
                    };

                    var qnaMaker = new QnAMaker(qnaEndpoint);
                    QnAServices.Add(qna.Name, qnaMaker);
                    break;
                }
        }
    }
}
```

然後在 **QnABot.cs** 中，我們將這個 QnAMaker 執行個體提供給 Bot。 如果您要存取自己的知識庫，請變更以下所示的「歡迎」訊息，為您的使用者提供實用的初始指示。 這個類別也是靜態變數 `QnAMakerKey` 的定義所在。 這會指向 .bot 檔案內包含存取 QnA Maker 知識庫所需連線資訊的區段。

```csharp
public class QnABot : IBot
{
    public static readonly string QnAMakerKey = "QnABot";

    private const string WelcomeText = @"This bot will introduce you to QnA Maker.
                                         Ask a question to get started.";
    private readonly BotServices _services;

    public QnABot(BotServices services)
    {
        _services = services ?? throw new System.ArgumentNullException(nameof(services));
        if (!_services.QnAServices.ContainsKey(QnAMakerKey))
        {
            throw new System.ArgumentException(
                $"Invalid configuration. Please check your '.bot' file for a QnA service named '{QnAMakerKey}'.");
        }
    }
}
```

# <a name="javascripttabjs"></a>[JavaScript](#tab/js)

在我們的範例中，啟動程式碼位於 **index.js** 檔案、Bot 邏輯的程式碼位於 **bot.js** 檔案，而其他組態資訊位於 **qnamaker.bot** 檔案。

在 **index.js** 檔案中，我們會讀入組態資訊以產生 QnA Maker 服務及初始化 Bot。

將 `QNA_CONFIGURATION` 的值更新為 .bot 檔案中的 "name": 值。 這是進入 .bot 檔案 "type": "qna" 區段的關鍵，其中包含用來存取 QnA Maker 知識庫的連線參數。

```js
// Name of the QnA Maker service stored in "name" field of .bot file. 
const QNA_CONFIGURATION = '<SERVICE_NAME_FROM_.BOT_FILE>';

// Get endpoint and QnA Maker configurations by service name.
const endpointConfig = botConfig.findServiceByNameOrId(BOT_CONFIGURATION);
const qnaConfig = botConfig.findServiceByNameOrId(QNA_CONFIGURATION);

// Map the contents to the required format for `QnAMaker`.
const qnaEndpointSettings = {
    knowledgeBaseId: qnaConfig.kbId,
    endpointKey: qnaConfig.endpointKey,
    host: qnaConfig.hostname
};

// Create adapter...

// Create the QnAMakerBot.
let bot;
try {
    bot = new QnAMakerBot(qnaEndpointSettings, {});
} catch (err) {
    console.error(`[botInitializationError]: ${ err }`);
    process.exit();
}
```

然後我們會建立 HTTP 伺服器並接聽傳入要求，這會對我們的 Bot 邏輯產生呼叫。

```js
// Create HTTP server.
let server = restify.createServer();
server.listen(process.env.port || process.env.PORT || 3978, function() {
    console.log(`\n${ server.name } listening to ${ server.url }.`);
    console.log(`\nGet Bot Framework Emulator: https://aka.ms/botframework-emulator.`);
    console.log(`\nTo talk to your bot, open qnamaker.bot file in the emulator.`);
});

// Listen for incoming requests.
server.post('/api/messages', (req, res) => {
    adapter.processActivity(req, res, async (turnContext) => {
        await bot.onTurn(turnContext);
    });
});
```

---

## <a name="calling-qna-maker-from-your-bot"></a>從您的 Bot 呼叫 QnA Maker

# <a name="ctabcs"></a>[C#](#tab/cs)

當您的 Bot 需要來自 QnAMaker 的解答時，從您的 Bot 程式碼呼叫 `GetAnswersAsync()`，以根據目前的內容取得適當解答。 如果您要存取自己的知識庫，請變更以下的「沒有解答」 訊息，為您的使用者提供實用的指示。

```csharp
// Check QnA Maker model
var response = await _services.QnAServices[QnAMakerKey].GetAnswersAsync(turnContext);
if (response != null && response.Length > 0)
{
    await turnContext.SendActivityAsync(response[0].Answer, cancellationToken: cancellationToken);
}
else
{
    var msg = @"No QnA Maker answers were found. This example uses a QnA Maker Knowledge Base that focuses on smart light bulbs.
                To see QnA Maker in action, ask the bot questions like 'Why won't it turn on?' or 'I need help'.";
    await turnContext.SendActivityAsync(msg, cancellationToken: cancellationToken);
}

    /// ...
```

# <a name="javascripttabjs"></a>[JavaScript](#tab/js)

<!-- 4.2 uses `generateAnswer`, whereas 4.3 will use `getAnswers`. Change here and in the code when 4.3 goes live.
In the **bot.js** file, we pass the user's input to the QnA Maker service's `getAnswers` method to get answers from the knowledge base. If you are accessing your own knowledge base, change the _no answers_ and _welcome_ messages below to provide useful instructions for your users.
 -->
在 **bot.js** 檔案中，我們會將使用者的輸入傳遞至 QnA Maker 服務的 `generateAnswer` 方法，以從知識庫中取得解答。 如果您要存取自己的知識庫，請變更以下的「沒有解答」和「歡迎」訊息，為您的使用者提供實用的指示。

```javascript
const { ActivityTypes, TurnContext } = require('botbuilder');
const { QnAMaker, QnAMakerEndpoint, QnAMakerOptions } = require('botbuilder-ai');

/**
 * A simple bot that responds to utterances with answers from QnA Maker.
 * If an answer is not found for an utterance, the bot responds with help.
 */
class QnAMakerBot {
    /**
     * The QnAMakerBot constructor requires one argument (`endpoint`) which is used to create an instance of `QnAMaker`.
     * @param {QnAMakerEndpoint} endpoint The basic configuration needed to call QnA Maker. In this sample the configuration is retrieved from the .bot file.
     * @param {QnAMakerOptions} config An optional parameter that contains additional settings for configuring a `QnAMaker` when calling the service.
     */
    constructor(endpoint, qnaOptions) {
        this.qnaMaker = new QnAMaker(endpoint, qnaOptions);
    }

    /**
     * Every conversation turn for our QnAMakerBot will call this method.
     * @param {TurnContext} turnContext Contains all the data needed for processing the conversation turn.
     */
    async onTurn(turnContext) {
        // By checking the incoming Activity type, the bot only calls QnA Maker in appropriate cases.
        if (turnContext.activity.type === ActivityTypes.Message) {
            // Perform a call to the QnA Maker service to retrieve matching Question and Answer pairs.
            const qnaResults = await this.qnaMaker.generateAnswer(turnContext);

            // If an answer was received from QnA Maker, send the answer back to the user.
            if (qnaResults[0]) {
                await turnContext.sendActivity(qnaResults[0].answer);

            // If no answers were returned from QnA Maker, reply with help.
            } else {
                await turnContext.sendActivity('No QnA Maker answers were found. This example uses a QnA Maker Knowledge Base that focuses on smart light bulbs. To see QnA Maker in action, ask the bot questions like "Why won\'t it turn on?" or "I need help."');
            }

        // If the Activity is a ConversationUpdate, send a greeting message to the user.
        } else if (turnContext.activity.type === ActivityTypes.ConversationUpdate &&
                   turnContext.activity.recipient.id !== turnContext.activity.membersAdded[0].id) {
            await turnContext.sendActivity('Welcome to the QnA Maker sample! Ask me a question and I will try to answer it.');

        // Respond to all other Activity types.
        } else if (turnContext.activity.type !== ActivityTypes.ConversationUpdate) {
            await turnContext.sendActivity(`[${ turnContext.activity.type }]-type activity detected.`);
        }
    }
}

module.exports.QnAMakerBot = QnAMakerBot;
```

---

## <a name="test-the-bot"></a>測試 Bot

在您的電腦本機執行範例。 如需指示，請參閱 [C#](https://github.com/Microsoft/BotBuilder-Samples/tree/master/samples/csharp_dotnetcore/11.qnamaker) 或 [JS](https://github.com/Microsoft/BotBuilder-Samples/blob/master/samples/javascript_nodejs/11.qnamaker/README.md) 範例的讀我檔案。

在模擬器中，將訊息傳送給 Bot，如下所示。

![測試 qna 範例](~/media/emulator-v4/qna-test-bot.png)


## <a name="next-steps"></a>後續步驟

QnA Maker 可以結合其他認知服務，讓 Bot 具有更強大的功能。 分派工具可讓您在 Bot 中結合 QnA 和 Language Understanding (LUIS)。

> [!div class="nextstepaction"]
> [使用分派工具結合 LUIS 和 QnA 服務](./bot-builder-tutorial-dispatch.md)
