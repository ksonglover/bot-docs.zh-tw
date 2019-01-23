---
title: 重複使用對話方塊 | Microsoft Docs
description: 了解如何在適用於 Node.js 的 Bot Framework SDK 和 C# 中，使用對話容器將 Bot 邏輯模組化。
keywords: 複合控制項, 模組化 Bot 邏輯
author: v-ducvo
ms.author: v-ducvo
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 11/08/2018
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 3f4b2dd49b738132affd19fea8fd5dbfbd6ff99e
ms.sourcegitcommit: b15cf37afc4f57d13ca6636d4227433809562f8b
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 01/11/2019
ms.locfileid: "54224563"
---
# <a name="reuse-dialogs"></a>重複使用對話方塊

[!INCLUDE [pre-release-label](../includes/pre-release-label.md)]

假設您正在建立可處理多項工作的旅館 Bot，例如問候使用者、預訂晚餐座位、點菜、設定鬧鐘、顯示目前天氣等等。 您可以在 Bot 中使用一個對話方塊物件來處理各項工作，但這可能會讓您的對話方塊程式碼太大而且雜亂。

您可以將對話流程的各部分變成元件對話方塊，也就是將大型對話方塊集合細分成更容易管理的片段。 這可以簡化程式碼，並可更輕鬆地對其進行偵錯，也可允許多個小組同時在 Bot 上執行作業。
您也可以在其他對話方塊集合和 Bot 中建立元件對話方塊的程式庫，以便重複使用。

在此範例中，我們會建立旅館 Bot，結合入住、叫醒及訂位元件對話方塊。

本文會以有關管理[簡單](bot-builder-dialog-manage-conversation-flow.md)及[複雜](bot-builder-dialog-manage-complex-conversation-flow.md)對話流程的資訊作為基礎。

## <a name="create-your-project"></a>建立專案

我們會從 Echo Bot 的範本開始，並在專案中包含對話方塊程式庫。

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

我們先從 **EchoBot** 範本開始。 如需指示，請參閱[適用於 .NET 的快速入門](../dotnet/bot-builder-dotnet-sdk-quickstart.md)。

若要使用對話，請為專案或解決方案安裝 `Microsoft.Bot.Builder.Dialogs` NuGet 套件。
然後視需要在程式碼檔案中的 using 陳述式，參考對話方塊程式庫。

將 **EchoBotWithCounter.cs** 檔案重新命名為 **HotelBot.cs**，並將類別重新命名為 **HotelBot**。

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

我們先從 **Echo** 範本開始。 如需指示，請參閱[適用於 JavaScript 的快速入門](../javascript/bot-builder-javascript-quickstart.md)。

可以從 npm 下載 `botbuilder-dialogs` 程式庫。 若要安裝 `botbuilder-dialogs` 程式庫，請執行下列 npm 命令：

```cmd
npm install --save botbuilder-dialogs
```

---

## <a name="managing-state"></a>管理狀態

有許多方法可針對使用複合對話方塊的 Bot 設定狀態管理。 在此 Bot 中，每個元件對話方塊都會傳回作為對話方塊結果的物件。 呼叫內容會管理傳回的值。 如需狀態管理的詳細資訊，請參閱[使用對話和使用者屬性來儲存狀態](bot-builder-howto-v4-state.md)。

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

每個對話方塊都會收集某些資訊，而 Bot 的回合處理常式或主功能表會將這些資訊儲存到使用者狀態中。 我們針對入住、訂位和鬧鈴設定定義元件對話方塊。 每一個對話方塊都會傳回適當類別的物件。 請將下列每個類別新增至您的專案，以作為新的 C# 類別模組。

```csharp
/// <summary>
/// User state information.
/// </summary>
public class UserInfo
{
    public GuestInfo Guest { get; set; }
    public TableInfo Table { get; set; }
    public WakeUpInfo WakeUp { get; set; }
}

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
```

將 **EchoBotAccessors.cs** 重新命名為 **BotAccessors.cs**，並更新類別名稱 **BotAccessors**。

接著，使用此程式碼更新檔案。 我們需要存取子來取得這兩個對話方塊狀態，以及我們所收集的使用者資訊。

```csharp
using System;
using Microsoft.Bot.Builder;
using Microsoft.Bot.Builder.Dialogs;

/// <summary>
/// Contains the state objects and the state property accessors for the bot.
/// </summary>
public class BotAccessors
{
    // The property accessor keys to use.
    public const string UserInfoAccessorName = "HotelBot.UserInfo";
    public const string DialogStateAccessorName = "HotelBot.DialogState";

    /// <summary>
    /// Initializes a new instance of the <see cref="BotAccessors"/> class.
    /// Contains the <see cref="ConversationState"/> and associated <see cref="IStatePropertyAccessor{T}"/>.
    /// </summary>
    /// <param name="conversationState">The state object that stores the counter.</param>
    public BotAccessors(ConversationState conversationState, UserState userState)
    {
        ConversationState = conversationState ?? throw new ArgumentNullException(nameof(conversationState));
        UserState = userState ?? throw new ArgumentNullException(nameof(userState));
    }

    /// <summary>Gets or sets the state property accessor for the user information we're tracking.</summary>
    /// <value>Accessor for user information.</value>
    public IStatePropertyAccessor<UserInfo> UserInfoAccessor { get; set; }

    /// <summary>Gets or sets the state property accessor for the dialog state.</summary>
    /// <value>Accessor for the dialog state.</value>
    public IStatePropertyAccessor<DialogState> DialogStateAccessor { get; set; }

    /// <summary>Gets the conversation state for the bot.</summary>
    /// <value>The conversation state for the bot.</value>
    public ConversationState ConversationState { get; }

    /// <summary>Gets the user state for the bot.</summary>
    /// <value>The user state for the bot.</value>
    public UserState UserState { get; }
}
```

在 **Startup.cs** 檔案中，更新 `ConfigureServices` 方法中的程式碼，該方法用來設定狀態和 Bot 的狀態屬性存取子。

```csharp
using Microsoft.Bot.Builder.Dialogs;

public void ConfigureServices(IServiceCollection services)
{
    services.AddBot<HotelBot>(options =>
    {
        //...

        // The Memory Storage used here is for local bot debugging only. When the bot
        // is restarted, everything stored in memory will be gone.
        IStorage dataStore = new MemoryStorage();

        // Create conversation and user state objects.
        options.State.Add(new ConversationState(dataStore));
        options.State.Add(new UserState(dataStore));
    });

    // Create and register state accessors.
    // Accessors created here are passed into the IBot-derived class on every turn.
    services.AddSingleton<BotAccessors>(sp =>
    {
        var options = sp.GetRequiredService<IOptions<BotFrameworkOptions>>().Value;
        var conversationState = options.State.OfType<ConversationState>().FirstOrDefault();
        var userState = options.State.OfType<UserState>().FirstOrDefault();

        // Create the custom state accessor.
        // State accessors enable other components to read and write individual properties of state.
        var accessors = new BotAccessors(conversationState, userState)
        {
            UserInfoAccessor = userState.CreateProperty<UserInfo>(BotAccessors.UserInfoAccessorName),
            DialogStateAccessor = conversationState.CreateProperty<DialogState>(BotAccessors.DialogStateAccessorName),
        };

        return accessors;
    });
}
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

每個對話方塊都會收集某些資訊，而 Bot 的回合處理常式或主功能表會將這些資訊儲存到使用者狀態中。 我們針對入住、訂位和鬧鈴設定定義元件對話方塊。 每一個對話方塊都會傳回具有適當屬性的物件。 我們會將這些屬性彙總在 Bot 的使用者狀態中。

在您的 **bot.js** 檔案中，將 Bot 建構函式更新為建立狀態屬性存取子，即可追蹤使用者狀態和對話方塊狀態。

```javascript
// Define the identifiers for our state property accessors.
const DIALOG_STATE_PROPERTY = 'dialogStatePropertyAccessor';
const USER_INFO_PROPERTY = 'userInfoPropertyAccessor';

constructor(conversationState, userState) {
    // Record the conversation and user state management objects.
    this.conversationState = conversationState;
    this.userState = userState;

    // Create our state property accessors.
    this.dialogStateAccessor = conversationState.createProperty(DIALOG_STATE_PROPERTY);
    this.userInfoAccessor = userState.createProperty(USER_INFO_PROPERTY);
}
```

在您的 **index.js** 檔案中，更新來自 `botbuilder` 程式庫的匯入類別，以及更新用來建立狀態物件和 Bot 的程式碼：

```javascript
// Import required bot services.
const { BotFrameworkAdapter, MemoryStorage, ConversationState, UserState } = require('botbuilder');
```

```javascript
// Define state store for your bot.
const memoryStorage = new MemoryStorage();

// Create conversation and user state with in-memory storage provider.
const conversationState = new ConversationState(memoryStorage);
const userState = new UserState(memoryStorage);

// Create the bot.
const myBot = new MyBot(conversationState, userState);
```

---

## <a name="define-the-check-in-component-dialog"></a>定義入住元件對話方塊

首先，從簡單的入住對話方塊開始，詢問使用者姓名以及要住宿的房間。 我們會建立用來延伸 `ComponentDialog` 的 `CheckInDialog` 類別。 這個類別具有可定義根對話方塊名稱的建構函式，我們會將其定義為三步驟的瀑布對話方塊。

以下是入住對話方塊的步驟。

1. 詢問來賓姓名。
1. 詢問來賓想要住宿的房間。
1. 傳送確認訊息，完成對話方塊。

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

使用下列程式碼，將 `CheckInDialog` 類別新增至您的專案。

在這整個對話方塊中，我們可以寫入本機狀態物件，其可透過步驟內容的 `Values` 屬性來存取。 對話方塊完成之後，會清除本機狀態物件。 因此，我們會從包含來賓資訊的對話方塊中傳回值。

```csharp
using System.Threading;
using System.Threading.Tasks;
using Microsoft.Bot.Builder;
using Microsoft.Bot.Builder.Dialogs;

public class CheckInDialog : ComponentDialog
{
    private const string GuestKey = nameof(CheckInDialog);
    private const string TextPrompt = "textPrompt";

    // You can start this from the parent using the dialog's ID.
    public CheckInDialog(string id)
        : base(id)
    {
        InitialDialogId = Id;

        // Define the prompts used in this conversation flow.
        AddDialog(new TextPrompt(TextPrompt));

        // Define the conversation flow using a waterfall model.
        WaterfallStep[] waterfallSteps = new WaterfallStep[]
        {
            NameStepAsync,
            RoomStepAsync,
            FinalStepAsync,
        };
        AddDialog(new WaterfallDialog(Id, waterfallSteps));
    }

    private static async Task<DialogTurnResult> NameStepAsync(
        WaterfallStepContext step,
        CancellationToken cancellationToken = default(CancellationToken))
    {
        // Clear the guest information and prompt for the guest's name.
        step.Values[GuestKey] = new GuestInfo();
        return await step.PromptAsync(
            TextPrompt,
            new PromptOptions
            {
                Prompt = MessageFactory.Text("What is your name?"),
            },
            cancellationToken);
    }

    private static async Task<DialogTurnResult> RoomStepAsync(
        WaterfallStepContext step,
        CancellationToken cancellationToken = default(CancellationToken))
    {
        // Save the name and prompt for the room number.
        string name = step.Result as string;
        ((GuestInfo)step.Values[GuestKey]).Name = name;
        return await step.PromptAsync(
            TextPrompt,
            new PromptOptions
            {
                Prompt = MessageFactory.Text($"Hi {name}. What room will you be staying in?"),
            },
            cancellationToken);
    }

    private static async Task<DialogTurnResult> FinalStepAsync(
        WaterfallStepContext step,
        CancellationToken cancellationToken = default(CancellationToken))
    {
        // Save the room number and "sign off".
        string room = step.Result as string;
        ((GuestInfo)step.Values[GuestKey]).Room = room;

        await step.Context.SendActivityAsync(
            "Great, enjoy your stay!",
            cancellationToken: cancellationToken);

        // End the dialog, returning the guest info.
        return await step.EndDialogAsync(
            (GuestInfo)step.Values[GuestKey],
            cancellationToken);
    }
}
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

建立 **checkInDialog.js** 檔案，並將可延伸 `ComponentDialog` 的 `CheckInDialog` 類別新增至其中。

在這整個對話方塊中，我們可以寫入本機狀態物件，其可透過步驟內容的 `values` 屬性來存取。 對話方塊完成之後，會清除本機狀態物件。 因此，我們會從包含來賓資訊的對話方塊中傳回值。

```JavaScript
const { ComponentDialog, TextPrompt, WaterfallDialog } = require('botbuilder-dialogs');

class CheckInDialog extends ComponentDialog {
    constructor(dialogId) {
        super(dialogId);

        // ID of the child dialog that should be started anytime the component is started.
        this.initialDialogId = dialogId;

        // Define the prompts used in this conversation flow.
        this.addDialog(new TextPrompt('textPrompt'));

        // Define the conversation flow using a waterfall model.
        this.addDialog(new WaterfallDialog(dialogId, [
            async function (step) {
                // Clear the guest information and prompt for the guest's name.
                step.values.guestInfo = {};
                return await step.prompt('textPrompt', "What is your name?");
            },
            async function (step) {
                // Save the name and prompt for the room number.
                step.values.guestInfo.userName = step.result;
                return await step.prompt('textPrompt', `Hi ${step.result}. What room will you be staying in?`);
            },
            async function (step) {
                // Save the room number and "sign off".
                step.values.guestInfo.roomNumber = step.result;
                await step.context.sendActivity(`Great! Enjoy your stay in room ${step.result}!`);

                // End the dialog, returning the guest info.
                return await step.endDialog(step.values.guestInfo);
            }
        ]));
    }
}

exports.CheckInDialog = CheckInDialog;
```

---

## <a name="define-the-reserve-table-and-wake-up-component-dialogs"></a>定義訂位和叫醒元件對話方塊

使用元件對話方塊的其中一個優點是能夠一起使用不同對話方塊。 由於每個 `DialogSet` 都會維護一組獨特的 `dialogs`，共用或交叉參考 `dialogs` 無法輕鬆完成。 這就是元件對話方塊發揮功用之處。 您可以將對話流程中複雜難解的部分，封裝至元件對話方塊中，這可以簡化對話方塊的管理和維護。 我們將建立其他兩個元件對話方塊：一個用來詢問使用者晚餐想要預訂的桌位，另一個用來建立叫醒服務。 同樣地，我們會為每個對話方塊使用個別的類別，而每個對話方塊都可延伸主要的 `ComponentDialog`。

我們會設計為在對話方塊啟動時，對話方塊就會以對話方塊選項來接受來賓資訊。

以下是訂位對話方塊的步驟。

1. 詢問要預訂的桌位。
1. 傳送確認訊息並完成對話方塊，然後傳回桌位號碼。

以下是叫醒對話方塊的步驟。

1. 詢問要為使用者設定的叫醒時間。
1. 傳送確認訊息並完成對話方塊，然後傳回鬧鈴時間。

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

使用下列程式碼，將 `ReserveTableDialog` 類別新增至您的專案。

我們會從啟動對話方塊時傳入的選項物件中，得到來賓姓名。 然後，我們會從對話方塊中傳回值，此時對話方塊中會包含桌位號碼。

```csharp
using System.Threading;
using System.Threading.Tasks;
using Microsoft.Bot.Builder;
using Microsoft.Bot.Builder.Dialogs;
using Microsoft.Bot.Builder.Dialogs.Choices;

public class ReserveTableDialog : ComponentDialog
{
    private const string TablePrompt = "choicePrompt";

    public ReserveTableDialog(string id)
        : base(id)
    {
        InitialDialogId = Id;

        // Define the prompts used in this conversation flow.
        AddDialog(new ChoicePrompt(TablePrompt));

        // Define the conversation flow using a waterfall model.
        WaterfallStep[] waterfallSteps = new WaterfallStep[]
        {
                TableStepAsync,
                FinalStepAsync,
        };
        AddDialog(new WaterfallDialog(Id, waterfallSteps));
    }

    private static async Task<DialogTurnResult> TableStepAsync(
        WaterfallStepContext step,
        CancellationToken cancellationToken = default(CancellationToken))
    {
        string greeting = step.Options is GuestInfo guest
                && !string.IsNullOrWhiteSpace(guest?.Name)
                ? $"Welcome {guest.Name}" : "Welcome";

        string prompt = $"{greeting}, How many diners will be at your table?";
        string[] choices = new string[] { "1", "2", "3", "4", "5", "6" };
        return await step.PromptAsync(
            TablePrompt,
            new PromptOptions
            {
                Prompt = MessageFactory.Text(prompt),
                Choices = ChoiceFactory.ToChoices(choices),
            },
            cancellationToken);
    }

    private static async Task<DialogTurnResult> FinalStepAsync(
        WaterfallStepContext step,
        CancellationToken cancellationToken = default(CancellationToken))
    {
        // ChoicePrompt returns a FoundChoice object.
        string table = (step.Result as FoundChoice).Value;

        // Send a confirmation message.
        await step.Context.SendActivityAsync(
            $"Sounds great;  we will reserve a table for you for {table} diners.",
            cancellationToken: cancellationToken);

        // End the dialog, returning the table info.
        return await step.EndDialogAsync(
            new TableInfo { Number = table },
            cancellationToken);
    }
}
```

使用下列程式碼，將 `SetAlarmDialog` 類別新增至您的專案。

我們會從啟動對話方塊時傳入的選項物件中，得到來賓的房間號碼。 然後，我們會從對話方塊中傳回值，對話方塊中會包含來賓想要設定的叫醒時間。

```csharp
using System.Collections.Generic;
using System.Linq;
using System.Threading;
using System.Threading.Tasks;
using Microsoft.Bot.Builder;
using Microsoft.Bot.Builder.Dialogs;

public class SetAlarmDialog : ComponentDialog
{
    private const string AlarmPrompt = "dateTimePrompt";

    public SetAlarmDialog(string id)
        : base(id)
    {
        InitialDialogId = Id;

        // Define the prompts used in this conversation flow.
        // Ideally, we'd add validation to this prompt.
        AddDialog(new DateTimePrompt(AlarmPrompt));

        // Define the conversation flow using a waterfall model.
        WaterfallStep[] waterfallSteps = new WaterfallStep[]
        {
                AlarmStepAsync,
                FinalStepAsync,
        };

        AddDialog(new WaterfallDialog(Id, waterfallSteps));
    }

    private static async Task<DialogTurnResult> AlarmStepAsync(
        WaterfallStepContext step,
        CancellationToken cancellationToken = default(CancellationToken))
    {
        string greeting = step.Options is GuestInfo guest
                && !string.IsNullOrWhiteSpace(guest?.Name)
                ? $"Hi {guest.Name}" : "Hi";

        string prompt = $"{greeting}. When would you like your alarm set for?";
        return await step.PromptAsync(
            AlarmPrompt,
            new PromptOptions { Prompt = MessageFactory.Text(prompt) },
            cancellationToken);
    }

    private static async Task<DialogTurnResult> FinalStepAsync(
        WaterfallStepContext step,
        CancellationToken cancellationToken = default(CancellationToken))
    {
        // Ambiguous responses can generate multiple results.
        var resolution = (step.Result as IList<DateTimeResolution>)?.FirstOrDefault();

        // Time ranges have a start and no value.
        var alarm = resolution.Value ?? resolution.Start;
        string roomNumber = (step.Options as GuestInfo)?.Room;

        // Send a confirmation message.
        await step.Context.SendActivityAsync(
            $"Your alarm is set to {alarm} for room {roomNumber}.",
            cancellationToken: cancellationToken);

        // End the dialog, returning the alarm info.
        return await step.EndDialogAsync(
            new WakeUpInfo { Time = alarm },
            cancellationToken);
    }
}
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

建立 **reserveTableDialog.js** 檔案，並將可延伸 `ComponentDialog` 的 `ReserveTableDialog` 類別新增至其中。

我們會從啟動對話方塊時傳入的選項物件中，得到來賓姓名。 然後，我們會從對話方塊中傳回值，此時對話方塊中會包含桌位號碼。

```JavaScript
const { ComponentDialog, ChoicePrompt, WaterfallDialog } = require('botbuilder-dialogs');

class ReserveTableDialog extends ComponentDialog {
    constructor(dialogId) {
        super(dialogId);

        // ID of the child dialog that should be started anytime the component is started.
        this.initialDialogId = dialogId;

        // Define the prompts used in this conversation flow.
        this.addDialog(new ChoicePrompt('choicePrompt'));

        // Define the conversation flow using a waterfall model.
        this.addDialog(new WaterfallDialog(dialogId, [
            async function (step) {
                // Welcome the user and ask for their table preference.
                const greeting = step.options && step.options.userName ? `Welcome ${step.options.userName}` : `Welcome`;

                const promptOptions = {
                    prompt: `${greeting}, How many diners will be at your table?`,
                    reprompt: 'That was not a valid choice, please select a table size between 1 and 6 guests.',
                    choices: ['1', '2', '3', '4', '5', '6']
                };
                return await step.prompt('choicePrompt', promptOptions);
            },
            async function (step) {
                const choice = step.result;

                // Send a confirmation message.
                const tableNumber = choice.value;
                await step.context.sendActivity(`Sounds great, we will reserve a table for you for ${tableNumber} diners.`);

                // End the dialog, returning the table info.
                return await step.endDialog({ table: tableNumber });
            }
        ]));
    }
}

exports.ReserveTableDialog = ReserveTableDialog;
```

建立 **setAlarmDialog.js** 檔案，並將可延伸 `ComponentDialog` 的 `SetAlarmDialog` 類別新增至其中。

我們會從啟動對話方塊時傳入的選項物件中，得到來賓的房間號碼。 然後，我們會從對話方塊中傳回值，對話方塊中會包含來賓想要設定的叫醒時間。

```JavaScript
const { ComponentDialog, DateTimePrompt, WaterfallDialog } = require('botbuilder-dialogs');

class SetAlarmDialog extends ComponentDialog {
    constructor(dialogId) {
        super(dialogId);

        // ID of the child dialog that should be started anytime the component is started.
        this.initialDialogId = dialogId;

        // Define the prompts used in this conversation flow.
        this.dialogs.add(new DateTimePrompt('datePrompt'));

        this.dialogs.add(new WaterfallDialog(dialogId, [
            async function (step) {
                step.values.wakeUp = {};
                if (step.options && step.options.roomNumber) {
                    step.values.roomNumber = step.options.roomNumber;
                }

                const greeting = step.options && step.options.userName ? `Hi ${step.options.userName}` : `Hi`;
                return await step.prompt('datePrompt', `${greeting}. What time would you like your alarm to be set?`);
            },
            async function (step) {
                // Ambiguous responses can generate multiple results.
                const resolution = step.result[0];

                // Time ranges have a start and no value.
                const alarm = resolution.value ? resolution.value : resolution.start;
                const roomNumber = step.values.roomNumber;

                // Send a confirmation message.
                await step.context.sendActivity(`Your alarm is set to ${alarm} for room ${roomNumber}.`);

                // End the dialog, returning the alarm info.
                return await step.endDialog({ alarm: alarm });
            }]));

        // Defining the prompt used in this conversation flow
    }
}

exports.SetAlarmDialog = SetAlarmDialog;
```

---

## <a name="add-the-component-dialogs-to-the-bot"></a>將元件對話方塊新增到 Bot

我們已經定義了三個元件對話方塊，現在可以將這些對話方塊用在 Bot 中。

- 三個對話方塊都已新增至 Bot 的主對話方塊集合中。
- 開始新對話時，我們沒有使用中的對話方塊，因此會使用 Bot 的 on turn 邏輯。
- 如果我們沒有使用者的來賓資訊，我們會啟動入住對話方塊。
- 一旦擁有來賓資訊時，就會由主對話方塊接管，並讓使用者可多次啟動訂位和叫醒對話方塊。

我們將更新 Bot on-turn 處理常式的邏輯。

1. 取得使用者狀態。
1. 繼續任何使用中的對話方塊。
1. 如果對話方塊已完成此回合，這應該是入住對話方塊。
   1. 儲存來賓資訊。
   1. 啟動主對話方塊。
1. 如果 Bot 尚未傳送訊息給使用者，則不會有正在執行的使用中對話方塊。
    1. 如果我們沒有來賓的資訊，則會啟動入住對話方塊。
    1. 否則會啟動主對話方塊。
1. 儲存任何可能已發生的狀態變更。

以下是主對話方塊的步驟。

1. 詢問來賓想要執行的動作：訂位或設定叫醒服務。
1. 啟動對應的子對話方塊，或傳送_無法辨識輸入_訊息，並從第一個步驟開始。
1. 處理子對話方塊的傳回值，並重新啟動主對話方塊。

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

在 **HotelBot.cs** 檔案中更新 using 陳述式。

```csharp
using System.Collections.Generic;
using System.Threading;
using System.Threading.Tasks;
using Microsoft.Bot.Builder;
using Microsoft.Bot.Builder.Dialogs;
using Microsoft.Bot.Schema;
using Microsoft.Extensions.Logging;
```

更新初始化程式碼，並定義 Bot 的對話方塊集合和主對話方塊。

```csharp
// Define the IDs for the dialogs in the bot's dialog set.
private const string MainDialogId = "mainDialog";
private const string CheckInDialogId = "checkInDialog";
private const string TableDialogId = "tableDialog";
private const string AlarmDialogId = "alarmDialog";

// Define the dialog set for the bot.
private readonly DialogSet _dialogs;

// Define the state accessors and the logger for the bot.
private readonly BotAccessors _accessors;
private readonly ILogger _logger;

/// <summary>
/// Initializes a new instance of the <see cref="HotelBot"/> class.
/// </summary>
/// <param name="accessors">Contains the objects to use to manage state.</param>
/// <param name="loggerFactory">A <see cref="ILoggerFactory"/> that is hooked to the Azure App Service provider.</param>
public HotelBot(BotAccessors accessors, ILoggerFactory loggerFactory)
{
    if (loggerFactory == null)
    {
        throw new System.ArgumentNullException(nameof(loggerFactory));
    }

    _logger = loggerFactory.CreateLogger<HotelBot>();
    _logger.LogTrace($"{nameof(HotelBot)} turn start.");
    _accessors = accessors ?? throw new System.ArgumentNullException(nameof(accessors));

    // Define the steps of the main dialog.
    WaterfallStep[] steps = new WaterfallStep[]
    {
        MenuStepAsync,
        HandleChoiceAsync,
        LoopBackAsync,
    };

    // Create our bot's dialog set, adding a main dialog and the three component dialogs.
    _dialogs = new DialogSet(_accessors.DialogStateAccessor)
        .Add(new WaterfallDialog(MainDialogId, steps))
        .Add(new CheckInDialog(CheckInDialogId))
        .Add(new ReserveTableDialog(TableDialogId))
        .Add(new SetAlarmDialog(AlarmDialogId));
}

private static async Task<DialogTurnResult> MenuStepAsync(
    WaterfallStepContext stepContext,
    CancellationToken cancellationToken = default(CancellationToken))
{
    // Present the user with a set of "suggested actions".
    List<string> menu = new List<string> { "Reserve Table", "Wake Up" };
    await stepContext.Context.SendActivityAsync(
        MessageFactory.SuggestedActions(menu, "How can I help you?"),
        cancellationToken: cancellationToken);
    return Dialog.EndOfTurn;
}

private async Task<DialogTurnResult> HandleChoiceAsync(
    WaterfallStepContext stepContext,
    CancellationToken cancellationToken = default(CancellationToken))
{
    // Get the user's info. (Since the type factory is null, this will throw if state does not yet have a value for user info.)
    UserInfo userInfo = await _accessors.UserInfoAccessor.GetAsync(stepContext.Context, null, cancellationToken);

    // Check the user's input and decide which dialog to start.
    // Pass in the guest info when starting either of the child dialogs.
    string choice = (stepContext.Result as string)?.Trim()?.ToLowerInvariant();
    switch (choice)
    {
        case "reserve table":
            return await stepContext.BeginDialogAsync(TableDialogId, userInfo.Guest, cancellationToken);

        case "wake up":
            return await stepContext.BeginDialogAsync(AlarmDialogId, userInfo.Guest, cancellationToken);

        default:
            // If we don't recognize the user's intent, start again from the beginning.
            await stepContext.Context.SendActivityAsync(
                "Sorry, I don't understand that command. Please choose an option from the list.");
            return await stepContext.ReplaceDialogAsync(MainDialogId, null, cancellationToken);
    }
}

private async Task<DialogTurnResult> LoopBackAsync(
    WaterfallStepContext stepContext,
    CancellationToken cancellationToken = default(CancellationToken))
{
    // Get the user's info. (Because the type factory is null, this will throw if state does not yet have a value for user info.)
    UserInfo userInfo = await _accessors.UserInfoAccessor.GetAsync(stepContext.Context, null, cancellationToken);

    // Process the return value from the child dialog.
    switch (stepContext.Result)
    {
        case TableInfo table:
            // Store the results of the reserve-table dialog.
            userInfo.Table = table;
            await _accessors.UserInfoAccessor.SetAsync(stepContext.Context, userInfo, cancellationToken);
            break;
        case WakeUpInfo alarm:
            // Store the results of the set-wake-up-call dialog.
            userInfo.WakeUp = alarm;
            await _accessors.UserInfoAccessor.SetAsync(stepContext.Context, userInfo, cancellationToken);
            break;
        default:
            // We shouldn't get here, since these are no other branches that get this far.
            break;
    }

    // Restart the main menu dialog.
    return await stepContext.ReplaceDialogAsync(MainDialogId, null, cancellationToken);
}
```

然後，更新 Bot 的回合處理常式，以使用對話方塊集合。

```csharp
public async Task OnTurnAsync(ITurnContext turnContext, CancellationToken cancellationToken = default(CancellationToken))
{
    if (turnContext.Activity.Type == ActivityTypes.Message)
    {
        // Establish dialog state from the conversation state.
        DialogContext dc = await _dialogs.CreateContextAsync(turnContext, cancellationToken);

        // Get the user's info.
        UserInfo userInfo = await _accessors.UserInfoAccessor.GetAsync(turnContext, () => new UserInfo(), cancellationToken);

        // Continue any current dialog.
        DialogTurnResult dialogTurnResult = await dc.ContinueDialogAsync();

        // Process the result of any complete dialog.
        if (dialogTurnResult.Status is DialogTurnStatus.Complete)
        {
            switch (dialogTurnResult.Result)
            {
                case GuestInfo guestInfo:
                    // Store the results of the check-in dialog.
                    userInfo.Guest = guestInfo;
                    await _accessors.UserInfoAccessor.SetAsync(turnContext, userInfo, cancellationToken);
                    break;
                default:
                    // We shouldn't get here, since the main dialog is designed to loop.
                    break;
            }
        }

        // Every dialog step sends a response, so if no response was sent,
        // then no dialog is currently active.
        else if (!turnContext.Responded)
        {
            if (string.IsNullOrEmpty(userInfo.Guest?.Name))
            {
                // If we don't yet have the guest's info, start the check-in dialog.
                await dc.BeginDialogAsync(CheckInDialogId, null, cancellationToken);
            }
            else
            {
                // Otherwise, start our bot's main dialog.
                await dc.BeginDialogAsync(MainDialogId, null, cancellationToken);
            }
        }

        // Save the new turn count into the conversation state.
        await _accessors.ConversationState.SaveChangesAsync(turnContext, false, cancellationToken);
        await _accessors.UserState.SaveChangesAsync(turnContext, false, cancellationToken);
    }
    else
    {
        await turnContext.SendActivityAsync($"{turnContext.Activity.Type} event detected");
    }
}
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

在我們的 Bot 檔案 (**bot.js**) 中，我們不只要從 SDK 匯入要使用的類別，還要匯入為元件對話方塊定義的類別。

```JavaScript
const { ActivityTypes, MessageFactory } = require('botbuilder');
const { DialogSet, WaterfallDialog, Dialog, DialogTurnStatus } = require('botbuilder-dialogs');

// Import our component dialogs.
const { CheckInDialog } = require("./checkInDialog");
const { ReserveTableDialog } = require("./reserveTableDialog");
const { SetAlarmDialog } = require("./setAlarmDialog");
```

我們也需要建立對話方塊集合，並新增我們要在其中使用的所有對話方塊。

我們會將主對話方塊的瀑布步驟定義為類別中的函式，而不是內嵌函式。 我們會在這些函式上使用 `bind()`，因此，`this` 可從函式內部進行正確解析。

以下是我們已更新的 Bot 建構函式。

```JavaScript
constructor(conversationState, userState) {
    // Record the conversation and user state management objects.
    this.conversationState = conversationState;
    this.userState = userState;

    // Create our state property accessors.
    this.dialogStateAccessor = conversationState.createProperty(DIALOG_STATE_PROPERTY);
    this.userInfoAccessor = userState.createProperty(USER_INFO_PROPERTY);

    // Create our bot's dialog set, adding a main dialog and the three component dialogs.
    this.dialogs = new DialogSet(this.dialogStateAccessor)
        .add(new CheckInDialog('checkInDialog'))
        .add(new ReserveTableDialog('reserveTableDialog'))
        .add(new SetAlarmDialog('setAlarmDialog'))
        .add(new WaterfallDialog('mainDialog', [
            this.promptForChoice.bind(this),
            this.startChildDialog.bind(this),
            this.saveResult.bind(this)
    ]));
}
```

在 Bot 建構函式下方，新增下列程式碼以實作主對話方塊的步驟。

```JavaScript
async promptForChoice(step) {
    const menu = ["Reserve Table", "Wake Up"];
    await step.context.sendActivity(MessageFactory.suggestedActions(menu, 'How can I help you?'));
    return Dialog.EndOfTurn;
}

async startChildDialog(step) {
    // Get the user's info.
    const user = await this.userInfoAccessor.get(step.context);
    // Check the user's input and decide which dialog to start.
    // Pass in the guest info when starting either of the child dialogs.
    switch (step.result) {
        case "Reserve Table":
            return await step.beginDialog('reserveTableDialog', user.guestInfo);
            break;
        case "Wake Up":
            return await step.beginDialog('setAlarmDialog', user.guestInfo);
            break;
        default:
            await step.context.sendActivity("Sorry, I don't understand that command. Please choose an option from the list.");
            return await step.replaceDialog('mainDialog');
            break;
    }
}

async saveResult(step) {
    // Process the return value from the child dialog.
    if (step.result) {
        const user = await this.userInfoAccessor.get(step.context);
        if (step.result.table) {
            // Store the results of the reserve-table dialog.
            user.table = step.result.table;
        } else if (step.result.alarm) {
            // Store the results of the set-wake-up-call dialog.
            user.alarm = step.result.alarm;
        }
        await this.userInfoAccessor.set(step.context, user);
    }
    // Restart the main menu dialog.
    return await step.replaceDialog('mainDialog'); // Show the menu again
}
```

並立即更新 Bot 的回合處理常式：

```JavaScript
async onTurn(turnContext) {
    if (turnContext.activity.type === ActivityTypes.Message) {
        const user = await this.userInfoAccessor.get(turnContext, {});
        const dc = await this.dialogs.createContext(turnContext);
        const dialogTurnResult = await dc.continueDialog();
        if (dialogTurnResult.status === DialogTurnStatus.complete) {
            user.guestInfo = dialogTurnResult.result;
            await this.userInfoAccessor.set(turnContext, user);
            await dc.beginDialog('mainDialog');
        } else if (!turnContext.responded) {
            if (!user.guestInfo) {
                await dc.beginDialog('checkInDialog');
            } else {
                await dc.beginDialog('mainDialog');
            }
        }
        // Save state changes
        await this.conversationState.saveChanges(turnContext);
        await this.userState.saveChanges(turnContext);
    }
}
```

---

如您所見，元件對話方塊已新增至 Bot 的主對話方塊中，類似於將[提示](bot-builder-prompts.md)新增至對話方塊的方式。 您可以視需要將子對話方塊新增至主對話方塊中。 每個模組會新增額外的功能和服務，讓 Bot 將其提供給使用者。

## <a name="next-steps"></a>後續步驟

現在您已了解如何使用元件對話方塊，讓我們看看如何使用 Language Understanding (LUIS) 以協助 Bot 決定何時開始對話方塊。

> [!div class="nextstepaction"]
> [使用 LUIS 來理解語言](./bot-builder-howto-v4-luis.md)
