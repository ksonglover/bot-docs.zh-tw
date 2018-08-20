---
title: 使用對話方塊容器建立模組化 Bot 邏輯 | Microsoft Docs
description: 了解如何在適用於 Node.js 的 Bot 建立器 SDK 和 C# 中，使用對話方塊容器將 Bot 邏輯模組化。
keywords: 複合控制項, 模組化 Bot 邏輯
author: v-ducvo
ms.author: v-ducvo
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 4/27/2018
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 2441a32167618ebb08e6a43d68d74076c3351d8f
ms.sourcegitcommit: f576981342fb3361216675815714e24281e20ddf
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/18/2018
ms.locfileid: "39300226"
---
# <a name="create-modular-bot-logic-with-a-dialog-container"></a>使用對話方塊容器建立模組化 Bot 邏輯

[!INCLUDE [pre-release-label](../includes/pre-release-label.md)]

假設您正在建立可處理多項工作的旅館 Bot，例如問候使用者、預訂晚餐座位、點菜、設定鬧鐘、顯示目前天氣等等。 您可以在 Bot 中使用一個對話方塊物件來處理各項工作，但這可能會讓 Bot 主要檔案中的對話方塊程式碼太大而且雜亂。

您可以透過模組化解決此問題。 模組化可簡化您的程式碼，也更容易對其進行偵錯。 此外，您也可以將它分成數個區段，讓多個小組同時處理 Bot。 我們可以透過使用對話方塊容器將 Bot 拆解成元件，來建立可管理多個對話流程的 Bot。 我們會建立幾個基本的對話流程，並示範如何使用對話方塊容器將它們結合在一起。

在此範例中，我們會建立旅館 Bot，結合入住、叫醒及訂位模組。

## <a name="managing-state"></a>管理狀態

有許多方法可針對使用複合對話方塊的 Bot 設定狀態管理。 在這個 Bot 中，我們會示範其中一個方法。

如需狀態管理的詳細資訊，請參閱[使用對話和使用者屬性來儲存狀態](bot-builder-howto-v4-state.md)。

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

每個對話方塊都會收集某些資訊，我們會將這些資訊儲存到使用者狀態中。 我們會為每個對話方塊定義類別，並在使用者狀態類別中使用這些類別作為屬性。

```csharp
/// <summary>
/// State information associated with the check-in dialog.
/// </summary>
public class GuestInfo
{
    public string Name { get; set; }
    public string Room { get; set; }
}

/// <summary>
/// State information associated with the reserve-table dialog.
/// </summary>
public class TableInfo
{
    public string Number { get; set; }
}

/// <summary>
/// State information associated with the wake-up call dialog.
/// </summary>
public class WakeUpInfo
{
    public string Time { get; set; }
}

/// <summary>
/// User state information.
/// </summary>
public class UserInfo
{
    public GuestInfo Guest { get; set; }
    public TableInfo Table { get; set; }
    public WakeUpInfo WakeUp { get; set; }
}
```

在 Bot 回合中，對話方塊集合的 `CreateContext` 方法會建立對話方塊狀態。
這個方法會將回合內容和狀態物件當做參數。

對於對話方塊，這個狀態物件必須實作 `IDictionary<string, object>` 介面。 於這個 Bot 只會使用對話狀態來容納對話方塊狀態，因此對話狀態類別可以是簡單的字典。

```csharp
using System.Collections.Generic;

/// <summary>
/// Conversation state information.
/// We are also using this directly for dialog state, which needs an <see cref="IDictionary{string, object}"/> object.
/// </summary>
public class ConversationInfo : Dictionary<string, object> { }
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

為了追蹤使用者的輸入，我們會從父代對話方塊傳入 `userState` 物件作為對話方塊參數。 在每個對話方塊類別中，建構函式會內建對話方塊，讓您將資訊儲存到 `userState`。 在這整個對話方塊中，我們可以在 `dc.activeDialog.state` 物件上寫入定義為屬性的本機狀態物件，作為使用者輸入資訊。 對話方塊完成之後，會清除本機狀態物件。 因此，我們會將本機狀態物件儲存至父代的 `userState`，這可在您與使用者的所有對話中保存使用者的相關資訊。 

---

## <a name="define-a-modular-check-in-dialog"></a>定義模組化入住對話方塊

首先，從簡單的入住對話方塊開始，詢問使用者姓名以及要住宿的房間。 若要模組化這項工作，我們需建立可延伸 `DialogContainer` 的 `CheckIn` 類別。 這個類別具有可定義根對話方塊名稱的建構函式，其定義為三步驟的「瀑布圖」。 對話方塊物件的簽章和建構與標準瀑布圖相同。

**入住對話步驟**

1. 詢問來賓姓名。
1. 詢問來賓想要住宿的房間。
1. 傳送確認訊息，完成對話方塊。

如需對話方塊和瀑布圖的詳細資訊，請參閱[使用對話方塊來管理對話流程](bot-builder-dialog-manage-conversation-flow.md)。

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

`CheckIn` 類別具有私用建構函式，可定義入住對話的步驟、建立單一執行個體，並在靜態 `Instance` 屬性中公開。

在這整個對話方塊中，我們可以寫入本機狀態物件，其可透過對話方塊內容 `dc.ActiveDialog.State` 的屬性來存取。 對話方塊完成之後，會清除本機狀態物件。 因此，我們會將本機狀態物件儲存至 Bot 的 `userState`，這可在您與使用者的所有對話中保存使用者的相關資訊。

如需狀態管理的詳細資訊，請參閱[使用對話和使用者屬性來儲存狀態](bot-builder-howto-v4-state.md)。 

**CheckIn.cs**
```csharp
using Microsoft.Bot.Builder.Core.Extensions;
using Microsoft.Bot.Builder.Dialogs;
using Microsoft.Recognizers.Text;

namespace HotelBot
{
    public class CheckIn : DialogContainer
    {
        public const string Id = "checkIn";

        private const string GuestKey = nameof(CheckIn);

        public static CheckIn Instance { get; } = new CheckIn();

        // You can start this from the parent using the dialog's ID.
        private CheckIn() : base(Id)
        {
            // Define the conversation flow using a waterfall model.
            this.Dialogs.Add(Id, new WaterfallStep[]
            {
                async (dc, args, next) =>
                {
                    // Clear the guest information and prompt for the guest's name.
                    dc.ActiveDialog.State[GuestKey] = new GuestInfo();
                    await dc.Prompt("textPrompt", "What is your name?");
                },
                async (dc, args, next) =>
                {
                    // Save the name and prompt for the room number.
                    var name = args["Value"] as string;

                    var guestInfo = dc.ActiveDialog.State[GuestKey];
                    ((GuestInfo)guestInfo).Name = name;

                    await dc.Prompt("numberPrompt",
                        $"Hi {name}. What room will you be staying in?");
                },
                async (dc, args, next) =>
                {
                    // Save the room number and "sign off".
                    var room = (string)args["Text"];

                    var guestInfo = dc.ActiveDialog.State[GuestKey];
                    ((GuestInfo)guestInfo).Room = room;

                    await dc.Context.SendActivity("Great, enjoy your stay!");

                    // Save dialog state to user state and end the dialog.
                    var userState = UserState<UserInfo>.Get(dc.Context);
                    userState.Guest = (GuestInfo)guestInfo;

                    await dc.End();
                }
            });

            // Define the prompts used in this conversation flow.
            this.Dialogs.Add("textPrompt", new TextPrompt());
            this.Dialogs.Add("numberPrompt", new NumberPrompt<int>(Culture.English));
        }
    }
}
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

**checkIn.js**
```JavaScript
const { DialogContainer, DialogSet, TextPrompt, NumberPrompt } = require('botbuilder-dialogs');

class CheckIn extends DialogContainer {
    constructor(userState) {
        // Dialog ID of 'checkIn' will start when class is called in the parent
        super('checkIn');

        // Defining the conversation flow using a waterfall model
        this.dialogs.add('checkIn', [
            async function (dc) {
                // Create a new local guestInfo state object
                dc.activeDialog.state.guestInfo = {};
                await dc.context.sendActivity("What is your name?");
            },
            async function (dc, name){
                // Save the name 
                dc.activeDialog.state.guestInfo.userName = name;
                await dc.prompt('numberPrompt', `Hi ${name}. What room will you be staying in?`);
            },
            async function (dc, room){
                // Save the room number
                dc.activeDialog.state.guestInfo.room = room
                await dc.context.sendActivity(`Great! Enjoy your stay!`);

                // Save dialog's state object to the parent's state object
                const user = userState.get(dc.context);
                user.guestInfo = dc.activeDialog.state.guestInfo;
                await dc.end();
            }
        ]);
        // Defining the prompt used in this conversation flow
        this.dialogs.add('textPrompt', new TextPrompt());
        this.dialogs.add('numberPrompt', new NumberPrompt());
    }
}
exports.CheckIn = CheckIn;
```

---

## <a name="define-modular-reserve-table-and-wake-up-dialogs"></a>定義模組化的訂位及叫醒對話方塊

使用對話方塊容器的其中一個優點是能夠一起撰寫對話方塊。 由於每個 `DialogSet` 都會維護一組獨特的 `dialogs`，共用或交叉參考 `dialogs` 無法輕鬆完成。 這就是對話方塊容器發揮功用之處。 您可以使用對話方塊容器來建立複合對話方塊，讓跨對話方塊管理對話流程更加容易。 為了方便說明，讓我們再建立兩個對話方塊：一個用來詢問使用者晚餐想要預訂的桌位，另一個用來建立叫醒服務。 同樣地，我們會為每個對話方塊使用個別的類別，而每個對話方塊都可延伸 `DialogContainer`。

**訂位對話方塊步驟**

1. 詢問要預訂的桌位。
1. 傳送確認訊息，完成對話方塊。

**叫醒對話方塊步驟**

1. 詢問要為使用者設定的叫醒時間。
1. 傳送確認訊息，完成對話方塊。

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

**ReserveTable.cs**
```csharp
using Microsoft.Bot.Builder.Core.Extensions;
using Microsoft.Bot.Builder.Dialogs;
using Microsoft.Recognizers.Text;
using System.Linq;
using Choice = Microsoft.Bot.Builder.Prompts.Choices.Choice;
using FoundChoice = Microsoft.Bot.Builder.Prompts.Choices.FoundChoice;

namespace HotelBot
{
    public class ReserveTable : DialogContainer
    {
        public const string Id = "reserveTable";

        private const string TableKey = "Table";

        public static ReserveTable Instance { get; } = new ReserveTable();

        private ReserveTable() : base(Id)
        {
            this.Dialogs.Add(Id, new WaterfallStep[]
            {
                async (dc, args, next) =>
                {
                    // Clear the table information and prompt for the table number.
                    dc.ActiveDialog.State[TableKey] = new TableInfo();

                    var guestInfo = UserState<UserInfo>.Get(dc.Context).Guest;

                    var prompt = $"Welcome {guestInfo.Name}, " +
                        $"which table would you like to reserve?";
                    var choices = new string[] { "1", "2", "3", "4", "5", "6" };
                    await dc.Prompt("choicePrompt", prompt,
                        new ChoicePromptOptions
                        {
                            Choices = choices.Select(s => new Choice { Value = s }).ToList()
                        });
                },
                async (dc, args, next) =>
                {
                    // Save the table number and "sign off".
                    var table = (args["Value"] as FoundChoice).Value;

                    var tableInfo = dc.ActiveDialog.State[TableKey];
                    ((TableInfo)tableInfo).Number = table;

                    await dc.Context.SendActivity(
                        $"Sounds great; we will reserve table number {table} for you.");

                    // Save dialog state to user state and end the dialog.
                    var userState = UserState<UserInfo>.Get(dc.Context);
                    userState.Table = (TableInfo)tableInfo;

                    await dc.End();
                }
            });

            // Define the prompts used in this conversation flow.
            this.Dialogs.Add("choicePrompt", new ChoicePrompt(Culture.English));
        }
    }
}
```

**WakeUp.cs**
```csharp
using Microsoft.Bot.Builder.Core.Extensions;
using Microsoft.Bot.Builder.Dialogs;
using Microsoft.Recognizers.Text;
using System.Collections.Generic;
using System.Linq;
using DateTimeResolution = Microsoft.Bot.Builder.Prompts.DateTimeResult.DateTimeResolution;

namespace HotelBot
{
    public class WakeUp : DialogContainer
    {
        public const string Id = "wakeUp";

        private const string WakeUpKey = "WakeUp";

        public static WakeUp Instance { get; } = new WakeUp();

        private WakeUp() : base(Id)
        {
            this.Dialogs.Add(Id, new WaterfallStep[]
            {
                async (dc, args, next) =>
                {
                    // Clear the wake up call information and prompt for the alarm time.
                   dc.ActiveDialog.State[WakeUpKey] = new WakeUpInfo();

                    var guestInfo = UserState<UserInfo>.Get(dc.Context).Guest;

                    await dc.Prompt("datePrompt", $"Hi {guestInfo.Name}, " +
                        $"what time would you like your alarm set for?");
                },
                async (dc, args, next) =>
                {
                    // Save the alarm time and "sign off".
                    var resolution = (List<DateTimeResolution>)args["Resolution"];

                    var wakeUp = dc.ActiveDialog.State[WakeUpKey];
                    ((WakeUpInfo)wakeUp).Time = resolution?.FirstOrDefault()?.Value;

                    var userState = UserState<UserInfo>.Get(dc.Context);
                    await dc.Context.SendActivity(
                        $"Your alarm is set to {((WakeUpInfo)wakeUp).Time}" +
                        $" for room {userState.Guest.Room}.");

                    // Save dialog state to user state and end the dialog.
                    userState.WakeUp = (WakeUpInfo)wakeUp;

                    await dc.End();
                }
            });

            // Define the prompts used in this conversation flow.
            this.Dialogs.Add("datePrompt", new DateTimePrompt(Culture.English));
        }
    }
}
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

**reserveTable.js**
```JavaScript
const { DialogContainer, DialogSet, ChoicePrompt } = require('botbuilder-dialogs');

class ReserveTable extends DialogContainer {
    constructor(userState) {
        // Dialog ID of 'reserve_table' will start when class is called in the parent
        super('reserve_table'); 

        // Defining the conversation flow using a waterfall model
        this.dialogs.add('reserve_table', [
            async function (dc, args) {
                // Get the user state from context
                const user = userState.get(dc.context);

                // Create a new local reserveTable state object
                dc.activeDialog.state.reserveTable = {};

                const prompt = `Welcome ${user.guestInfo.userName}, which table would you like to reserve?`;
                const choices = ['1', '2', '3', '4', '5', '6'];
                await dc.prompt('choicePrompt', prompt, choices);
            },
            async function(dc, choice){
                // Save the table number
                dc.activeDialog.state.reserveTable.tableNumber = choice.value;
                await dc.context.sendActivity(`Sounds great, we will reserve table number ${choice.value} for you.`);
                
                // Get the user state from context
                const user = userState.get(dc.context);
                // Persist dialog's state object to the parent's state object
                user.reserveTable = dc.activeDialog.state.reserveTable;

                // End the dialog
                await dc.end();
            }
        ]);

        // Defining the prompt used in this conversation flow
        this.dialogs.add('choicePrompt', new ChoicePrompt());
    }
}
exports.ReserveTable = ReserveTable;
```

**wakeUp.js**
```JavaScript
const { DialogContainer, DialogSet, DatetimePrompt } = require('botbuilder-dialogs');

class WakeUp extends DialogContainer {
    constructor(userState) {
        // Dialog ID of 'wakeup' will start when class is called in the parent
        super('wakeUp');

        this.dialogs.add('wakeUp', [
            async function (dc, args) {
                // Get the user state from context
                const user = userState.get(dc.context); 

                // Create a new local reserveTable state object
                dc.activeDialog.state.wakeUp = {};  
                             
                await dc.prompt('datePrompt', `Hello, ${user.guestInfo.userName}. What time would you like your alarm to be set?`);
            },
            async function (dc, time){
                // Get the user state from context
                const user = userState.get(dc.context);

                // Save the time
                dc.activeDialog.state.wakeUp.time = time[0].value

                await dc.context.sendActivity(`Your alarm is set to ${time[0].value} for room ${user.guestInfo.room}`);
                
                // Save dialog's state object to the parent's state object
                user.wakeUp = dc.activeDialog.state.wakeUp;

                // End the dialog
                await dc.end();
            }]);

        // Defining the prompt used in this conversation flow
        this.dialogs.add('datePrompt', new DatetimePrompt());
    }
}
exports.WakeUp = WakeUp;
```

---

## <a name="add-modular-dialogs-to-a-bot"></a>新增模組化對話方塊到 Bot 中

主要的 Bot 檔案會將這三個模組化對話方塊繫結至一個 Bot。

- 三個對話方塊都已新增至 Bot 的主對話方塊集合中。
- 訂位和叫醒對話已內建至主對話方塊的對話流程中。
- 開始新對話時，我們沒有使用中的對話方塊，因此會使用 Bot 的 on turn 邏輯。

**Bot 的 on-turn 處理常式**

每當 Bot 處於使用中對話方塊之外，就會檢查使用者狀態。
1. 如果它已擁有來賓的資訊，則會啟動主要對話方塊。
1. 否則，就會啟動主要對話方塊的入住子對話方塊。

**主要對話方塊步驟**

1. 詢問來賓想要執行的動作：訂位或設定叫醒服務。
1. 啟動對應的子對話方塊，或傳送_無法辨識輸入_訊息，並跳到下一個步驟。
1. 跳回此對話方塊的開頭。


# <a name="ctabcsharp"></a>[C#](#tab/csharp)

系統會使用對話方塊內容的 `Continue` 方法來更新對話方塊流程。 這個方法會執行對話方塊堆疊上瀑布圖的下一個步驟。

**HotelBot.cs**
```csharp
using System.Collections.Generic;
using System.Threading.Tasks;
using Microsoft.Bot;
using Microsoft.Bot.Builder;
using Microsoft.Bot.Builder.Core.Extensions;
using Microsoft.Bot.Builder.Dialogs;
using Microsoft.Bot.Schema;

namespace HotelBot
{
    public class HotelBot : IBot
    {
        private const string MainMenuId = "mainMenu";

        private DialogSet _dialogs { get; } = ComposeMainDialog();

        /// <summary>
        /// Every Conversation turn for our bot calls this method. 
        /// </summary>
        /// <param name="context">The current turn context.</param>        
        public async Task OnTurn(ITurnContext context)
        {
            if (context.Activity.Type is ActivityTypes.Message)
            {
                // Get the user and conversation state from the turn context.
                var userInfo = UserState<UserInfo>.Get(context);
                var conversationInfo = ConversationState<ConversationInfo>.Get(context);

                // Establish dialog state from the conversation state.
                var dc = _dialogs.CreateContext(context, conversationInfo);

                // Continue any current dialog.
                await dc.Continue();

                // Every turn sends a response, so if no response was sent,
                // then there no dialog is currently active.
                if (!context.Responded)
                {
                    // If we don't yet have the guest's info, start the check-in dialog.
                    if (string.IsNullOrEmpty(userInfo?.Guest?.Name))
                    {
                        await dc.Begin(CheckIn.Id);
                    }
                    // Otherwise, start our bot's main dialog.
                    else
                    {
                        await dc.Begin(MainMenuId);
                    }
                }
            }
        }

        /// <summary>
        /// Composes a main dialog for our bot.
        /// </summary>
        /// <returns>A new main dialog.</returns>
        private static DialogSet ComposeMainDialog()
        {
            var dialogs = new DialogSet();

            dialogs.Add(MainMenuId, new WaterfallStep[]
            {
                async (dc, args, next) =>
                {
                    var menu = new List<string> { "Reserve Table", "Wake Up" };
                    await dc.Context.SendActivity(MessageFactory.SuggestedActions(menu, "How can I help you?"));
                },
                async (dc, args, next) =>
                {
                    // Decide which dialog to start.
                    var result = (args["Activity"] as Activity)?.Text?.Trim().ToLowerInvariant();
                    switch (result)
                    {
                        case "reserve table":
                            await dc.Begin(ReserveTable.Id);
                            break;
                        case "wake up":
                            await dc.Begin(WakeUp.Id);
                            break;
                        default:
                            await dc.Context.SendActivity("Sorry, I don't understand that command. Please choose an option from the list below.");
                            await next();
                            break;
                    }
                },
                async (dc, args, next) =>
                {
                    // Show the main menu again.
                    await dc.Replace(MainMenuId);
                }
            });

            // Add our child dialogs.
            dialogs.Add(CheckIn.Id, CheckIn.Instance);
            dialogs.Add(ReserveTable.Id, ReserveTable.Instance);
            dialogs.Add(WakeUp.Id, WakeUp.Instance);

            return dialogs;
        }
    }
}
```

最後，我們需要更新 `StartUp` 類別的 `ConfigureServices` 方法，以連接 Bot 並新增狀態中介軟體。

**Startup.cs**
```csharp
// This method gets called by the runtime. Use this method to add services to the container.
public void ConfigureServices(IServiceCollection services)
{
    services.AddBot<HotelBot>(options =>
    {
        options.CredentialProvider = new ConfigurationCredentialProvider(Configuration);

        options.Middleware.Add(new CatchExceptionMiddleware<Exception>(async (context, exception) =>
        {
            await context.TraceActivity($"{nameof(HotelBot)} Exception", exception);
            await context.SendActivity("Sorry, it looks like something went wrong!");
        }));

        // Add state management middleware for conversation and user state.
        var path = Path.Combine(Path.GetTempPath(), nameof(HotelBot));
        if (!Directory.Exists(path))
        {
            Directory.CreateDirectory(path);
        }
        IStorage storage = new FileStorage(path);

        options.Middleware.Add(new ConversationState<ConversationInfo>(storage));
        options.Middleware.Add(new UserState<UserInfo>(storage));
    });
}
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

系統會使用對話方塊內容的 `continue` 方法來更新對話方塊流程。 這個方法會執行對話方塊堆疊上瀑布圖的下一個步驟。

**app.js**
```JavaScript
const {BotFrameworkAdapter, FileStorage, ConversationState, UserState, BotStateSet, MessageFactory} = require("botbuilder");
const {DialogSet} = require("botbuilder-dialogs");
const restify = require("restify");
var azure = require('botbuilder-azure'); 

// Create server
let server = restify.createServer();
server.listen(process.env.port || process.env.PORT || 3978, function () {
    console.log(`${server.name} listening to ${server.url}`);
});

// Create adapter
const adapter = new BotFrameworkAdapter({
    appId: process.env.MICROSOFT_APP_ID,
    appPassword: process.env.MICROSOFT_APP_PASSWORD
});

//Memory Storage
const storage = new FileStorage("c:/temp");
// ConversationState lasts for the entirety of a conversation then gets disposed of
const convoState = new ConversationState(storage);

// UserState persists information about the user across all of the conversations you have with that user
const userState  = new UserState(storage);

adapter.use(new BotStateSet(convoState, userState));

// Listen for incoming requests 
server.post('/api/messages', (req, res) => {
    adapter.processActivity(req, res, async (context) => {
        const isMessage = context.activity.type === 'message';

        // State will store all of your information 
        const convo = convoState.get(context);
        const user = userState.get(context); // userState will not be used in this example

        const dc = dialogs.createContext(context, convo);
        // Continue the current dialog if one is currently active
        await dc.continue(); 

        // Default action
        if (!context.responded && isMessage) {

            // Getting the user info from the state
            const userinfo = userState.get(dc.context); 
            // If guest info is undefined prompt the user to check in
            if(!userinfo.guestInfo){
                await dc.begin('checkInPrompt');
            }else{
                await dc.begin('mainMenu'); 
            }           
        }
    });
});

const dialogs = new DialogSet();
dialogs.add('mainMenu', [
    async function (dc, args) {
        const menu = ["Reserve Table", "Wake Up"];
        await dc.context.sendActivity(MessageFactory.suggestedActions(menu));    
    },
    async function (dc, result){
        // Decide which module to start
        switch(result){
            case "Reserve Table":
                await dc.begin('reservePrompt');
                break;
            case "Wake Up":
                await dc.begin('wakeUpPrompt');
                break;
            default:
                await dc.context.sendActivity("Sorry, i don't understand that command. Please choose an option from the list below.");
                break;            
        }
    },
    async function (dc, result){
        await dc.replace('mainMenu'); // Show the menu again
    }

]);

// Importing the dialogs 
const checkIn = require("./checkIn");
dialogs.add('checkInPrompt', new checkIn.CheckIn(userState));

const reserve_table = require("./reserveTable");
dialogs.add('reservePrompt', new reserve_table.ReserveTable(userState));

const wake_up = require("./wake_up");
dialogs.add('wakeUpPrompt', new wake_up.WakeUp(userState));
```

---
<!-- TODO: These dialogs are not fully modularized, as there are cross dependencies:
    - Importantly, the dialogs need to know details about the bot's user state.
-->

如您所見，模組化對話方塊已新增至 Bot 的主對話方塊中，類似於將[提示](bot-builder-prompts.md)新增至對話方塊的方式。 您可以視需要將子對話方塊新增至主對話方塊中。 每個模組會新增額外的功能和服務，讓 Bot 將其提供給使用者。

## <a name="next-steps"></a>後續步驟

現在您已了解如何使用對話方塊將 Bot 邏輯模組化，讓我們看看如何使用 Language Understanding (LUIS) 以協助 Bot 決定何時開始對話方塊。

> [!div class="nextstepaction"]
> [使用 LUIS 來理解語言](./bot-builder-howto-v4-luis.md)
