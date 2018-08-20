---
title: 使用 QnA Maker | Microsoft Docs
description: 了解如何在 Bot 中使用 QnA Maker。
keywords: 問題和解答, QnA, 常見問題集, 中介軟體
author: ivorb
ms.author: v-ivorb
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 03/13/2018
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 78bc2c849a2c1900da33c7419693a7ff84c43cb0
ms.sourcegitcommit: dcbc8ad992a3e242a11ebcdf0ee99714d919a877
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/30/2018
ms.locfileid: "39352947"
---
# <a name="how-to-use-qna-maker"></a>如何使用 QnA Maker

若要對 Bot 新增簡單的問題和解答支援，您可以使用 [QnA Maker](https://qnamaker.ai/) 服務。

在撰寫您自己的 QnA Maker 服務時，其中一項基本需求是在服務中植入問題和解答。 許多時候，問題和解答早已存在於常見問題集或其他文件等內容中。 其他時候，您則會想要以更自然的對話方式自訂問題的解答。 

## <a name="create-a-qna-maker-service"></a>建立 QnA Maker 服務
先建立帳戶，並登入 [QnA Maker](https://qnamaker.ai/)。 然後，瀏覽至 [建立知識庫]。 按一下 [建立 QnA 服務]，然後遵循指示來建立 Azure QnA 服務。

![QnA 影像 1](media/QnA_1.png)

系統會將您重新導向至[建立 QnA Maker](https://ms.portal.azure.com/#create/Microsoft.CognitiveServicesQnAMaker)。 完成表單，然後按一下 [建立]。

![QnA 影像 2](media/QnA_2.png)
 
在 Azure 入口網站中建立 QnA 服務後，系統會在 [資源管理] 標題下向您提供金鑰，但您可能會忽略。 請繼續進行步驟 2 來將 Azure QnA 服務連線。 重新整理頁面，以選取您剛才建立的 Azure 服務，並輸入知識庫的名稱。

![QnA 影像 3](media/QnA_3.png)

按一下 [建立 KB]。 輸入您自己的問題和解答，或複製下列範例。 

![QnA 影像 4](media/QnA_4.png)

或者，您也可以選擇 [填入 KB]，並提供檔案或 URL。 用於產生簡單 QnA Maker 服務的來源檔案範例[在此](https://aka.ms/qna-tsv)。

在新增 QnA 組或填入 KB 後，按一下 [儲存並訓練]。 完成後，在 [發佈] 索引標籤上，按一下 [發佈]。

若要將 QnA 服務連線至 Bot，您需要內含知識庫識別碼和 QnA Maker 訂用帳戶金鑰的 HTTP 要求字串。 從發佈結果複製 HTTP 要求範例。 

![QnA 影像 5](media/QnA_5.png)

## <a name="installing-packages"></a>安裝套件

開始撰寫程式碼之前，請先確定您有 QnA Maker 所需的套件。

# <a name="ctabcs"></a>[C#](#tab/cs)

對以下 NuGet 套件的 v4 發行前版本[新增參考](https://docs.microsoft.com/en-us/nuget/tools/package-manager-ui) \(機器翻譯\)：

* `Microsoft.Bot.Builder.Ai.QnA`

# <a name="javascripttabjs"></a>[JavaScript](#tab/js)

這些任一項服務都可以使用 botbuilder-ai 套件新增到您的 Bot。 可以透過 npm 將此套件新增到您的專案：

* `npm install --save botbuilder@preview`
* `npm install --save botbuilder-ai@preview`

---


## <a name="using-qna-maker"></a>使用 QnA Maker

QnA Maker 會先新增為中介軟體。 然後，我們可以在 Bot 邏輯內使用結果。

# <a name="ctabcs"></a>[C#](#tab/cs)

更新 `Startup.cs` 檔案中的 `ConfigureServices` 方法，以新增 `QnAMakerMiddleware` 物件。 您可以藉由直接將知識庫新增至 Bot 的中介軟體堆疊，來對 Bot 進行設定，使其針對從使用者收到的每則訊息檢查知識庫。


```csharp
using Microsoft.AspNetCore.Builder;
using Microsoft.AspNetCore.Hosting;
using Microsoft.Bot.Builder.Ai.Qna;
using Microsoft.Bot.Builder.BotFramework;
using Microsoft.Bot.Builder.Core.Extensions;
using Microsoft.Bot.Builder.Integration.AspNet.Core;
using Microsoft.Extensions.Configuration;
using Microsoft.Extensions.DependencyInjection;
using Microsoft.Extensions.Logging;
using System;

public void ConfigureServices(IServiceCollection services)
{
    services.AddSingleton(_ => Configuration);
    services.AddBot<AiBot>(options =>
    {
        options.CredentialProvider = new ConfigurationCredentialProvider(Configuration);

        var endpoint = new QnAMakerEndpoint
        {
           KnowledgeBaseId = "YOUR-KB-ID",
           // Get the Host from the HTTP request example at https://www.qnamaker.ai
           // For GA services: https://<Service-Name>.azurewebsites.net/qnamaker
           // For Preview services: https://westus.api.cognitive.microsoft.com/qnamaker/v2.0           
           Host = "YOUR-HTTP-REQUEST-HOST",
           EndpointKey = "YOUR-QNA-MAKER-SUBSCRIPTION-KEY"
        };
        options.Middleware.Add(new QnAMakerMiddleware(endpoint));
    });
}
```



編輯 EchoBot.cs 檔案中的程式碼，讓 `OnTurn` 在 QnA Maker 中介軟體未對使用者的問題傳送回應時，傳送後援訊息：

```csharp
using System.Threading.Tasks;
using Microsoft.Bot;
using Microsoft.Bot.Builder;
using Microsoft.Bot.Schema;

namespace Bot_Builder_Echo_Bot_QnA
{
    public class EchoBot : IBot
    {    
        public async Task OnTurn(ITurnContext context)
        {
            // This bot is only handling Messages
            if (context.Activity.Type == ActivityTypes.Message)
            {             
                if (!context.Responded)
                {
                    // QnA didn't send the user an answer
                    await context.SendActivity("Sorry, I couldn't find a good match in the KB.");

                }
            }
        }
    }
}
```


如需 Bot 範例，請參閱 [QnA Maker 範例](https://aka.ms/qna-cs-bot-sample)。

# <a name="javascripttabjs"></a>[JavaScript](#tab/js)

首先，要求/匯入 [QnAMaker](https://github.com/Microsoft/botbuilder-js/tree/master/doc/botbuilder-ai/classes/botbuilder_ai.qnamaker.md) 類別：

```js
const { QnAMaker } = require('botbuilder-ai');
```

根據 QnA 服務的 HTTP 要求，使用字串將 `QnAMaker` 初始化來加以建立。 您可以從 [QnA Maker 入口網站](https://qnamaker.ai)的 [設定] > [部署詳細資料] 底下複製服務的要求範例。


端視 QnA Maker 服務使用的是 GA 版還是預覽版的 QnA Maker，字串格式會有所不同。 複製 HTTP 要求範例，然後取得在初始化 `QnAMaker` 時所要使用的知識庫識別碼、訂用帳戶金鑰和主機。

<!--
**Preview**
```js
const qnaEndpointString = 
    // Replace xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx with your knowledge base ID
    "POST /knowledgebases/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/generateAnswer\r\n" + 
    "Host: https://westus.api.cognitive.microsoft.com/qnamaker/v2.0\r\n" +
    // Replace xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx with your QnAMaker subscription key
    "Ocp-Apim-Subscription-Key: xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx\r\n"
const qna = new QnAMaker(qnaEndpointString);
```

**GA**
```js
const qnaEndpointString = 
    // Replace xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx with your knowledge base ID
    "POST /knowledgebases/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/generateAnswer\r\n" + 
    // Replace <Service-Name> to match the Azure URL where your service is hosted
    "Host: https://<Service-Name>.azurewebsites.net/qnamaker\r\n" +
    // Replace xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx with your QnAMaker subscription key
    "Authorization: EndpointKey xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx\r\n"
const qna = new QnAMaker(qnaEndpointString);
```
-->
<bpt id="p1">**</bpt>Preview<ept id="p1">**</ept>
```js
const qna = new QnAMaker(
    {
        knowledgeBaseId: '<KNOWLEDGE-BASE-ID>',
        endpointKey: '<QNA-SUBSCRIPTION-KEY>',
        host: 'https://westus.api.cognitive.microsoft.com/qnamaker/v2.0'
    },
    {
        // set this to true to send answers from QnA Maker
        answerBeforeNext: true
    }
);
```
**GA**
```js
const qna = new QnAMaker(
    {
        knowledgeBaseId: '<KNOWLEDGE-BASE-ID>',
        endpointKey: '<QNA-SUBSCRIPTION-KEY>',
        host: 'https://<Service-Name>.azurewebsites.net/qnamaker'
    },
    {
        answerBeforeNext: true
    }
);
```
您可以藉由直接將 QnA Maker 新增至 Bot 的中介軟體堆疊，來對 Bot 進行設定，使其自動呼叫 QnA Maker：

```js
// Add QnA Maker as middleware
adapter.use(qna);

// Listen for incoming requests 
server.post('/api/messages', (req, res) => {
    // Route received request to adapter for processing
    adapter.processActivity(req, res, async (context) => {
        // If `!context.responded`, that means an answer wasn't found for the user's utterance.
        // In this case, we send the user a fallback message.
        if (context.activity.type === 'message' && !context.responded) {
            await context.sendActivity('No QnA Maker answers were found.');
        } else if (context.activity.type !== 'message') {
            await context.sendActivity(`[${context.activity.type} event detected]`);
        }
    });
});
```

在初始化 `QnAMaker` 時將 `answerBeforeNext` 初始化為 `true`，代表 QnA Maker 中介軟體會先自動回應其是否找到解答，然後才會進入到 Bot 在 `processActivity` 中的主要邏輯。 如果相反地，您將 `answerBeforeNext` 設定為 `false`，Bot 則只會在 `processActivity` 中的 Bot 主要邏輯全都執行後，且在 Bot 尚未回覆使用者的情況下，才會呼叫 QnA Maker。 

## <a name="calling-qna-maker-without-using-middleware"></a>呼叫 QnA Maker 而不使用中介軟體

在上述範例中，`adapter.use(qna);` 陳述式代表 QnA 正以中介軟體的形式執行，因此，會對 Bot 所接收的每則訊息做出回應。 若要提升對於 QnA Maker 呼叫方式和時間的控制能力，您可以直接從 Bot 邏輯內呼叫 `qna.answer()`，而不是將它安裝為中介軟體。

移除 `adapter.use(qna);` 陳述式，然後使用下列程式碼直接呼叫 QnA Maker。

```js
// Listen for incoming activity 
server.post('/api/messages', (req, res) => {
    // Route received activity to adapter for processing
    adapter.processActivity(req, res, async (context) => {
        if (context.activity.type === 'message') {
            var handled = await qna.answer(context)
                if (!handled) {
                    await context.sendActivity(`I'm sorry. I didn't understand.`);
                }
        }
    });
});
```

QnA Maker 的另一種自訂方法是使用 `qna.generateAnswer()`。 這個方法可讓您從 QnA Maker 中取得更多關於解答的詳細資料。


```js
// Listen for incoming activity 
server.post('/api/messages', (req, res) => {
    // Route received activity to adapter for processing
    adapter.processActivity(req, res, async (context) => {
        if (context.activity.type === 'message') {
            // Get all the answers QnA Maker finds
            var results = await qna.generateAnswer(context.activity.text);
                if (results && results.length > 0) {
                    await context.sendActivity(results[0].answer);
                } else {
                    await context.sendActivity(`I don't know.`);
                }
    
        }
    });
});
```
---

對 Bot 問問題，以查看 QnA Maker 服務的回覆。

![QnA 影像 6](media/QnA_6.png)



## <a name="next-steps"></a>後續步驟

QnA Maker 可以結合其他認知服務，讓 Bot 具有更強大的功能。 分派工具可讓您在 Bot 中結合 QnA 和 Language Understanding (LUIS)。

> [!div class="nextstepaction"]
> [使用分派工具結合 LUIS 應用程式和 QnA 服務](./bot-builder-tutorial-dispatch.md)
