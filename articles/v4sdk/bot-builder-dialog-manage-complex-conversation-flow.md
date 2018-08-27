---
title: 使用對話方塊管理複雜的對話流程 | Microsoft Docs
description: 了解如何在適用於 Node.js 的 Bot 建立器 SDK 中，使用對話方塊來管理複雜的對話流程。
keywords: 複雜對話流程, 重複, 迴圈, 選單, 對話方塊, 提示, 瀑布, 對話方塊集
author: v-ducvo
ms.author: v-ducvo
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 7/27/2018
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 304de6783a268140bf74f95d96cd9c24e12c4c05
ms.sourcegitcommit: 1abc32353c20acd103e0383121db21b705e5eec3
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/21/2018
ms.locfileid: "40236356"
---
# <a name="manage-complex-conversation-flows-with-dialogs"></a>使用對話方塊來管理複雜的對話流程

[!INCLUDE [pre-release-label](~/includes/pre-release-label.md)]

您可以使用對話方塊程式庫管理簡單和複雜的對話流程。 在[簡單對話流程](bot-builder-dialog-manage-conversation-flow.md)中，使用者從「瀑布圖」的第一個步驟開始，持續進行到最後一個步驟，對話交換即完成。 在本文中，我們會使用對話方塊來管理更複雜的對話，其中部分可以進行分支和迴圈。 若要這樣做，我們會使用對話方塊內容的 _replace_ 方法，以及在對話方塊不同部分間傳遞引數。

<!-- TODO: We need a dialogs conceptual topic to link to, so we can reference that here, in place of describing what they are and what their features are in a how-to topic. -->

為了讓您可以更充分地控制「對話方塊堆疊」，**對話方塊**程式庫提供了 _replace_ 方法。 這個方法可讓您從堆疊中移除目前的對話方塊，將新對話方塊推送至堆疊的頂端，並啟動新對話方塊。 這項功能可讓您提供更複雜的對話。 這些技巧可用來建立各種複雜的對話。 萬一對話複雜度增加到讓對話方塊變得難以管理，可以建立您自己的控制邏輯流程，來追蹤使用者的對話。

<!-- TODO: This is probably a good place to add a link to the modular/composite dialogs topic. -->

在本文中，我們將建立旅館 Bot 的對話方塊，來賓可以用其來預訂餐廳，或訂餐並送到自己的房間。 最上層的對話方塊提供來賓這兩個選項。 如果來賓想要預訂餐廳，最上層的對話方塊就會啟動 [餐廳預訂] 對話方塊。 如果來賓想要訂購晚餐，最上層的對話方塊就會啟動 [晚餐訂購] 對話方塊。 [晚餐訂購] 對話方塊會先要求來賓從菜單中選取食物項目，然後詢問房間號碼。 選取食物項目也是一個對話方塊，以便讓來賓選取多個項目，接著再處理其晚餐訂單。

此圖說明本文中將建立的對話方塊，以及對話彼此的關聯。

![本文中所使用的對話方塊說明](~/media/complex-conversation-flows.png)

## <a name="how-to-branch"></a>如何分支

對話方塊內容會維持_對話方塊堆疊_，對於堆疊中的每個對話方塊，追蹤哪個步驟是下一步。 _begin_ 方法會將對話方塊推送至堆疊頂端，而 _end_ 方法會將頂端的對話方塊從堆疊中移除。

對話方塊可以藉由呼叫對話方塊內容的 _begin_ 方法，並提供新對話方塊識別碼，在同一對話方塊集內啟動新的對話方塊 (也稱為分支)。 原始對話方塊仍在堆疊中，但呼叫對話方塊內容的 _continue_ 方法只會傳送到堆疊頂端的對話方塊，也就是_作用中的對話方塊_。 當對話方塊從堆疊中移除時，對話方塊內容會從停止的地方，繼續使用堆疊中瀑布圖的下一個步驟。

因此，藉由包含在對話方塊中，可以有條件地從一組可能的對話方塊中選擇對話方塊來啟動的步驟，您即可在對話流程內建立分支。

## <a name="how-to-loop"></a>如何執行迴圈

對話方塊內容的 _replace_ 方法可讓您取代堆疊頂端的對話方塊。 舊對話方塊的狀態會遭捨棄，而新對話方塊會從頭開始。 藉由以對話方塊本身來取代對話，您可以使用這個方法來建立迴圈。 不過，為了[保存狀態](bot-builder-howto-v4-state.md)，您必須在對 _replace_ 方法的呼叫中將資訊傳遞至對話方塊的新執行個體。 在下列範例中，您會看到晚餐訂單購物車會儲存在對話狀態方塊中，而當 `orderPrompt` 對話方塊重複時，目前的對話方塊狀態會傳入，以便新對話方塊的狀態可以繼續新增項目。 如果您想要將這些資訊儲存在其他狀態中，請參閱[保存使用者資料](bot-builder-tutorial-persist-user-inputs.md)。

## <a name="create-the-dialogs-for-the-hotel-bot"></a>建立旅館 Bot 的對話方塊

在本節中，我們將針對所描述的旅館 Bot 建立對話方塊來管理對話。

### <a name="install-the-dialogs-library"></a>安裝 dialogs 程式庫

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

我們先從基本的 EchoBot 範本開始。 如需指示，請參閱[適用於 .NET 的快速入門](~/dotnet/bot-builder-dotnet-quickstart.md)。

若要使用對話，請為專案或解決方案安裝 `Microsoft.Bot.Builder.Dialogs` NuGet 套件。
然後視需要在程式碼檔案中的 using 陳述式，參考對話方塊程式庫。

```csharp
using Microsoft.Bot.Builder.Dialogs;
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

可以從 NPM 下載 `botbuilder-dialogs` 程式庫。 若要安裝 `botbuilder-dialogs` 程式庫，請執行下列 NPM 命令：

```cmd
npm install --save botbuilder-dialogs
```

若要在 Bot 中使用**對話**，請將其包含在 Bot 程式碼中。 例如，將下列內容新增至您的 **app.js** 檔案中：

```javascript
const botbuilder_dialogs = require('botbuilder-dialogs');
```

---

### <a name="create-a-dialog-set"></a>建立對話方塊集

建立_對話方塊集_，我們會將此範例的所有對話方塊加入其中。

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

建立 **HotelDialog** 類別，並新增所需的 using 陳述式。

```csharp
using Microsoft.Bot.Builder.Core.Extensions;
using Microsoft.Bot.Builder.Dialogs;
using Microsoft.Bot.Builder.Prompts.Choices;
using Microsoft.Bot.Schema;
using Microsoft.Recognizers.Text;
using System;
using System.Collections.Generic;
using System.Linq;
```

從 **DialogSet** 衍生類別，並定義識別碼和金鑰，以用來識別此對話方塊集的對話方塊、提示和狀態資訊。

```csharp
/// <summary>Contains the set of dialogs and prompts for the hotel bot.</summary>
public class HotelDialog : DialogSet
{
    /// <summary>The ID of the top-level dialog.</summary>
    public const string MainMenu = "mainMenu";

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

若要在 Bot 中使用**對話**，請將其包含在 Bot 程式碼中。

從 **app.js** 檔案參考程式庫。

```javascript
const botbuilder_dialogs = require('botbuilder-dialogs');
```

然後建立對話方塊集。

```javascript
const dialogs = new botbuilder_dialogs.DialogSet();
```

---

### <a name="add-the-prompts-to-the-set"></a>將提示新增至集合

我們將使用 **ChoicePrompt** 向來賓詢問要訂購晚餐或預訂餐廳，以及晚餐的菜單要選取哪個選項。 若是訂購晚餐，接著會使用 **NumberPrompt** 詢問來賓的房號。

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

在 **HotelDialog** 建構函式中，新增兩個提示。

```csharp
// Add the prompts.
this.Add(Inputs.Choice, new ChoicePrompt(Culture.English));
this.Add(Inputs.Number, new NumberPrompt<int>(Culture.English));
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

將這兩個提示新增至對話方塊集。

```javascript
dialogs.add('choicePrompt', new botbuilder_dialogs.ChoicePrompt());
dialogs.add('numberPrompt', new botbuilder_dialogs.NumberPrompt());
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
    /// <summary>The text to show the guest for this option.</summary>
    public string Description { get; set; }

    /// <summary>The ID of the associated dialog for this option.</summary>
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

    /// <summary>The name of the meal item or the request.</summary>
    public string Name { get; set; }

    /// <summary>The price of the meal item; or NaN for a request.</summary>
    public double Price { get; set; }

    /// <summary>The text to show the guest for this option.</summary>
    public string Description => (double.IsNaN(Price)) ? Name : $"{Name} - ${Price:0.00}";
}

/// <summary>Contains the lists used to present options to the guest.</summary>
private static class Lists
{
    /// <summary>The options for the top-level dialog.</summary>
    public static List<WelcomeChoice> WelcomeOptions { get; } = new List<WelcomeChoice>
    {
        new WelcomeChoice { Description = "Order dinner", DialogName = Dialogs.OrderDinner },
        new WelcomeChoice { Description = "Reserve a table", DialogName = Dialogs.ReserveTable },
    };

    private static List<string> WelcomeList { get; } = WelcomeOptions.Select(x => x.Description).ToList();

    /// <summary>The choices to present in the choice prompt for the top-level dialog.</summary>
    public static List<Choice> WelcomeChoices { get; } = ChoiceFactory.ToChoices(WelcomeList);

    /// <summary>The reprompt action for the top-level dialog.</summary>
    public static Activity WelcomeReprompt
    {
        get
        {
            var reprompt = MessageFactory.SuggestedActions(WelcomeList, "Please choose an option");
            reprompt.AttachmentLayout = AttachmentLayoutTypes.List;
            return reprompt as Activity;
        }
    }

    /// <summary>The options for the food-selection dialog.</summary>
    public static List<MenuChoice> MenuOptions { get; } = new List<MenuChoice>
    {
        new MenuChoice { Name = "Potato Salad", Price = 5.99 },
        new MenuChoice { Name = "Tuna Sandwich", Price = 6.89 },
        new MenuChoice { Name = "Clam Chowder", Price = 4.50 },
        new MenuChoice { Name = MenuChoice.Process, Price = double.NaN },
        new MenuChoice { Name = MenuChoice.Cancel, Price = double.NaN },
    };

    private static List<string> MenuList { get; } = MenuOptions.Select(x => x.Description).ToList();

    /// <summary>The choices to present in the choice prompt for the food-selection dialog.</summary>
    public static List<Choice> MenuChoices { get; } = ChoiceFactory.ToChoices(MenuList);

    /// <summary>The reprompt action for the food-selection dialog.</summary>
    public static Activity MenuReprompt
    {
        get
        {
            var reprompt = MessageFactory.SuggestedActions(MenuList, "Please choose an option");
            reprompt.AttachmentLayout = AttachmentLayoutTypes.List;
            return reprompt as Activity;
        }
    }
}
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

建立 **dinnerMenu** 常數以包含這項資訊。

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

這個對話會使用 `ChoicePrompt` 顯示功能表，然後等候使用者選擇選項。 當使用者選擇 `Order dinner` 或 `Reserve a table`，其會開始適當選擇的對話方塊，而在該對話方塊完成時，它不只是在最後一個步驟結束對話方塊，而讓使用者對當下情況感到困惑，此 `mainMenu` 對話方塊會自行重複，以再次啟動 `mainMenu` 對話方塊。 有了這個簡單的結構，Bot 一律會顯示功能表，使用者即可了解其選擇為何。

**MainMenu** 對話方塊包含下列瀑布圖步驟。

* 在瀑布圖的第一個步驟中，我們會初始化或清除對話方塊狀態、歡迎來賓，並要求來賓從可用選項中選擇：`Order dinner` 或 `Reserve a table`。
* 在第二個步驟中，我們要擷取來賓的選取項目，並啟動與該選取項目相關聯的子對話方塊。 子對話方塊結束後，這個對話方塊會繼續進行下一個步驟。
* 在最後一個步驟中，我們使用 **DialogContext.Replace** 方法將此對話方塊取代為新的執行個體，可有效地將 [歡迎] 對話方塊變成迴圈功能表。

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

```csharp
// Add the main welcome dialog.
this.Add(MainMenu, new WaterfallStep[]
{
    async (dc, args, next) =>
    {
        // Greet the guest and ask them to choose an option.
        await dc.Context.SendActivity("Welcome to Contoso Hotel and Resort.");
        await dc.Prompt(Inputs.Choice, "How may we serve you today?", new ChoicePromptOptions
        {
            Choices = Lists.WelcomeChoices,
            RetryPromptActivity = Lists.WelcomeReprompt,
        });
    },
    async (dc, args, next) =>
    {
        // Begin a child dialog associated with the chosen option.
        var choice = (FoundChoice)args["Value"];
        var dialogId = Lists.WelcomeOptions[choice.Index].DialogName;

        await dc.Begin(dialogId, dc.ActiveDialog.State);
    },
    async (dc, args, next) =>
    {
        // Start this dialog over again.
        await dc.Replace(MainMenu);
    },
});
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

```javascript
// Display a menu and ask user to choose a menu item. Direct user to the item selected otherwise, show
// the menu again.
dialogs.add('mainMenu', [
    async function(dc){
        await dc.context.sendActivity("Welcome to Contoso Hotel and Resort.");
        await dc.prompt('choicePrompt', "How may we serve you today?", ['Order Dinner', 'Reserve a table']);
    },
    async function(dc, result){
        if(result.value.match(/order dinner/ig)){
            await dc.begin('orderDinner');
        }
        else if(result.value.match(/reserve a table/ig)){
            await dc.begin('reserveTable');
        }
    },
    async function(dc, result){
        // Start over
        await dc.endAll().begin('mainMenu');
    }
]);
```

---

### <a name="create-the-order-dinner-dialog"></a>建立 [訂購晚餐] 對話方塊

在 [訂購晚餐] 對話方塊中，我們會歡迎來賓使用「晚餐訂購服務」，並立即開始 [食物選取項目] 對話方塊，這會在下一節中討論。 重要的是，如果來賓要求此服務處理其訂單，[食物選取項目] 對話方塊會傳回訂單上的項目清單。 若要完成整個程序，此對話方塊接著會詢問要送餐的房間號碼，然後在結束前傳送確認訊息。 如果來賓取消訂單，則此對話方塊不會詢問房間號碼，並會立即結束。

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

針對 **HotelDialog** 類別，新增可用於追蹤來賓晚餐訂單的資料結構。

```csharp
/// <summary>Contains the guest's dinner order.</summary>
private class OrderCart : List<MenuChoice> { }
```

在 **HotelDialog** 建構函式中，新增 [訂購晚餐] 對話方塊。

* 在第一個步驟中，我們會傳送歡迎訊息，並啟動 [食物選取項目] 對話方塊。
* 在第二個步驟中，我們會檢查 [食物選取項目] 對話是否傳回訂單購物車。
  * 如果是，提示來賓輸入房號，並儲存購物車資訊。
  * 如果否，假設來賓已取消訂單，則呼叫 **DialogContext.End** 結束此對話方塊。
* 在最後一個步驟中，記錄房號，並在結束前傳送確認訊息。 在這個步驟中，Bot 會將訂單轉送至處理服務。

```csharp
// Add the order-dinner dialog.
this.Add(Dialogs.OrderDinner, new WaterfallStep[]
{
    async (dc, args, next) =>
    {
        await dc.Context.SendActivity("Welcome to our Dinner order service.");

        // Start the food selection dialog.
        await dc.Begin(Dialogs.OrderPrompt);
    },
    async (dc, args, next) =>
    {
        if (args.TryGetValue(Outputs.OrderCart, out object arg) && arg is OrderCart cart)
        {
            // If there are items in the order, record the order and ask for a room number.
            dc.ActiveDialog.State[Outputs.OrderCart] = cart;
            await dc.Prompt(Inputs.Number, "What is your room number?", new PromptOptions
            {
                RetryPromptString = "Please enter your room number."
            });
        }
        else
        {
            // Otherwise, assume the order was cancelled by the guest and exit.
            await dc.End();
        }
    },
    async (dc, args, next) =>
    {
        // Get and save the guest's answer.
        var roomNumber = args["Text"] as string;
        dc.ActiveDialog.State[Outputs.RoomNumber] = roomNumber;

        // Process the dinner order using the collected order cart and room number.

        await dc.Context.SendActivity($"Thank you. Your order will be delivered to room {roomNumber} within 45 minutes.");
        await dc.End();
    },
});
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

```javascript
// Order dinner:
// Help user order dinner from a menu
dialogs.add('orderDinner', [
    async function (dc){
        await dc.context.sendActivity("Welcome to our Dinner order service.");
        
        await dc.begin('orderPrompt', dc.activeDialog.state.orderCart); // Prompt for orders
    },
    async function (dc, result) {
        if(result == "Cancel"){
            await dc.end();
        }
        else { 
            await dc.prompt('numberPrompt', "What is your room number?");
        }
    },
    async function(dc, result){
        await dc.context.sendActivity(`Thank you. Your order will be delivered to room ${result} within 45 minutes.`);
        await dc.end();
    }
]);
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

在 **HotelDialog** 建構函式中，新增 [食物選取項目] 對話方塊。

* 在第一個步驟中，我們會初始化對話方塊狀態。 如果對話方塊的輸入引數包含購物車資訊，我們會將其儲存到對話方塊狀態中；否則，建立空的購物車，然後將其加入。 然後，我們會提示來賓從晚餐菜單中進行選擇。
* 在下一個步驟中，則看來賓挑選的選項：
  * 如果是處理訂單的要求，則檢查購物車是否包含任何項目。
    * 如果有，結束對話方塊，傳回購物車內容。
    * 如果沒有，傳送錯誤訊息給來賓，然後從對話方塊開頭重新開始。
  * 如果是要求取消訂單，則結束對話方塊，傳回空的字典。
  * 如果是晚餐項目，將其加入購物車、傳送狀態訊息，並啟動對話方塊，然後傳入目前的訂單狀態。

```csharp
// Add the food-selection dialog.
this.Add(Dialogs.OrderPrompt, new WaterfallStep[]
    {
        async (dc, args, next) =>
        {
            if (args is null || !args.ContainsKey(Outputs.OrderCart))
            {
                // First time through, initialize the order state.
                dc.ActiveDialog.State[Outputs.OrderCart] = new OrderCart();
                dc.ActiveDialog.State[Outputs.OrderTotal] = 0.0;
            }
            else
            {
                // Otherwise, set the order state to that of the arguments.
                dc.ActiveDialog.State = new Dictionary<string, object>(args);
            }

            await dc.Prompt(Inputs.Choice, "What would you like?", new ChoicePromptOptions
            {
                Choices = Lists.MenuChoices,
                RetryPromptActivity = Lists.MenuReprompt,
            });
        },
        async (dc, args, next) =>
        {
            // Get the guest's choice.
            var choice = (FoundChoice)args["Value"];
            var option = Lists.MenuOptions[choice.Index];

            // Get the current order from dialog state.
            var cart = (OrderCart)dc.ActiveDialog.State[Outputs.OrderCart];

            if (option.Name is MenuChoice.Process)
            {
                if (cart.Count > 0)
                {
                    // If there are any items in the order, then exit this dialog,
                    // and return the list of selected food items.
                    await dc.End(new Dictionary<string, object>
                    {
                        [Outputs.OrderCart] = cart
                    });
                }
                else
                {
                    // Otherwise, send an error message and restart from
                    // the beginning of this dialog.
                    await dc.Context.SendActivity(
                        "Your cart is empty. Please add at least one item to the cart.");
                    await dc.Replace(Dialogs.OrderPrompt);
                }
            }
            else if (option.Name is MenuChoice.Cancel)
            {
                await dc.Context.SendActivity("Your order has been cancelled.");

                // Exit this dialog, returning an empty property bag.
                dc.ActiveDialog.State.Clear();
                await dc.End(new Dictionary<string, object>());
            }
            else
            {
                // Add the selected food item to the order and update the order total.
                cart.Add(option);
                var total = (double)dc.ActiveDialog.State[Outputs.OrderTotal] + option.Price;
                dc.ActiveDialog.State[Outputs.OrderTotal] = total;

                await dc.Context.SendActivity($"Added {option.Name} (${option.Price:0.00}) to your order." +
                    Environment.NewLine + Environment.NewLine +
                    $"Your current total is ${total:0.00}.");

                // Present the order options again, passing in the current order state.
                await dc.Replace(Dialogs.OrderPrompt, dc.ActiveDialog.State);
            }
        },
    });
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

```javascript
// Helper dialog to repeatedly prompt user for orders
dialogs.add('orderPrompt', [
    async function(dc, orderCart){
        // Define a new cart of one does not exists
        if(!orderCart){
            // Initialize a new cart
            // convoState = conversationState.get(dc.context);
            dc.activeDialog.state.orderCart = {
                orders: [],
                total: 0
            };
        }
        else {
            dc.activeDialog.state.orderCart = orderCart;
        }
        await dc.prompt('choicePrompt', "What would you like?", dinnerMenu.choices);
    },
    async function(dc, choice){
        // Get state object
        // convoState = conversationState.get(dc.context);

        if(choice.value.match(/process order/ig)){
            if(dc.activeDialog.state.orderCart.orders.length > 0) {
                // Process the order
                // ...
                dc.activeDialog.state.orderCart = undefined; // Reset cart
                await dc.context.sendActivity("Processing your order.");
                await dc.end();
            }
            else {
                await dc.context.sendActivity("Your cart was empty. Please add at least one item to the cart.");
                // Ask again
                await dc.replace('orderPrompt');
            }
        }
        else if(choice.value.match(/cancel/ig)){
            //dc.activeDialog.state.orderCart = undefined; // Reset cart
            await dc.context.sendActivity("Your order has been canceled.");
            await dc.end(choice.value);
        }
        else {
            var item = dinnerMenu[choice.value];

            // Only proceed if user chooses an item from the menu
            if(!item){
                await dc.context.sendActivity("Sorry, that is not a valid item. Please pick one from the menu.");
                
                // Ask again
                await dc.replace('orderPrompt');
            }
            else {
                // Add the item to cart
                dc.activeDialog.state.orderCart.orders.push(item);
                dc.activeDialog.state.orderCart.total += item.Price;

                await dc.context.sendActivity(`Added to cart: ${choice.value}. <br/>Current total: $${dc.activeDialog.state.orderCart.total}`);

                // Ask again
                await dc.replace('orderPrompt', dc.activeDialog.state.orderCart);
            }
        }
    }
]);
```

上述範例程式碼會顯示主要的 `orderDinner` 對話使用名為 `orderPrompt` 的協助程式對話來處理使用者的選擇。 `orderPrompt` 對話方塊會顯示食物項目選單、要求使用者選擇項目、將項目新增至購物車，然後在迴圈中再次提示。 這可讓使用者將多個項目新增至他們的訂單。 對話會重複直到使用者選擇 `Process order` 或 `Cancel`。 到時候，執行權會交回給父對話方塊 (`orderDinner` 對話方塊)。 `orderDinner` 對話會進行一些最後關頭的整理，如果使用者想要處理訂單的話；否則它會結束，並將執行權傳回給其父對話 (例如 `mainMenu`)。 `mainMenu` 對話接著會繼續執行最後一個步驟，也就是單純重新顯示主功能表選項。

---
### <a name="create-the-reserve-table-dialog"></a>建立 [預訂餐廳] 對話方塊

<!-- TODO: Update the "Manage simple conversation flows" topic to use a reserveTable dialog, and then reference that from here. -->

為了讓此範例短一點，在這裡只示範 `reserveTable` 對話方塊的最低實作。

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

```csharp
// Add the table-reservation dialog.
this.Add(Dialogs.ReserveTable, new WaterfallStep[]
    {
        // Replace this waterfall with your reservation steps.
        async (dc, args, next) =>
        {
            await dc.Context.SendActivity("Your table has been reserved.");
            await dc.End();
        }
    });
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

```javascript
// Reserve a table:
// Help the user to reserve a table

dialogs.add('reserveTable', [
    // Replace this waterfall with your reservation steps.
    async function(dc, args, next){
        await dc.context.sendActivity("Your table has been reserved");
        await dc.end();
    }
]);
```

---

## <a name="next-steps"></a>後續步驟

您可以將其他選項 (如「詳細資訊」或「說明」) 提供到選單的選擇中，以增強這個 Bot。 如需實作這些中斷類型的詳細資訊，請參閱[處理使用者中斷](bot-builder-howto-handle-user-interrupt.md)。

既然您已了解如何使用對話方塊、提示和瀑布來管理複雜的對話流程，讓我們看看如何將對話方塊 (例如 `orderDinner` 和 `reserveTable` 對話方塊) 分成個別物件，而不是將它們全部堆在一個大型對話集中。

> [!div class="nextstepaction"]
> [使用複合控制項建立模組化 Bot 邏輯](bot-builder-compositcontrol.md)
