---
title: 處理使用者中斷 | Microsoft Docs
description: 深入了解如何處理使用者中斷和直接對話流程。
keywords: 中斷, 切換主題, 中斷
author: v-ducvo
ms.author: v-ducvo
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 04/17/2018
ms.reviewer: ''
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: fff4f8e2a4d2d86cf440bee7ab40216e93a8c8c5
ms.sourcegitcommit: 1abc32353c20acd103e0383121db21b705e5eec3
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/21/2018
ms.locfileid: "42756569"
---
# <a name="handle-user-interruptions"></a>處理使用者中斷

[!INCLUDE [pre-release-label](../includes/pre-release-label.md)]

處理使用者中斷是強固 Bot 很重要的層面。 您也許會認為使用者會乖乖逐步遵循您定義的對話流程，但是他們很有可能會改變心意，或者在程序中途詢問問題而不是回答問題。 在這些情況下，您的 Bot 要如何處理使用者的輸入？ 使用者體驗會是如何？ 如何維護使用者狀態資料？

這些問題沒有正確答案，因為每種情況對於您的 Bot 設計來處理的案例都是獨一無二的。 在本主題中，我們將會探索一些處理使用者插斷的常見方式，並且建議在您的 Bot 中加以實作的一些方法。

## <a name="handle-expected-interruptions"></a>處理預期的插斷

程序性對話流程具有一組核心步驟，您會想要引導使用者進行這些步驟，與這些步驟不同的任何使用者動作都是潛在的中斷。 在一般流程中，會有您可以預期的中斷。

**餐廳訂位** 在餐廳訂位 Bot 中，核心步驟可能是向使用者詢問日期和時間、派對規模及訂位名稱。 在該程序中，您可以預期到的一些中斷可能包括：

* `cancel`：可結束程序。
* `help`：可提供此程序相關的其他指引。
* `more info`：可提供提示與建議，或提供餐廳訂位的替代方式 (例如：連絡用電子郵件地址或電話號碼)。
* `show list of available tables`：如果這是選項；會顯示在使用者希望的日期和時間可供選擇的桌位清單。

**訂購晚餐** 在訂購晚餐 Bot 中，核心步驟是提供選單項目清單，並且讓使用者可將項目新增至購物車。 在此程序中，您可以預期到的一些中斷可能包括：

* `cancel`：可結束訂購程序。
* `more info`：可提供每個選單項目的飲食詳細資料。
* `help`：可提供如何使用系統的說明。
* `process order`：可處理訂單。

您可用**建議的動作**清單或提示的形式，將這些項目提供給使用者，讓使用者至少知道可以傳送哪些 Bot 能夠了解的命令。

例如，在訂購晚餐流程中，您可以提供預期的中斷以及選單項目。 在此情況下，選單項目是以 `choices` 陣列的形式傳送。

# <a name="ctabcsharptab"></a>[C#](#tab/csharptab)

```cs
public class dinnerItem
{
    public string Description;
    public double Price;
}

public class dinnerMenu
{
    static public Dictionary<string, dinnerItem> dinnerChoices = new Dictionary<string, dinnerItem>
    {
        { "potato salad", new dinnerItem { Description="Potato Salad", Price=5.99 } },
        { "tuna sandwich", new dinnerItem { Description="Tuna Sandwich", Price=6.89 } },
        { "clam chowder", new dinnerItem { Description="Clam Chowder", Price=4.50 } }
    };

    static public string[] choices = new string[] {"Potato Salad", "Tuna Sandwich", "Clam Chowder", "more info", "Process order", "help", "Cancel"};
}
```

# <a name="javascripttabjstab"></a>[JavaScript](#tab/jstab)

```javascript
var dinnerMenu = {
    choices: ["Potato Salad - $5.99", "Tuna Sandwich - $6.89", "Clam Chowder - $4.50",
            "more info", "Process order", "Cancel"],
    "Potato Salad - $5.99": {
        Description: "Potato Salad",
        Price: 5.99
    },
    "Tuna Sandwich - $6.89": {
        Description: "Tuna Sandwich",
        Price: 6.89
    },
    "Clam Chowder - $4.50": {
        Description: "Clam Chowder",
        Price: 4.50
    }
}
```

---

在您的訂購邏輯中，您可以使用字串比對或規則運算式來檢查它們。

# <a name="ctabcsharptab"></a>[C#](#tab/csharptab)

首先，我們需要定義協助程式來追蹤我們的訂單

```cs
// Helper class for storing the order in the dictionary
public class Orders
{
    public double total;
    public string order;
    public bool processOrder;

    // Initialize order values
    public Orders()
    {
        total = 0;
        order = "";
        processOrder = false;
    }
}
```

然後，將對話新方塊增至您的 Bot。

```cs
dialogs.Add("orderPrompt", new WaterfallStep[]
{
    async (dc, args, next) =>
    {
        // Prompt the user
        await dc.Prompt("choicePrompt",
            "What would you like for dinner?",
            new ChoicePromptOptions
            {
                Choices = dinnerMenu.choices.Select( s => new Choice { Value = s }).ToList(),
                RetryPromptString = "I'm sorry, I didn't understand that. What would you " +
                    "like for dinner?"
            });
    },
    async(dc, args, next) =>
    {
        var convo = ConversationState<Dictionary<string,object>>.Get(dc.Context);

        // Get the user's choice from the previous prompt
        var response = (args["Value"] as FoundChoice).Value.ToLower();

        if(response == "process order")
        {
            try
            {
                var order = convo["order"];

                await dc.Context.SendActivity("Order is on it's way!");

                // In production, you may want to store something more helpful,
                // such as send order off to be made
                (order as Orders).processOrder = true;

                // Once it's submitted, clear the current order
                convo.Remove("order");
                await dc.End();
            }
            catch
            {
                await dc.Context.SendActivity("Your order is empty, please add your order choice");
                // Ask again
                await dc.Replace("orderPrompt");
            }
        }
        else if(response == "cancel" )
        {
            // Get rid of current order
            convo.Remove("order");
            await dc.Context.SendActivity("Your order has been canceled");
            await dc.End();
        }
        else if(response == "more info")
        {
            // Send more information about the options
            var msg = "More info: <br/>" +
                "Potato Salad: contains 330 calaries per serving. Cost: 5.99 <br/>"
                + "Tuna Sandwich: contains 700 calaries per serving. Cost: 6.89 <br/>"
                + "Clam Chowder: contains 650 calaries per serving. Cost: 4.50";
            await dc.Context.SendActivity(msg);

            // Ask again
            await dc.Replace("orderPrompt");
        }
        else if(response == "help")
        {
            // Provide help information
            await dc.Context.SendActivity("To make an order, add as many items to your cart as " +
                "you like then choose the \"Process order\" option to check out.");

            // Ask again
            await dc.Replace("orderPrompt");
        }
        else
        {
            // Unlikely to get past the prompt verification, but this will catch
            // anything that isn't a valid menu choice
            if(!dinnerMenu.dinnerChoices.ContainsKey(response))
            {
                await dc.Context.SendActivity("Sorry, that is not a valid item. " +
                    "Please pick one from the menu.");

                // Ask again
                await dc.Replace("orderPrompt");
            }
            else {
                // Add the item to cart
                Orders currentOrder;

                // If there is a current order going, add to it. If not, start a new one
                try
                {
                    currentOrder = convo["order"] as Orders;
                }
                catch
                {
                    convo["order"] = new Orders();
                    currentOrder = convo["order"] as Orders;
                }

                // Add to the current order
                currentOrder.order += (dinnerMenu.dinnerChoices[$"{response}"].Description) + ", ";
                currentOrder.total += (double)dinnerMenu.dinnerChoices[$"{response}"].Price;

                // Save back to the conversation state
                convo["order"] = currentOrder;

                await dc.Context.SendActivity($"Added to cart. Current order: " +
                    $"{currentOrder.order} " +
                    $"<br/>Current total: ${currentOrder.total}");

                // Ask again to allow user to add more items or process order
                await dc.Replace("orderPrompt");
            }
        }
    }
});
```

# <a name="javascripttabjstab"></a>[JavaScript](#tab/jstab)

```javascript
// Helper dialog to repeatedly prompt user for orders
dialogs.add('orderPrompt', [
    async function(dc){
        await dc.prompt('choicePrompt', "What would you like?", dinnerMenu.choices);
    },
    async function(dc, choice){
        if(choice.value.match(/process order/ig)){
            if(orderCart.orders.length > 0) {
                // Process the order
                dc.context.activity.conversation.dinnerOrder = orderCart;

                await dc.end();
            }
            else {
                await dc.context.sendActivity("Your cart was empty. Please add at least one item to the cart.");
                // Ask again
                await dc.replace('orderPrompt');
            }
        }
        else if(choice.value.match(/cancel/ig)){
            orderCart.clear(context);
            await dc.context.sendActivity("Your order has been canceled.");
            await dc.end(choice.value);
        }
        else if(choice.value.match(/more info/ig)){
            var msg = "More info: <br/>Potato Salad: contains 330 calaries per serving. <br/>"
                + "Tuna Sandwich: contains 700 calaries per serving. <br/>" 
                + "Clam Chowder: contains 650 calaries per serving."
            await dc.context.sendActivity(msg);

            // Ask again
            await dc.replace('orderPrompt');
        }
        else if(choice.value.match(/help/ig)){
            var msg = `Help: <br/>To make an order, add as many items to your cart as you like then choose the "Process order" option to check out.`
            await dc.context.sendActivity(msg);

            // Ask again
            await dc.replace('orderPrompt');
        }
        else {
            var choice = dinnerMenu[choice.value];

            // Only proceed if user chooses an item from the menu
            if(!choice){
                await dc.context.sendActivity("Sorry, that is not a valid item. Please pick one from the menu.");

                // Ask again
                await dc.replace('orderPrompt');
            }
            else {
                // Add the item to cart
                orderCart.orders.push(choice);
                orderCart.total += dinnerMenu[choice.value].Price;

                await dc.context.sendActivity(`Added to cart: ${choice.value}. <br/>Current total: $${orderCart.total}`);

                // Ask again
                await dc.replace('orderPrompt');
            }
        }
    }
]);
```

---

## <a name="handle-unexpected-interruptions"></a>處理未預期的中斷

有些中斷不在您 Bot 設計要執行的功能範圍內。
雖然您無法預期所有中斷，但可以針對 Bot 進行程式設計，來處理一些中斷的模式。

### <a name="switching-topic-of-conversations"></a>切換對話主題

如果使用者正在對話中，而且想要切換至另一個對話，怎麼辦？ 例如，您的 Bot 可以處理餐廳訂位和訂購晚餐。
當使用者處於_餐廳訂位_流程中時，使用者並非回答「派對中有多少人？」的問題，而是傳送「訂購晚餐」訊息。 在此情況下，使用者改變了心意並且想要參與訂購晚餐對話。 您應該如何處理此中斷？

您可以將主題切換為訂購晚餐流程，或者可以藉由告知使用者您正在等待數量並且重新提示他們，讓問題固定下來。 如果您允許他們切換主題，接著就必須決定是否要儲存進度，讓使用者可以從他們離開的地方再開始；或者可以刪除已收集的所有資訊，讓他們在下一次訂位的時候從頭開始。 如需管理使用者狀態資料的詳細資訊，請參閱[使用對話和使用者屬性來儲存狀態](bot-builder-howto-v4-state.md)。

### <a name="apply-artificial-intelligence"></a>套用人工智慧

針對範圍之外的中斷，您可以嘗試猜測使用者的想法。 您可以使用 AI 服務 (例如 QnAMaker、LUIS) 或您的自訂邏輯，然後針對 Bot 所判斷的使用者需求的提供建議，來達到這項目的。

例如，在餐廳訂位流程當中，使用者說：「我想要訂購漢堡」。 Bot 不會知道要如何在這個對話流程中處理這個訊息。 因為目前的流程與訂購毫無相關，Bot 的其他對話命令是「訂購晚餐」，所以 Bot 不知道要怎麼處理這個輸入。 舉例來說，如果您套用 LUIS，您可以訓練模型來辨識使用者想要訂購餐點 (例如：LUIS 可以傳回 "orderFood" 意圖)。 因此，Bot 就可以回應：「您似乎想要訂購餐點。 您要切換到我們的訂購晚餐程序嗎？」 如需訓練 LUIS 並且偵測使用者意圖的詳細資訊，請參閱[使用 LUIS 理解語言](bot-builder-howto-v4-luis.md)。

### <a name="default-response"></a>預設回應

如果所有解決方案都失敗，您可以傳送通用預設回應，而不是毫無作為，讓使用者臆測究竟發生什麼事。 預設回應應該告知使用者 Bot 可以理解哪些命令，讓使用者可以回到正軌。

您可以檢查 Bot 邏輯結尾的內容**已回應**旗標，以查看 Bot 是否在使用者等待時將任何項目傳回給使用者。 如果 Bot 會處理使用者的輸入，但是不會回應，有可能是 Bot 不知道該如何處理輸入。 在此情況下，您可以攔截它，並且將預設訊息傳送給使用者。

此 Bot 的預設值是對使用者顯示 `mainMenu` 對話方塊。 遇到 Bot 發生這種情形時，使用者會有何種體驗全操之在您。

# <a name="ctabcsharptab"></a>[C#](#tab/csharptab)

```cs
if(!context.Responded)
{
    await dc.EndAll().Begin("mainMenu");
}
```

# <a name="javascripttabjstab"></a>[JavaScript](#tab/jstab)

```javascript
// Check to see if anyone replied. If not then clear all the stack and present the main menu
if (!context.responded) {
    await dc.endAll().begin('mainMenu');
}
```

---
