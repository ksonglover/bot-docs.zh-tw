---
title: 使用對話管理對話流程 | Microsoft Docs
description: 了解如何在適用於 Node.js 的 Bot Builder SDK 中，使用對話來管理對話流程。
keywords: 對話流程, 對話, 提示, 瀑布, 對話集合
author: v-ducvo
ms.author: v-ducvo
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 5/8/2018
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 99184ba71072c159c598c7f68289c42a51926795
ms.sourcegitcommit: f576981342fb3361216675815714e24281e20ddf
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/18/2018
ms.locfileid: "39299367"
---
# <a name="manage-conversation-flow-with-dialogs"></a>使用對話管理對話流程
[!INCLUDE [pre-release-label](../includes/pre-release-label.md)]


管理對話流程是建置 Bot 的必要工作。 如果您使用 Bot Builder SDK，則可使用**對話**來管理對話流程。

對話類似於程式中的函式。 它通常是設計來執行特定作業，並且可視需要隨時叫用它。 您可以將多個對話鏈結在一起，只處理您想要 Bot 處理的任何對話流程。 Bot Builder SDK 的 **dialogs** 程式庫包含**提示**和**瀑布**之類的內建功能，可協助您透過對話來管理對話流程。 prompts 程式庫提供各種提示，您可以用來要求使用者提供不同類型的資訊。 瀑布可讓您將多個步驟結合在序列中。

本文將說明如何建立 dialogs 物件，並新增提示和瀑布步驟到對話集合裡，來管理簡單的對話流程和複雜的對話流程。 

## <a name="install-the-dialogs-library"></a>安裝 dialogs 程式庫

# <a name="ctabcsharp"></a>[C#](#tab/csharp)
若要使用對話，請為專案或解決方案安裝 `Microsoft.Bot.Builder.Dialogs` NuGet 套件。
然後在程式碼檔案中使用陳述式參考 dialogs 程式庫。 例如︰

```csharp
using Microsoft.Bot.Builder.Dialogs;
```

# <a name="javascripttabjs"></a>[JavaScript](#tab/js)
可以從 NPM 下載 `botbuilder-dialogs` 程式庫。 若要安裝 `botbuilder-dialogs` 程式庫，請執行下列 NPM 命令：

```cmd
npm install --save botbuilder-dialogs
```

若要在 Bot 中使用 **dialogs**，請將它包含在 Bot 程式碼中。 例如︰

**app.js**

```javascript
const botbuilder_dialogs = require('botbuilder-dialogs');
```
---

## <a name="create-a-dialog-stack"></a>建立對話堆疊

若要使用對話，您必須先建立「對話集合」。

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

`Microsoft.Bot.Builder.Dialogs` 程式庫提供 `DialogSet` 類別。
對於對話集合，您可以新增具名的對話及對話集合，之後再依名稱存取它們。

```csharp
IDialog dialog = null;
// Initialize dialog.

DialogSet dialogs = new DialogSet();
dialogs.Add("dialog name", dialog);
```

# <a name="javascripttabjs"></a>[JavaScript](#tab/js)

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

對話名稱 (例如 `addTwoNumbers`) 在每個對話集合內必須是唯一的。 您可以視需要在每一個集合中定義多個對話。

對話程式庫會定義下列對話：
-   **提示**對話，對話使用至少兩個函式，一個用來提示使用者輸入，而另一個用來處理輸入。
    您可以使用**瀑布**模型將這些串在一起。
-   **瀑布**對話定義一系列的_瀑布步驟_，它們會依序執行。
    瀑布對話可以有單一步驟，在此情況下它可以視為簡單的單一步驟對話。

## <a name="create-a-single-step-dialog"></a>建立單一步驟對話

單一步驟對話可用來擷取單一回合的對話流程。 這個範例會建立一個 Bot，偵測使用者是否說了類似「1 + 2」的話，然後開始 `addTwoNumbers` 對話，回覆「1 + 2 = 3」。 

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

值會以 `IDictionary<string,object>` 屬性包的方式傳入對話，再從對話傳回。

若要在對話集合內建立簡單的對話，請使用 `Add` 方法。 以下會新增名為 `addTwoNumbers` 的單一步驟瀑布。

此步驟會假設被傳入的對話引數包含 `first` 和 `second` 屬性，代表要相加的數字。

從 EchoBot 範本著手。 然後在您的 Bot 類別新增程式碼，將對話新增至建構函式中。
```csharp
public class EchoBot : IBot
{
    private DialogSet _dialogs;

    public EchoBot()
    {
        _dialogs = new DialogSet();
        _dialogs.Add("addTwoNumbers", new WaterfallStep[]
        {              
            async (dc, args, next) =>
            {
                double sum = (double)args["first"] + (double)args["second"];
                await dc.Context.SendActivity($"{args["first"]} + {args["second"]} = {sum}");
                await dc.End();
            }
        });
    }

    // The rest of the class definition is omitted here but would include OnTurn()
}

```

### <a name="pass-arguments-to-the-dialog"></a>將引數傳遞至對話

若要在 Bot 的 `OnTurn` 方法內呼叫對話，請修改 `OnTurn` 以包含下列內容：
```cs
public async Task OnTurn(ITurnContext context)
{
    // This bot is only handling Messages
    if (context.Activity.Type == ActivityTypes.Message)
    {
        // Get the conversation state from the turn context
        var state = context.GetConversationState<EchoState>();

        // create a dialog context
        var dialogCtx = _dialogs.CreateContext(context, state);

        // Bump the turn count. 
        state.TurnCount++;

        await dialogCtx.Continue();
        if (!context.Responded)
        {
            // Call a helper function that identifies if the user says something 
            // like "2 + 3" or "1.25 + 3.28" and extract the numbers to add            
            if (TryParseAddingTwoNumbers(context.Activity.Text, out double first, out double second))
            { 
                var dialogArgs = new Dictionary<string, object>
                {
                    ["first"] = first,
                    ["second"] = second
                };                        
                await dialogCtx.Begin("addTwoNumbers", dialogArgs);
            }
            else
            {
                // Echo back to the user whatever they typed.
                await context.SendActivity($"Turn: {state.TurnCount}. You said '{context.Activity.Text}'");
            }
        }
    }
}
```

將協助程式函式新增至 Bot 類別。 協助程式函式只有使用簡單的規則運算式來偵測使用者的訊息是否為將 2 個數字相加的要求。

```cs
// Recognizes if the message is a request to add 2 numbers, in the form: number + number, 
// where number may have optionally have a decimal point.: 1 + 1, 123.99 + 45, 0.4+7. 
// For the sake of simplicity it doesn't handle negative numbers or numbers like 1,000 that contain a comma.
// If you need more robust number recognition, try System.Recognizers.Text
public bool TryParseAddingTwoNumbers(string message, out double first, out double second)
{
    // captures a number with optional -/+ and optional decimal portion
    const string NUMBER_REGEXP = "([-+]?(?:[0-9]+(?:\\.[0-9]+)?|\\.[0-9]+))";
    // matches the plus sign with optional spaces before and after it
    const string PLUSSIGN_REGEXP = "(?:\\s*)\\+(?:\\s*)";
    const string ADD_TWO_NUMBERS_REGEXP = NUMBER_REGEXP + PLUSSIGN_REGEXP + NUMBER_REGEXP;
    var regex = new Regex(ADD_TWO_NUMBERS_REGEXP);
    var matches = regex.Matches(message);
    var succeeded = false;
    first = 0;
    second = 0;
    if (matches.Count == 0)
    {
        succeeded = false;
    }
    else
    {
        var matched = matches[0];
        if ( System.Double.TryParse(matched.Groups[1].Value, out first) 
            && System.Double.TryParse(matched.Groups[2].Value, out second))
        {
            succeeded = true;
        } 
    }
    return succeeded;
}
```

如果您正在使用 EchoBot 範本，請修改 **EchoState.cs** 中的 `EchoState` 類別，如下所示：

```cs
/// <summary>
/// Class for storing conversation state.
/// This bot only stores the turn count in order to echo it to the user
/// </summary>
public class EchoState: Dictionary<string, object>
{
    private const string TurnCountKey = "TurnCount";
    public EchoState()
    {
        this[TurnCountKey] = 0;            
    }

    public int TurnCount
    {
        get { return (int)this[TurnCountKey]; }
        set { this[TurnCountKey] = value; }
    }
}
```

# <a name="javascripttabjs"></a>[JavaScript](#tab/js)

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
        if (isMessage) {
            const state = conversationState.get(context);
            const count = state.count === undefined ? state.count = 0 : ++state.count;

            // create a dialog context
            const dc = dialogs.createContext(context, state);

            // MatchesAdd2Numbers checks if the message matches a regular expression
            // and if it does, returns an array of the numbers to add
            var numbers = await MatchesAdd2Numbers(context.activity.text); 
            if (numbers != null && numbers.length >=2 )
            {    
                await dc.begin('addTwoNumbers', numbers);
            }
            else {
                // Just echo back the user's message if they're not adding numbers
                return context.sendActivity(`Turn ${count}: You said "${context.activity.text}"`); 
            }           
        } else {
            return context.sendActivity(`[${context.activity.type} event detected]`);
        }
        if (!context.responded) {
            await dc.continue();
            // if the dialog didn't send a response
            if (!context.responded && isMessage) {
                await dc.context.sendActivity(`Hi! I'm the add 2 numbers bot. Say something like "what's 1+2?"`);
            }
        }
    });
});
```

將協助程式函式新增至 **app.js**。 協助程式函式只有使用簡單的規則運算式來偵測使用者的訊息是否為將 2 個數字相加的要求。 如果規則運算式相符，它會傳回陣列，其中包含要相加的數字。

```javascript
async function MatchesAdd2Numbers(message) {
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

請嘗試在 Bot Framework Emulator 中執行 Bot，並對它說出「1+1 等於多少」等字詞。 to it.

![執行 Bot](./media/how-to-dialogs/bot-output-add-numbers.png)



## <a name="using-dialogs-to-guide-the-user-through-steps"></a>使用對話引導使用者執行步驟

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

### <a name="create-a-composite-dialog"></a>建立複合對話

下列程式碼片段取自 botbuilder-dotnet 存放庫中的 [Microsoft.Bot.Samples.Dialog.Prompts](https://github.com/Microsoft/botbuilder-dotnet/tree/master/samples/MIcrosoft.Bot.Samples.Dialog.Prompts) 程式碼範例。

在 Startup.cs 中：
1.  將 Bot 重新命名為 `DialogContainerBot`。
1.  使用簡單的字典作為 Bot 對話狀態的屬性包。

```csharp
public void ConfigureServices(IServiceCollection services)
{
    services.AddBot<DialogContainerBot>(options =>
    {
        options.CredentialProvider = new ConfigurationCredentialProvider(Configuration);
        options.Middleware.Add(new ConversationState<Dictionary<string, object>>(new MemoryStorage()));
    });
}
```

將 `EchoBot` 重新命名為 `DialogContainerBot`。

在 `DialogContainerBot.cs` 中，定義設定檔對話的類別。

```csharp
public class ProfileControl : DialogContainer
{
    public ProfileControl()
        : base("fillProfile")
    {
        Dialogs.Add("fillProfile", 
            new WaterfallStep[]
            {
                async (dc, args, next) =>
                {
                    dc.ActiveDialog.State = new Dictionary<string, object>();
                    await dc.Prompt("textPrompt", "What's your name?");
                },
                async (dc, args, next) =>
                {
                    dc.ActiveDialog.State["name"] = args["Value"];
                    await dc.Prompt("textPrompt", "What's your phone number?");
                },
                async (dc, args, next) =>
                {
                    dc.ActiveDialog.State["phone"] = args["Value"];
                    await dc.End(dc.ActiveDialog.State);
                }
            }
        );
        Dialogs.Add("textPrompt", new Builder.Dialogs.TextPrompt());
    }
}
```

然後，在 Bot 定義內，宣告 Bot 主要對話的欄位，並在 Bot 的建構函式中加以初始化。
Bot 的主要對話包含設定檔對話。

```csharp
private DialogSet _dialogs;

public DialogContainerBot()
{
    _dialogs = new DialogSet();

    _dialogs.Add("getProfile", new ProfileControl());
    _dialogs.Add("firstRun",
        new WaterfallStep[]
        {
            async (dc, args, next) =>
            {
                    await dc.Context.SendActivity("Welcome! We need to ask a few questions to get started.");
                    await dc.Begin("getProfile");
            },
            async (dc, args, next) =>
            {
                await dc.Context.SendActivity($"Thanks {args["name"]} I have your phone number as {args["phone"]}!");
                await dc.End();
            }
        }
    );
}
```

在 bot 的 `OnTurn` 方法中：
-   在對話開始時，問候使用者。
-   每當我們從使用者取得訊息時，初始化並「繼續」主要對話。

    如果對話未產生回應，假設它已提早完成或尚未開始，然後「開始」它，並指定集合中要開始的對話名稱。

```csharp
public async Task OnTurn(ITurnContext turnContext)
{
    try
    {
        switch (turnContext.Activity.Type)
        {
            case ActivityTypes.ConversationUpdate:
                foreach (var newMember in turnContext.Activity.MembersAdded)
                {
                    if (newMember.Id != turnContext.Activity.Recipient.Id)
                    {
                        await turnContext.SendActivity("Hello and welcome to the Composite Control bot.");
                    }
                }
                break;

            case ActivityTypes.Message:
                var state = ConversationState<Dictionary<string, object>>.Get(turnContext);
                var dc = _dialogs.CreateContext(turnContext, state);

                await dc.Continue();

                if (!turnContext.Responded)
                {
                    await dc.Begin("firstRun");
                }

                break;
        }
    }
    catch (Exception e)
    {
        await turnContext.SendActivity($"Exception: {e.Message}");
    }
}

```

# <a name="javascripttabjs"></a>[JavaScript](#tab/js)

### <a name="create-a-dialog-with-waterfall-steps"></a>使用瀑布步驟建立對話

對話包含使用者與 Bot 之間交換的一系列訊息。 當 Bot 的目標是引導使用者完成一系列的步驟時，您可以使用**瀑布**模型來定義對話的步驟。

**瀑布**是特定的對話實作，最常用來向使用者收集資訊或引導使用者完成一系列的工作。 這些工作會實作為函式陣列，其中第一個函式的結果將作為引數傳遞至下一個函式，依此類推。 每個函式通常代表整個程序的其中一個步驟。 在每個步驟中，Bot 會[提示](bot-builder-prompts.md)使用者提供輸入、等候回應，然後將結果傳遞給下一個步驟。

例如，下列程式碼範例會定義陣列中的三個函式，代表**瀑布**的三個步驟：

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

**瀑布**步驟的特徵標記如下所示：

| 參數 | 說明 |
| ---- | ----- |
| `context` | 對話內容。 |
| `args` | 選擇性，它包含傳入步驟的引數。 |
| `next` | 選擇性，讓您繼續進行瀑布下一步驟的方法。 當您呼叫此方法時，可以提供 *args* 引數，這樣可讓您將引數傳遞至瀑布的下一個步驟。 |

每個步驟必須在傳回之前呼叫下列其中一個方法：*next()*、*dialogs.prompt()*、*dialogs.end()*、*dialogs.begin()* 或 *Promise.resolve()*，否則 Bot 會卡在該步驟。 也就是說，如果函式不會傳回其中一個方法，則所有的使用者輸入會導致每次使用者傳送訊息給 Bot 時便重新執行此步驟。

當您到達瀑布尾端時，最佳做法是使用 `end()` 方法傳回，讓對話可以從堆疊中取出。 如需詳細資訊，請參閱[結束對話](#end-a-dialog)一節。 同樣地，要一個步驟前往下一個步驟時，瀑布步驟必須以提示字元結束，或是明確地呼叫 `next()` 函式以推進瀑布。 

### <a name="start-a-dialog"></a>開始對話

若要開始對話，請將您要開始的 *dialogId* 傳遞到 `begin()`、`prompt()` 或 `replace()` 方法。 **begin** 方法會將對話推上堆疊的頂端，而 **replace** 方法會將目前對話從堆疊取出，並將取代的對話推送至堆疊上。

開始對話而不使用引數：

```javascript
// Start the 'greetings' dialog.
await dc.begin('greetings');
```

開始對話並使用引數：

```javascript
// Start the 'greetings' dialog with the 'userName' passed in. 
await dc.begin('greetings', userName);
```

開始**提示**對話：

```javascript
// Start a 'choicePrompt' dialog with choices passed in as an array of colors to choose from.
await dc.prompt('choicePrompt', `choice: select a color`, ['red', 'green', 'blue']);
```

根據您要開始的提示類型，提示的引數特徵標記可能會不同。 **DialogSet.prompt** 方法是一個協助程式方法。 這個方法會接受引數，並建構適當的提示選項，然後它會呼叫 **begin** 方法來開始提示對話。

取代堆疊上的對話：

```javascript
// End the current dialog and start the 'mainMenu' dialog.
await dc.replace('mainMenu'); // Can optionally passed in an 'args' as the second argument.
```

如何使用 **replace()** 方法的更多詳細資料位於下面的[重複對話](#repeat-a-dialog)和[對話迴圈](#dialog-loops)節。

## <a name="end-a-dialog"></a>結束對話

將對話從堆疊取出並選擇性地傳回結果給父對話，可以結束對話。 父對話會以任何傳回的結果，叫用其 **Dialog.resume()** 方法。

最佳做法是在對話結尾明確地呼叫 `end()` 方法，不過這並非必要；因為當您到達瀑布結尾時，對話會自動會從堆疊中取出。

結束對話：

```javascript
// End the current dialog by popping it off the stack
await dc.end();
```

結束對話並將選擇性的引數傳遞給父對話：

```javascript
// End the current dialog and pass a result to the parent dialog
await dc.end(result);
```

或者，您也可以傳回已解析的承諾來結束對話：

```javascript
await Promise.resolve();
```

呼叫 `Promise.resolve()` 會導致對話結束，並從堆疊取出。 不過，這個方法不會呼叫父對話繼續執行。 呼叫 `Promise.resolve()` 之後，執行會停止，且 Bot 將會從使用者傳送訊息給 Bot 時父對話離開的地方繼續。 這可能不是理想的對話結束使用者體驗。 請考慮使用 `end()` 或 `replace()` 結束對話，讓您的 Bot 可以繼續與使用者互動。

### <a name="clear-the-dialog-stack"></a>清除對話堆疊

如果您想要將所有對話從堆疊取出，您可以藉由呼叫 `dc.endAll()` 方法清除對話堆疊。

```javascript
// Pop all dialogs from the current stack.
await dc.endAll();
```

### <a name="repeat-a-dialog"></a>重複對話

若要重複對話，請使用 `dialogs.replace()` 方法。

```javascript
// End the current dialog and start the 'mainMenu' dialog.
await dc.replace('mainMenu'); 
```

如果您想要預設顯示主功能表，可使用下列步驟建立 `mainMenu` 對話：

```javascript
// Display a menu and ask user to choose a menu item. Direct user to the item selected.
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
        else {
            // Repeat the menu
            await dc.replace('mainMenu');
        }
    },
    async function(dc, result){
        // Start over
        await dc.endAll().begin('mainMenu');
    }
]);

dialogs.add('choicePrompt', new botbuilder_dialogs.ChoicePrompt());
```

這個對話會使用 `ChoicePrompt` 顯示功能表，然後等候使用者選擇選項。 當使用者選擇 `Order Dinner` 或 `Reserve a table`　時，它會開始適當選擇的對話；該工作完成時，此對話會自行重複，而不是只在最後一個步驟結束對話。

### <a name="dialog-loops"></a>對話迴圈

使用 `replace()`　方法的另一種方式是藉由模擬迴圈。 以此案例為例。 如果您想要允許使用者將多個功能表項目新增至購物車，可以重複功能表選項，直到使用者完成訂購。

```javascript
// Order dinner:
// Help user order dinner from a menu

var dinnerMenu = {
    choices: ["Potato Salad - $5.99", "Tuna Sandwich - $6.89", "Clam Chowder - $4.50", 
        "More info", "Process order", "Cancel", "Help"],
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

// The order cart
var orderCart = {
    orders: [],
    total: 0,
    clear: function(dc) {
        this.orders = [];
        this.total = 0;
        dc.context.activity.conversation.orderCart = null;
    }
};

dialogs.add('orderDinner', [
    async function (dc){
        await dc.context.sendActivity("Welcome to our Dinner order service.");
        orderCart.clear(dc); // Clears the cart.

        await dc.begin('orderPrompt'); // Prompt for orders
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

// Helper dialog to repeatedly prompt user for orders
dialogs.add('orderPrompt', [
    async function(dc){
        await dc.prompt('choicePrompt', "What would you like?", dinnerMenu.choices);
    },
    async function(dc, choice){
        if(choice.value.match(/process order/ig)){
            if(orderCart.orders.length > 0) {
                // Process the order
                // ...
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

// Define prompts
// Generic prompts
dialogs.add('textPrompt', new botbuilder_dialogs.TextPrompt());
dialogs.add('numberPrompt', new botbuilder_dialogs.NumberPrompt());
dialogs.add('dateTimePrompt', new botbuilder_dialogs.DatetimePrompt());
dialogs.add('choicePrompt', new botbuilder_dialogs.ChoicePrompt());

```

上述範例程式碼會顯示主要的 `orderDinner` 對話使用名為 `orderPrompt` 的協助程式對話來處理使用者的選擇。 `orderPrompt` 對話會顯示功能表、要求使用者選擇項目、將項目新增至購物車，然後再次提示。 這可讓使用者將多個項目新增至他們的訂單。 對話會重複直到使用者選擇 `Process order` 或 `Cancel`。 到時候，執行權會交回給父對話 (例如 `orderDinner`)。 `orderDinner` 對話會進行一些最後關頭的整理，如果使用者想要處理訂單的話；否則它會結束，並將執行權傳回給其父對話 (例如 `mainMenu`)。 `mainMenu` 對話接著會繼續執行最後一個步驟，也就是單純重新顯示主功能表選項。

---

## <a name="next-steps"></a>後續步驟

既然您了解如何使用**對話**、**提示**和**瀑布**來管理對話流程，讓我們看看如何可以將對話分成模組化的工作，而不是將它們全部一起堆在主要 Bot 邏輯的 `dialogs` 物件。

> [!div class="nextstepaction"]
> [使用複合控制項建立模組化 Bot 邏輯](bot-builder-compositcontrol.md)