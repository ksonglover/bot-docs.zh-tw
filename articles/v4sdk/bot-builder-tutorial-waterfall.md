---
redirect_url: /bot-framework/bot-builder-prompts
ms.openlocfilehash: e7cfbad19290b3ef61d40dc90493db8f530a9a4e
ms.sourcegitcommit: 4661b9bb31d74731dbbb16e625be088b44ba5899
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 11/16/2018
ms.locfileid: "51826935"
---
<a name="--"></a><!--
---
標題：詢問使用者問題 | Microsoft Docs 描述：了解如何使用瀑布模型在 Bot Builder SDK 要求使用者輸入多項資料。
關鍵字：瀑布, 對話, 詢問問題, 提示 author: v-ducvo ms.author: v-ducvo manager: kamrani ms.topic: article ms.service: bot-service ms.subservice: sdk ms.date: 5/10/2018 monikerRange: 'azure-bot-service-4.0'
---

# <a name="ask-the-user-questions"></a>向使用者提出問題

[!INCLUDE [pre-release-label](../includes/pre-release-label.md)]

基本上，Bot 是根據與使用者的交談而建立的。 交談具有[多種形式](bot-builder-conversations.md)，可能簡短也可能複雜，可能是發問也可能是回答。 交談的形式取決於多種因素，不過所有因素皆與交談有關。

本教學課程會逐步引導您建立交談，第一步先從向可多次進行交談的 Bot 提出簡單問題開始。 我們以向餐廳訂位做為範例，不過您可以進一步想像，Bot 還可以透過進行交談執行多種工作，例如提交訂單、回答常見問題、進行預約等等。

互動式聊天 Bot 能夠回應使用者輸入或要求使用者輸入特定內容。 本教學課程將說明如何使用 `Dialogs` 中的 `Prompts` 程式庫，向使用者提出問題。 您可以將[對話方塊](../bot-service-design-conversation-flow.md)視為定義交談結構的容器，在[操作說明文章](bot-builder-prompts.md)中，您還可以深入了解對話方塊中的提示。

## <a name="prerequisite"></a>必要條件

本教學課程中的程式碼，會透過您在[開始使用](~/bot-service-quickstart.md)體驗中製作的**基本型 Bot** 加以建立。

## <a name="get-the-package"></a>取得套件

# <a name="ctabcstab"></a>[C#](#tab/cstab)

透過 Nuget 封包管理員安裝 **Microsoft.Bot.Builder.Dialogs** 套件。

# <a name="javascripttabjstab"></a>[JavaScript](#tab/jstab)
瀏覽至您的 Bot 專案資料夾，然後安裝來自 NPM 的 `botbuilder-dialogs` 套件：

```cmd
npm install --save botbuilder-dialogs@preview
```

---

## <a name="import-package-to-bot"></a>將套件匯入至 Bot

# <a name="ctabcstab"></a>[C#](#tab/cstab)

將參照同時加入您的 Bot 程式碼中的對話方塊和提示。

```cs
// ...
using Microsoft.Bot.Builder.Dialogs;
using Microsoft.Bot.Builder.Prompts;
// ...
```


# <a name="javascripttabjstab"></a>[JavaScript](#tab/jstab)

開啟 **app.js** 檔案，並在 Bot 程式碼中加入 `botbuilder-dialogs` 程式庫。

```javascript
const botbuilder_dialogs = require('botbuilder-dialogs');
```

---

如此，您就能存取可用於向使用者發問的 `DialogSet` 和 `Prompts` 程式庫。 `DialogSet` 是我們在**瀑布**模式中建構的對話方塊集合。 這只是表示一個接一個的對話方塊。

## <a name="instantiate-a-dialogs-object"></a>執行個體化對話方塊物件

執行個體化 `dialogs` 物件。 您必須使用此對話方塊物件來管理問答程序。

# <a name="ctabcstab"></a>[C#](#tab/cstab)
在您的 Bot 類別中宣告成員變數，並在 Bot 的建構函式中加以執行個體化。 
```cs
public class MyBot : IBot
{
    private readonly DialogSet dialogs;
    public MyBot()
    {
        dialogs = new DialogSet();
    }
    // The rest of the class definition is omitted here
}
```

# <a name="javascripttabjstab"></a>[JavaScript](#tab/jstab)

```javascript
const dialogs = new botbuilder_dialogs.DialogSet();
```
---

## <a name="define-a-waterfall-dialog"></a>定義瀑布對話方塊

若要發問，您至少需要一個兩步驟的**瀑布**對話方塊。 在此範例中，您將會建立一個兩步驟**瀑布**對話方塊，其中第一步是詢問使用者名稱，第二步則是按照該名稱來問候使用者。 

# <a name="ctabcstab"></a>[C#](#tab/cstab)

修改 Bot 的建構函式以新增對話方塊：
```csharp
public MyBot()
{
    dialogs = new DialogSet();
    dialogs.Add("greetings", new WaterfallStep[]
    {
        async (dc, args, next) =>
        {
            // Prompt for the guest's name.
            await dc.Prompt("textPrompt","What is your name?");
        },
        async(dc, args, next) =>
        {
            await dc.Context.SendActivity($"Hi {args["Text"]}!");
            await dc.End();
        }
    });
}
```

# <a name="javascripttabjstab"></a>[JavaScript](#tab/jstab)

```javascript
// Greet user:
// Ask for the user name and then greet them by name.
dialogs.add('greetings',[
    async function (dc){
        await dc.prompt('textPrompt', 'What is your name?');
    },
    async function(dc, results){
        var userName = results;
        await dc.context.sendActivity(`Hello ${userName}!`);
        await dc.end(); // Ends the dialog
    }
]);
```

---

問題須透過來自 `Prompts` 程式庫的 `textPrompt` 方式發問。 `Prompts` 程式庫提供一組提示，以便向使用者詢問各類資訊。 如需其他提示類型的詳細資訊，請參閱[提示使用者進行輸入](~/v4sdk/bot-builder-prompts.md)。

若要讓提示正常運作，您必須新增具有 dialogId `textPrompt` 的 `dialogs` 物件來新增提示，並藉由 `TextPrompt()` 建構函示加以建立。

# <a name="ctabcstab"></a>[C#](#tab/cstab)

```cs
public MyBot()
{
    dialogs = new DialogSet();
    dialogs.Add("greetings", new WaterfallStep[]
    {
        async (dc, args, next) =>
        {
            // Prompt for the guest's name.
            await dc.Prompt("textPrompt","What is your name?");
        },
        async(dc, args, next) =>
        {
            await dc.Context.SendActivity($"Hi {args["Text"]}!");
            await dc.End();
        }
    });
    // add the prompt, of type TextPrompt
    dialogs.Add("textPrompt", new Builder.Dialogs.TextPrompt());
}

```
使用者回答問題後，步驟 2 的參數 `args` 隨即出現回應。

# <a name="javascripttabjstab"></a>[JavaScript](#tab/jstab)

```javascript
dialogs.add('textPrompt', new botbuilder_dialogs.TextPrompt());
```
使用者回答問題後，步驟 2 的參數 `results` 隨即出現回應。 在此情況下，系統會將 `results` 指派給本機變數 `userName`。 在稍後的教學課程中，我們會說明如何將使用者的輸入內容留存在您選擇的儲存體中。

---


您已經定義了要發問的 `dialogs`，接下來必須呼叫對話方塊才能啟動提示程序。

## <a name="start-the-dialog"></a>啟動對話方塊

# <a name="ctabcstab"></a>[C#](#tab/cstab)

請依以下方式修改 Bot 邏輯：

```cs

public async Task OnTurn(ITurnContext context)
{
    // We'll cover state later, in the next tutorial
    var state = ConversationState<Dictionary<string, object>>.Get(context);
    var dc = dialogs.CreateContext(context, state);
    if (context.Activity.Type == ActivityTypes.Message)
    {
        await dc.Continue();
        
        if(!context.Responded)
        {
            await dc.Begin("greetings");
        }
    }
}
```

Bot 邏輯會存入 `OnTurn()` 方法中。 使用者只要說「您好」，Bot 隨即啟動 `greetings` 對話方塊。 `greetings` 對話方塊的第一步驟會提示使用者輸入自己的名稱。 使用者會傳送具有自己名稱的回覆做為訊息活動，其結果會透過 `dc.Continue()` 方法傳送至瀑布的第二步驟。 您所定義的瀑布的第二步驟，會按照使用者的名稱問候對方，並結束此對話方塊。 

# <a name="javascripttabjstab"></a>[JavaScript](#tab/jstab)

將您的**基本** Bot `processActivity()` 方法依照以下範例修改：

```javascript
// Listen for incoming activity 
server.post('/api/messages', (req, res) => {
    adapter.processActivity(req, res, async (context) => {
        const isMessage = (context.activity.type === 'message');
        // State will store all of your information 
        const convo = conversationState.get(context);
        const dc = dialogs.createContext(context, convo);

        if (isMessage) {
            // Check for valid intents
            if(context.activity.text.match(/hi/ig)){
                await dc.begin('greetings');
            }
        }

        if(!context.responded){
            // Continue executing the "current" dialog, if any.
            await dc.continue();

            if(!context.responded && isMessage){
                // Default message
                await context.sendActivity("Hi! I'm a simple bot. Please say 'Hi'.");
            }
        }
    });
});
```

Bot 邏輯會存入 `processActivity()` 方法中。 使用者只要說「您好」，Bot 隨即啟動 `greetings` 對話方塊。 `greetings` 對話方塊的第一步驟會提示使用者輸入自己的名稱。 使用者會在回覆中加入自己的名稱，當作活動的 `text` 訊息。 由於訊息與任何預期的意圖均不符，且 Bot 未傳送任何回覆給使用者，因此結果會透過 `dc.continue()` 方法傳送至瀑布的第二步驟。 您所定義的瀑布的第二步驟，會按照使用者的名稱問候對方，並結束此對話方塊。 舉例來說，若第二步驟並未傳送問候訊息給使用者，則 `processActivity` 方法就會傳送*預設訊息*給使用者來結束對話。

---



## <a name="define-a-more-complex-waterfall-dialog"></a>定義更複雜的瀑布對話方塊

我們已經說明瀑布對話方塊的運作和建立方式，接下來，我們要嘗試建立更複雜的對話方塊，並以完成餐廳訂位做為目標。

若要管理 餐廳訂位要求，您必須定義具有四個步驟的**瀑布**對話方塊。 在此交談中，除了 `TextPrompt` 之外，您也會使用 `DatetimePrompt` 和 `NumberPrompt`。



# <a name="ctabcstab"></a>[C#](#tab/cstab)

從使用 Echo Bot 範本開始，並將 Bot 重新命名為 CafeBot。 新增 `DialogSet` 和幾個靜態成員變數。

```cs

namespace CafeBot
{
    public class CafeBot : IBot
    {
        private readonly DialogSet dialogs;

        // Usually, we would save the dialog answers to our state object, which will be covered in a later tutorial.
        // For purpose of this example, let's use the three static variables to store our reservation information.
        static DateTime reservationDate;
        static int partySize;
        static string reservationName;

        // the rest of the class definition is omitted here
        // but is discussed in the rest of this article
    }
}
```

然後定義您的 `reserveTable` 對話方塊。 您可以在 Bot 類別建構函式中新增對話方塊。
```cs
public CafeBot()
{
    dialogs = new DialogSet();

    // Define our dialog
    dialogs.Add("reserveTable", new WaterfallStep[]
    {
        async (dc, args, next) =>
        {
            // Prompt for the guest's name.
            await dc.Context.SendActivity("Welcome to the reservation service.");

            await dc.Prompt("dateTimePrompt", "Please provide a reservation date and time.");
        },
        async(dc, args, next) =>
        {
            var dateTimeResult = ((DateTimeResult)args).Resolution.First();

            reservationDate = Convert.ToDateTime(dateTimeResult.Value);
            
            // Ask for next info
            await dc.Prompt("partySizePrompt", "How many people are in your party?");

        },
        async(dc, args, next) =>
        {
            partySize = (int)args["Value"];

            // Ask for next info
            await dc.Prompt("textPrompt", "Whose name will this be under?");
        },
        async(dc, args, next) =>
        {
            reservationName = args["Text"];
            string msg = "Reservation confirmed. Reservation details - " +
            $"\nDate/Time: {reservationDate.ToString()} " +
            $"\nParty size: {partySize.ToString()} " +
            $"\nReservation name: {reservationName}";
            await dc.Context.SendActivity(msg);
            await dc.End();
        }
    });

     // Add a prompt for the reservation date
     dialogs.Add("dateTimePrompt", new Microsoft.Bot.Builder.Dialogs.DateTimePrompt(Culture.English));
     // Add a prompt for the party size
     dialogs.Add("partySizePrompt", new Microsoft.Bot.Builder.Dialogs.NumberPrompt<int>(Culture.English));
     // Add a prompt for the user's name
     dialogs.Add("textPrompt", new Microsoft.Bot.Builder.Dialogs.TextPrompt());
}
```


# <a name="javascripttabjstab"></a>[JavaScript](#tab/jstab)

`reserveTable` 對話方塊如下所示：

```javascript
// Reserve a table:
// Help the user to reserve a table
var reservationInfo = {};

dialogs.add('reserveTable', [
    async function(dc, args, next){
        await dc.context.sendActivity("Welcome to the reservation service.");

        reservationInfo = {}; // Clears any previous data
        await dc.prompt('dateTimePrompt', "Please provide a reservation date and time.");
    },
    async function(dc, result){
        reservationInfo.dateTime = result[0].value;

        // Ask for next info
        await dc.prompt('partySizePrompt', "How many people are in your party?");
    },
    async function(dc, result){
        reservationInfo.partySize = result;

        // Ask for next info
        await dc.prompt('textPrompt', "Whose name will this be under?");
    },
    async function(dc, result){
        reservationInfo.reserveName = result;
        
        // Reservation confirmation
        var msg = `Reservation confirmed. Reservation details: 
            <br/>Date/Time: ${reservationInfo.dateTime} 
            <br/>Party size: ${reservationInfo.partySize} 
            <br/>Reservation name: ${reservationInfo.reserveName}`;
        await dc.context.sendActivity(msg);
        await dc.end();
    }
]);
```

---

`reserveTable` 對話方塊的交談流程，會透過瀑布的 3 個步驟向使用者詢問 3 個問題。 步驟四則會處理最後一個問題的答案，並將訂位確認訊息傳送給使用者。



# <a name="ctabcstab"></a>[C#](#tab/cstab)
`reserveTable` 對話方塊的每個瀑布步驟都會利用提示要求使用者提供資訊。 以下程式碼可在對話方塊集中新增提示。

```cs
dialogs.Add("dateTimePrompt", new Microsoft.Bot.Builder.Dialogs.DateTimePrompt(Culture.English));
dialogs.Add("partySizePrompt", new Microsoft.Bot.Builder.Dialogs.NumberPrompt<int>(Culture.English));
dialogs.Add("textPrompt", new Microsoft.Bot.Builder.Dialogs.TextPrompt());
```

# <a name="javascripttabjstab"></a>[JavaScript](#tab/jstab)

若要讓此瀑布開始運作，您還必須為 `dialogs` 物件新增以下提示：

```javascript
// Define prompts
// Generic prompts
dialogs.add('textPrompt', new botbuilder_dialogs.TextPrompt());
dialogs.add('dateTimePrompt', new botbuilder_dialogs.DatetimePrompt());
dialogs.add('partySizePrompt', new botbuilder_dialogs.NumberPrompt());
```

---

現在，您已經可以將這個提示連接到 Bot 邏輯中。

## <a name="start-the-dialog"></a>啟動對話方塊

# <a name="ctabcstab"></a>[C#](#tab/cstab)
修改 Bot 的 `OnTurn` 以加入下列程式碼：
```cs
public async Task OnTurn(ITurnContext context)
{
    if (context.Activity.Type == ActivityTypes.Message)
    {
        // The type parameter PropertyBag inherits from 
        // Dictionary<string,object>
        var state = ConversationState<Dictionary<string, object>>.Get(context);
        var dc = dialogs.CreateContext(context, state);
        await dc.Continue();

        // Additional logic can be added to enter each dialog depending on the message received
        
        if(!context.Responded)
        {
            if (context.Activity.Text.ToLowerInvariant().Contains("reserve table"))
            {
                await dc.Begin("reserveTable");
            }
            else
            {
                await context.SendActivity($"You said '{context.Activity.Text}'");
            }
        }
    }
}
```


在 **Startup.cs** 中，將 ConversationState 中介軟體的初始化作業變更為使用衍生自 `Dictionary<string,object>` (而非衍生自 `EchoState`) 的類別。

例如，在 `Configure()` 中：
```cs
options.Middleware.Add(new ConversationState<Dictionary<string, object>>(dataStore));
```


# <a name="javascripttabjstab"></a>[JavaScript](#tab/jstab)

若要掌握使用者提出這項要求的意圖，請在 `processActivity()` 方法中加入幾行程式碼。 將您 Bot 的 `processActivity()` 方法依照以下範例修改：

```javascript
// Listen for incoming activity 
server.post('/api/messages', (req, res) => {
    adapter.processActivity(req, res, async (context) => {
        const isMessage = (context.activity.type === 'message');
        // State will store all of your information 
        const convo = conversationState.get(context);
        const dc = dialogs.createContext(context, convo);

        if (isMessage) {
            // Check for valid intents
            if(context.activity.text.match(/hi/ig)){
                await dc.begin('greetings');
            }
            else if(context.activity.text.match(/reserve table/ig)){
                await dc.begin('reserveTable');
            }
        }

        if(!context.responded){
            // Continue executing the "current" dialog, if any.
            await dc.continue();

            if(!context.responded && isMessage){
                // Default message
                await context.sendActivity("Hi! I'm a simple bot. Please say 'Hi' or 'Reserve table'.");
            }
        }
    });
});
```

在執行時，每當使用者傳送包含字串 `reserve table` 的訊息，Bot 就會啟動 `reserveTable` 交談。

---



## <a name="next-steps"></a>後續步驟

在本教學課程中， Bot 會將使用者的輸入內容儲存在 Bot 的變數中。 如果想要儲存或留存此資訊，您必須將狀態新增至中介軟體層。 在下一個教學課程中，我們會進一步探討如何留存使用者狀態資料。 

> [!div class="nextstepaction"]
> [留存使用者資料](bot-builder-tutorial-persist-user-inputs.md)

-->
