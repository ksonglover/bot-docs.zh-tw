---
title: 處理使用者動作 | Microsoft Docs
description: 了解如何使用適用於 Node.js 的 Bot Framework SDK，藉由讓 Bot 聽取和處理包含某些關鍵字的使用者輸入，來處理使用者動作。
author: DucVo
ms.author: v-ducvo
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: 7ca595b1c24769addfbdf7975c48d3a052c4a2de
ms.sourcegitcommit: b15cf37afc4f57d13ca6636d4227433809562f8b
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 01/11/2019
ms.locfileid: "54226003"
---
# <a name="handle-user-actions"></a>處理使用者動作

[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]

> [!div class="op_single_selector"]
> - [.NET](../dotnet/bot-builder-dotnet-global-handlers.md)
> - [Node.js](../nodejs/bot-builder-nodejs-dialog-actions.md)

使用者通常會嘗試使用「help」、「cancel」或「start over」等關鍵字，來存取 Bot 內的某些功能。 使用者會在對話中間這麼做，而 Bot 這時會預期不同的回應。 藉由實作**動作**，您可以將 Bot 設計為會以更妥善的方式處理這類要求。 處理常式會檢查使用者輸入中是否有您指定的關鍵字 (例如，「help」、「cancel」或「start over」)，並做出適當回應。 

![使用者的說話方式](../media/designing-bots/capabilities/trigger-actions.png)

## <a name="action-types"></a>動作類型

下表列出您可以附加至對話方塊的動作類型。 每個動作名稱的連結，會帶您前往可提供更多該動作詳細資料的章節。

| 動作 | 影響範圍 | 說明 |
|------|------| ---- |
| [triggerAction](#bind-a-triggeraction) | 全域 | 將動作繫結至對話，其會清除對話堆疊，並將自身推至堆疊底部。 使用 `onSelectAction` 選項可覆寫這個預設行為。 |
| [customAction](#bind-a-customaction) | 全域 | 將自訂動作繫結至 Bot，其可處理資訊或採取動作，而不會影響對話方塊堆疊。 使用 `onSelectAction` 選項可自訂此動作的功能。 |
[beginDialogAction](#bind-a-begindialogaction) | 內容相關 | 將動作繫結至對話，其會在觸發時開始另一段對話。 系統會將開始中的對話方塊推至堆疊上，並於結束後消失。 |
[reloadAction](#bind-a-reloadaction) | 內容相關 | 將動作繫結至對話方塊，其會在觸發時重新載入對話方塊。 您可以使用 `reloadAction` 來處理使用者語句，像是「start over」。 |
[cancelAction](#bind-a-cancelaction) | 內容相關 | 將動作繫結至對話方塊，其會在觸發時取消對話方塊。 您可以使用 `cancelAction` 來處理使用者語句，像是「cancel」或「nevermind」。 |
[endConversationAction](#bind-an-endconversationaction) | 內容相關 | 將動作繫結至對話方塊，其會在觸發時結束與使用者的對話。 您可以使用 `endConversationAction` 來處理使用者語句，像是「goodbye」。 |

## <a name="action-precedence"></a>動作優先順序 

當 Bot 收到來自使用者的語句時，它會檢查對話方塊堆疊上所有已註冊動作的語句。 比對會在堆疊中由上而下進行。 如果沒有相符者，則會檢查所有全域動作 `matches` 的選項語句。

若您在不同內容中使用相同命令，動作優先順序就很重要。 例如，您可以對 Bot 使用「Help」命令來獲得一般說明。 您也可以針對每個工作使用「Help」，但這些「Help」命令會與每個工作的內容相關。 如需詳細說明這一點的實用範例，請參閱[回應使用者輸入](bot-builder-nodejs-dialog-manage-conversation-flow.md#respond-to-user-input)。

## <a name="bind-actions-to-dialog"></a>將動作繫結至對話方塊

使用者說出的語句或按下按鈕的動作可以*觸發*與[對話方塊](https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.dialog.html)相關聯的動作。
如果指定「matches」，此動作將會聽取使用者所說出會觸發動作的字組或片語。  `matches` 選項可採用規則運算式或[辨識器][RecognizeIntent]名稱。
若要將動作繫結至按下按鈕，請使用 [CardAction.dialogAction()][CardAction] 來觸發動作。

動作皆「可鏈結」，以讓您將所需數量的動作繫結至對話方塊。

### <a name="bind-a-triggeraction"></a>繫結 triggerAction

若要將 [triggerAction][triggerAction] 繫結至對話方塊，請執行下列動作：

```javascript
// Order dinner.
bot.dialog('orderDinner', [
    //...waterfall steps...
])
// Once triggered, will clear the dialog stack and pushes
// the 'orderDinner' dialog onto the bottom of stack.
.triggerAction({
    matches: /^order dinner$/i
});
```

將 `triggerAction` 繫結至對話方塊便會向 Bot 註冊。 一經觸發，`triggerAction` 就會清除對話方塊堆疊，並將已觸發的對話方塊推至堆疊上。 此動作最適合用來切換[對話主題](bot-builder-nodejs-dialog-manage-conversation-flow.md#change-the-topic-of-conversation)，或用來允許使用者要求任意的獨立工作。 如果您想要覆寫此動作會清除對話方塊堆疊的行為，請將 `onSelectAction` 選項新增至 `triggerAction`。

下列程式碼片段說明如何在不清除對話方塊堆疊的情況下，從全域內容提供一般說明。

```javascript
bot.dialog('help', function (session, args, next) {
    //Send a help message
    session.endDialog("Global help menu.");
})
// Once triggered, will start a new dialog as specified by
// the 'onSelectAction' option.
.triggerAction({
    matches: /^help$/i,
    onSelectAction: (session, args, next) => {
        // Add the help dialog to the top of the dialog stack 
        // (override the default behavior of replacing the stack)
        session.beginDialog(args.action, args);
    }
});
```

在此情況下，`triggerAction` 會附加至 `help` 對話方塊本身 (而不是 `orderDinner` 對話)。 `onSelectAction` 選項可讓您在不清除對話堆疊的情況下，啟動這個對話方塊。 這可讓您處理全域要求，例如「help」、「about」、「support」等等。請注意，`onSelectAction` 選項會明確地呼叫 `session.beginDialog` 方法來啟動已觸發的對話方塊。 已觸發對話方塊的識別碼會透過 `args.action` 提供。 請勿將對話方塊識別碼 (例如：「help」) 手工編碼至這個方法，否則，您可能會遇到執行階段錯誤。 如果您想要對 `orderDinner` 工作本身觸發內容相關的說明訊息，則請考慮將 `beginDialogAction` 改為附加到 `orderDinner` 對話方塊。

### <a name="bind-a-customaction"></a>繫結 customAction

不同於其他動作類型，`customAction` 並未定義任何預設動作。 您要負責定義這個動作的作用。 使用 `customAction` 的好處是您可以選擇處理使用者要求，而不必管理對話方塊堆疊。 當 `customAction` 觸發時，`onSelectAction` 選項可以處理要求，而不必將新的對話方塊推至堆疊上。 此動作完成後，控制權會交回給堆疊最上方的對話方塊，然後 Bot 便可繼續。

您可以使用 `customAction` 提供一般且快速的動作要求，例如，「What is the temperature outside right now?」、「What time is it right now in Paris?」、「Remind me to buy milk at 5pm today」等等。這些是 Bot 所能執行，且不必管理堆疊的一般動作。

`customAction` 的另一個主要差異是，它會繫結至 Bot 而不是對話方塊。

下列程式碼範例說明如何將 `customAction` 繫結至聽取要求的 `bot`，以設定提醒。

```javascript
bot.customAction({
    matches: /remind|reminder/gi,
    onSelectAction: (session, args, next) => {
        // Set reminder...
        session.send("Reminder is set.");
    }
})
```

### <a name="bind-a-begindialogaction"></a>繫結 beginDialogAction

將 `beginDialogAction` 繫結至對話方塊會向對話註冊動作。 這個方法會在觸發時啟動另一個對話方塊。 此動作行為類似於呼叫 [beginDialog](https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.session.html#begindialog) 方法。 系統會將新的對話方塊推至對話方塊堆疊最上方，因此，新對話不會自動結束目前的工作。 當新的對話方塊結束後，目前的工作就會繼續。 

下列程式碼片段說明如何將 [beginDialogAction][beginDialogAction] 繫結至對話方塊。

```javascript
// Order dinner.
bot.dialog('orderDinner', [
    //...waterfall steps...
])
// Once triggered, will start the 'showDinnerCart' dialog.
// Then, the waterfall will resumed from the step that was interrupted.
.beginDialogAction('showCartAction', 'showDinnerCart', {
    matches: /^show cart$/i
});

// Show dinner items in cart
bot.dialog('showDinnerCart', function(session){
    for(var i = 1; i < session.conversationData.orders.length; i++){
        session.send(`You ordered: ${session.conversationData.orders[i].Description} for a total of $${session.conversationData.orders[i].Price}.`);
    }

    // End this dialog
    session.endDialog(`Your total is: $${session.conversationData.orders[0].Price}`);
});
```

如果您需要將額外引數傳遞至新的對話方塊，您可以在動作中新增 [`dialogArgs`](https://docs.botframework.com/en-us/node/builder/chat-reference/interfaces/_botbuilder_d_.idialogactionoptions#dialogargs) 選項。

使用上述範例，您就可以將它修改為接受透過 `dialogArgs` 傳遞的引數。

```javascript
// Order dinner.
bot.dialog('orderDinner', [
    //...waterfall steps...
])
// Once triggered, will start the 'showDinnerCart' dialog.
// Then, the waterfall will resumed from the step that was interrupted.
.beginDialogAction('showCartAction', 'showDinnerCart', {
    matches: /^show cart$/i,
    dialogArgs: {
        showTotal: true;
    }
});

// Show dinner items in cart with the option to show total or not.
bot.dialog('showDinnerCart', function(session, args){
    for(var i = 1; i < session.conversationData.orders.length; i++){
        session.send(`You ordered: ${session.conversationData.orders[i].Description} for a total of $${session.conversationData.orders[i].Price}.`);
    }

    if(args && args.showTotal){
        // End this dialog with total.
        session.endDialog(`Your total is: $${session.conversationData.orders[0].Price}`);
    }
    else{
        session.endDialog(); // Ends without a message.
    }
});
```

### <a name="bind-a-reloadaction"></a>繫結 reloadAction

將 `reloadAction` 繫結至對話方塊便會將其註冊至對話中方塊。 將這個動作繫結至對話方塊，會讓對話方塊在觸發動作時重新啟動。 觸發此動作類似於呼叫 [replaceDialog](https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.session.html#replacedialog) 方法。 這適合用來實作邏輯以處理使用者語句 (像是「start over」) 或建立[迴圈](bot-builder-nodejs-dialog-replace.md#repeat-an-action)。

下列程式碼片段說明如何將 [reloadAction][reloadAction] 繫結至對話方塊。

```javascript
// Order dinner.
bot.dialog('orderDinner', [
    //...waterfall steps...
])
// Once triggered, will restart the dialog.
.reloadAction('startOver', 'Ok, starting over.', {
    matches: /^start over$/i
});
```

如果您需要將額外引數傳遞至重新載入的對話方塊，您可以在動作中新增 [`dialogArgs`](https://docs.botframework.com/en-us/node/builder/chat-reference/interfaces/_botbuilder_d_.idialogactionoptions#dialogargs) 選項。 此選項會傳遞至 `args` 參數。 重寫上述程式碼範例以便在重新載入動作上收到引數，會看起來像這樣：

```javascript
// Order dinner.
bot.dialog('orderDinner', [
    function(session, args, next){
        if(args && args.isReloaded){
            // Reload action was triggered.
        }

        session.send("Lets order some dinner!");
        builder.Prompts.choice(session, "Dinner menu:", dinnerMenu);
    }
    //...other waterfall steps...
])
// Once triggered, will restart the dialog.
.reloadAction('startOver', 'Ok, starting over.', {
    matches: /^start over$/i,
    dialogArgs: {
        isReloaded: true;
    }
});
```

### <a name="bind-a-cancelaction"></a>繫結 cancelAction

繫結 `cancelAction` 會將其註冊至對話方塊中。 一經觸發，此動作就會突然結束對話方塊。 對話方塊結束後，父對話方塊會以已繼續程式碼 (指出其已 `canceled`) 來繼續進行。 這個動作可讓您處理「nevermind」或「cancel」之類的語句。 如果您需要改為以程式設計方式取消對話方塊，請參閱[取消對話方塊](bot-builder-nodejs-dialog-replace.md#cancel-a-dialog)。 如需*已繼續程式碼*的詳細資訊，請參閱[提示結果](bot-builder-nodejs-dialog-prompt.md#prompt-results)。 

下列程式碼片段說明如何將 [cancelAction][cancelAction] 繫結至對話方塊。

```javascript
// Order dinner.
bot.dialog('orderDinner', [
    //...waterfall steps...
])
//Once triggered, will end the dialog.
.cancelAction('cancelAction', 'Ok, cancel order.', {
    matches: /^nevermind$|^cancel$|^cancel.*order/i
});
```

### <a name="bind-an-endconversationaction"></a>繫結 endConversationAction

繫結 `endConversationAction` 會將其註冊至對話方塊中。 一經觸發，這個動作就會結束與使用者的對話。 觸發此動作類似於呼叫 [endConversation](https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.session.html#endconversation) 方法。 交談結束後，適用於 Node.js 的 Bot Framework SDK 會清除對話堆疊和保存的狀態資料。 如需所保存狀態資料的詳細資訊，請參閱[管理狀態資料](bot-builder-nodejs-state.md)。

下列程式碼片段說明如何將 [endConversationAction][endConversationAction] 繫結至對話方塊。

```javascript
// Order dinner.
bot.dialog('orderDinner', [
    //...waterfall steps...
])
// Once triggered, will end the conversation.
.endConversationAction('endConversationAction', 'Ok, goodbye!', {
    matches: /^goodbye$/i
});
```

## <a name="confirm-interruptions"></a>確認中斷

上述絕大多數 (雖然不是全部) 的動作都會中斷一般對話流程。 許多動作具有干擾性，所以應謹慎處理。 例如，`triggerAction`、`cancelAction` 或 `endConversationAction` 會清除對話方塊堆疊。 如果使用者不小心觸發上述其中一個動作，他們就必須讓工作重新開始。 為了確保使用者真的想要觸發這些動作，您可以將`confirmPrompt`選項新增到這些動作。 `confirmPrompt` 會詢問使用者是否確定要取消或結束目前的工作。 它可讓使用者改變心意，並繼續原本的程序。

下面的程式碼片段顯示 [cancelAction][cancelAction] 與 [confirmPrompt](http://docs.botframework.com/en-us/node/builder/chat-reference/interfaces/_botbuilder_d_.itriggeractionoptions#confirmprompt)，可確保使用者真的想要取消訂單程序。

```javascript
// Order dinner.
bot.dialog('orderDinner', [
    //...waterfall steps...
])
// Confirm before triggering the action.
// Once triggered, will end the dialog. 
.cancelAction('cancelAction', 'Ok, cancel order.', {
    matches: /^nevermind$|^cancel$|^cancel.*order/i,
    confirmPrompt: "Are you sure?"
});
```

此動作觸發後會詢問使用者「是否確定？」 使用者必須回答「是」才能繼續該動作，回答「否」則可取消動作並從原來的地方繼續。

## <a name="next-steps"></a>後續步驟

**動作**可讓您預期使用者的要求，並讓 Bot 妥善處理這些要求。 這些動作中有許多會干擾目前的對話流程。 如果您想要讓使用者能夠跳出並繼續對話流程，您需要先儲存使用者狀態再跳出。讓我們更加深入地了解如何儲存使用者狀態和管理狀態資料。

> [!div class="nextstepaction"]
> [管理狀態資料](bot-builder-nodejs-state.md)


[triggerAction]: https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.dialog.html#triggeraction

[cancelAction]: https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.dialog.html#cancelaction

[reloadAction]: https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.dialog.html#reloadaction

[beginDialogAction]: https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.dialog.html#begindialogaction

[endConversationAction]: https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.dialog.html#endconversationaction

[RecognizeIntent]: bot-builder-nodejs-recognize-intent-messages.md

[CardAction]: https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.cardaction#dialogaction
