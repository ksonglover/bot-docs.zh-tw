---
title: 使用 QnA Maker 回答問題 | Microsoft Docs
description: 了解如何在 Bot 中使用 QnA Maker。
keywords: 問題和答案, QnA, 常見問題集, QnA Maker
author: ivorb
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 11/06/2019
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: b58307732ae973719231987a35eab6374ea704e4
ms.sourcegitcommit: 312a4593177840433dfee405335100ce59aac347
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 11/12/2019
ms.locfileid: "73933615"
---
# <a name="use-qna-maker-to-answer-questions"></a>使用 QnA Maker 回答問題

[!INCLUDE[applies-to](../includes/applies-to.md)]

QnA Maker 會透過您的資料提供對話式的問題和解答層。 這可讓您的 Bot 傳送問題給 QnA Maker 並接收回應，而不需要您來剖析和解譯問題的意圖。

在建立您自己的 QnA Maker 服務時，其中一項基本需求是在服務中植入問題和解答。 在許多案例中，問題和解答已存在於常見問題集或其他文件的內容中；但有些時候，您可能會想要以更自然的對話方式來自訂問題的解答。

## <a name="prerequisites"></a>必要條件

- 本文中的程式碼是以 QnA Maker 範例為基礎。 您需要其採用 **[C#](https://aka.ms/cs-qna) 或 [JavaScript](https://aka.ms/js-qna-sample)** 的複本。
- [QnA Maker](https://www.qnamaker.ai/) 帳戶
- [Bot 基本概念](bot-builder-basics.md)、[QnA Maker](https://docs.microsoft.com/azure/cognitive-services/qnamaker/overview/overview) 及[管理 Bot 資源](bot-file-basics.md)的知識。

## <a name="about-this-sample"></a>關於此範例

為了讓 Bot 可使用 QnA Maker，您必須先在 [QnA Maker](https://www.qnamaker.ai/) 上建立知識庫，我們將在下一節中討論這部分。 然後，您的 Bot 可以對其傳送使用者的查詢，讓其在回應中提供問題的最佳答案。

## <a name="ctabcs"></a>[C#](#tab/cs)

![QnABot 邏輯流程](./media/qnabot-logic-flow.png)

系統會針對每個收到的使用者輸入呼叫 `OnMessageActivityAsync`。 呼叫後，該方法就會在範例程式碼的 `appsetting.json` 檔案內存取儲存的 `_configuration` 資訊，以尋找值來連線到預先設定的 QnA Maker 知識庫。

## <a name="javascripttabjs"></a>[JavaScript](#tab/js)

![QnABot JS 邏輯流程](./media/qnabot-js-logic-flow.png)

系統會針對每個收到的使用者輸入呼叫 `OnMessage`。 呼叫後，該方法就會存取您的 `qnamaker` 連接器，該連接器已使用您範例程式碼中 `.env` 檔案所提供的值預先設定。  qnamaker 方法 `getAnswers` 會將您的 Bot 連線到外部的 QnA Maker 知識庫。

---

使用者輸入會傳送至您的知識庫，並向使用者顯示最適合的回傳解答。

## <a name="create-a-qna-maker-service-and-publish-a-knowledge-base"></a>建立 QnA Maker 服務及發佈知識庫

第一步是建立 QnA Maker 服務。 依照 QnA Maker [文件](https://docs.microsoft.com/azure/cognitive-services/qnamaker/how-to/set-up-qnamaker-service-azure)中所述的步驟在 Azure 中建立服務。

接下來，您將使用 `smartLightFAQ.tsv` 檔案 (位於範例專案的 CognitiveModels 資料夾中) 來建立知識庫。 用於建立、定型及發佈 QnA Maker[知識庫](https://docs.microsoft.com/azure/cognitive-services/qnamaker/quickstarts/create-publish-knowledge-base)的步驟會列於 QnA Maker 文件中。 當您遵循這些步驟時，請將您的 KB 命名為 `qna`，並使用 `smartLightFAQ.tsv` 檔案來填入 KB。

> 注意： 本文也可用來存取您自己使用者所開發的 QnA Maker 知識庫。

## <a name="obtain-values-to-connect-your-bot-to-the-knowledge-base"></a>取得值，將您的 Bot 連線到知識庫

1. 在 [QnA Maker](https://www.qnamaker.ai/) 網站上，選取您的知識庫。
1. 開啟您的知識庫，然後選取 [設定]  。 記錄針對「服務名稱」  顯示的值。 使用 QnA Maker 入口網站介面時，此值很適合用於尋找您感興趣的知識庫。 但不會用於將 Bot 應用程式連線到此知識庫。
1. 從 Postman 範例 HTTP 要求，向下捲動以尋找 [部署詳細資料]  並記錄下列值：
   - POST /knowledgebases/\<knowledge-base-id>/generateAnswer
   - 主機：\<your-hostname> / / 以 /qnamaker 結尾的完整 URL
   - 授權：EndpointKey \<your-endpoint-key>

您主機名稱的完整 URL 字串看起來會類似 "https://< >.azure.net/qnamaker"。 這三個值會為您的應用程式提供透過 Azure QnA 服務連線到 QnA Maker 知識庫所需的資訊。  

## <a name="update-the-settings-file"></a>更新 settings 檔案

首先，將存取您知識庫所需的資訊 (包括主機名稱、端點金鑰和知識庫識別碼 (kbId)) 新增至設定檔。 這些是您在 QnA Maker 中從知識庫 [設定]  索引標籤儲存的值。

如果您未將其部署於生產環境，可將應用程式識別碼和密碼欄位保留空白。

> [!NOTE]
> 如果您要將 QnA Maker 知識庫的存取權新增到現有 Bot 應用程式中，請務必為 QnA 項目新增具參考價值的標題。 此區段內的 "name" 值會提供從您的應用程式存取此資訊所需的金鑰。

## <a name="ctabcs"></a>[C#](#tab/cs)

### <a name="update-your-appsettingsjson-file"></a>更新 appsettings.json 檔案

[!code-csharp[appsettings](~/../botbuilder-samples/samples/csharp_dotnetcore/11.qnamaker/appsettings.json)]

## <a name="javascripttabjs"></a>[JavaScript](#tab/js)

### <a name="update-your-env-file"></a>更新 .env 檔案

[!code-javascript[.env file](~/../botbuilder-samples/samples/javascript_nodejs/11.qnamaker/.env)]

---

## <a name="set-up-the-qna-maker-instance"></a>設定 QnA Maker 執行個體

首先，我們會建立用來存取 QnA Maker 知識庫的物件。

## <a name="ctabcs"></a>[C#](#tab/cs)

請確定您已為專案安裝 **Microsoft.Bot.Builder.AI.QnA** NuGet 套件。

在 **QnABot.cs** 的 `OnMessageActivityAsync` 方法中，我們會建立 QnAMaker 執行個體。 `QnABot` 類別中也會加入連線資訊名稱 (儲存在上述的 `appsettings.json` 中)。 如果您已在設定檔中選擇不同的知識庫連線資訊名稱，請務必在此更新名稱，以反映您所選擇的名稱。

**Bots/QnABot.cs** [!code-csharp[qna connection](~/../botbuilder-samples/samples/csharp_dotnetcore/11.qnamaker/Bots/QnABot.cs?range=32-39)]

## <a name="javascripttabjs"></a>[JavaScript](#tab/js)

請確認您的專案已安裝 **botbuilder-ai** npm 套件。

在我們的範例中，Bot 邏輯的程式碼會位在 **QnABot.js** 檔案中。

在 **QnABot.js** 檔案中，我們可使用 .env 檔案所提供的連線資訊來連線到 QnA Maker 服務：_this.qnaMaker_。

**bots/QnABot.js** [!code-javascript[QnAMaker](~/../botbuilder-samples/samples/javascript_nodejs/11.qnamaker/bots/QnABot.js?range=12-16)]

---

## <a name="calling-qna-maker-from-your-bot"></a>從您的 Bot 呼叫 QnA Maker

## <a name="ctabcs"></a>[C#](#tab/cs)

當您的 Bot 需要來自 QnAMaker 的解答時，從您的 Bot 程式碼呼叫 `GetAnswersAsync()`，以根據目前的內容取得適當解答。 如果您要存取自己的知識庫，請變更以下的_找不到解答_訊息，為您的使用者提供實用的指示。

**Bots/QnABot.cs** [!code-csharp[qna get answers](~/../botbuilder-samples/samples/csharp_dotnetcore/11.qnamaker/Bots/QnABot.cs?range=43-52)]

## <a name="javascripttabjs"></a>[JavaScript](#tab/js)

在 **QnABot.js** 檔案中，我們會將使用者的輸入傳遞至 QnA Maker 服務的 `getAnswers` 方法，以從知識庫中取得解答。 如果 QnA Maker 傳回回應，該回應會顯示給使用者。 否則，使用者會收到「找不到 QnA Maker 解答」的訊息。

**bots/QnABot.js** [!code-javascript[OnMessage](~/../botbuilder-samples/samples/javascript_nodejs/11.qnamaker/bots/QnABot.js?range=46-55)]

---

## <a name="test-the-bot"></a>測試 Bot

在您的電腦本機執行範例。 如果您尚未安裝 [Bot Framework Emulator](https://github.com/Microsoft/BotFramework-Emulator/blob/master/README.md#download)，請進行安裝。 如需進一步指示，請參閱 [C# 範例](https://aka.ms/cs-qna)或 [Javascript 範例](https://aka.ms/js-qna-sample)的讀我檔案。

啟動模擬器、連線到您的 Bot 並傳送如下所示的訊息。

![測試 qna 範例](../media/emulator-v4/qna-test-bot.png)

## <a name="additional-information"></a>其他資訊

### <a name="multi-turn-prompts"></a>多回合提示

QnA Maker 支援後續追蹤提示，也稱為多回合提示。
如果 QnA Maker 知識庫需要使用者的其他回應，QnA Maker 會傳送您可以用來提示使用者的內容資訊。 此資訊也會用來對 QnA Maker 服務進行任何後續追蹤呼叫。
在 4.6 版中，Bot Framework SDK 已新增此功能的支援。

若要建立這類知識庫，請參閱 QnA Maker 文件，以了解如何[使用後續追蹤提示建立多個對話回合](https://aka.ms/qnamaker-multiturn-conversation)。 <!--To learn how to incorporate multi-turn support in your bot, take a look at the QnA Maker Multi-turn [[**C#**](https://aka.ms/cs-qna-multiturn) | [**JS**](https://aka.ms/js-qna-multiturn)] sample.-->

<!--TODO: Update code based on final sample 
The following code snippets come from the proof-of-concept **multi-turn QnA Maker prompts** sample for
[**C#**](https://github.com/microsoft/BotBuilder-Samples/tree/master/experimental/qnamaker-prompting/csharp_dotnetcore) and
[**JavaScript**](https://github.com/microsoft/BotBuilder-Samples/tree/master/experimental/qnamaker-prompting/javascript_nodejs).
-->
<!--
#### Sample code

This sample uses a custom QnA dialog to track state for QnA Maker and handle the user's input and QnA Maker's response. When the user sends a message to the bot, the bot treats the input as either an initial query or a response to a follow-up question. The bot starts or continues its QnA dialog, which tracks the QnA Maker context information.

1. When the dialog **starts**, it makes an initial call to the QnA Maker service. QnA Maker context information **is not** included in the call.
1. When the dialog **continues**, it makes a follow-up call to the QnA Maker service. QnA Maker context information included **is** included in the call.
1. In either case, the dialog evaluates the query results.
   - Each QnA Maker result includes either an answer or a follow-up question to the user's initial query. (This sample uses only the first result.)
   - If the result includes follow-up prompts, the dialog prompts the user, saves the context information, and stays on the dialog stack, waiting for additional information from the user.
   - Otherwise, the result represents an answer, the dialog sends the answer and ends.

This sample implements this across two dialog classes:

- The base _functional dialog_ defines the begin, continue, and state logic for the dialog, notably, the _run state machine_ method.
- The derived _QnA dialog_ defines the logic to call QnA Maker, evaluate its response, and send an answer or follow-up question to the user.

#### QnA Maker context

You need to track context information for the QnA Maker service.
The dialog funnels incoming activities through its _run state machine_ method, which:

1. Gets previous QnA Maker context, if any, from state.
1. Calls its _process_ method to call QnA Maker and generates a response for the user.
1. If the result was a follow-up question, sends the question, saves new QnA Maker context to state, and waits for more input on the next turn.
1. If the result was an answer, sends the answer and ends the dialog.

##### [C#](#tab/csharp)

**Dialogs\FunctionDialogBase.cs** defines the **RunStateMachineAsync** method.

```csharp
private async Task<DialogTurnResult> RunStateMachineAsync(DialogContext dialogContext, CancellationToken cancellationToken)
{
     // Get the Process function's current state from the dialog state
     var oldState = GetPersistedState(dialogContext.ActiveDialog);

     // Run the Process function.
     var (newState, output, result) = await ProcessAsync(oldState, dialogContext.Context.Activity).ConfigureAwait(false);

     // If we have output to send then send it.
     foreach (var activity in output)
     {
          await dialogContext.Context.SendActivityAsync(activity).ConfigureAwait(false);
     }

     // If we have new state then we must still be running.
     if (newState != null)
     {
          // Add the state returned from the Process function to the dialog state.
          dialogContext.ActiveDialog.State[FunctionStateName] = newState;

          // Return Waiting indicating this dialog is still in progress.
          return new DialogTurnResult(DialogTurnStatus.Waiting);
     }
     else
     {
          // The Process function indicates it's completed by returning null for the state.
          return await dialogContext.EndDialogAsync(result).ConfigureAwait(false);
     }
}
```

##### [JavaScript](#tab/javascript)

**dialogs/functionDialogBase.js** defines the **runStateMachine** method.

```javascript
async runStateMachine(dc) {

     var oldState = this.getPersistedState(dc.activeDialog);

     var processResult = await this.processAsync(oldState, dc.context.activity);

     var newState = processResult[0];
     var output = processResult[1];
     var result = processResult[2];

     await dc.context.sendActivity(output);

     if(newState != null){
          dc.activeDialog.state[functionStateName] = newState;
          return { status: DialogTurnStatus.waiting };
     }
     else{
          return await dc.endDialog();
     }
}
```

---

#### QnA Maker input and response

You need to provide any previous context information when you call the QnA Maker service.
The dialog handles the call to QnA Maker in its _process_ method, which:

1. Makes the call to QnA Maker, passing in the previous context, if any.
   - This sample uses a _query QnA service_ helper method to format the parameters and make the call.
   - Importantly, if this is a follow-up call to QnA Maker, the _QnA Maker options_ should include values for the _QnA request context_ and the _QnA question ID_.
1. Gets QnA Maker's response and any follow-up prompts from the top result.
1. If the result was a follow-up question:
   - Generates a hero card that contains the question and options for the user's response.
   - Includes the hero card in a message activity.
   - Returns the message activity and the new QnA Maker context.
1. If the result was an answer, returns a message activity that contains the answer.

You can format a follow-up activity in many ways. This sample uses a hero card. However, using suggested actions would be another option.

##### [C#](#tab/csharp)

**Dialogs\QnADialog.cs** defines the **RunStateMachineAsync** method.

```csharp
protected override async Task<(object newState, IEnumerable<Activity> output, object result)> ProcessAsync(object oldState, Activity inputActivity)
{
     Activity outputActivity = null;
     QnABotState newState = null;

     var query = inputActivity.Text;
     var qnaResult = await _qnaService.QueryQnAServiceAsync(query, (QnABotState)oldState);
     var qnaAnswer = qnaResult[0].Answer;
     var prompts = qnaResult[0].Context?.Prompts;

     if (prompts == null || prompts.Length < 1)
     {
          outputActivity = MessageFactory.Text(qnaAnswer);
     }
     else
     {
          // Set bot state only if prompts are found in QnA result
          newState = new QnABotState
          {
          PreviousQnaId = qnaResult[0].Id,
          PreviousUserQuery = query
          };

          outputActivity = CardHelper.GetHeroCard(qnaAnswer, prompts);
     }

     return (newState, new Activity[] { outputActivity }, null);
}
```

##### [JavaScript](#tab/javascript)

**dialogs/qnaDialog.js** defines the **runStateMachine** method.

```javascript
async processAsync(oldState, activity){

     var newState = null;
     var query = activity.text;
     var qnaResult = await QnAServiceHelper.queryQnAService(query, oldState);
     var qnaAnswer = qnaResult[0].answer;

     var prompts = null;
     if(qnaResult[0].context != null){
          prompts = qnaResult[0].context.prompts;
     }

     var outputActivity = null;
     if(prompts == null || prompts.length < 1){
          outputActivity = MessageFactory.text(qnaAnswer);
     }
     else{
          var newState = {
               PreviousQnaId: qnaResult[0].id,
               PreviousUserQuery: query
          }

          outputActivity = CardHelper.GetHeroCard(qnaAnswer, prompts);
     }

     return [newState, outputActivity , null];
}  
```

---
-->
## <a name="next-steps"></a>後續步驟

QnA Maker 可以結合其他認知服務，讓 Bot 具有更強大的功能。 分派工具可讓您在 Bot 中結合 QnA 和 Language Understanding (LUIS)。

> [!div class="nextstepaction"]
> [使用分派工具結合 LUIS 和 QnA 服務](./bot-builder-tutorial-dispatch.md)
