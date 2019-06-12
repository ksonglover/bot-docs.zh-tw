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
ms.date: 05/23/2019
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 5c8b164aa97ff4ea74acbd6765c72ea0c3f1ad3b
ms.sourcegitcommit: e276008fb5dd7a37554e202ba5c37948954301f1
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/05/2019
ms.locfileid: "66693655"
---
# <a name="tutorial-use-qna-maker-in-your-bot-to-answer-questions"></a>教學課程：在 Bot 中使用 QnA Maker 回答問題

[!INCLUDE [applies-to-v4](../includes/applies-to.md)]

您可以使用 QnA Maker 服務和知識庫，對 Bot 新增問與答支援。 當您建立知識庫時，您會在其中植入問題與答案。

在本教學課程中，您了解如何：

> [!div class="checklist"]
> * 建立 QnA Maker 服務和知識庫
> * 將知識庫資訊新增至設定檔
> * 更新 Bot 以查詢知識庫
> * 重新發佈 Bot

如果您沒有 Azure 訂用帳戶，請在開始前建立[免費帳戶](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) 。

## <a name="prerequisites"></a>必要條件

* 在[上一個教學課程](bot-builder-tutorial-basic-deploy.md)中建立的 Bot。 我們會將問與答功能新增至 Bot。
* 熟悉一下 [QnA Maker](https://qnamaker.ai/) 很有幫助。 我們將使用 QnA Maker 入口網站來建立、訓練及發佈要搭配 Bot 使用的知識庫。
* 熟悉如何使用 Azure Bot 服務來[建立 QnA Bot](https://aka.ms/azure-create-qna)。

您應該也已具備上一個教學課程的必要條件。

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
   1. 按一下 [建立知識庫]  以建立知識庫。
1. **儲存並訓練**知識庫。
1. **發佈**知識庫。

發佈 QnA Maker 應用程式後，選取 [設定]  索引標籤，然後捲動至 [部署詳細資料]。 記錄以下來自 _Postman_ 範例 HTTP 要求的值。

```text
POST /knowledgebases/<knowledge-base-id>/generateAnswer
Host: <your-hostname>  // NOTE - this is a URL ending in /qnamaker.
Authorization: EndpointKey <qna-maker-resource-key>
```

您主機名稱的完整 URL 字串看起來會類似 "https://< >.azure.net/qnamaker"。

在下一個步驟中，您會在 `appsettings.json` 或 `.env` 檔案中用到這些值。

您的知識庫現在可供您的 Bot 使用。

## <a name="add-knowledge-base-information-to-your-bot"></a>將知識庫資訊新增至 Bot
從 Bot Framework v4.3 開始，Azure 不會再於您下載的 Bot 原始程式碼中提供 .bot 檔案。 使用下列指示將 CSharp 或 JavaScript Bot 連線到您的知識庫。

## <a name="ctabcsharp"></a>[C#](#tab/csharp)

在 appsetting.json 檔案中新增下列值：

```json
{
   "MicrosoftAppId": "",
  "MicrosoftAppPassword": "",
  "ScmType": "None",
  
  "QnAKnowledgebaseId": "<knowledge-base-id>",
  "QnAAuthKey": "<qna-maker-resource-key>",
  "QnAEndpointHostName": "<your-hostname>" // This is a URL ending in /qnamaker
}
```

## <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

在 .env 檔案中新增下列值：

```
MicrosoftAppId=""
MicrosoftAppPassword=""
ScmType=None

QnAKnowledgebaseId="<knowledge-base-id>"
QnAAuthKey="<qna-maker-resource-key>"
QnAEndpointHostName="<your-hostname>" // This is a URL ending in /qnamaker
```

---

| 欄位 | 值 |
|:----|:----|
| QnAKnowledgebaseId | QnA Maker 入口網站為您產生的知識庫識別碼。 |
| QnAAuthKey | QnA Maker 入口網站為您產生的端點金鑰。 |
| QnAEndpointHostName | QnA Maker 入口網站所產生的主機 URL。 使用完整的 URL，其開頭為 `https://` 並以 `/qnamaker` 結尾。 完整的 URL 字串看起來會類似 "https://< >.azure.net/qnamaker"。 |

現在儲存您的編輯。

## <a name="update-your-bot-to-query-the-knowledge-base"></a>更新 Bot 以查詢知識庫

更新您的初始化程式碼，以載入您知識庫的服務資訊。

## <a name="ctabcsharp"></a>[C#](#tab/csharp)

1. 將 **Microsoft.Bot.Builder.AI.QnA** NuGet 套件新增至專案。
1. 將 **Microsoft.Extensions.Configuration** NuGet 套件新增至專案。
1. 在 **Startup.cs** 檔案中，新增這些命名空間參考。

   **Startup.cs**
   ```csharp
   using Microsoft.Bot.Builder.AI.QnA;
   using Microsoft.Extensions.Configuration;
   ```
1. 然後修改 _ConfigureServices_ 方法並建立 QnAMakerEndpoint，其會連線到 **appsettings.json** 檔案中定義的知識庫。

   **Startup.cs**
   ```csharp
   // Create QnAMaker endpoint as a singleton
   services.AddSingleton(new QnAMakerEndpoint
   {
      KnowledgeBaseId = Configuration.GetValue<string>($"QnAKnowledgebaseId"),
      EndpointKey = Configuration.GetValue<string>($"QnAAuthKey"),
      Host = Configuration.GetValue<string>($"QnAEndpointHostName")
    });

   ```
1. 在 **EchoBot.cs** 檔案中，新增下列命名空間參考。

   **EchoBot.cs**
   ```csharp
   using System.Linq;
   using Microsoft.Bot.Builder.AI.QnA;
   ```

1. 新增 `EchoBotQnA` 連接器並在 Bot 的建構函式中加以初始化。

   **EchoBot.cs**
   ```csharp
   public QnAMaker EchoBotQnA { get; private set; }
   public EchoBot(QnAMakerEndpoint endpoint)
   {
      // connects to QnA Maker endpoint for each turn
      EchoBotQnA = new QnAMaker(endpoint);
   }
   ```
1. 新增下列程式碼，以在 _OnMembersAddedAsync( )_ 方法下方建立 _AccessQnAMaker( )_ 方法：

   **EchoBot.cs**
   ```csharp
   private async Task AccessQnAMaker(ITurnContext<IMessageActivity> turnContext, CancellationToken cancellationToken)
   {
      var results = await EchoBotQnA.GetAnswersAsync(turnContext);
      if (results.Any())
      {
         await turnContext.SendActivityAsync(MessageFactory.Text("QnA Maker Returned: " + results.First().Answer), cancellationToken);
      }
      else
      {
         await turnContext.SendActivityAsync(MessageFactory.Text("Sorry, could not find an answer in the Q and A system."), cancellationToken);
      }
   }
   ```
1. 現在在 _OnMessageActivityAsync( )_ 內呼叫您的新方法 _AccessQnAMaker( )_ ，如下所示：

   **EchoBot.cs**
   ```csharp
   protected override async Task OnMessageActivityAsync(ITurnContext<IMessageActivity> turnContext, CancellationToken cancellationToken)
   {
      // First send the user input to your QnA Maker knowledgebase
      await AccessQnAMaker(turnContext, cancellationToken);
      ...
   }
   ```

## <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

1. 將終端機或命令提示字元開啟至您專案的根目錄。
1. 將 **botbuilder-ai** npm 套件新增至您的專案。
   ```shell
   npm i botbuilder-ai
   ```

1. 在 **index.js** 中，於 // Create Adapter 區段後面，新增下列程式碼以讀取產生 QnA Maker 服務所需的 .env 檔案組態資訊。

   **index.js**
   ```javascript
   // Map knowledgebase endpoint values from .env file into the required format for `QnAMaker`.
   const configuration = {
      knowledgeBaseId: process.env.QnAKnowledgebaseId,
      endpointKey: process.env.QnAAuthKey,
      host: process.env.QnAEndpointHostName
   };

   ```

1. 更新 Bot 建構以傳入 QnA 服務組態資訊。

   **index.js**
   ```javascript
   // Create the main dialog.
   const myBot = new MyBot(configuration, {});
   ```

1. 在 **bot.js** 檔案中，為 QnAMaker 新增此 require 陳述式

   **bot.js**
   ```javascript
   const { QnAMaker } = require('botbuilder-ai');
   ```

1. 將建構函式修改為立即接收建立 QnAMaker 連接器所需的已傳遞組態參數，若未提供這些參數，則擲回錯誤。

   **bot.js**
   ```javascript
      class MyBot extends ActivityHandler {
         constructor(configuration, qnaOptions) {
            super();
            if (!configuration) throw new Error('[QnaMakerBot]: Missing parameter. configuration is required');
            // now create a qnaMaker connector.
            this.qnaMaker = new QnAMaker(configuration, qnaOptions);
   ```

1. 最後，將下列程式碼新增至 onMessage( ) 呼叫，該呼叫會將每個使用者輸入傳遞至 QnA Maker 知識庫並將 QnA Maker 回應傳回給使用者以在知識庫中查詢答案。
 
    **bot.js**
    ```javascript
   // send user input to QnA Maker.
   const qnaResults = await this.qnaMaker.getAnswers(turnContext);

   // If an answer was received from QnA Maker, send the answer back to the user.
   if (qnaResults[0]) {
      await turnContext.sendActivity(`QnAMaker returned response: ' ${ qnaResults[0].answer}`);
   } 
   else { 
      // If no answers were returned from QnA Maker, reply with help.
      await turnContext.sendActivity('No QnA Maker response was returned.'
           + 'This example uses a QnA Maker Knowledge Base that focuses on smart light bulbs. '
           + `Ask the bot questions like "Why won't it turn on?" or "I need help."`);
   }
   ```
---

### <a name="test-the-bot-locally"></a>在本機測試 Bot

此時您的 Bot 應該能夠回答一些問題。 在本機執行 Bot 並且在模擬器中開啟。

![測試 qna 範例](./media/qna-test-bot.png)

## <a name="republish-your-bot"></a>重新發佈 Bot

我們現在可以將您的 Bot 重新發佈回到 Azure。

> [!IMPORTANT]
> 建立專案檔案的壓縮檔之前，請確定您「位在」  正確的資料夾中。 
> - 針對 C# Bot，這是具有 .csproj 檔案的資料夾。 
> - 針對 JS Bot，這是具有 app.js 或 index.js 檔案的資料夾。 
>
> 選取所有檔案並在該資料夾中壓縮，然後在該資料夾中執行命令。
>
> 如果您的根資料夾位置不正確，**Bot 將無法在 Azure 入口網站中執行**。

## <a name="ctabcsharp"></a>[C#](#tab/csharp)
```cmd
az webapp deployment source config-zip --resource-group <resource-group-name> --name <bot-name-in-azure> --src "c:\bot\mybot.zip"
```

## <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

[!INCLUDE [publish snippet](~/includes/deploy/snippet-publish-js.md)]

---

### <a name="test-the-published-bot"></a>測試已發佈的 Bot

發佈 Bot 之後，讓 Azure 有幾分鐘的時間來更新並啟動 Bot。

使用模擬器來測試 Bot 的生產端點，或使用 Azure 入口網站在網路聊天中測試 Bot。
在任一情況下，您應會看見如同在本機測試時的相同行為。

## <a name="clean-up-resources"></a>清除資源

<!-- In the first tutorial, we should tell them to use a new resource group, so that it is easy to clean up resources. We should also mention in this step in the first tutorial not to clean up resources if they are continuing with the sequence. -->

如果您不打算繼續使用此應用程式，請使用下列步驟來刪除相關聯的資源：

1. 在 Azure 入口網站中，開啟您 Bot 的資源群組。
1. 按一下 [刪除資源群組]  來刪除群組及其包含的所有資源。
1. 在確認窗格中，輸入資源群組名稱，然後按一下 [刪除]  。

## <a name="next-steps"></a>後續步驟

如需如何將功能新增至 Bot 的相關資訊，請參閱開發操作說明一節中的**傳送及接收文字訊息**一文和其他文章。
> [!div class="nextstepaction"]
> [傳送及接收文字訊息](bot-builder-howto-send-messages.md)
