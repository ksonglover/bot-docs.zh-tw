---
title: 處理使用者中斷 | Microsoft Docs
description: 深入了解如何處理使用者中斷和直接對話流程。
keywords: 中斷, 切換主題, 中斷
author: ivorb
ms.author: v-ivorb
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 04/18/2019
ms.reviewer: ''
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 9089334823c1c57c8ace48531c767c3f966b3355
ms.sourcegitcommit: aea57820b8a137047d59491b45320cf268043861
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 04/22/2019
ms.locfileid: "59905011"
---
# <a name="handle-user-interruptions"></a>處理使用者中斷

[!INCLUDE[applies-to](../includes/applies-to.md)]

處理中斷是強固 Bot 很重要的層面。

您也許會認為使用者會乖乖逐步遵循您定義的對話流程，但是他們很有可能會改變心意，或者在程序中途詢問問題而不是回答問題。 在這些情況下，您的 Bot 要如何處理使用者的輸入？ 使用者體驗會是如何？ 如何維護使用者狀態資料？ 處理中斷表示確定 Bot 已準備好處理這種情況。

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

我們會將對話集定義為 **DialogSet** 的子類別。

```csharp
using System;
using System.Collections.Generic;
using System.Linq;
using System.Threading;
using System.Threading.Tasks;
using Microsoft.Bot.Builder;
using Microsoft.Bot.Builder.Dialogs;
using Microsoft.Bot.Builder.Dialogs.Choices;

public class OrderDinnerDialogs : DialogSet
{
    public OrderDinnerDialogs(IStatePropertyAccessor<DialogState> dialogStateAccessor)
        : base(dialogStateAccessor)
    {
    }
}
```

我們會定義幾個內部類別來描述功能表。

```cs
/// <summary>
/// Contains information about an item on the menu.
/// </summary>
public class DinnerItem
{
    public string Description { get; set; }

    public double Price { get; set; }
}

/// <summary>
/// Describes the dinner menu, including the items on the menu and options for
/// interrupting the ordering process.
/// </summary>
public class DinnerMenu
{
    /// <summary>Gets the items on the menu.</summary>
    public static Dictionary<string, DinnerItem> MenuItems { get; } = new Dictionary<string, DinnerItem>
    {
        ["Potato salad"] = new DinnerItem { Description = "Potato Salad", Price = 5.99 },
        ["Tuna sandwich"] = new DinnerItem { Description = "Tuna Sandwich", Price = 6.89 },
        ["Clam chowder"] = new DinnerItem { Description = "Clam Chowder", Price = 4.50 },
    };

    /// <summary>Gets all the "interruptions" the bot knows how to process.</summary>
    public static List<string> Interrupts { get; } = new List<string>
    {
        "More info", "Process order", "Help", "Cancel",
    };

    /// <summary>Gets all of the valid inputs a user can make.</summary>
    public static List<string> Choices { get; }
        = MenuItems.Select(c => c.Key).Concat(Interrupts).ToList();
}
```

# <a name="javascripttabjstab"></a>[JavaScript](#tab/jstab)

我們先從基本的 EchoBot 範本開始。 如需指示，請參閱[適用於 JavaScript 的快速入門](~/javascript/bot-builder-javascript-quickstart.md)。

可以從 NPM 下載 `botbuilder-dialogs` 程式庫。 若要安裝 `botbuilder-dialogs` 程式庫，請執行下列 npm 命令：

```cmd
npm install --save botbuilder-dialogs
```

在 **bot.js** 檔案中，參考我們將會參考的類別，並定義我們將用於對話方塊的識別碼。

```javascript
const { ActivityTypes } = require('botbuilder');
const { DialogSet, ChoicePrompt, WaterfallDialog, DialogTurnStatus } = require('botbuilder-dialogs');

// Name for the dialog state property accessor.
const DIALOG_STATE_PROPERTY = 'dialogStateProperty';

// Name of the order-prompt dialog.
const ORDER_PROMPT = 'orderingDialog';

// Name for the choice prompt for use in the dialog.
const CHOICE_PROMPT = 'choicePrompt';
```

定義我們想要顯示在排序對話方塊上的選項。

```javascript
// The options on the dinner menu, including commands for the bot.
const dinnerMenu = {
    choices: ["Potato Salad - $5.99", "Tuna Sandwich - $6.89", "Clam Chowder - $4.50",
        "Process order", "Cancel", "More info", "Help"],
    "Potato Salad - $5.99": {
        description: "Potato Salad",
        price: 5.99
    },
    "Tuna Sandwich - $6.89": {
        description: "Tuna Sandwich",
        price: 6.89
    },
    "Clam Chowder - $4.50": {
        description: "Clam Chowder",
        price: 4.50
    }
}
```

---

在您的訂購邏輯中，您可以使用字串比對或規則運算式來檢查它們。

# <a name="ctabcsharptab"></a>[C#](#tab/csharptab)

首先，我們需要定義協助程式來追蹤我們的訂單。

```cs
/// <summary>Helper class for storing the order.</summary>
public class Order
{
    public double Total { get; set; } = 0.0;

    public List<DinnerItem> Items { get; set; } = new List<DinnerItem>();

    public bool ReadyToProcess { get; set; } = false;

    public bool OrderProcessed { get; set; } = false;
}
```

新增一些常數，以追蹤我們需要的識別碼。

```csharp
/// <summary>The ID of the top-level dialog.</summary>
public const string MainDialogId = "mainMenu";

/// <summary>The ID of the choice prompt.</summary>
public const string ChoicePromptId = "choicePrompt";

/// <summary>The ID of the order card value, tracked inside the dialog.</summary>
public const string OrderCartId = "orderCart";
```

更新建構函式，以新增選擇提示和瀑布對話。 我們也會定義可實作瀑布步驟的方法。

```cs
public OrderDinnerDialogs(IStatePropertyAccessor<DialogState> dialogStateAccessor)
    : base(dialogStateAccessor)
{
    // Add a choice prompt for the dialog.
    Add(new ChoicePrompt(ChoicePromptId));

    // Define and add the main waterfall dialog.
    WaterfallStep[] steps = new WaterfallStep[]
    {
        PromptUserAsync,
        ProcessInputAsync,
    };

    Add(new WaterfallDialog(MainDialogId, steps));
}

/// <summary>
/// Defines the first step of the main dialog, which is to ask for input from the user.
/// </summary>
/// <param name="stepContext">The current waterfall step context.</param>
/// <param name="cancellationToken">The cancellation token.</param>
/// <returns>The task to perform.</returns>
private async Task<DialogTurnResult> PromptUserAsync(WaterfallStepContext stepContext, CancellationToken cancellationToken)
{
    // Initialize order, continuing any order that was passed in.
    Order order = (stepContext.Options is Order oldCart && oldCart != null)
        ? new Order
        {
            Items = new List<DinnerItem>(oldCart.Items),
            Total = oldCart.Total,
            ReadyToProcess = oldCart.ReadyToProcess,
            OrderProcessed = oldCart.OrderProcessed,
        }
        : new Order();

    // Set the order cart in dialog state.
    stepContext.Values[OrderCartId] = order;

    // Prompt the user.
    return await stepContext.PromptAsync(
        "choicePrompt",
        new PromptOptions
        {
            Prompt = MessageFactory.Text("What would you like for dinner?"),
            RetryPrompt = MessageFactory.Text(
                "I'm sorry, I didn't understand that. What would you like for dinner?"),
            Choices = ChoiceFactory.ToChoices(DinnerMenu.Choices),
        },
        cancellationToken);
}

/// <summary>
/// Defines the second step of the main dialog, which is to process the user's input, and
/// repeat or exit as appropriate.
/// </summary>
/// <param name="stepContext">The current waterfall step context.</param>
/// <param name="cancellationToken">The cancellation token.</param>
/// <returns>The task to perform.</returns>
private async Task<DialogTurnResult> ProcessInputAsync(WaterfallStepContext stepContext, CancellationToken cancellationToken)
{
    // Get the order cart from dialog state.
    Order order = stepContext.Values[OrderCartId] as Order;

    // Get the user's choice from the previous prompt.
    string response = (stepContext.Result as FoundChoice).Value;

    if (response.Equals("process order", StringComparison.InvariantCultureIgnoreCase))
    {
        order.ReadyToProcess = true;

        await stepContext.Context.SendActivityAsync(
            "Your order is on it's way!",
            cancellationToken: cancellationToken);

        // In production, you may want to store something more helpful.
        // "Process" the order and exit.
        order.OrderProcessed = true;
        return await stepContext.EndDialogAsync(null, cancellationToken);
    }
    else if (response.Equals("cancel", StringComparison.InvariantCultureIgnoreCase))
    {
        // Cancel the order.
        await stepContext.Context.SendActivityAsync(
            "Your order has been canceled",
            cancellationToken: cancellationToken);

        // Exit without processing the order.
        return await stepContext.EndDialogAsync(null, cancellationToken);
    }
    else if (response.Equals("more info", StringComparison.InvariantCultureIgnoreCase))
    {
        // Send more information about the options.
        string message = "More info: <br/>" +
            "Potato Salad: contains 330 calories per serving. Cost: 5.99 <br/>"
            + "Tuna Sandwich: contains 700 calories per serving. Cost: 6.89 <br/>"
            + "Clam Chowder: contains 650 calories per serving. Cost: 4.50";
        await stepContext.Context.SendActivityAsync(
            message,
            cancellationToken: cancellationToken);

        // Continue the ordering process, passing in the current order cart.
        return await stepContext.ReplaceDialogAsync(MainDialogId, order, cancellationToken);
    }
    else if (response.Equals("help", StringComparison.InvariantCultureIgnoreCase))
    {
        // Provide help information.
        string message = "To make an order, add as many items to your cart as " +
            "you like. Choose the `Process order` to check out. " +
            "Choose `Cancel` to cancel your order and exit.";
        await stepContext.Context.SendActivityAsync(
            message,
            cancellationToken: cancellationToken);

        // Continue the ordering process, passing in the current order cart.
        return await stepContext.ReplaceDialogAsync(MainDialogId, order, cancellationToken);
    }

    // We've checked for expected interruptions. Check for a valid item choice.
    if (!DinnerMenu.MenuItems.ContainsKey(response))
    {
        await stepContext.Context.SendActivityAsync("Sorry, that is not a valid item. " +
            "Please pick one from the menu.");

        // Continue the ordering process, passing in the current order cart.
        return await stepContext.ReplaceDialogAsync(MainDialogId, order, cancellationToken);
    }
    else
    {
        // Add the item to cart.
        DinnerItem item = DinnerMenu.MenuItems[response];
        order.Items.Add(item);
        order.Total += item.Price;

        // Acknowledge the input.
        await stepContext.Context.SendActivityAsync(
            $"Added `{response}` to your order; your total is ${order.Total:0.00}.",
            cancellationToken: cancellationToken);

        // Continue the ordering process, passing in the current order cart.
        return await stepContext.ReplaceDialogAsync(MainDialogId, order, cancellationToken);
    }
}
```

# <a name="javascripttabjstab"></a>[JavaScript](#tab/jstab)

在 Bot 建構函式中定義您的對話方塊。 請注意，程式碼會先檢查並處理中斷，然後繼續進行下一個邏輯步驟。

```javascript
constructor(conversationState) {
    this.dialogStateAccessor = conversationState.createProperty(DIALOG_STATE_PROPERTY);
    this.conversationState = conversationState;

    this.dialogs = new DialogSet(this.dialogStateAccessor)
        .add(new ChoicePrompt(CHOICE_PROMPT))
        .add(new WaterfallDialog(ORDER_PROMPT, [
            async (step) => {
                if (step.options && step.options.orders) {
                    // If an order cart was passed in, continue to use it.
                    step.values.orderCart = step.options;
                } else {
                    // Otherwise, start a new cart.
                    step.values.orderCart = {
                        orders: [],
                        total: 0
                    };
                }
                return await step.prompt(CHOICE_PROMPT, "What would you like?", dinnerMenu.choices);
            },
            async (step) => {
                const choice = step.result;
                if (choice.value.match(/process order/ig)) {
                    if (step.values.orderCart.orders.length > 0) {
                        // If the cart is not empty, process the order by returning the order to the parent context.
                        await step.context.sendActivity("Your order has been processed.");
                        return await step.endDialog(step.values.orderCart);
                    } else {
                        // Otherwise, prompt again.
                        await step.context.sendActivity("Your cart was empty. Please add at least one item to the cart.");
                        return await step.replaceDialog(ORDER_PROMPT);
                    }
                } else if (choice.value.match(/cancel/ig)) {
                    // Cancel the order, and return "cancel" to the parent context.
                    await step.context.sendActivity("Your order has been canceled.");
                    return await step.endDialog("cancelled");
                } else if (choice.value.match(/more info/ig)) {
                    // Provide more information, and then continue the ordering process.
                    var msg = "More info: <br/>Potato Salad: contains 330 calories per serving. <br/>"
                        + "Tuna Sandwich: contains 700 calories per serving. <br/>"
                        + "Clam Chowder: contains 650 calories per serving."
                    await step.context.sendActivity(msg);
                    return await step.replaceDialog(ORDER_PROMPT, step.values.orderCart);
                } else if (choice.value.match(/help/ig)) {
                    // Provide help information, and then continue the ordering process.
                    var msg = `Help: <br/>`
                        + `To make an order, add as many items to your cart as you like then choose `
                        + `the "Process order" option to check out.`
                    await step.context.sendActivity(msg);
                    return await step.replaceDialog(ORDER_PROMPT, step.values.orderCart);
                } else {
                    // The user has chosen a food item from the menu. Add the item to cart.
                    var item = dinnerMenu[choice.value];
                    step.values.orderCart.orders.push(item.description);
                    step.values.orderCart.total += item.price;

                    await step.context.sendActivity(`Added ${item.description} to your cart. <br/>`
                        + `Current total: $${step.values.orderCart.total}`);

                    // Continue the ordering process.
                    return await step.replaceDialog(ORDER_PROMPT, step.values.orderCart);
                }
            }
        ]));
}
```

更新回合處理常式，以呼叫對話方塊並顯示排序程序的結果。

```javascript
async onTurn(turnContext) {
    if (turnContext.activity.type === ActivityTypes.Message) {
        let dc = await this.dialogs.createContext(turnContext);
        let dialogTurnResult = await dc.continueDialog();
        if (dialogTurnResult.status === DialogTurnStatus.complete) {
            // The dialog completed this turn.
            const result = dialogTurnResult.result;
            if (!result || result === "cancelled") {
                await turnContext.sendActivity('You cancelled your order.');
            } else {
                await turnContext.sendActivity(`Your order came to $${result.total}`);
            }
        } else if (!turnContext.responded) {
            // No dialog was active.
            await turnContext.sendActivity("Let's order dinner...");
            await dc.cancelAllDialogs();
            await dc.beginDialog(ORDER_PROMPT);
        } else {
            // The dialog is active.
        }
    } else {
        await turnContext.sendActivity(`[${turnContext.activity.type} event detected]`);
    }
    // Save state changes
    await this.conversationState.saveChanges(turnContext);
}
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

針對範圍之外的中斷，您可以嘗試猜測使用者的想法。 您可以使用 AI 服務 (例如 QnA Maker、LUIS) 或您的自訂邏輯，然後針對 Bot 所判斷的使用者需求的提供建議，來達到這項目的。

例如，在餐廳訂位流程當中，使用者說：「我想要訂購漢堡」。 Bot 不會知道要如何在這個對話流程中處理這個訊息。 因為目前的流程與訂購毫無相關，Bot 的其他對話命令是「訂購晚餐」，所以 Bot 不知道要怎麼處理這個輸入。 舉例來說，如果您套用 LUIS，就可以訓練模型來辨識使用者想要訂購餐點 (例如：LUIS 可以傳回 "orderFood" 意圖)。 因此，Bot 就可以回應：「您似乎想要訂購餐點。 您要切換到我們的訂購晚餐程序嗎？」 如需訓練 LUIS 並且偵測使用者意圖的詳細資訊，請參閱[使用 LUIS 理解語言](bot-builder-howto-v4-luis.md)。

### <a name="default-response"></a>預設回應

如果所有解決方案都失敗，您應該傳送預設回應，而不是毫無作為，讓使用者臆測究竟發生什麼事。 預設回應應該告知使用者 Bot 可以理解哪些命令，讓使用者可以回到正軌。

您可以檢查 Bot 邏輯結尾的內容**已回應**旗標，以查看 Bot 是否在使用者等待時將任何項目傳回給使用者。 如果 Bot 會處理使用者的輸入，但是不會回應，有可能是 Bot 不知道該如何處理輸入。 在此情況下，您可以攔截它，並且將預設訊息傳送給使用者。

如果 Bot 的預設值是為使用者提供 `mainMenu` 對話，您則必須決定使用者在這種情形下會有何種 Bot 體驗。

# <a name="ctabcsharptab"></a>[C#](#tab/csharptab)

```cs
// Check whether we replied. If not then clear the dialog stack and present the main menu.
if (!turnContext.Responded)
{
    await dc.CancelAllDialogsAsync(cancellationToken);
    await dc.BeginDialogAsync(OrderDinnerDialogs.MainDialogId, null, cancellationToken);
}
```

# <a name="javascripttabjstab"></a>[JavaScript](#tab/jstab)

```javascript
// Check to see if anyone replied. If not then clear all the stack and present the main menu
if (!context.responded) {
    await dc.cancelAllDialogs();
    await step.beginDialog('mainMenu');
}
```

---

## <a name="handling-global-interruptions"></a>處理全域中斷

在上述範例中，我們會處理可能在特定對話方塊中的特定回合發生的中斷。 如果我們想要處理全域中斷 - 隨時可能發生的種類，該怎麼做？

做法是將中斷處理邏輯放入 Bot 的主要處理常式中 - 這是可處理傳入 `turnContext` 並決定其處理方式的函式。

在以下範例中，Bot 進行的「第一件事」是檢查傳入訊息文字中是否有使用者需要協助或想要取消的徵象 - Bot 很常遇到的兩種中斷情形。 在這項檢查完成之後，Bot 會呼叫 `dc.continueDialog()` 以處理仍然擱置的任何使用中對話方塊。

# <a name="ctabcsharptab"></a>[C#](#tab/csharptab)

```cs
// Check for top-level interruptions.
string utterance = turnContext.Activity.Text.Trim().ToLowerInvariant();

if (utterance == "help")
{
    // Start a general help dialog. Dialogs already on the stack remain and will continue
    // normally if the help dialog exits normally.
    await dc.BeginDialogAsync(OrderDinnerDialogs.HelpDialogId, null, cancellationToken);
}
else if (utterance == "cancel")
{
    // Cancel any dialog on the stack.
    await turnContext.SendActivityAsync("Canceled.", cancellationToken: cancellationToken);
    await dc.CancelAllDialogsAsync(cancellationToken);
}
else
{
    await dc.ContinueDialogAsync(cancellationToken);

    // Check whether we replied. If not then clear the dialog stack and present the main menu.
    if (!turnContext.Responded)
    {
        await dc.CancelAllDialogsAsync(cancellationToken);
        await dc.BeginDialogAsync(OrderDinnerDialogs.MainDialogId, null, cancellationToken);
    }
}
```

# <a name="javascripttabjstab"></a>[JavaScript](#tab/jstab)

我們會先處理中斷，再將使用者回應傳送至對話方塊。

此程式碼片段假設我們的對話方塊集中有 `helpDialog` 和`mainMenu`。

```javascript
const utterance = (context.activity.text || '').trim();

// Let's look for some interruptions first!
if (utterance.match(/help/ig)) {
    // Launch a new help dialog if the user asked for help
    await dc.beginDialog('helpDialog');
} else if (utterance.match(/cancel/ig)) {
    // Cancel any active dialogs if the user says cancel
    await dc.context.sendActivity('Canceled.');
    await dc.cancelAllDialogs();
}

// If the bot hasn't yet responded...
if (!context.responded) {
    // Continue any active dialog, which might send a response...
    await dc.continueDialog();

    // Finally, if the bot still hasn't sent a response, send instructions.
    if (!context.responded) {
        await dc.cancelAllDialogs();
        await context.sendActivity("Starting the main menu...");
        await dc.beginDialog('mainMenu');
    }
}
```

---
