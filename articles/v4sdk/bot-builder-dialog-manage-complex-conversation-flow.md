---
title: 使用對話方塊管理複雜的對話流程 | Microsoft Docs
description: 了解如何在適用於 Node.js 的 Bot 建立器 SDK 中，使用對話方塊來管理複雜的對話流程。
keywords: 複雜對話流程, 重複, 迴圈, 選單, 對話方塊, 提示, 瀑布, 對話方塊集
author: v-ducvo
ms.author: v-ducvo
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 10/03/2018
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 871bfd9f8d693c5082fe1ccf38349f4d3d46ece2
ms.sourcegitcommit: b8bd66fa955217cc00b6650f5d591b2b73c3254b
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/15/2018
ms.locfileid: "49326565"
---
# <a name="manage-complex-conversation-flows-with-dialogs"></a>使用對話方塊來管理複雜的對話流程

[!INCLUDE [pre-release-label](~/includes/pre-release-label.md)]

在上一篇文章中，我們已示範如何使用對話方塊程式庫來管理簡單的對話。 在[簡單對話流程](bot-builder-dialog-manage-conversation-flow.md)中，使用者從「瀑布圖」的第一個步驟開始，持續進行到最後一個步驟，對話交換即完成。 在本文中，我們會使用對話方塊來管理更複雜的對話，其中部分可以進行分支和迴圈。 為完成此作業，我們將使用對話方塊內容上定義的各種方法及瀑布步驟內容，而且我們會在對話方塊的不同部分之間傳遞引數。

請參閱[對話方塊程式庫](bot-builder-concept-dialog.md)，以深入了解對話方塊的背景資訊。

為了讓您可以更充分地控制「對話方塊堆疊」，**對話方塊**程式庫提供了 _replace dialog_ 方法。 此方法可讓您將目前使用中的對話方塊交換成另一個對話方塊，同時維持對話的狀態或流程。 如果您要建立更複雜的互動，_begin dialog_ 和 _replace dialog_ 方法可讓您建立必要的分支和迴圈。 如果您的對話複雜性升高到難以管理瀑布對話方塊的情況，請使用[元件對話方塊](bot-builder-compositcontrol.md)，或根據基底 `Dialog` 類別建置自訂對話方塊管理類別，來研究此狀況。

在本文中，我們將針對旅館門房 Bot 建立範例對話，來賓可能會使用此 Bot 來取得常見服務：預訂旅館餐廳和透過客房服務訂餐。  這每一個功能 (包含將其連接在一起的功能表) 會建立為對話方塊集合中的對話方塊。

Bot 最上層的對話方塊會提供來賓這兩個選項。 如果來賓想要訂位，最上層的對話方塊就會使用 _begin dialog_ 非同步方法來啟動訂位對話方塊。 如果來賓想要預訂客房服務，最上層的對話方塊會改為啟動 [晚餐訂購] 對話方塊。

[晚餐訂購] 對話方塊會先要求來賓從菜單中選取食物項目，然後詢問房間號碼。 食物項目的選取「也是」對話方塊，來賓從菜單中選取項目時，此對話方塊會受到呼叫並執行多次，直到送出晚餐訂單為止。

此圖說明本文中將建立的對話方塊，以及對話彼此的關聯。

![本文中所使用的對話方塊說明](~/media/complex-conversation-flows.png)

## <a name="how-to-branch"></a>如何分支

對話方塊內容會維持_對話方塊堆疊_，對於堆疊中的每個對話方塊，追蹤哪個步驟是下一步。 「開始對話方塊」方法會將對話方塊推送至堆疊頂端，而其「結束對話方塊」方法則會將頂端的對話方塊從堆疊中移除。

若要建立分支，請從一組子對話方塊中選取一個來開始。 如需有關此概念的詳細資訊，請參閱[建立對話分支](bot-builder-concept-dialog.md#branch-a-conversation)。

## <a name="how-to-loop"></a>如何執行迴圈

對話方塊內容的 _replace dialog_ 方法可讓您取代堆疊頂端的對話方塊。 舊對話方塊的狀態會遭捨棄，而新對話方塊會從頭開始。 藉由以對話方塊本身來取代對話，您可以使用這個方法來建立迴圈。 不過，為了[保存 Bot 已收集的任何資訊](bot-builder-howto-v4-state.md)，您必須在對 _replace dialog_ 方法的呼叫中，將該資訊傳遞至對話方塊的新執行個體。

在下列範例中，您會看到客房服務訂單會儲存在對話狀態方塊中，而當 `orderPrompt` 對話方塊重複時，目前的對話方塊狀態會傳入，以便新對話方塊的狀態可以繼續新增項目。 如果您想要將此資訊儲存在對話方塊以外的 Bot 狀態，請參閱如何[保存使用者資料](bot-builder-tutorial-persist-user-inputs.md)。

## <a name="create-the-dialogs-for-the-hotel-bot"></a>建立旅館 Bot 的對話方塊

在本節中，我們將針對所描述的旅館 Bot 建立對話方塊來管理對話。

### <a name="install-the-dialogs-library"></a>安裝 dialogs 程式庫

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

我們先從基本的 EchoBot 範本開始。 如需指示，請參閱[適用於 .NET 的快速入門](../dotnet/bot-builder-dotnet-sdk-quickstart.md)。

若要使用對話，請為專案或解決方案安裝 `Microsoft.Bot.Builder.Dialogs` NuGet 套件。
然後視需要在程式碼檔案中的 using 陳述式，參考對話方塊程式庫。

```csharp
using Microsoft.Bot.Builder.Dialogs;
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

我們先從 EchoBot 範本開始。 如需指示，請參閱[適用於 JavaScript 的快速入門](../javascript/bot-builder-javascript-quickstart.md)。



可以從 npm 下載 `botbuilder-dialogs` 程式庫。 若要安裝 `botbuilder-dialogs` 程式庫，請執行下列 npm 命令：

```cmd
npm install --save botbuilder-dialogs
```

若要在 Bot 中使用**對話**，請將其包含在 Bot 程式碼中。 例如，將下列內容新增至您的 **index.js** 檔案中：

```javascript
const { DialogSet } = require('botbuilder-dialogs');
```

以及將下列內容新增至 **bot.js** 檔案：

```javascript
const { DialogSet, NumberPrompt, ChoicePrompt, WaterfallDialog } = require('botbuilder-dialogs');
```

---

### <a name="create-a-dialog-set"></a>建立對話方塊集

建立_對話方塊集_，我們會將此範例的所有對話方塊加入其中。

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

建立 **HotelDialogs** 類別，並新增所需的 using 陳述式。

```csharp
using System;
using System.Collections.Generic;
using System.Linq;
using System.Threading;
using System.Threading.Tasks;
using Microsoft.Bot.Builder;
using Microsoft.Bot.Builder.Dialogs;
using Microsoft.Bot.Builder.Dialogs.Choices;
using Microsoft.Bot.Schema;
```

從 **DialogSet** 衍生類別。 包含使用 `IStatePropertyAccessor<DialogState>` 參數的建構函式，以管理對話方塊集合的執行個體內部狀態。 此外，請定義識別碼和金鑰，以用來識別此對話方塊集的對話方塊、提示和狀態資訊。

```csharp
/// <summary>Contains the set of dialogs and prompts for the hotel bot.</summary>
public class HotelDialogs : DialogSet
{
    /// <summary>The ID of the top-level dialog.</summary>
    public const string MainMenu = "mainMenu";

    public HotelDialogs(IStatePropertyAccessor<DialogState> dialogStateAccessor)
        : base(dialogStateAccessor)
    {
    }

    /// <summary>Contains the IDs for the other dialogs in the set.</summary>
    private static class Dialogs
    {
        public const string OrderDinner = "orderDinner";
        public const string OrderPrompt = "orderPrompt";
        public const string ReserveTable = "reserveTable";
    }

    /// <summary>Contains the IDs for the prompts used by the dialogs.</summary>
    private static class Inputs
    {
        public const string Choice = "choicePrompt";
        public const string Number = "numberPrompt";
    }

    /// <summary>Contains the keys used to manage dialog state.</summary>
    private static class Outputs
    {
        public const string OrderCart = "orderCart";
        public const string OrderTotal = "orderTotal";
        public const string RoomNumber = "roomNumber";
    }
}
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

在 **index.js** 檔案中，新增程式碼以建立管理對話方塊狀態的狀態屬性存取子，並使用此存取子來建立供 Bot 使用的對話方塊集合。

```javascript
// Create conversation state with in-memory storage provider.
const conversationState = new ConversationState(memoryStorage);
const dialogStateAccessor = conversationState.createProperty('dialogState');

// Create a dialog set for the bot.
const dialogSet = new DialogSet(dialogStateAccessor);

// Create the bot.
const bot = new MyBot(conversationState, dialogSet)
```

接著，更新活動處理呼叫以使用 Bot 物件。

```javascript
// Listen for incoming requests.
server.post('/api/messages', (req, res) => {
    adapter.processActivity(req, res, async (context) => {
        // Route to the bot's turn handler.
        await bot.onTurn(context);
    });
});
```

---

### <a name="add-the-prompts-to-the-set"></a>將提示新增至集合

我們將使用 **ChoicePrompt** 向來賓詢問要訂購晚餐或預訂餐廳，以及晚餐的菜單要選取哪個選項。 若是訂購晚餐，接著會使用 **NumberPrompt** 詢問來賓的房號。

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

在 **HotelDialogs** 建構函式中，新增兩個提示。

```csharp
// Add the prompts.
Add(new ChoicePrompt(Inputs.Choice));
Add(new NumberPrompt<int>(Inputs.Number));
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

更新 Bot 建構函式，並將兩個提示新增至對話方塊集合。

```javascript
constructor(conversationState, dialogSet) {
    // Creates a new state accessor property.
    // See https://aka.ms/about-bot-state-accessors to learn more about the bot state and state accessors.
    this.countProperty = conversationState.createProperty(TURN_COUNTER_PROPERTY);
    this.conversationState = conversationState;
    this.dialogSet = dialogSet;

    this.dialogSet.add(new ChoicePrompt('choicePrompt'));
    this.dialogSet.add(new NumberPrompt('numberPrompt'));
}
```

---

### <a name="define-some-of-the-supporting-information"></a>定義一些支援的資訊

由於我們需要晚餐菜單上每個選項的相關資訊，現在讓我們來設定。

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

建立內部靜態**清單**類別以包含這項資訊。 我們也會建立內部 **WelcomeChoice** 和 **MenuChoice** 類別以包含每個選項的相關資訊。

處理這個時，我們將在最上層的歡迎對話方塊中新增選項清單的資訊，也會建立稍後提示來賓輸入此資訊時會使用的支援清單。 這是一些新增的前置工作，可讓對話方塊程式碼更簡單。

```csharp
/// <summary>Describes an option for the top-level dialog.</summary>
private class WelcomeChoice
{
    /// <summary>Gets or sets the text to show the guest for this option.</summary>
    public string Description { get; set; }

    /// <summary>Gets or sets the ID of the associated dialog for this option.</summary>
    public string DialogName { get; set; }
}

/// <summary>Describes an option for the food-selection dialog.</summary>
/// <remarks>We have two types of options. One represents meal items that the guest
/// can add to their order. The other represents a request to process or cancel the
/// order.</remarks>
private class MenuChoice
{
    /// <summary>The request text for cancelling the meal order.</summary>
    public const string Cancel = "Cancel order";

    /// <summary>The request text for processing the meal order.</summary>
    public const string Process = "Process order";

    /// <summary>Gets or sets the name of the meal item or the request.</summary>
    public string Name { get; set; }

    /// <summary>Gets or sets the price of the meal item; or NaN for a request.</summary>
    public double Price { get; set; }

    /// <summary>Gets the text to show the guest for this option.</summary>
    public string Description => double.IsNaN(Price) ? Name : $"{Name} - ${Price:0.00}";
}
```

```csharp
/// <summary>Contains the lists used to present options to the guest.</summary>
private static class Lists
{
    /// <summary>Gets the options for the top-level dialog.</summary>
    public static List<WelcomeChoice> WelcomeOptions { get; } = new List<WelcomeChoice>
    {
        new WelcomeChoice { Description = "Order dinner", DialogName = Dialogs.OrderDinner },
        new WelcomeChoice { Description = "Reserve a table", DialogName = Dialogs.ReserveTable },
    };

    private static readonly List<string> _welcomeList = WelcomeOptions.Select(x => x.Description).ToList();

    /// <summary>Gets the choices to present in the choice prompt for the top-level dialog.</summary>
    public static IList<Choice> WelcomeChoices { get; } = ChoiceFactory.ToChoices(_welcomeList);

    /// <summary>Gets the reprompt action for the top-level dialog.</summary>
    public static Activity WelcomeReprompt
    {
        get
        {
            var reprompt = MessageFactory.SuggestedActions(_welcomeList, "Please choose an option");
            reprompt.AttachmentLayout = AttachmentLayoutTypes.List;
            return reprompt as Activity;
        }
    }

    /// <summary>Gets the options for the food-selection dialog.</summary>
    public static List<MenuChoice> MenuOptions { get; } = new List<MenuChoice>
    {
        new MenuChoice { Name = "Potato Salad", Price = 5.99 },
        new MenuChoice { Name = "Tuna Sandwich", Price = 6.89 },
        new MenuChoice { Name = "Clam Chowder", Price = 4.50 },
        new MenuChoice { Name = MenuChoice.Process, Price = double.NaN },
        new MenuChoice { Name = MenuChoice.Cancel, Price = double.NaN },
    };

    private static readonly List<string> _menuList = MenuOptions.Select(x => x.Description).ToList();

    /// <summary>Gets the choices to present in the choice prompt for the food-selection dialog.</summary>
    public static IList<Choice> MenuChoices { get; } = ChoiceFactory.ToChoices(_menuList);

    /// <summary>Gets the reprompt action for the food-selection dialog.</summary>
    public static Activity MenuReprompt
    {
        get
        {
            var reprompt = MessageFactory.SuggestedActions(_menuList, "Please choose an option");
            reprompt.AttachmentLayout = AttachmentLayoutTypes.List;
            return reprompt as Activity;
        }
    }
}
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

在 **bot.js** 檔案中，建立 **dinnerMenu** 常數以包含這項資訊。

```javascript
const dinnerMenu = {
    choices: ["Potato Salad - $5.99", "Tuna Sandwich - $6.89", "Clam Chowder - $4.50",
        "Process order", "Cancel"],
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

### <a name="create-the-welcome-dialog"></a>建立 [歡迎] 對話方塊

這個對話方塊會使用 `ChoicePrompt` 顯示功能表，然後等候使用者選擇選項。 當使用者選擇 `Order dinner` 或 `Reserve a table` 時，該項目就會啟動適當的對話方塊。 完成子對話方塊時，對話方塊不會在最後一個步驟中直接結束，讓使用者疑惑發生了什麼事，而是藉由再次啟動 `mainMenu` 對話方塊來建立 `mainMenu` 對話方塊的迴圈。 有了這個簡單的結構，Bot 一律會顯示功能表，使用者即可了解其選擇為何。

`mainMenu` 對話方塊包含下列瀑布步驟：

* 在瀑布圖的第一個步驟中，我們會初始化或清除對話方塊狀態、歡迎來賓，並要求來賓從可用選項中選擇：`Order dinner` 或 `Reserve a table`。
* 在第二個步驟中，我們會擷取來賓的選取項目，並啟動與該選取項目相關聯的子對話方塊。 子對話方塊結束後，這個對話方塊會繼續進行下一個步驟。
* 在最後一個步驟中，我們使用 _replace dialog_ 方法將此對話方塊取代為新的執行個體，可有效地將 [歡迎] 對話方塊變成迴圈功能表。

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

在建構函式中，新增瀑布對話方塊。 然後定義瀑布步驟。 在這裡，我們已在巢狀類別中定義這些步驟。

在 **HotelDialogs** 建構函式中。

```csharp
// Define the steps for and add the main welcome dialog.
WaterfallStep[] welcomeDialogSteps = new WaterfallStep[]
{
    MainDialogSteps.PresentMenuAsync,
    MainDialogSteps.ProcessInputAsync,
    MainDialogSteps.RepeatMenuAsync,
};

Add(new WaterfallDialog(MainMenu, welcomeDialogSteps));
```

在 **HotelDialogs** 類別中，新增步驟定義。

```csharp
/// <summary>
/// Contains the waterfall dialog steps for the order-dinner dialog.
/// </summary>
private static class MainDialogSteps
{
    public static async Task<DialogTurnResult> PresentMenuAsync(
        WaterfallStepContext stepContext,
        CancellationToken cancellationToken)
    {
        // Greet the guest and ask them to choose an option.
        await stepContext.Context.SendActivityAsync(
            "Welcome to Contoso Hotel and Resort.",
            cancellationToken: cancellationToken);
        return await stepContext.PromptAsync(
            Inputs.Choice,
            new PromptOptions
            {
                Prompt = MessageFactory.Text("How may we serve you today?"),
                RetryPrompt = Lists.WelcomeReprompt,
                Choices = Lists.WelcomeChoices,
            },
            cancellationToken);
    }

    public static async Task<DialogTurnResult> ProcessInputAsync(
        WaterfallStepContext stepContext,
        CancellationToken cancellationToken)
    {
        // Begin a child dialog associated with the chosen option.
        var choice = (FoundChoice)stepContext.Result;
        var dialogId = Lists.WelcomeOptions[choice.Index].DialogName;

        return await stepContext.BeginDialogAsync(dialogId, null, cancellationToken);
    }

    public static async Task<DialogTurnResult> RepeatMenuAsync(
        WaterfallStepContext stepContext,
        CancellationToken cancellationToken)
    {
        // Start this dialog over again.
        return await stepContext.ReplaceDialogAsync(MainMenu, null, cancellationToken);
    }
}
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

在 Bot 建構函式中，新增 `mainMenu` 瀑布對話方塊。

```javascript
// Display a menu and ask user to choose a menu item. Direct user to the item selected otherwise, show
// the menu again.
this.dialogSet.add(new WaterfallDialog('mainMenu', [
    async function (step) {
        // Welcome the user and send a prompt.
        await step.context.sendActivity("Welcome to Contoso Hotel and Resort.");
        return await step.prompt('choicePrompt', "How may we serve you today?", ['Order Dinner', 'Reserve a table']);
    },
    async function (step) {
        // Handle the user's response to the previous prompt and branch the dialog.
        if (step.result.value.match(/order dinner/ig)) {
            return await step.beginDialog('orderDinner');
        } else if (step.result.value.match(/reserve a table/ig)) {
            return await step.beginDialog('reserveTable');
        }
    },
    async function (step) {
        // Calling replaceDialog will loop the main menu
        return await step.replaceDialog('mainMenu');
    }
]));
```

---

### <a name="create-the-order-dinner-dialog"></a>建立 [訂購晚餐] 對話方塊

在 [訂購晚餐] 對話方塊中，我們會歡迎來賓使用「晚餐訂購服務」，並立即開始 [食物選取項目] 對話方塊，這會在下一節中討論。 重要的是，如果來賓要求此服務處理其訂單，[食物選取項目] 對話方塊會傳回訂單上的項目清單。 若要完成此程序，此對話方塊接著會詢問要送餐的房間號碼，然後再傳送確認訊息。 如果來賓取消訂單，則此對話方塊不會詢問房間號碼，並會立即結束。

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

針對 **HotelDialog** 類別，新增可用於追蹤來賓晚餐訂單的資料結構。 由於這會保存在對話狀態中，因此類別需要預設建構函式來進行序列化。

```csharp
/// <summary>Contains the guest's dinner order.</summary>
private class OrderCart : List<MenuChoice>
{
    public OrderCart() : base() { }

    public OrderCart(OrderCart other) : base(other) { }
}
```

在建構函式中，新增瀑布對話方塊。 然後定義瀑布步驟。 在這裡，我們已在巢狀類別中定義這些步驟。

在 **HotelDialogs** 建構函式中。

```csharp
// Define the steps for and add the order-dinner dialog.
WaterfallStep[] orderDinnerDialogSteps = new WaterfallStep[]
{
    OrderDinnerSteps.StartFoodSelectionAsync,
    OrderDinnerSteps.GetRoomNumberAsync,
    OrderDinnerSteps.ProcessOrderAsync,
};

Add(new WaterfallDialog(Dialogs.OrderDinner, orderDinnerDialogSteps));
```

在 **HotelDialogs** 類別中，新增步驟定義。

* 在第一個步驟中，我們會傳送歡迎訊息，並啟動 [食物選取項目] 對話方塊。
* 在第二個步驟中，我們會檢查 [食物選取項目] 對話是否傳回訂單購物車。
  * 如果是，提示來賓輸入房號，並儲存購物車資訊。
  * 如果否，假設來賓已取消訂單，則呼叫 _end dialog_ 方法結束此對話方塊。
* 在最後一個步驟中，記錄房號，並在結束前傳送確認訊息。 在這個步驟中，Bot 會將訂單轉送至處理服務。

```csharp
/// <summary>
/// Contains the waterfall dialog steps for the order-dinner dialog.
/// </summary>
private static class OrderDinnerSteps
{
    public static async Task<DialogTurnResult> StartFoodSelectionAsync(
        WaterfallStepContext stepContext,
        CancellationToken cancellationToken)
    {
        await stepContext.Context.SendActivityAsync(
            "Welcome to our Dinner order service.",
            cancellationToken: cancellationToken);

        // Start the food selection dialog.
        return await stepContext.BeginDialogAsync(Dialogs.OrderPrompt, null, cancellationToken);
    }

    public static async Task<DialogTurnResult> GetRoomNumberAsync(
        WaterfallStepContext stepContext,
        CancellationToken cancellationToken)
    {
        if (stepContext.Result != null && stepContext.Result is OrderCart cart)
        {
            // If there are items in the order, record the order and ask for a room number.
            stepContext.Values[Outputs.OrderCart] = cart;
            return await stepContext.PromptAsync(
                Inputs.Number,
                new PromptOptions
                {
                    Prompt = MessageFactory.Text("What is your room number?"),
                    RetryPrompt = MessageFactory.Text("Please enter your room number."),
                },
                cancellationToken);
        }
        else
        {
            // Otherwise, assume the order was cancelled by the guest and exit.
            return await stepContext.EndDialogAsync(null, cancellationToken);
        }
    }

    public static async Task<DialogTurnResult> ProcessOrderAsync(
        WaterfallStepContext stepContext,
        CancellationToken cancellationToken)
    {
        // Get and save the guest's answer.
        var roomNumber = (int)stepContext.Result;
        stepContext.Values[Outputs.RoomNumber] = roomNumber;

        // Process the dinner order using the collected order cart and room number.

        await stepContext.Context.SendActivityAsync(
            $"Thank you. Your order will be delivered to room {roomNumber} within 45 minutes.",
            cancellationToken: cancellationToken);
        return await stepContext.EndDialogAsync(null, cancellationToken);
    }
}
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

在 Bot 建構函式中，新增 `orderDinner` 瀑布對話方塊。

```javascript
// Order dinner:
// Help user order dinner from a menu
this.dialogSet.add(new WaterfallDialog('orderDinner', [
    async function (step) {
        await step.context.sendActivity("Welcome to our dinner order service.");

        return await step.beginDialog('orderPrompt', step.values.orderCart = {
            orders: [],
            total: 0
        }); // Prompt for orders
    },
    async function (step) {
        if (step.result == "Cancel") {
            return await step.endDialog();
        } else {
            return await step.prompt('numberPrompt', "What is your room number?");
        }
    },
    async function (step) {
        await step.context.sendActivity(`Thank you. Your order will be delivered to room ${step.result} within 45 minutes.`);
        return await step.endDialog();
    }
]));
```

---

### <a name="create-the-order-prompt-dialog"></a>建立 [訂單提示] 對話方塊

在 [食物選取項目] 對話方塊中，我們會提供來賓選項清單，其中包含可訂購的晚餐項目以及兩個訂單處理要求。 這個對話方塊會重複執行，直到來賓選擇讓 Bot 進行處理，或取消訂單。

* 來賓選取晚餐項目時，我們會將其加入_購物車_。
* 如果來賓選擇處理訂單，我們會先檢查購物車是否是空的。
  * 如果是空的，我們會傳送錯誤訊息，並繼續執行迴圈。
  * 如果不是，則結束此對話方塊，將購物車資訊傳回至父對話方塊。
* 如果來賓選擇取消訂單，則會結束對話方塊，但不傳回購物車資訊。

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

在建構函式中，新增瀑布對話方塊。 然後定義瀑布步驟。 在這裡，我們已在巢狀類別中定義這些步驟。

在 **HotelDialogs** 建構函式中。

```csharp
// Define the steps for and add the order-prompt dialog.
WaterfallStep[] orderPromptDialogSteps = new WaterfallStep[]
{
    OrderPromptSteps.PromptForItemAsync,
    OrderPromptSteps.ProcessInputAsync,
};

Add(new WaterfallDialog(Dialogs.OrderPrompt, orderPromptDialogSteps));
```

* 在第一個步驟中，我們會初始化對話方塊狀態。 如果對話方塊的輸入引數包含購物車資訊，我們會將其儲存到對話方塊狀態中；否則，建立空的購物車，然後將其加入。 然後，我們會提示來賓從晚餐菜單中進行選擇。
* 在下一個步驟中，則看來賓挑選的選項：
  * 如果是處理訂單的要求，則檢查購物車是否包含任何項目。
    * 如果有，結束對話方塊，傳回購物車內容。
    * 如果沒有，傳送錯誤訊息給來賓，然後從對話方塊開頭重新開始。
  * 如果是要求取消訂單，則結束對話方塊，傳回空的字典。
  * 如果是晚餐項目，將其加入購物車、傳送狀態訊息，並啟動對話方塊，然後傳入目前的訂單狀態。

在 **HotelDialogs** 類別中，新增步驟定義。

```csharp
/// <summary>
/// Contains the waterfall dialog steps for the order-prompt dialog.
/// </summary>
private static class OrderPromptSteps
{
    public static async Task<DialogTurnResult> PromptForItemAsync(
        WaterfallStepContext stepContext,
        CancellationToken cancellationToken)
    {
        // First time through, options will be null.
        var cart = (stepContext.Options is OrderCart oldCart && oldCart != null)
            ? new OrderCart(oldCart) : new OrderCart();

        stepContext.Values[Outputs.OrderCart] = cart;
        stepContext.Values[Outputs.OrderTotal] = cart.Sum(item => item.Price);

        return await stepContext.PromptAsync(
            Inputs.Choice,
            new PromptOptions
            {
                Prompt = MessageFactory.Text("What would you like?"),
                RetryPrompt = Lists.MenuReprompt,
                Choices = Lists.MenuChoices,
            },
            cancellationToken);
    }

    public static async Task<DialogTurnResult> ProcessInputAsync(
        WaterfallStepContext stepContext,
        CancellationToken cancellationToken)
    {
        // Get the guest's choice.
        var choice = (FoundChoice)stepContext.Result;
        var menuOption = Lists.MenuOptions[choice.Index];

        // Get the current order from dialog state.
        var cart = (OrderCart)stepContext.Values[Outputs.OrderCart];

        if (menuOption.Name is MenuChoice.Process)
        {
            if (cart.Count > 0)
            {
                // If there are any items in the order, then exit this dialog,
                // and return the list of selected food items.
                return await stepContext.EndDialogAsync(cart, cancellationToken);
            }
            else
            {
                // Otherwise, send an error message and restart from
                // the beginning of this dialog.
                await stepContext.Context.SendActivityAsync(
                    "Your cart is empty. Please add at least one item to the cart.",
                    cancellationToken: cancellationToken);
                return await stepContext.ReplaceDialogAsync(Dialogs.OrderPrompt, null, cancellationToken);
            }
        }
        else if (menuOption.Name is MenuChoice.Cancel)
        {
            await stepContext.Context.SendActivityAsync(
                "Your order has been cancelled.",
                cancellationToken: cancellationToken);

            // Exit this dialog, returning null.
            return await stepContext.EndDialogAsync(null, cancellationToken);
        }
        else
        {
            // Add the selected food item to the order and update the order total.
            cart.Add(menuOption);
            var total = (double)stepContext.Values[Outputs.OrderTotal] + menuOption.Price;
            stepContext.Values[Outputs.OrderTotal] = total;

            await stepContext.Context.SendActivityAsync(
                $"Added {menuOption.Name} (${menuOption.Price:0.00}) to your order." +
                    Environment.NewLine + Environment.NewLine +
                    $"Your current total is ${total:0.00}.",
                cancellationToken: cancellationToken);

            // Present the order options again, passing in the current order state.
            return await stepContext.ReplaceDialogAsync(Dialogs.OrderPrompt, cart);
        }
    }
}
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

在 Bot 建構函式中，新增 `orderPrompt` 瀑布對話方塊。

```javascript
// Helper dialog to repeatedly prompt user for orders
this.dialogSet.add(new WaterfallDialog('orderPrompt', [
    async function (step) {
        // Define a new cart of one does not exists
        step.values.orderCart = step.options;

        return await step.prompt('choicePrompt', "What would you like?", dinnerMenu.choices);
    },
    async function (step) {
        const choice = step.result;
        if (choice.value.match(/process order/ig)) {
            if (step.values.orderCart.orders.length > 0) {
                // Process the order
                await step.context.sendActivity("Processing your order.");
                // ...
                step.values.orderCart = undefined; // Reset cart
                return await step.endDialog();
            } else {
                await step.context.sendActivity("Your cart was empty. Please add at least one item to the cart.");
                // Ask again
                return await step.replaceDialog('orderPrompt', step.values.orderCart);
            }
        } else if (choice.value.match(/cancel/ig)) {
            await step.context.sendActivity("Your order has been canceled.");
            return await step.endDialog(choice.value);
        } else {
            var item = dinnerMenu[choice.value];

            // Only proceed if user chooses an item from the menu
            if (!item) {
                await step.context.sendActivity("Sorry, that is not a valid item. Please pick one from the menu.");

                // Ask again
                return await step.replaceDialog('orderPrompt', step.values.orderCart);
            } else {
                // Add the item to cart
                step.values.orderCart.orders.push(item);
                step.values.orderCart.total += item.Price;

                await step.context.sendActivity(`Added to cart: ${choice.value}. <br/>Current total: $${step.values.orderCart.total}`);

                // Ask again
                return await step.replaceDialog('orderPrompt', step.values.orderCart);
            }
        }
    }
]));
```

上述範例程式碼會顯示主要的 `orderDinner` 對話使用名為 `orderPrompt` 的協助程式對話來處理使用者的選擇。 `orderPrompt` 對話方塊會顯示食物項目選單、要求使用者選擇項目、將項目新增至購物車，然後在迴圈中再次提示。 這可讓使用者將多個項目新增至他們的訂單。 對話會重複直到使用者選擇 `Process order` 或 `Cancel`。 到時候，執行權會交回給父對話方塊 (`orderDinner` 對話方塊)。 `orderDinner` 對話會進行一些最後關頭的整理，如果使用者想要處理訂單的話；否則它會結束，並將執行權傳回給其父對話 (例如 `mainMenu`)。 `mainMenu` 對話接著會繼續執行最後一個步驟，也就是單純重新顯示主功能表選項。

---

### <a name="create-the-reserve-table-dialog"></a>建立 [預訂餐廳] 對話方塊

<!-- TODO: Update the "Manage simple conversation flows" topic to use a reserveTable dialog, and then reference that from here. -->

為了讓此範例短一點，在這裡只示範 `reserveTable` 對話方塊的最低實作。

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

在建構函式中，新增瀑布對話方塊。 然後定義瀑布步驟。 在這裡，我們已在巢狀類別中定義這些步驟。

在 **HotelDialogs** 建構函式中。

```csharp
// Define the steps for and add the reserve-table dialog.
WaterfallStep[] reserveTableDialogSteps = new WaterfallStep[]
{
    ReserveTableSteps.StubAsync,
};

Add(new WaterfallDialog(Dialogs.ReserveTable, reserveTableDialogSteps));
```

在 **HotelDialogs** 類別中，新增步驟定義。

```csharp
/// <summary>
/// Contains the waterfall dialog steps for the reserve-table dialog.
/// </summary>
private static class ReserveTableSteps
{
    public static async Task<DialogTurnResult> StubAsync(
        WaterfallStepContext stepContext,
        CancellationToken cancellationToken)
    {
        await stepContext.Context.SendActivityAsync(
            "Your table has been reserved.",
            cancellationToken: cancellationToken);

        return await stepContext.EndDialogAsync(null, cancellationToken);
    }
}
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

在 Bot 建構函式中，新增 `reserveTable` 瀑布對話方塊的預留位置。

```javascript
// Reserve a table:
// Help the user to reserve a table
this.dialogSet.add(new WaterfallDialog('reserveTable', [
    // Replace this waterfall with your reservation steps.
    async function(step){
        await step.context.sendActivity("Your table has been reserved");
        await step.endDialog();
    }
]));
```

---

### <a name="update-the-bot-code-to-call-the-dialogs"></a>更新 Bot 程式碼來呼叫對話方塊

更新 Bot 回合處理常式程式碼以呼叫對話方塊。

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

將 **EchoBotAccessors.cs** 重新命名為 **BotAccessors.cs**，並將類別名稱從 `EchoBotAccessors` 變更為 `BotAccessors`。 更新 using 陳述式和類別定義，以提供使用此 Bot 時需要的狀態屬性存取子。

```csharp
using System;
using Microsoft.Bot.Builder;
using Microsoft.Bot.Builder.Dialogs;
```

```csharp
/// <summary>
/// This class is created as a Singleton and passed into the IBot-derived constructor.
///  - See <see cref="EchoWithCounterBot"/> constructor for how that is injected.
///  - See the Startup.cs file for more details on creating the Singleton that gets
///    injected into the constructor.
/// </summary>
public class BotAccessors
{
    /// <summary>
    /// Initializes a new instance of the <see cref="BotAccessors"/> class.
    /// Contains the <see cref="ConversationState"/> and associated <see cref="IStatePropertyAccessor{T}"/>.
    /// </summary>
    /// <param name="conversationState">The state object that stores the counter.</param>
    public BotAccessors(ConversationState conversationState)
    {
        ConversationState = conversationState ?? throw new ArgumentNullException(nameof(conversationState));
    }

    /// <summary>
    /// Gets the <see cref="IStatePropertyAccessor{T}"/> name used for the <see cref="DialogState"/> accessor.
    /// </summary>
    /// <remarks>Accessors require a unique name.</remarks>
    /// <value>The accessor name for the dialog state accessor.</value>
    public static string DialogStateAccessorName { get; } = $"{nameof(BotAccessors)}.DialogState";

    /// <summary>
    /// Gets or sets the DialogState property accessor.
    /// </summary>
    /// <value>
    /// The DialogState property accessor.
    /// </value>
    public IStatePropertyAccessor<DialogState> DialogStateAccessor { get; set; }

    /// <summary>
    /// Gets the <see cref="ConversationState"/> object for the conversation.
    /// </summary>
    /// <value>The <see cref="ConversationState"/> object.</value>
    public ConversationState ConversationState { get; }
}
```

更新 **Startup.cs** 檔案以設定 `BotAccessors` singleton。

1. 更新 using 陳述式。

    ```csharp
    using System;
    using System.Linq;
    using Microsoft.AspNetCore.Builder;
    using Microsoft.AspNetCore.Hosting;
    using Microsoft.Bot.Builder;
    using Microsoft.Bot.Builder.Dialogs;
    using Microsoft.Bot.Builder.Integration;
    using Microsoft.Bot.Builder.Integration.AspNet.Core;
    using Microsoft.Bot.Configuration;
    using Microsoft.Bot.Connector.Authentication;
    using Microsoft.Extensions.Configuration;
    using Microsoft.Extensions.DependencyInjection;
    using Microsoft.Extensions.Logging;
    using Microsoft.Extensions.Options;
    ```

1. 更新 `ConfigureServices` 方法中註冊 Bot 狀態屬性存取子的部分。

    ```csharp
    // Create and register state accesssors.
    // Acessors created here are passed into the IBot-derived class on every turn.
    services.AddSingleton<BotAccessors>(sp =>
    {
        var options = sp.GetRequiredService<IOptions<BotFrameworkOptions>>().Value;
        var conversationState = options.State.OfType<ConversationState>().FirstOrDefault();

        // Create the custom state accessor.
        // State accessors enable other components to read and write individual properties of state.
        var accessors = new BotAccessors(conversationState)
        {
            DialogStateAccessor = conversationState.CreateProperty<DialogState>(BotAccessors.DialogStateAccessorName),
        };

        return accessors;
    });
    ```

將 EchoWithCounterBot.cs 檔案重新命名為 HotelBot.cs，並將類別名稱從 EchoWithCounterBot 變更為 HotelBot。

1. 更新 Bot 的初始化程式碼。

    ```csharp
    private readonly BotAccessors _accessors;
    private readonly HotelDialogs _dialogs;
    private readonly ILogger _logger;

    /// <summary>
    /// Initializes a new instance of the <see cref="HotelBot"/> class.
    /// </summary>
    /// <param name="accessors">A class containing <see cref="IStatePropertyAccessor{T}"/> used to manage state.</param>
    /// <param name="loggerFactory">A <see cref="ILoggerFactory"/> that is hooked to the Azure App Service provider.</param>
    public HotelBot(BotAccessors accessors, ILoggerFactory loggerFactory)
    {
        _logger = loggerFactory.CreateLogger<HotelBot>();
        _logger.LogTrace("EchoBot turn start.");
        _accessors = accessors;
        _dialogs = new HotelDialogs(_accessors.DialogStateAccessor);
    }
    ```

1. 更新 Bot 的回合處理常式，使其執行該對話方塊。

    ```csharp
    public async Task OnTurnAsync(ITurnContext turnContext, CancellationToken cancellationToken = default(CancellationToken))
    {
        var dc = await _dialogs.CreateContextAsync(turnContext, cancellationToken);

        if (turnContext.Activity.Type == ActivityTypes.Message)
        {
            await dc.ContinueDialogAsync(cancellationToken);
            if (!turnContext.Responded)
            {
                await dc.BeginDialogAsync(HotelDialogs.MainMenu, null, cancellationToken);
            }
        }
        else if (turnContext.Activity.Type == ActivityTypes.ConversationUpdate)
        {
            var activity = turnContext.Activity.AsConversationUpdateActivity();
            if (activity.MembersAdded.Any(member => member.Id != activity.Recipient.Id))
            {
                await dc.BeginDialogAsync(HotelDialogs.MainMenu, null, cancellationToken);
            }
        }

        await _accessors.ConversationState.SaveChangesAsync(turnContext, false, cancellationToken);
    }
    ```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

```javascript
async onTurn(turnContext) {
    let dc = await this.dialogSet.createContext(turnContext);

    // See https://aka.ms/about-bot-activity-message to learn more about the message and other activity types.
    if (turnContext.activity.type === ActivityTypes.Message) {

        await dc.continueDialog();

        if (!turnContext.responded) {
            await dc.beginDialog('mainMenu');
        }
    } else if (turnContext.activity.type === ActivityTypes.ConversationUpdate) {
        // Do we have any new members added to the conversation?
        if (turnContext.activity.membersAdded.length !== 0) {
            // Iterate over all new members added to the conversation
            for (var idx in turnContext.activity.membersAdded) {
                // Greet anyone that was not the target (recipient) of this message.
                // Since the bot is the recipient for events from the channel,
                // context.activity.membersAdded === context.activity.recipient.Id indicates the
                // bot was added to the conversation, and the opposite indicates this is a user.
                if (turnContext.activity.membersAdded[idx].id !== turnContext.activity.recipient.id) {
                    // Start the dialog.
                    await dc.beginDialog('mainMenu');
                }
            }
        }
    }

    // Save state changes
    await this.conversationState.saveChanges(turnContext);
}
```

---

## <a name="next-steps"></a>後續步驟

您可以將其他選項 (如「詳細資訊」或「說明」) 提供到選單的選擇中，以增強這個 Bot。 如需實作這些中斷類型的詳細資訊，請參閱[處理使用者中斷](bot-builder-howto-handle-user-interrupt.md)。

既然您已了解如何使用對話方塊、提示和瀑布來管理複雜的對話流程，讓我們看看如何將對話方塊 (例如 `orderDinner` 和 `reserveTable` 對話方塊) 分成個別物件，而不是將它們全部堆在一個大型對話集中。

> [!div class="nextstepaction"]
> [使用複合控制項建立模組化 Bot 邏輯](bot-builder-compositcontrol.md)
