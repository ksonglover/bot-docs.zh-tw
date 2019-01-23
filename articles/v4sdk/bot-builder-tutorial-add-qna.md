---
title: 可讓 Bot 回答問題的 Azure Bot Service 教學課程 | Microsoft Docs
description: 在 Bot 中使用 QnA Maker 回答問題的教學課程。
keywords: QnA Maker, 問題和答案, 知識庫
author: JonathanFingold
ms.author: v-jofing
manager: kamrani
ms.topic: tutorial
ms.service: bot-service
ms.subservice: sdk
ms.date: 01/15/2019
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: b1c34531ee60b2ce9037f42e4f5a7093501cf83a
ms.sourcegitcommit: bdb981c0b11ee99d128e30ae0462705b2dae8572
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 01/17/2019
ms.locfileid: "54360945"
---
# <a name="tutorial-use-qna-maker-in-your-bot-to-answer-questions"></a>教學課程：在 Bot 中使用 QnA Maker 回答問題

您可以使用 QnA Maker 服務和知識庫，對 Bot 新增問與答支援。 當您建立知識庫時，您會在其中植入問題與答案。

在本教學課程中，您了解如何：

> [!div class="checklist"]
> * 建立 QnA Maker 服務和知識庫
> * 將知識庫資訊新增至 .bot 檔案
> * 更新 Bot 以查詢知識庫
> * 重新發佈 Bot

如果您沒有 Azure 訂用帳戶，請在開始前建立[免費帳戶](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) 。

## <a name="prerequisites"></a>必要條件

* 在[上一個教學課程](bot-builder-tutorial-basic-deploy.md)中建立的 Bot。 我們會將問與答功能新增至 Bot。
* 熟悉一下 QnA Maker 很有幫助。 我們將使用 QnA Maker 入口網站來建立、訓練及發佈要搭配 Bot 使用的知識庫。

您應該已經有上一個教學課程的必要條件：

[!INCLUDE [deployment prerequisites snippet](~/includes/deploy/snippet-prerequisite.md)]

## <a name="sign-in-to-qna-maker-portal"></a>登入 QnA Maker 入口網站

<!-- This and the next step are close duplicates of what's in the QnA How-To -->

使用您的 Azure 認證登入 [QnA Maker 入口網站](https://qnamaker.ai/)。

## <a name="create-a-qna-maker-service-and-knowledge-base"></a>建立 QnA Maker 服務和知識庫

我們將從 [Microsoft/BotBuilder-Samples](https://github.com/Microsoft/BotBuilder-Samples) 存放庫中的 QnA Maker 範例匯入現有的知識庫定義。

1. 將範例存放庫複製到您的電腦。
1. 在 QnA Maker 入口網站中，**建立知識庫**。
   1. 如有必要，請建立 QnA 服務。 (您可以在此教學課程中使用現有的 QnA Maker 服務或建立新的服務)。如需詳細的 QnA Maker 指示，請參閱[建立 QnA Maker 服務](https://docs.microsoft.com/en-us/azure/cognitive-services/qnamaker/how-to/set-up-qnamaker-service-azure)和[建立、訓練及發佈 QnA Maker 知識庫](https://docs.microsoft.com/en-us/azure/cognitive-services/qnamaker/quickstarts/create-publish-knowledge-base)。
   1. 將 QnA 服務連線到知識庫。
   1. 為您的知識庫命名。
   1. 若要填入您的知識庫，請使用範例存放庫中的 **BotBuilder-Samples\samples\csharp_dotnetcore\11.qnamaker\CognitiveModels\smartLightFAQ.tsv** 檔案。
   1. 按一下 [建立知識庫] 以建立知識庫。
1. **儲存並訓練**知識庫。
1. **發佈**知識庫。

   您的知識庫現在可供您的 Bot 使用。 記錄知識庫識別碼、端點金鑰和主機名稱。 您在下一個步驟中需要這些資訊。

## <a name="add-knowledge-base-information-to-your-bot-file"></a>將知識庫資訊新增至 .bot 檔案

將存取您的知識庫所需的資訊新增至 .bot 檔案。

1. 在編輯器中開啟 .bot 檔案。
1. 將 `qna` 元素新增至 `services` 陣列。

    ```json
    {
        "type": "qna",
        "name": "<your-knowledge-base-name>",
        "kbId": "<your-knowledge-base-id>",
        "hostname": "<your-qna-service-hostname>",
        "endpointKey": "<your-knowledge-base-endpoint-key>",
        "subscriptionKey": "<your-azure-subscription-key>",
        "id": "<a-unique-id>"
    }
    ```

    | 欄位 | 值 |
    |:----|:----|
    | type | 必須是 `qna`。 這表示此服務項目描述 QnA 知識庫。 |
    | name | 您指派給知識庫的名稱。 |
    | kbId | QnA Maker 入口網站為您產生的知識庫識別碼。 |
    | 主機名稱 | QnA Maker 入口網站所產生的主機 URL。 使用完整的 URL，其開頭為 `https://` 並以 `/qnamaker` 結尾。 |
    | endpointKey | QnA Maker 入口網站為您產生的端點金鑰。 |
    | subscriptionKey | 在 Azure 中建立 QnA Maker 服務時所使用的訂用帳戶識別碼。 |
    | id | 尚未用於 .bot 檔案中所列的其中一項其他服務的唯一識別碼，例如 "201"。 |

1. 儲存您的編輯。

## <a name="update-your-bot-to-query-the-knowledge-base"></a>更新 Bot 以查詢知識庫

更新您的初始化程式碼，以載入您知識庫的服務資訊。

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

1. 將 **Microsoft.Bot.Builder.AI.QnA** NuGet 套件新增至專案。
1. 將您可實作 **IBot** 的類別重新命名為 `QnaBot`。
1. 將包含 Bot 存取子的類別重新命名為 `QnaBotAccessors`。
1. 在 **Startup.cs** 檔案中，新增這些命名空間參考。
    ```csharp
    using System.Collections.Generic;
    using System.Linq;
    using Microsoft.Bot.Builder.AI.QnA;
    using Microsoft.Bot.Builder.Integration;
    ```
1. 然後，修改 **ConfigureServices** 方法，以初始化及註冊 **.bot** 檔案中所定義的知識庫。 請注意，以下前幾行已從其前面的 `services.AddBot<QnaBot>(options =>` 呼叫本文中移除。
    ```csharp
    public void ConfigureServices(IServiceCollection services)
    {
        var secretKey = Configuration.GetSection("botFileSecret")?.Value;
        var botFilePath = Configuration.GetSection("botFilePath")?.Value;

        // Loads .bot configuration file and adds a singleton that your Bot can access through dependency injection.
        var botConfig = BotConfiguration.Load(botFilePath ?? @".\jfEchoBot.bot", secretKey);
        services.AddSingleton(sp => botConfig ?? throw new InvalidOperationException($"The .bot config file could not be loaded. ({botConfig})"));

        // Initialize the QnA knowledge bases for the bot.
        services.AddSingleton(sp => {
            var qnaServices = new List<QnAMaker>();
            foreach (var qnaService in botConfig.Services.OfType<QnAMakerService>())
            {
                qnaServices.Add(new QnAMaker(qnaService));
            }
            return qnaServices;
        });

        services.AddBot<QnaBot>(options =>
        {
            // Retrieve current endpoint.
            // ...
        });

        // Create and register state accessors.
        // ...
    }
    ```
1. 在 **QnaBot.cs** 檔案中，新增這些命名空間參考。
    ```csharp
    using System.Collections.Generic;
    using Microsoft.Bot.Builder.AI.QnA;
    ```
1. 新增 `_qnaServices` 屬性並在 Bot 的建構函式中加以初始化。
    ```csharp
    private readonly List<QnAMaker> _qnaServices;

    /// ...
    public QnaBot(QnaBotAccessors accessors, List<QnAMaker> qnaServices, ILoggerFactory loggerFactory)
    {
        // ...
        _qnaServices = qnaServices;
    }
    ```
1. 修改回合處理常式，以針對使用者的輸入查詢任何已註冊的知識庫。 當您的 Bot 需要來自 QnAMaker 的解答時，從您的 Bot 程式碼呼叫 `GetAnswersAsync`，以根據目前的內容取得適當解答。 如果您要存取自己的知識庫，請變更以下的「沒有解答」 訊息，為您的使用者提供實用的指示。
    ```csharp
    public async Task OnTurnAsync(ITurnContext turnContext, CancellationToken cancellationToken = default(CancellationToken))
    {
        if (turnContext.Activity.Type == ActivityTypes.Message)
        {
            foreach(var qnaService in _qnaServices)
            {
                var response = await qnaService.GetAnswersAsync(turnContext);
                if (response != null && response.Length > 0)
                {
                    await turnContext.SendActivityAsync(
                        response[0].Answer,
                        cancellationToken: cancellationToken);
                    return;
                }
            }

            var msg = "No QnA Maker answers were found. This example uses a QnA Maker knowledge base that " +
                "focuses on smart light bulbs. Ask the bot questions like 'Why won't it turn on?' or 'I need help'.";

            await turnContext.SendActivityAsync(msg, cancellationToken: cancellationToken);
        }
        else
        {
            await turnContext.SendActivityAsync($"{turnContext.Activity.Type} event detected");
        }
    }
    ```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

1. 將終端機或命令提示字元開啟至您專案的根目錄。
1. 將 **botbuilder-ai** npm 套件新增至您的專案。
    ```shell
    npm i botbuilder-ai
    ```
1. 在 **index.js** 檔案中，新增此 require 陳述式。
    ```javascript
    const { QnAMaker } = require('botbuilder-ai');
    ```
1. 讀入組態資訊以產生 QnA Maker 服務。
    ```javascript
    // Read bot configuration from .bot file.
    // ...

    // Initialize the QnA knowledge bases for the bot.
    // Assume each QnA entry in the .bot file is well defined.
    const qnaServices = [];
    botConfig.services.forEach(s => {
        if (s.type == 'qna') {
            const endpoint = {
                knowledgeBaseId: s.kbId,
                endpointKey: s.endpointKey,
                host: s.hostname
            };
            const options = {};
            qnaServices.push(new QnAMaker(endpoint, options));
        }
    });

    // Get bot endpoint configuration by service name
    // ...
    ```
1. 更新 Bot 建構以傳入 QnA 服務。
    ```javascript
    // Create the bot.
    const myBot = new MyBot(qnaServices);
    ```
1. 在 **bot.js** 檔案中，新增建構函式。
    ```javascript
    constructor(qnaServices) {
        this.qnaServices = qnaServices;
    }
    ```
1. 然後，更新回合處理常式，以查詢知識庫中的答案。
    ```javascript
    async onTurn(turnContext) {
        if (turnContext.activity.type === ActivityTypes.Message) {
            for (let i = 0; i < this.qnaServices.length; i++) {
                // Perform a call to the QnA Maker service to retrieve matching Question and Answer pairs.
                const qnaResults = await this.qnaServices[i].getAnswers(turnContext);

                // If an answer was received from QnA Maker, send the answer back to the user and exit.
                if (qnaResults[0]) {
                    await turnContext.sendActivity(qnaResults[0].answer);
                    return;
                }
            }
            // If no answers were returned from QnA Maker, reply with help.
            await turnContext.sendActivity('No QnA Maker answers were found. '
                + 'This example uses a QnA Maker Knowledge Base that focuses on smart light bulbs. '
                + `Ask the bot questions like "Why won't it turn on?" or "I need help."`);
        } else {
            await turnContext.sendActivity(`[${ turnContext.activity.type } event detected]`);
        }
    }
    ```

---

### <a name="test-the-bot-locally"></a>在本機測試 Bot

此時您的 Bot 應該能夠回答一些問題。 在本機執行 Bot 並且在模擬器中開啟。

![測試 qna 範例](~/media/emulator-v4/qna-test-bot.png)

## <a name="re-publish-your-bot"></a>重新發佈 Bot

我們現在可以重新發佈您的 Bot。

[!INCLUDE [publish snippet](~/includes/deploy/snippet-publish.md)]

### <a name="test-the-published-bot"></a>測試已發佈的 Bot

發佈 Bot 之後，讓 Azure 有幾分鐘的時間來更新並啟動 Bot。

1. 使用模擬器來測試 Bot 的生產端點，或使用 Azure 入口網站在網路聊天中測試 Bot。

   在任一情況下，您應會看見如同在本機測試時的相同行為。

## <a name="clean-up-resources"></a>清除資源

<!-- In the first tutorial, we should tell them to use a new resource group, so that it is easy to clean up resources. We should also mention in this step in the first tutorial not to clean up resources if they are continuing with the sequence. -->

如果您不打算繼續使用此應用程式，請使用下列步驟來刪除相關聯的資源：

1. 在 Azure 入口網站中，開啟您 Bot 的資源群組。
1. 按一下 [刪除資源群組] 來刪除群組及其包含的所有資源。
1. 在確認窗格中，輸入資源群組名稱，然後按一下 [刪除]。

## <a name="next-steps"></a>後續步驟

如需如何將功能新增至 Bot 的相關資訊，請參閱開發操作說明一節中的文章。
> [!div class="nextstepaction"]
> [後續步驟按鈕](bot-builder-howto-send-messages.md)
