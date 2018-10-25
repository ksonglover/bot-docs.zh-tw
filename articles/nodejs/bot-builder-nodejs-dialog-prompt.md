---
title: 提示使用者輸入 | Microsoft Docs
description: 瞭解如何使用提示來收集適用於 Node.js 的 Bot Builder SDK 使用者輸入。
author: v-ducvo
ms.author: v-ducvo
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: d448862720d159ee58883edfb42aab211d72e81a
ms.sourcegitcommit: b78fe3d8dd604c4f7233740658a229e85b8535dd
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/24/2018
ms.locfileid: "50000345"
---
# <a name="prompt-for-user-input"></a>提示使用者輸入

[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]

適用於 Node.js 的 Bot Builder SDK 提供一組內建的提示以簡化收集使用者的輸入。 

A *提示* 每當 Bot 需要使用者輸入會使用。 您可以透過像是瀑布般連接的提示來使用提示要求使用者輸入一個系列。 您可以搭配提示使用[瀑布](bot-builder-nodejs-dialog-waterfall.md)來協助您在在 Bot [管理對話工作流程](bot-builder-nodejs-manage-conversation-flow.md)。 

文章將協助您瞭解提示的運作方式，以及如何使用它們來向使用者收集資訊。

## <a name="prompts-and-responses"></a>提示與回應

當您需要使用者輸入時，您可以傳送提示，等候使用者回應輸入，然後處理後並回傳回應給使用者。

下列程式碼範例會提示使用者輸入其名稱，並以問候訊息加以回應。

```javascript
bot.dialog('greetings', [
    // Step 1
    function (session) {
        builder.Prompts.text(session, 'Hi! What is your name?');
    },
    // Step 2
    function (session, results) {
        session.endDialog(`Hello ${results.response}!`);
    }
]);
```

使用此基本建構，您可以透過增加 Bot 所要求的提示與回應來建構您的對話工作流程。

## <a name="prompt-results"></a>提示結果 

內建的提示會在傳回使用者的回應中`results.response` 欄位作為[對話](bot-builder-nodejs-dialog-overview.md)應用。 JSON 物件中，回應會回傳`results.response.entity`欄位。 任何一種[對話的處理常式](bot-builder-nodejs-dialog-overview.md#dialog-handlers)都可以接收提示的結果。 一旦 Bot 收到回應時，它可以使用它或將它藉由呼叫[`session.endDialogWithResult`][EndDialogWithResult]方法回傳給呼叫對話方塊。

下列程式碼範例顯示如何使用`session.endDialogWithResult`方法來將提示結果回傳呼叫對話方塊。 在此範例中，`greetings`對話方塊使用`askName`對話方塊回傳的提示結果依照名稱對用戶進行問候。

```javascript
// Ask the user for their name and greet them by name.
bot.dialog('greetings', [
    function (session) {
        session.beginDialog('askName');
    },
    function (session, results) {
        session.endDialog(`Hello ${results.response}!`);
    }
]);
bot.dialog('askName', [
    function (session) {
        builder.Prompts.text(session, 'Hi! What is your name?');
    },
    function (session, results) {
        session.endDialogWithResult(results);
    }
]);
```

## <a name="prompt-types"></a>提示類型
適用於 Node.js 的 Bot Builder SDK 包含了許多不同類型的內建提示。 

|**提示類型**     | **說明** |     
| ------------------ | --------------- |
|[Prompts.text](#promptstext) | 要求使用者輸入文字字串。 |     
|[Prompts.confirm](#promptsconfirm) | 要求使用者確認動作。| 
|[Prompts.number](#promptsnumber) | 要求使用者輸入數字。     |
|[Prompts.number](#promptstime) | 要求使用者輸入時間或日期/時間。      |
|[Prompts.number](#promptschoice) | 要求使用者選擇選項清單。    |
|[Prompts.attachment](#promptsattachment) | 要求使用者上傳圖片或影片。|       

下列各區段提供關於各類型提示的其他詳細資料。

### <a name="promptstext"></a>Prompts.text

使用 [Prompts.text()][PromptsText] 方法來要求使用者提供**的文字字串**。 提示會以 [IPromptTextResult][IPromptTextResult] 回傳使用者的回應。

```javascript
builder.Prompts.text(session, "What is your name?");
```

### <a name="promptsconfirm"></a>Prompts.confirm

使用 [Prompts.confirm()][PromptsConfirm] 方法來要求使用者確認動作並回應**是/否**。 提示會以 [IPromptConfirmResult](http://docs.botframework.com/en-us/node/builder/chat-reference/interfaces/_botbuilder_d_.ipromptconfirmresult.html) 回傳使用者的回應。

```javascript
builder.Prompts.confirm(session, "Are you sure you wish to cancel your order?");
```

### <a name="promptsnumber"></a>Prompts.number

使用 [Prompts.number()][PromptsNumber] 方法來要求使用者提供**數字**。 提示會以 [IPromptConfirmResult](http://docs.botframework.com/en-us/node/builder/chat-reference/interfaces/_botbuilder_d_.ipromptnumberresult.html) 回傳使用者的回應。

```javascript
builder.Prompts.number(session, "How many would you like to order?");
```

### <a name="promptstime"></a>Prompts.time

使用 [Prompts.time()][PromptsTime] 方法來要求使用者提供**時間**或是**日期/時間**。 提示會以 [IPromptTimeResult](http://docs.botframework.com/en-us/node/builder/chat-reference/interfaces/_botbuilder_d_.iprompttimeresult.html) 回傳使用者的回應。 此架構使用 [Chrono](https://github.com/wanasit/chrono) 資料庫來剖析使用者的回應，並支援 (例如「再 5 分鐘 」) 的相對回應和非相對回應 (例如「6 月 6 號下午 2 點」)。

[Results.response](http://docs.botframework.com/en-us/node/builder/chat-reference/interfaces/_botbuilder_d_.iprompttimeresult.html#response) 欄位中，它代表使用者的回應，包含指定日期和時間的[實體](http://docs.botframework.com/en-us/node/builder/chat-reference/interfaces/_botbuilder_d_.ientity.html)物件。 若要解決日期和時間變成 JavaScript`Date` 物件，請使用 [EntityRecognizer.resolveTime()](http://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.entityrecognizer.html#resolvetime) 方法。

> [!TIP] 
> 使用者輸入的時間會根據 Bot 伺服器的時區時間轉換成 UTC 時間。 由於伺服器可能位於與使用者不同的時區，因此，請務必將時區納入考量。 若要轉換成使用者當地的日期和時間，可以考慮詢問使用者位於哪個時區。

```javascript
bot.dialog('createAlarm', [
    function (session) {
        session.dialogData.alarm = {};
        builder.Prompts.text(session, "What would you like to name this alarm?");
    },
    function (session, results, next) {
        if (results.response) {
            session.dialogData.name = results.response;
            builder.Prompts.time(session, "What time would you like to set an alarm for?");
        } else {
            next();
        }
    },
    function (session, results) {
        if (results.response) {
            session.dialogData.time = builder.EntityRecognizer.resolveTime([results.response]);
        }

        // Return alarm to caller  
        if (session.dialogData.name && session.dialogData.time) {
            session.endDialogWithResult({ 
                response: { name: session.dialogData.name, time: session.dialogData.time } 
            }); 
        } else {
            session.endDialogWithResult({
                resumed: builder.ResumeReason.notCompleted
            });
        }
    }
]);
```

### <a name="promptschoice"></a>Prompts.choice

使用[Prompts.choice()] [ PromptsChoice]方法來要求使用者**從選項清單中選擇**。 使用者可以透過輸入他們所選擇的選項相關的號碼來傳達其選取項目，或輸入他們所選擇的選項名稱。 支援的選項名稱的完整名稱或部分名稱。 在提示字元以 [IPromptChoiceResult](http://docs.botframework.com/en-us/node/builder/chat-reference/interfaces/_botbuilder_d_.ipromptchoiceresult.html) 回傳使用者的回應。 

為指定呈現給使用者的清單樣式，請設定 [IPromptOptions.listStyle](http://docs.botframework.com/en-us/node/builder/chat-reference/interfaces/_botbuilder_d_.ipromptoptions.html#liststyle) 屬性。 以下表格顯示`ListStyle`此屬性的列舉值。


`ListStyle`列舉值如下所示：

| 索引 | 名稱 | 說明 |
| ---- | ---- | ---- |
| 0 | None | 轉譯任何清單。 當列表作為提示的一部分時使用。 |
| 1 | 內嵌 | 選項呈現為「1 的內嵌清單。 紅色，2。 綠色或 3。 藍色」。 |
| 2 | list | 選項呈現為編號的清單。 |
| 3 | 按鈕 | 選項會顯示為支援按鈕通道的按鈕。 其他通道會將其轉譯為文字。 |
| 4 | 自動 | 樣式會自動根據通道和選項數量進行選擇。 | 

您可以從這個列舉`builder`物件存取，或您可以提供索引選擇`ListStyle`。 例如，下列程式碼片段中的兩個敘述句都完成同樣的工作。

```javascript
// ListStyle passed in as Enum
builder.Prompts.choice(session, "Which color?", "red|green|blue", { listStyle: builder.ListStyle.button });

// ListStyle passed in as index
builder.Prompts.choice(session, "Which color?", "red|green|blue", { listStyle: 3 });
```

若要指定選項的清單，您可以使用直立線符號分隔 (`|`) 字串、字串陣列或物件對應。

直立線符號分隔的字串: 

```javascript
builder.Prompts.choice(session, "Which color?", "red|green|blue");
```

字串陣列：

```javascript
builder.Prompts.choice(session, "Which color?", ["red","green","blue"]);
```

物件對應： 

```javascript
var salesData = {
    "west": {
        units: 200,
        total: "$6,000"
    },
    "central": {
        units: 100,
        total: "$3,000"
    },
    "east": {
        units: 300,
        total: "$9,000"
    }
};

bot.dialog('getSalesData', [
    function (session) {
        builder.Prompts.choice(session, "Which region would you like sales for?", salesData); 
    },
    function (session, results) {
        if (results.response) {
            var region = salesData[results.response.entity];
            session.send(`We sold ${region.units} units for a total of ${region.total}.`); 
        } else {
            session.send("OK");
        }
    }
]);
```

### <a name="promptsattachment"></a>Prompts.attachment

使用[Prompts.attachment()][PromptsAttachment] 方法來要求使用者上傳像是影像或影片檔。 提示會以 [IPromptAttachmentResult](http://docs.botframework.com/en-us/node/builder/chat-reference/interfaces/_botbuilder_d_.ipromptattachmentresult.html) 回傳使用者的回應。

```javascript
builder.Prompts.attachment(session, "Upload a picture for me to transform.");
```

## <a name="next-steps"></a>後續步驟

既然您已經知道如何逐步執行透過使用者瀑布圖及提示他們提供資訊，讓我們看看該如何協助您更好管理對話工作流程。

> [!div class="nextstepaction"]
> [管理對話工作流程](bot-builder-nodejs-dialog-manage-conversation-flow.md)


[SendAttachments]: bot-builder-nodejs-send-receive-attachments.md
[SendCardWithButtons]: bot-builder-nodejs-send-rich-cards.md
[RecognizeUserIntent]: bot-builder-nodejs-recognize-intent-messages.md
[SaveUserData]: bot-builder-nodejs-save-user-data.md

[UniversalBot]: https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.universalbot.html
[ChatConnector]: https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.chatconnector.html
[Session]: https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.session


[SendTyping]: https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.session#sendtyping

[EndDialogWithResult]: https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.session.html#enddialogwithresult

[IPromptResult]: https://docs.botframework.com/en-us/node/builder/chat-reference/interfaces/_botbuilder_d_.ipromptresult.html

[Result_Response]: https://docs.botframework.com/en-us/node/builder/chat-reference/interfaces/_botbuilder_d_.ipromptresult.html#reponse

[ResumeReason]: https://docs.botframework.com/en-us/node/builder/chat-reference/enums/_botbuilder_d_.resumereason.html

[Result_Resumed]: https://docs.botframework.com/en-us/node/builder/chat-reference/interfaces/_botbuilder_d_.ipromptresult.html#resumed

[entity]: https://docs.botframework.com/en-us/node/builder/chat-reference/interfaces/_botbuilder_d_.ientity.html

[ResolveTime]: https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.entityrecognizer.html#resolvetime

[PromptsRef]: https://docs.botframework.com/en-us/node/builder/chat-reference/interfaces/_botbuilder_d_.__global.iprompts.html

[PromptsText]: https://docs.botframework.com/en-us/node/builder/chat-reference/interfaces/_botbuilder_d_.__global.iprompts.html#text

[IPromptTextResult]: https://docs.botframework.com/en-us/node/builder/chat-reference/interfaces/_botbuilder_d_.iprompttextresult.html

[PromptsConfirm]: https://docs.botframework.com/en-us/node/builder/chat-reference/interfaces/_botbuilder_d_.__global.iprompts.html#confirm

[IPromptConfirmResult]: https://docs.botframework.com/en-us/node/builder/chat-reference/interfaces/_botbuilder_d_.ipromptconfirmresult.html

[PromptsNumber]: https://docs.botframework.com/en-us/node/builder/chat-reference/interfaces/_botbuilder_d_.__global.iprompts.html#number

[IPromptNumberResult]: https://docs.botframework.com/en-us/node/builder/chat-reference/interfaces/_botbuilder_d_.ipromptnumberresult.html

[PromptsTime]: https://docs.botframework.com/en-us/node/builder/chat-reference/interfaces/_botbuilder_d_.__global.iprompts.html#time

[IPromptTimeResult]: https://docs.botframework.com/en-us/node/builder/chat-reference/interfaces/_botbuilder_d_.iprompttimeresult.html

[PromptsChoice]: https://docs.botframework.com/en-us/node/builder/chat-reference/interfaces/_botbuilder_d_.__global.iprompts.html#choice

[IPromptChoiceResult]: https://docs.botframework.com/en-us/node/builder/chat-reference/interfaces/_botbuilder_d_.ipromptchoiceresult.html

[PromptsAttachment]: https://docs.botframework.com/en-us/node/builder/chat-reference/interfaces/_botbuilder_d_.__global.iprompts.html#attachment

[IPromptAttachmentResult]: https://docs.botframework.com/en-us/node/builder/chat-reference/interfaces/_botbuilder_d_.ipromptattachmentresult.html
