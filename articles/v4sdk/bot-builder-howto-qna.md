---
title: 使用 QnA Maker 回答問題 | Microsoft Docs
description: 了解如何在 Bot 中使用 QnA Maker。
keywords: 問題和解答, QnA, 常見問題集, 中介軟體
author: ivorb
ms.author: v-ivorb
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: cognitive-services
ms.date: 10/08/2018
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 3d488cc2bb61ef460ed45707596cb7db9e6c23e8
ms.sourcegitcommit: b78fe3d8dd604c4f7233740658a229e85b8535dd
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/24/2018
ms.locfileid: "49999077"
---
# <a name="use-qna-maker-to-answer-questions"></a>使用 QnA Maker 回答問題

[!INCLUDE [pre-release-label](../includes/pre-release-label.md)]

您可以使用 QnA Maker 服務，對 Bot 新增問題和解答支援。 在建立您自己的 QnA Maker 服務時，其中一項基本需求是在服務中植入問題和解答。 許多時候，問題和解答早已存在於常見問題集或其他文件等內容中。 其他時候，您則會想要以更自然的對話方式自訂問題的解答。

## <a name="prerequisites"></a>必要條件
- 建立 [QnA Maker](https://www.qnamaker.ai/) 帳戶
- 下載 QnA Maker 範例 [[C#](https://aka.ms/cs-qna) | [JavaScript](https://aka.ms/js-qna-sample)]

## <a name="create-a-qna-maker-service-and-publish-a-knowledge-base"></a>建立 QnA Maker 服務及發佈知識庫

建立 QnA Maker 帳戶之後，請依照指示來建立 [QnA Maker 服務](https://docs.microsoft.com/en-us/azure/cognitive-services/qnamaker/how-to/set-up-qnamaker-service-azure)和[知識庫](https://docs.microsoft.com/en-us/azure/cognitive-services/qnamaker/quickstarts/create-publish-knowledge-base)。 

發佈您的知識庫之後，您需要記錄下列值，才能以程式設計方式將 Bot 連線至知識庫。
- 在 [QnA Maker](https://www.qnamaker.ai/) 網站上，選取您的知識庫。
- 開啟您的知識庫，然後選取 [設定]。 將針對「服務名稱」顯示的值記錄為 <your_kb_name>
- 向下捲動以尋找 [部署詳細資料] 及記錄下列值：
   - POST /knowledgebases/<your_knowledge_base_id>/generateAnswer
   - 主機： https://<you_hostname>.azurewebsites.net/qnamaker
   - 授權：EndpointKey <your_endpoint_key>

## <a name="installing-packages"></a>安裝套件

開始撰寫程式碼之前，請先確定您有 QnA Maker 所需的套件。

# <a name="ctabcs"></a>[C#](#tab/cs)

將下列 [NuGet 套件](https://docs.microsoft.com/en-us/nuget/tools/package-manager-ui)新增到您的 Bot。

* `Microsoft.Bot.Builder.AI.QnA`

# <a name="javascripttabjs"></a>[JavaScript](#tab/js)

QnA Maker 功能位於 `botbuilder-ai` 套件中。 可以透過 npm 將此套件新增到您的專案：

```shell
npm install --save botbuilder-ai
```

---

## <a name="using-cli-tools-to-update-your-bot-configuration"></a>使用 CLI 工具來更新 .bot 組態

取得知識庫存取值的替代方法是使用 [qnamaker](https://aka.ms/botbuilder-tools-qnaMaker) 和 [msbot](https://aka.ms/botbuilder-tools-msbot-readme) BotBuilder CLI 工具來取得有關您知識庫的中繼資料，並將其新增至您的 .bot 檔案。

1. 開啟終端機或命令提示字元，然後瀏覽至您 Bot 專案的根目錄。
2. 執行 `qnamaker init` 以建立 QnA Maker 資源檔 (**.qnamakerrc**)。 其會提示您輸入 QnA Maker 訂用帳戶金鑰。
3. 執行下列命令以下載您的中繼資料，並將其新增至 Bot 的組態檔。

    ```shell
    qnamaker get kb --kbId <your-kb-id> --msbot | msbot connect qna --stdin [--secret <your-secret>]
    ```
如果您已加密組態檔，則必須提供祕密金鑰以更新檔案。

## <a name="using-qna-maker"></a>使用 QnA Maker
初始化 Bot 時，會首度新增 QnA Maker 應用程式的參考。 然後，我們可以在 Bot 邏輯內呼叫。

# <a name="ctabcs"></a>[C#](#tab/cs)
開啟您稍早下載的 QnA Maker 範例。 我們會視需要修改此程式碼。
首先，將存取您的知識庫所需的資訊 (包括主機名稱、端點金鑰和知識庫識別碼 (KbId)) 新增至 `BotConfiguration.bot`。 這些是您在 QnA Maker 中從您知識庫的 [設定] 儲存的值。

```json
{
  "name": "QnABotSample",
  "services": [
    {
      "type": "endpoint",
      "name": "development",
      "endpoint": "http://localhost:3978/api/messages",
      "appId": "",
      "id": "1",
      "appPassword": ""
    },
    {
      "type": "qna",
      "name": "QnABot",
      "KbId": "<YOUR_KNOWLEDGE_BASE_ID>",
      "Hostname": "https://<YOUR_HOSTNAME>.azurewebsites.net/qnamaker",
      "EndpointKey": "<YOUR_ENDPOINT_KEY>"
    }
  ],
  "version": "2.0",
  "padlock": ""
}
```

接下來，我們會在 `Startup.cs` 中建立 QnA Maker 執行個體。 這會從 `BotConfiguration.bot` 檔案抓取上述的資訊。 這些字串也可以硬式編碼以供測試。

```csharp
private static BotServices InitBotServices(BotConfiguration config)
{
    var qnaServices = new Dictionary<string, QnAMaker>();
    foreach (var service in config.Services)
    {
        switch (service.Type)
        {
            case ServiceTypes.QnA:
            {
                // Create a QnA Maker that is initialized and suitable for passing
                // into the IBot-derived class (QnABot).
                var qna = (QnAMakerService)service;
                if (qna == null)
                {
                    throw new InvalidOperationException("The QnA service is not configured correctly in your '.bot' file.");
                }

                if (string.IsNullOrWhiteSpace(qna.KbId))
                {
                    throw new InvalidOperationException("The QnA KnowledgeBaseId ('kbId') is required to run this sample. Please update your '.bot' file.");
                }

                if (string.IsNullOrWhiteSpace(qna.EndpointKey))
                {
                    throw new InvalidOperationException("The QnA EndpointKey ('endpointKey') is required to run this sample. Please update your '.bot' file.");
                }

                if (string.IsNullOrWhiteSpace(qna.Hostname))
                {
                    throw new InvalidOperationException("The QnA Host ('hostname') is required to run this sample. Please update your '.bot' file.");
                }

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
    var connectedServices = new BotServices(qnaServices);
    return connectedServices;
}
```

然後我們必須將這個 QnAMaker 執行個體提供給 Bot。 開啟 `QnABot.cs`，並且在檔案頂端新增下列程式碼。 如果您要存取自己的知識庫，請變更以下所示的「歡迎」訊息，為您的使用者提供實用的初始指示。

```csharp
public class QnABot : IBot
{
    public static readonly string QnAMakerKey = "QnABot";
    private const string WelcomeText = @"This bot will introduce you to QnA Maker.
                                         Ask a quesiton to get started.";
    private readonly BotServices _services;
    public QnABot(BotServices services)
    {
        _services = services ?? throw new System.ArgumentNullException(nameof(services));
        Console.WriteLine($"{_services}");
        if (!_services.QnAServices.ContainsKey(QnAMakerKey))
        {
            throw new System.ArgumentException($"Invalid configuration. Please check your '.bot' file for a QnA service named '{QnAMakerKey}'.");
        }
    }
}
```

# <a name="javascripttabjs"></a>[JavaScript](#tab/js)
開啟您稍早下載的 QnA Maker 範例。 我們會視需要修改此程式碼。
在我們的範例中，啟動程式碼位於 **index.js** 檔案、Bot 邏輯的程式碼位於 **bot.js** 檔案，而其他組態資訊位於 **qnamaker.bot** 檔案。

遵循指示來建立知識庫以及更新 **.bot** 檔案之後，**qnamaker.bot** 檔案應該包含 QnA Maker 知識庫的服務項目。

```json
{
    "name": "qnamaker",
    "description": "",
    "services": [
        {
            "type": "endpoint",
            "name": "development",
            "id": "1",
            "appId": "",
            "appPassword": "",
            "endpoint": "http://localhost:3978/api/messages"
        },
        {
            "type": "qna",
            "name": "<YOUR_KB_NAME>",
            "kbId": "<YOUR_KNOWLEDGE_BASE_ID>",
            "endpointKey": "<YOUR_ENDPOINT_KEY>",
            "hostname": "https://<YOUR_HOSTNAME>.azurewebsites.net/qnamaker",
            "id": "221"
        }
    ],
    "padlock": "",
    "version": "2.0"
}
```

在 **index.js** 檔案中，我們會讀入組態資訊以產生 QnA Maker 服務及初始化 Bot。

將 `QNA_CONFIGURATION` 的值更新為您的知識庫名稱，因為其會出現在您的組態檔中。

```js
// QnA Maker knowledge base name as specified in .bot file.
const QNA_CONFIGURATION = '<YOUR_KB_NAME>';

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
            const qnaResults = await this.qnaMaker.generateAnswer(turnContext.activity.text);

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

對 Bot 問問題，以查看 QnA Maker 服務的回覆。 如需有關測試及發佈 QnA 服務的詳細資訊，請參閱有關[測試知識庫](https://docs.microsoft.com/en-us/azure/cognitive-services/qnamaker/how-to/test-knowledge-base)的 QnA Maker 文章。

## <a name="next-steps"></a>後續步驟

QnA Maker 可以結合其他認知服務，讓 Bot 具有更強大的功能。 分派工具可讓您在 Bot 中結合 QnA 和 Language Understanding (LUIS)。

> [!div class="nextstepaction"]
> [使用分派工具結合 LUIS 和 QnA 服務](./bot-builder-tutorial-dispatch.md)
