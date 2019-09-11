---
title: 對話方塊 | Microsoft Docs
description: 深入了解如何使用適用於 Node.js 的 Bot Framework SDK，取代重新提示輸入的對話及管理交談流程。
author: v-ducvo
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: 92782463a4f7d0de6d0fa30693542eea051366bd
ms.sourcegitcommit: a6d02ec4738e7fc90b7108934740e9077667f3c5
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/04/2019
ms.locfileid: "70299830"
---
# <a name="replace-dialogs"></a>取代對話方塊

[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]

當您需要在對話過程中驗證使用者輸入或者重複動作時，取代對話方塊的能力很有用。 透過適用於 Node.js 的 Bot Framework SDK，您可藉由使用 [`session.replaceDialog`](http://docs.botframework.com/node/builder/chat-reference/classes/_botbuilder_d_.session#replacedialog) 方法來取代對話。 這個方法可讓您結束目前的對話，不用傳回給來電者就可以新的對話加以取代。 

## <a name="create-custom-prompts-to-validate-input"></a>建立自訂提示來驗證輸入

適用於 Node.js 的 Bot Framework SDK 包括某些類型[提示](bot-builder-nodejs-dialog-prompt.md) (例如 `Prompts.time` 和 `Prompts.choice`) 的輸入驗證。 若要驗證您為了回應 `Prompts.text` 所收到的文字輸入，您必須建立自己的驗證邏輯和自訂提示。 

如果驗證必須符合您定義的特定值、模式、範圍或準則，建議您驗證輸入。 如果輸入驗證失敗，Bot 可以藉由使用 `session.replaceDialog` 方法，再次提示使用者該資訊。

下列程式碼範例示範如何建立自訂提示，來驗證使用者輸入的電話號碼。

```javascript
// This dialog prompts the user for a phone number. 
// It will re-prompt the user if the input does not match a pattern for phone number.
bot.dialog('phonePrompt', [
    function (session, args) {
        if (args && args.reprompt) {
            builder.Prompts.text(session, "Enter the number using a format of either: '(555) 123-4567' or '555-123-4567' or '5551234567'")
        } else {
            builder.Prompts.text(session, "What's your phone number?");
        }
    },
    function (session, results) {
        var matched = results.response.match(/\d+/g);
        var number = matched ? matched.join('') : '';
        if (number.length == 10 || number.length == 11) {
            session.userData.phoneNumber = number; // Save the number.
            session.endDialogWithResult({ response: number });
        } else {
            // Repeat the dialog
            session.replaceDialog('phonePrompt', { reprompt: true });
        }
    }
]);
```

在此範例中，系統一開始會提示使用者提供其電話號碼。 驗證邏輯會使用規則運算式，該規則運算式會比對使用者輸入的位數範圍。 如果輸入包含 10 或 11 位數，則該數字會在回應中傳回。 否則，系統會執行 `session.replaceDialog` 方法以重複 `phonePrompt` 對話方塊，提示使用者再次輸入；這次關於預期的輸入格式，會提供更精確的指引。

當您呼叫 `session.replaceDialog` 方法時，您會指定要重複的對話方塊名稱和引數清單。 在此範例中，引數清單包含 `{ reprompt: true }`，它會讓 Bot 根據是初始提示還是重新提示，提供不同的提示訊息，但是您可以指定 Bot 可能需要的任何引數。 

## <a name="repeat-an-action"></a>重複動作

有時候在對話當中您會想要重複對話方塊，讓使用者完成特定動作多次。 例如，如果您的 Bot 提供各種服務，您可能會在一開始顯示服務的選單，引導使用者完成要求服務的程序，然後再次顯示服務的選單，藉此讓使用者要求另一個服務。 若要達到此目的，您可以使用 [`session.replaceDialog`](https://docs.botframework.com/node/builder/chat-reference/classes/_botbuilder_d_.session#replacedialog) 方法，以再次顯示服務的選單，而不是使用 ['session.endConversation`](https://docs.botframework.com/node/builder/chat-reference/classes/_botbuilder_d_.session#endconversation) 方法來結束對話。 

下列範例示範如何使用 `session.replaceDialog` 方法來實作這種案例。 首先，定義服務的選單：

```javascript
// Main menu
var menuItems = { 
    "Order dinner": {
        item: "orderDinner"
    },
    "Dinner reservation": {
        item: "dinnerReservation"
    },
    "Schedule shuttle": {
        item: "scheduleShuttle"
    },
    "Request wake-up call": {
        item: "wakeupCall"
    },
}
```

`mainMenu` 對話方塊是由預設對話方塊叫用，因此會在對話開始時向使用者顯示選單。 此外，[`triggerAction`](https://docs.botframework.com/node/builder/chat-reference/classes/_botbuilder_d_.dialog.html#triggeraction) 會附加至 `mainMenu` 對話，每當使用者輸入「主選單」時呈現選單。 當使用者看到選單並選取一個選項時，系統會使用 `session.beginDialog` 方法叫用與使用者選項對應的對話方塊。

```javascript
var inMemoryStorage = new builder.MemoryBotStorage();

// This is a reservation bot that has a menu of offerings.
var bot = new builder.UniversalBot(connector, [
    function(session){
        session.send("Welcome to Contoso Hotel and Resort.");
        session.beginDialog("mainMenu");
    }
]).set('storage', inMemoryStorage); // Register in-memory storage 

// Display the main menu and start a new request depending on user input.
bot.dialog("mainMenu", [
    function(session){
        builder.Prompts.choice(session, "Main Menu:", menuItems);
    },
    function(session, results){
        if(results.response){
            session.beginDialog(menuItems[results.response.entity].item);
        }
    }
])
.triggerAction({
    // The user can request this at any time.
    // Once triggered, it clears the stack and prompts the main menu again.
    matches: /^main menu$/i,
    confirmPrompt: "This will cancel your request. Are you sure?"
});
```

在此範例中，如果使用者選擇選項 1 來訂購晚餐送到他們的房間，系統會叫用 `orderDinner` 對話方塊，並且會逐步引導使用者進行訂購晚餐的程序。 在此程序結束時，Bot 會確認訂單，並且使用 `session.replaceDialog` 方法再次顯示主選單。

```javascript
// Menu: "Order dinner"
// This dialog allows user to order dinner to be delivered to their hotel room.
bot.dialog('orderDinner', [
    function(session){
        session.send("Lets order some dinner!");
        builder.Prompts.choice(session, "Dinner menu:", dinnerMenu);
    },
    function (session, results) {
        if (results.response) {
            var order = dinnerMenu[results.response.entity];
            var msg = `You ordered: %(Description)s for a total of $${order.Price}.`;
            session.dialogData.order = order;
            session.send(msg);
            builder.Prompts.text(session, "What is your room number?");
        } 
    },
    function(session, results){
        if(results.response){
            session.dialogData.room = results.response;
            var msg = `Thank you. Your order will be delivered to room #${results.response}.`;
            session.send(msg);
            session.replaceDialog("mainMenu"); // Display the menu again.
        }
    }
])
.reloadAction(
    "restartOrderDinner", "Ok. Let's start over.",
    {
        matches: /^start over$/i
    }
)
.cancelAction(
    "cancelOrder", "Type 'Main Menu' to continue.", 
    {
        matches: /^cancel$/i,
        confirmPrompt: "This will cancel your order. Are you sure?"
    }
);
```

兩個觸發程序會附加至 `orderDinner` 對話方塊，讓使用者在訂購程序期間隨時「重頭開始」或「取消」訂購。 

第一個觸發程序是 [`reloadAction`](https://docs.botframework.com/node/builder/chat-reference/classes/_botbuilder_d_.dialog.html#reloadaction)，會讓使用者藉由傳送輸入「重頭開始」，再次重頭開始訂購程序。 當觸發程序符合「從頭開始」的聲音時，`reloadAction` 會從頭重新啟動對話。 

第二個觸發程序是 [`cancelAction`](https://docs.botframework.com/node/builder/chat-reference/classes/_botbuilder_d_.dialog.html#cancelaction)，會讓使用者藉由傳送輸入「取消」，完全中止訂購程序。 此觸發程序不會自動再次顯示主選單，而是改為傳送訊息，告訴使用者接下來透過說出「輸入 [主選單] 以繼續」的後續步驟。

### <a name="dialog-loops"></a>對話方塊迴圈

在上述範例中，使用者在每次訂購只能選取一個項目。 也就是說，如果使用者想要從選單訂購兩個項目，他們必須針對第一個項目完成整個訂購程序，然後針對第二個項目再次重複整個訂購程序。 

下列範例示範如何藉由將晚餐選單重構至個別對話方塊中，來改善先前的 Bot。 這麼做可讓 Bot 迴圈重複顯示晚餐選單，因此讓使用者可以在單一訂單內選擇多個項目。

首先，將 [結帳] 選項新增至選單中。 這個選項可以讓使用者結束項目選取程序，並繼續結帳程序。

```javascript
// The dinner menu
var dinnerMenu = { 
    //...other menu items...,
    "Check out": {
        Description: "Check out",
        Price: 0 // Order total. Updated as items are added to order.
    }
};
```

接下來，將訂購提示重構至它自己的對話方塊中，讓 Bot 能夠重複選單，並且讓使用者可將多個項目新增至訂單中。

```javascript
// Add dinner items to the list by repeating this dialog until the user says `check out`. 
bot.dialog("addDinnerItem", [
    function(session, args){
        if(args && args.reprompt){
            session.send("What else would you like to have for dinner tonight?");
        }
        else{
            // New order
            // Using the conversationData to store the orders
            session.conversationData.orders = new Array();
            session.conversationData.orders.push({ 
                Description: "Check out",
                Price: 0
            })
        }
        builder.Prompts.choice(session, "Dinner menu:", dinnerMenu);
    },
    function(session, results){
        if(results.response){
            if(results.response.entity.match(/^check out$/i)){
                session.endDialog("Checking out...");
            }
            else {
                var order = dinnerMenu[results.response.entity];
                session.conversationData.orders[0].Price += order.Price; // Add to total.
                var msg = `You ordered: ${order.Description} for a total of $${order.Price}.`;
                session.send(msg);
                session.conversationData.orders.push(order);
                session.replaceDialog("addDinnerItem", { reprompt: true }); // Repeat dinner menu
            }
        }
    }
])
.reloadAction(
    "restartOrderDinner", "Ok. Let's start over.",
    {
        matches: /^start over$/i
    }
);
```

在此範例中，訂單是儲存在範圍為目前對話 `session.conversationData.orders` 的 Bot 資料存放區中。 針對每筆新訂單，變數會以新陣列重新初始化，針對使用者選擇的每個項目，Bot 會將該項目新增至 `orders` 陣列中並且將價格新增至總計，總計會儲存在結帳的 `Price` 變數中。 當使用者完成選取訂單的項目時，他們可以說「結帳」，然後繼續訂購程序的其餘部分。

> [!NOTE]
> 如需 Bot 資料儲存體的詳細資訊，請參閱[管理狀態資料](bot-builder-nodejs-state.md)。 

最後，請更新 `orderDinner` 對話方塊內瀑布圖的第二個步驟，以處理訂單與確認。 

```javascript
// Menu: "Order dinner"
// This dialog allows user to order dinner and have it delivered to their room.
bot.dialog('orderDinner', [
    function(session){
        session.send("Lets order some dinner!");
        session.beginDialog("addDinnerItem");
    },
    function (session, results) {
        if (results.response) {
            // Display itemize order with price total.
            for(var i = 1; i < session.conversationData.orders.length; i++){
                session.send(`You ordered: ${session.conversationData.orders[i].Description} for a total of $${session.conversationData.orders[i].Price}.`);
            }
            session.send(`Your total is: $${session.conversationData.orders[0].Price}`);

            // Continue with the check out process.
            builder.Prompts.text(session, "What is your room number?");
        } 
    },
    function(session, results){
        if(results.response){
            session.dialogData.room = results.response;
            var msg = `Thank you. Your order will be delivered to room #${results.response}`;
            session.send(msg);
            session.replaceDialog("mainMenu");
        }
    }
])
//...attached triggers...
;
```

## <a name="cancel-a-dialog"></a>取消對話方塊

雖然 `session.replaceDialog` 方法可以用來將「目前」  對話方塊取代為新對話方塊，但是它無法用來取代位於對話方塊堆疊底下遠處的對話方塊。 若要取代對話方塊堆疊內，除了「目前」  對話方塊以外的對話方塊，請改為使用 [`session.cancelDialog`](http://docs.botframework.com/node/builder/chat-reference/classes/_botbuilder_d_.session#canceldialog) 方法。 

`session.cancelDialog` 方法可用來結束對話方塊，不論對話方塊在對話方塊堆疊中的哪個位置，並選擇性地在其位置叫用一個新的對話方塊。 若要呼叫 `session.cancelDialog` 方法，請指定要取消的對話方塊識別碼，並選擇性地指定要在其位置叫用的對話方塊識別碼。 例如，此程式碼片段會取消 `orderDinner` 對話方塊，並且以 `mainMenu` 對話方塊來加以取代：

```javascript
session.cancelDialog('orderDinner', 'mainMenu'); 
```

當呼叫 `session.cancelDialog` 方法時，對話方塊堆疊會向後搜尋，並且會取消第一個出現的對話方塊，導致該對話方塊及其子系對話方塊 (如果有的話) 從對話方塊堆疊中移除。 接著控制權會交回給呼叫對話方塊，讓該對話方塊可以檢查 [`results.resumed`](http://docs.botframework.com/node/builder/chat-reference/interfaces/_botbuilder_d_.ipromptresult.html#resumed) 程式碼是否等於 [`ResumeReason.notCompleted`](http://docs.botframework.com/node/builder/chat-reference/enums/_botbuilder_d_.resumereason.html#notcompleted)，以偵測取消。

除了在呼叫 `session.cancelDialog` 方法時指定要取消的對話方塊識別碼以外，您也可以改為針對要取消的對話方塊，指定以零為基礎的索引，其中索引代表對話方塊在對話方塊堆疊中的位置。 例如，下列程式碼片段會終止目前使用中的對話方塊 (索引 = 0)，並啟動 `mainMenu` 對話方塊來加以取代。 系統會在對話方塊堆疊的 0 位置叫用 `mainMenu` 對話方塊，因此該對話方塊會成為新的預設對話方塊。

```javascript
session.cancelDialog(0, 'mainMenu');
```

請考慮在上方[對話方塊迴圈](#dialog-loops)中所討論的範例。 當使用者到達項目選取選單時，該對話方塊 (`addDinnerItem`) 是對話方塊堆疊中的第四個對話方塊：`[default dialog, mainMenu, orderDinner, addDinnerItem]`。 您要如何讓使用者從 `addDinnerItem` 對話方塊中取消其訂單？ 如果您將 `cancelAction` 觸發程序附加至 `addDinnerItem` 對話方塊，只會讓使用者返回先前的對話方塊 (`orderDinner`)，這樣會將使用者送回 `addDinnerItem` 對話方塊。

這就是 `session.cancelDialog` 方法很有用的地方。 以[對話方塊迴圈範例](#dialog-loops)開始，新增「取消訂單」作為晚餐選單內的明確選項。

```javascript
// The dinner menu
var dinnerMenu = { 
    //...other menu items...,
    "Check out": {
        Description: "Check out",
        Price: 0      // Order total. Updated as items are added to order.
    },
    "Cancel order": { // Cancel the order and back to Main Menu
        Description: "Cancel order",
        Price: 0
    }
};
```

接著，更新 `addDinnerItem` 對話來檢查「取消訂單」要求。 如果偵測到「取消」，則使用 `session.cancelDialog` 方法來取消預設對話方塊 (亦即位於堆疊索引 0 的對話方塊)，並叫用 `mainMenu` 對話方塊來加以取代。 

```javascript
// Add dinner items to the list by repeating this dialog until the user says `check out`. 
bot.dialog("addDinnerItem", [
    //...waterfall steps...,
    // Last step
    function(session, results){
        if(results.response){
            if(results.response.entity.match(/^check out$/i)){
                session.endDialog("Checking out...");
            }
            else if(results.response.entity.match(/^cancel/i)){
                // Cancel the order and start "mainMenu" dialog.
                session.cancelDialog(0, "mainMenu");
            }
            else {
                //...add item to list and prompt again...
                session.replaceDialog("addDinnerItem", { reprompt: true }); // Repeat dinner menu.
            }
        }
    }
])
//...attached triggers...
;
```

藉由如此使用 `session.cancelDialog` 方法，您就可以實作 Bot 所需的任何對話流程。

## <a name="next-steps"></a>後續步驟

如您所見，若要取代堆疊中的對話方塊，可使用各種**動作**來完成這項工作。 動作讓您在管理對話流程方面有絕佳彈性。 讓我們進一步看看**動作**，以及如何更妥善地處理使用者動作。

> [!div class="nextstepaction"]
> [處理使用者動作](bot-builder-nodejs-dialog-actions.md)
