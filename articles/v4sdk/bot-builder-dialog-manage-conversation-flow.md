---
title: 使用對話方塊來管理簡單對話流程 | Microsoft Docs
description: 了解如何在適用於 Node.js 的 Bot 建立器 SDK 中，使用對話方塊來管理簡單對話流程。
keywords: 簡單對話流程, 對話方塊, 提示, 瀑布, 對話方塊集合
author: v-ducvo
ms.author: v-ducvo
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 8/2/2018
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 77162601f542e6faa8908bc71abc971eb99fcc93
ms.sourcegitcommit: 1abc32353c20acd103e0383121db21b705e5eec3
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/21/2018
ms.locfileid: "42756568"
---
# <a name="manage-simple-conversation-flow-with-dialogs"></a>使用對話方塊來管理簡單對話流程

[!INCLUDE [pre-release-label](../includes/pre-release-label.md)]

您可以使用對話方塊程式庫管理簡單和複雜的對話流程。 在簡單對話流程中，使用者從「瀑布」的第一個步驟開始，持續進行到最後一個步驟，對話式交換即完成。 對話方塊也可以處理[複雜對話流程](~/v4sdk/bot-builder-dialog-manage-complex-conversation-flow.md)，其中部分的對話方塊可以分支和執行迴圈。

<!-- TODO: We need a dialogs conceptual topic to link to, so we can reference that here, in place of describing what they are and what their features are in a how-to topic. -->

<!-- TODO: This paragraph belongs in a conceptual topic. --> 對話方塊類似於程式中的函式。 其設計訴求通常是依照特定順序執行特定作業，並可視需要隨時叫用。 使用對話方塊可讓 Bot 開發人員引導對話流程。 您可以將多個對話 (dialog) 鏈結在一起，以處理幾乎任何您想要讓 Bot 處理的對話 (conversation) 流程。 Bot 建立器 SDK 的 **Dialogs** 程式庫包含「提示」和「瀑布圖對話方塊」之類的內建功能，可協助您管理對話流程。 您可以使用提示來要求使用者提供不同類型的資訊。 您可以使用瀑布圖將多個步驟結合在序列中。

在本文中，我們會使用「對話方塊集合」來建立對話流程，其中包含提示和瀑布圖步驟。 我們有兩個範例對話方塊。 第一個是單一步驟對話方塊，可執行不需要使用者輸入的作業。 第二個是多步驟對話方塊，可提示使用者輸入某些資訊。

## <a name="install-the-dialogs-library"></a>安裝 dialogs 程式庫

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
npm install --save botbuilder-dialogs@preview
```

若要在 Bot 中使用**對話方塊**，請將其包含在 **app.js** 檔案中。

```javascript
const botbuilder_dialogs = require('botbuilder-dialogs');
```

---

## <a name="create-a-dialog-stack"></a>建立對話堆疊

在第一個範例中，我們將建立單一步驟對話方塊，其可將兩個數字加在一起並顯示結果。

若要使用對話，您必須先建立「對話集合」。

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

`Microsoft.Bot.Builder.Dialogs` 程式庫提供 `DialogSet` 類別。
建立 **AdditionDialog** 類別，並新增所需的 using 陳述式。
您可以將具名的對話方塊和幾組對話方塊新增至對話集合，之後再依其名稱存取。

```csharp
using Microsoft.Bot.Builder.Dialogs;
```

從 **DialogSet** 衍生類別，並定義識別碼和金鑰，用來識別此對話方塊集的對話方塊和輸入資訊。

```csharp
/// <summary>Defines a simple dialog for adding two numbers together.</summary>
public class AdditionDialog : DialogSet
{
    /// <summary>The ID of the main dialog in the set.</summary>
    public const string Main = "additionDialog";

    /// <summary>Defines the IDs of the input arguments.</summary>
    public struct Inputs
    {
        public const string First = "first";
        public const string Second = "second";
    }
}
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

`botbuilder-dialogs` 程式庫提供 `DialogSet` 類別。
**DialogSet** 類別會定義**對話堆疊**，並提供您簡單的介面來管理堆疊。
Bot Builder SDK 將堆疊實作為陣列。

若要建立 **DialogSet**，請執行下列動作：

```javascript
const dialogs = new botbuilder_dialogs.DialogSet();
```

上述呼叫會建立 **DialogSet**，其具有預設**對話堆疊**，名為 `dialogStack`。
如果您想要命名您的堆疊，可以將它傳入作為 **DialogSet()** 的參數。 例如︰

```javascript
const dialogs = new botbuilder_dialogs.DialogSet("myStack");
```

---

建立對話只會將對話定義新增至集合。 在使用 _begin_ 或 _replace_ 方法將對話推入至對話堆疊之前，不會執行對話。

對話名稱 (例如 `addTwoNumbers`) 在每個對話集合內必須是唯一的。 您可以視需要在每一個集合中定義多個對話。 如果您想要建立多個對話方塊集合，並讓其一起密切運作，請參閱[ 建立模組化 Bot 邏輯](bot-builder-compositcontrol.md)。

對話程式庫會定義下列對話：

* **提示**對話，對話使用至少兩個函式，一個用來提示使用者輸入，而另一個用來處理輸入。 您可以使用**瀑布**模型將這些串在一起。
* **瀑布**對話定義一系列的_瀑布步驟_，它們會依序執行。 瀑布對話可以有單一步驟，在此情況下它可以視為簡單的單一步驟對話。

## <a name="create-a-single-step-dialog"></a>建立單一步驟對話

單一步驟對話可用來擷取單一回合的對話流程。 這個範例會建立一個 Bot，偵測使用者是否說了類似「1 + 2」的話，然後開始 `addTwoNumbers` 對話，回覆「1 + 2 = 3」。

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

值會以 `IDictionary<string,object>` 屬性包的方式傳入對話，再從對話傳回。

若要在對話集合內建立簡單的對話，請使用 `Add` 方法。 以下會新增名為 `addTwoNumbers` 的單一步驟瀑布。

此步驟會假設被傳入的對話引數包含 `first` 和 `second` 屬性，代表要相加的數字。

將下列建構函式新增到 **AdditionDialog** 類別。

```csharp
/// <summary>Defines the steps of the dialog.</summary>
public AdditionDialog()
{
    Add(Main, new WaterfallStep[]
    {
        async (dc, args, next) =>
        {
            // Get the input from the arguments to the dialog and add them.
            var x =(double)args[Inputs.First];
            var y =(double)args[Inputs.Second];
            var sum = x + y;

            // Display the result to the user.
            await dc.Context.SendActivity($"{x} + {y} = {sum}");

            // End the dialog.
            await dc.End();
        }
    });
}
```

### <a name="pass-arguments-to-the-dialog"></a>將引數傳遞至對話

在您的 Bot 程式碼中，更新您的 using 陳述式。

```cs
using Microsoft.Bot;
using Microsoft.Bot.Builder;
using Microsoft.Bot.Builder.Core.Extensions;
using Microsoft.Bot.Schema;
using System.Collections.Generic;
using System.Text.RegularExpressions;
using System.Threading.Tasks;
```

將靜態屬性新增至新增對話方塊的類別。

```cs
private static AdditionDialog AddTwoNumbers { get; } = new AdditionDialog();
```

若要在 Bot 的 `OnTurn` 方法內呼叫對話，請修改 `OnTurn` 以包含下列內容：

```cs
public async Task OnTurn(ITurnContext context)
{
    // Handle any message activity from the user.
    if (context.Activity.Type is ActivityTypes.Message)
    {
        // Get the conversation state from the turn context.
        var conversationState = context.GetConversationState<ConversationData>();

        // Generate a dialog context for the addition dialog.
        var dc = AddTwoNumbers.CreateContext(context, conversationState.DialogState);

        // Call a helper function that identifies if the user says something
        // like "2 + 3" or "1.25 + 3.28" and extract the numbers to add.
        if (TryParseAddingTwoNumbers(context.Activity.Text, out double first, out double second))
        {
            // Start the dialog, passing in the numbers to add.
            var args = new Dictionary<string, object>
            {
                [AdditionDialog.Inputs.First] = first,
                [AdditionDialog.Inputs.Second] = second
            };
            await dc.Begin(AdditionDialog.Main, args);
        }
        else
        {
            // Echo back to the user whatever they typed.
            await context.SendActivity($"You said '{context.Activity.Text}'");
        }
    }
}
```

將 **TryParseAddingTwoNumbers** 協助程式函式新增至 Bot 類別。 協助程式函式只有使用簡單的規則運算式來偵測使用者的訊息是否為將 2 個數字相加的要求。

```cs
// Recognizes if the message is a request to add 2 numbers, in the form: number + number,
// where number may have optionally have a decimal point.: 1 + 1, 123.99 + 45, 0.4+7.
// For the sake of simplicity it doesn't handle negative numbers or numbers like 1,000 that contain a comma.
// If you need more robust number recognition, try System.Recognizers.Text
public static bool TryParseAddingTwoNumbers(string message, out double first, out double second)
{
    // captures a number with optional -/+ and optional decimal portion
    const string NUMBER_REGEXP = "([-+]?(?:[0-9]+(?:\\.[0-9]+)?|\\.[0-9]+))";

    // matches the plus sign with optional spaces before and after it
    const string PLUSSIGN_REGEXP = "(?:\\s*)\\+(?:\\s*)";

    const string ADD_TWO_NUMBERS_REGEXP = NUMBER_REGEXP + PLUSSIGN_REGEXP + NUMBER_REGEXP;

    var regex = new Regex(ADD_TWO_NUMBERS_REGEXP);
    var matches = regex.Matches(message);

    first = 0;
    second = 0;
    if (matches.Count > 0)
    {
        var matched = matches[0];
        if (double.TryParse(matched.Groups[1].Value, out first)
            && double.TryParse(matched.Groups[2].Value, out second))
        {
            return true;
        }
    }
    return false;
}
```

如果您使用 EchoBot 範本，請將 **EchoState** 類別的名稱變更為 **ConversationData**，並加以修改以包含下列項目。

```cs
using System.Collections.Generic;

/// <summary>
/// Class for storing conversation state.
/// </summary>
public class ConversationData
{
    /// <summary>Property for storing dialog state.</summary>
    public Dictionary<string, object> DialogState { get; set; } = new Dictionary<string, object>();
}
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

從[使用 Bot Builder SDK v4 建立 Bot](../javascript/bot-builder-javascript-quickstart.md) 中所述的 JS 範本著手。 在 **app.js** 中，新增陳述式來要求 `botbuilder-dialogs`。

```js
const {DialogSet} = require('botbuilder-dialogs');
```

在 **app.js** 中，新增下列程式碼來定義名為 `addTwoNumbers` 且屬於 `dialogs` 集合的簡單對話：

```javascript
const dialogs = new DialogSet("myDialogStack");

// Show the sum of two numbers.
dialogs.add('addTwoNumbers', [async function (dc, numbers){
        var sum = Number.parseFloat(numbers[0]) + Number.parseFloat(numbers[1]);
        await dc.context.sendActivity(`${numbers[0]} + ${numbers[1]} = ${sum}`);
        await dc.end();
    }]
);
```

將 **app.js** 中處理內送活動的程式碼，取代為下列程式碼。 Bot 會呼叫協助程式函式，來檢查內送訊息是否看起來像是將兩個數字相加的要求。 如果是，它會將引數中的數字傳遞給 `DialogContext.begin`。

```js
// Listen for incoming activity 
server.post('/api/messages', (req, res) => {
    // Route received activity to adapter for processing
    adapter.processActivity(req, res, async (context) => {
        const isMessage = context.activity.type === 'message';
        // State will store all of your information
        const convoState = conversationState.get(context);
        const dc = dialogs.createContext(context, convoState);

        if (isMessage) {
            // TryParseAddingTwoNumbers checks if the message matches a regular expression
            // and if it does, returns an array of the numbers to add
            var numbers = await TryParseAddingTwoNumbers(context.activity.text); 
            if (numbers != null && numbers.length >=2 )
            {
                await dc.begin('addTwoNumbers', numbers);
            }
            else {
                // Just echo back the user's message if they're not adding numbers
                const count = (convoState.count === undefined ? convoState.count = 0 : ++convoState.count);
                return context.sendActivity(`Turn ${count}: You said "${context.activity.text}"`);
            }
        }
        else {
            return context.sendActivity(`[${context.activity.type} event detected]`);
        }

        if (!context.responded) {
            await dc.continue();
            // if the dialog didn't send a response
            if (!context.responded && isMessage) {
                await dc.context.sendActivity(`Hi! I'm the add 2 numbers bot. Say something like "What's 2+3?"`);
            }
        }
    });
});

```

將協助程式函式新增至 **app.js**。 協助程式函式只有使用簡單的規則運算式來偵測使用者的訊息是否為將 2 個數字相加的要求。 如果規則運算式相符，它會傳回陣列，其中包含要相加的數字。

```javascript
async function TryParseAddingTwoNumbers(message) {
    const ADD_NUMBERS_REGEXP = /([-+]?(?:[0-9]+(?:\.[0-9]+)?|\.[0-9]+))(?:\s*)\+(?:\s*)([-+]?(?:[0-9]+(?:\.[0-9]+)?|\.[0-9]+))/i;
    let matched = ADD_NUMBERS_REGEXP.exec(message);
    if (!matched) {
        // message wasn't a request to add 2 numbers
        return null;
    }
    else {
        var numbers = [matched[1], matched[2]];
        return numbers;
    }
}
```

---

### <a name="run-the-bot"></a>執行 Bot

請嘗試在 Bot Framework Emulator 中執行 Bot，並說出「1+1 等於多少」等 字詞。

![執行 Bot](./media/how-to-dialogs/bot-output-add-numbers.png)

## <a name="using-dialogs-to-guide-the-user-through-steps"></a>使用對話引導使用者執行步驟

在下一個範例中，我們會建立多步驟對話方塊以提示使用者輸入資訊。

### <a name="create-a-dialog-with-waterfall-steps"></a>使用瀑布步驟建立對話

**瀑布**是特定的對話實作，最常用來向使用者收集資訊或引導使用者完成一系列的工作。 這些工作會實作為函式陣列，其中第一個函式的結果將作為引數傳遞至下一個函式，依此類推。 每個函式通常代表整個程序的其中一個步驟。 在每個步驟中，Bot 會[提示](bot-builder-prompts.md)使用者提供輸入、等候回應，然後將結果傳遞給下一個步驟。

例如，下列程式碼範例會定義陣列中的三個函式，代表**瀑布**的三個步驟。 在每個提示之後，Bot 會認可使用者的輸入，但不會儲存輸入。 如果您想要保存使用者輸入，請參閱[保存使用者資料](bot-builder-tutorial-persist-user-inputs.md)以取得詳細資訊。

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

這會顯示問候語對話方塊的建構函式，其中的 **GreetingDialog** 衍生自 **DialogSet**、**Inputs.Text** 包含我們用於 **TextPrompt** 物件的識別碼，而 **Main** 包含問候語對話本身的識別碼。

```csharp
public GreetingDialog()
{
    // Include a text prompt.
    Add(Inputs.Text, new TextPrompt());

    // Define the dialog logic for greeting the user.
    Add(Main, new WaterfallStep[]
    {
        async (dc, args, next) =>
        {
            // Ask for their name.
            await dc.Prompt(Inputs.Text, "What is your name?");
        },
        async (dc, args, next) =>
        {
            // Get the prompt result.
            var name = args["Text"] as string;

            // Acknowledge their input.
            await dc.Context.SendActivity($"Hi, {name}!");

            // Ask where they work.
            await dc.Prompt(Inputs.Text, "Where do you work?");
        },
        async (dc, args, next) =>
        {
            // Get the prompt result.
            var work = args["Text"] as string;

            // Acknowledge their input.
            await dc.Context.SendActivity($"{work} is a fun place.");

            // End the dialog.
            await dc.End();
        }
    });
}
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

```javascript
// Greet user:
// Ask for the user name and then greet them by name.
// Ask them where they work.
dialogs.add('greetings',[
    async function (dc){
        await dc.prompt('textPrompt', 'What is your name?');
    },
    async function(dc, results){
        var userName = results;
        await dc.context.sendActivity(`Hi ${userName}!`);
        await dc.prompt('textPrompt', 'Where do you work?');
    },
    async function(dc, results){
        var workPlace = results;
        await dc.context.sendActivity(`${workPlace} is a fun place.`);
        await dc.end(); // Ends the dialog
    }
]);

dialogs.add('textPrompt', new botbuilder_dialogs.TextPrompt());
```

---

**瀑布**步驟的特徵標記如下所示：

| 參數 | 說明 |
| :---- | :----- |
| `dc` | 對話內容。 |
| `args` | 選擇性，包含傳入步驟的引數。 |
| `next` | 選擇性，可讓您未經提示繼續進行瀑布下一步的方法。 您可以在呼叫此方法時提供 *args* 引數，這可讓您將引數傳遞至瀑布的下一個步驟。 |

每個步驟必須在傳回之前呼叫下列其中一個方法：*next()* 委派或其中一個對話方塊內容方法 *begin*, *end*、*prompt* 或 *replace*，否則 Bot 會卡在該步驟。 也就是說，如果函式並未透過其中一個方法完成，則所有的使用者輸入會導致每次使用者傳送訊息給 Bot 時，便重新執行此步驟。

當您到達瀑布尾端時，最佳做法是使用 _end_ 方法傳回，讓對話方塊可以從堆疊中取出。 如需詳細資訊，請參閱下面的[結束對話](#end-a-dialog)一節。 同樣地，從一個步驟前往下一個步驟時，瀑布步驟必須以提示字元結束，或是明確地呼叫 _next_ 委派以推進瀑布。

## <a name="start-a-dialog"></a>開始對話

若要開始對話方塊，請將您要開始的 *dialogId* 傳遞到對話方塊的 _begin_、_prompt_ 或 _replace_ 方法。 _begin_ 方法會將對話方塊推上堆疊的頂端，而 _replace_ 方法會將目前對話方塊從堆疊取出，並將取代的對話方塊推送至堆疊上。

開始對話而不使用引數：

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

```csharp
// Start the greetings dialog.
await dc.Begin("greetings");
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

```javascript
// Start the 'greetings' dialog.
await dc.begin('greetings');
```

---

開始對話並使用引數：

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

```csharp
// Start the greetings dialog, passing in a property bag.
await dc.Begin("greetings", args);
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

```javascript
// Start the 'greetings' dialog with the 'userName' passed in.
await dc.begin('greetings', userName);
```

---

開始**提示**對話：

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

在此，**Inputs.Text** 包含同一個對話方塊集合中 **TextPrompt** 的識別碼。

```csharp
// Ask a user for their name.
await dc.Prompt(Inputs.Text, "What is your name?");
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

```javascript
// Ask a user for their name.
await dc.prompt('textPrompt', "What is your name?");
```

---

根據您要開始的提示類型，提示的引數特徵標記可能會不同。 **DialogSet.prompt** 方法是一個協助程式方法。 這個方法會接受引數，並建構適當的提示選項，然後它會呼叫 **begin** 方法來開始提示對話。 如需提示的詳細資訊，請參閱[提示使用者進行輸入](bot-builder-prompts.md)。

## <a name="end-a-dialog"></a>結束對話

_end_ 方法可將對話方塊從堆疊取出，並將選擇性結果傳回給父代對話方塊，以結束對話方塊。

最佳做法是在對話方塊結尾明確地呼叫 _end_ 方法，不過這並非必要；因為當您到達瀑布結尾時，對話方塊會自動會從堆疊中取出。

結束對話：

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

```csharp
// End the current dialog by popping it off the stack.
await dc.End();
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

```javascript
// End the current dialog by popping it off the stack
await dc.end();
```

---

若要結束對話方塊並將資訊傳回給父代對話方塊，請包含屬性包引數。

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

```csharp
// End the current dialog and return information to the parent dialog.
await dc.end(new Dictionary<string, object>
    {
        ["property1"] = value1,
        ["property2"] = value2
    });
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

```javascript
// End the current dialog and pass a result to the parent dialog
await dc.end({
    "property1": value1,
    "property2": value2
});
```

---

## <a name="clear-the-dialog-stack"></a>清除對話堆疊

如果您想要將所有對話方塊從堆疊取出，您可以藉由呼叫 _end all_ 方法清除對話方塊堆疊。

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

```csharp
// Pop all dialogs from the current stack.
await dc.EndAll();
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

```javascript
// Pop all dialogs from the current stack.
await dc.endAll();
```

---

## <a name="repeat-a-dialog"></a>重複對話

若要重複對話方塊，請使用 _replace_ 方法。 對話方塊內容的 *replace* 方法會將目前對話方塊從堆疊取出，並將取代的對話方塊推送至堆疊上，開始進行該對話方塊。 這很適合用來處理[複雜對話流程](~/v4sdk/bot-builder-dialog-manage-complex-conversation-flow.md)，也是管理功能表的好方法。

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

```csharp
// End the current dialog and start the main menu dialog.
await dc.Replace("mainMenu");
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

```javascript
// End the current dialog and start the 'mainMenu' dialog.
await dc.replace('mainMenu');
```

---

## <a name="next-steps"></a>後續步驟

您現在已了解如何管理簡單對話流程，讓我們看看如何運用 _replace_ 方法來處理複雜對話流程。

> [!div class="nextstepaction"]
> [管理複雜交談流程](bot-builder-dialog-manage-complex-conversation-flow.md)
