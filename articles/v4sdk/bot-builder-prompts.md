---
title: 使用對話方塊提示收集使用者輸入 | Microsoft Docs
description: 了解如何在 Bot Framework SDK 中使用 Dialogs 程式庫來提示使用者輸入。
keywords: 提示, 提示, 使用者輸入, 對話方塊, AttachmentPrompt, ChoicePrompt, ConfirmPrompt, DatetimePrompt, NumberPrompt, TextPrompt, 重新提示, 驗證
author: JonathanFingold
ms.author: v-jofing
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 02/19/2019
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 68c01b0f12790393fe0ee7ae0bd28addf2d26ae7
ms.sourcegitcommit: 05ddade244874b7d6e2fc91745131b99cc58b0d6
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 02/21/2019
ms.locfileid: "56591119"
---
# <a name="gather-user-input-using-a-dialog-prompt"></a>使用對話方塊提示蒐集使用者輸入

[!INCLUDE [pre-release-label](../includes/pre-release-label.md)]

張貼問題來收集資訊是 Bot 與使用者互動時的其中一種主要方式。 *dialogs* 程式庫可讓您輕鬆地詢問問題，以及驗證回應以確保回應符合特定資料類型或符合自訂驗證規則。 本主題詳細說明如何從瀑布式對話建立並呼叫提示。

## <a name="prerequisites"></a>必要條件

- 本文中的程式碼是以 **DialogPromptBot** 範例為基礎。 您需要一份 [C# 範例](https://aka.ms/dialog-prompt-cs)或 [JS 範例](https://aka.ms/dialog-prompt-js)。
- 需要基本了解 [dialogs 程式庫](bot-builder-concept-dialog.md)以及如何[管理交談](bot-builder-dialog-manage-conversation-flow.md)。
- 用於測試的 [Bot Framework 模擬器](https://github.com/Microsoft/BotFramework-Emulator)。

## <a name="using-prompts"></a>使用提示

唯有對話方塊與提示位於相同對話方塊集時，對話方塊才可以使用提示。 在相同對話方塊集內的一個對話方塊或多個對話方塊中，您可以在多個步驟中使用相同的提示。 不過，您可在初始化階段建立自訂驗證與提示的關聯。 若要對同類型的提示使用不同的驗證，您則需要該提示 類型的多個執行個體，而每個執行個體都有自己的驗證碼。

### <a name="define-a-state-property-accessor-for-the-dialog-state"></a>定義對話狀態的狀態屬性存取子

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

本文中使用的對話提示範例會提示使用者輸入預訂資訊。 若要管理派對規模和日期，請在 DialogPromptBot.cs 檔案中，定義預訂資訊的內部類別。

```csharp
public class Reservation
{
    public int Size { get; set; }

    public string Location { get; set; }

    public string Date { get; set; }
}
```

接下來，新增預訂資料的狀態屬性存取子。

```csharp
public class DialogPromptBotAccessors
{
    public DialogPromptBotAccessors(ConversationState conversationState)
    {
        ConversationState = conversationState
            ?? throw new ArgumentNullException(nameof(conversationState));
    }

    public static string DialogStateAccessorKey { get; }
        = "DialogPromptBotAccessors.DialogState";
    public static string ReservationAccessorKey { get; }
        = "DialogPromptBotAccessors.Reservation";

    public IStatePropertyAccessor<DialogState> DialogStateAccessor { get; set; }
    public IStatePropertyAccessor<DialogPromptBot.Reservation> ReservationAccessor { get; set; }

    public ConversationState ConversationState { get; }
}
```

在 Startup.cs 中，更新 `ConfigureServices` 方法以設定存取子。

```csharp
public void ConfigureServices(IServiceCollection services)
{
    // ...

    IStorage dataStore = new MemoryStorage();
    var conversationState = new ConversationState(dataStore);

    // Create and register state accesssors.
    services.AddSingleton<DialogPromptBotAccessors>(sp =>
    {
        // State accessors enable other components to read and write individual properties of state.
        var accessors = new DialogPromptBotAccessors(conversationState)
        {
            DialogStateAccessor =
                conversationState.CreateProperty<DialogState>(
                    DialogPromptBotAccessors.DialogStateAccessorKey),
            ReservationAccessor =
                conversationState.CreateProperty<DialogPromptBot.Reservation>(
                    DialogPromptBotAccessors.ReservationAccessorKey),
        };

        return accessors;
    });

    // ...
}
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

不需要針對 JavaScript 變更 HTTP 服務程式碼，因此我們可讓 index.js 檔案保持原狀。

在 bot.js 中，我們包含了對話提示 Bot 所需的 `require` 陳述式。

```javascript
const { ActivityTypes } = require('botbuilder');
const { DialogSet, WaterfallDialog, NumberPrompt, DateTimePrompt, ChoicePrompt, DialogTurnStatus }
    = require('botbuilder-dialogs');
```

新增狀態屬性存取子、對話和提示的識別碼。

```javascript
// Define identifiers for our state property accessors.
const DIALOG_STATE_ACCESSOR = 'dialogStateAccessor';
const RESERVATION_ACCESSOR = 'reservationAccessor';

// Define identifiers for our dialogs and prompts.
const RESERVATION_DIALOG = 'reservationDialog';
const SIZE_RANGE_PROMPT = 'rangePrompt';
const LOCATION_PROMPT = 'locationPrompt';
const RESERVATION_DATE_PROMPT = 'reservationDatePrompt';
```

---

### <a name="create-a-dialog-set-and-prompts"></a>建立對話集和提示

一般而言，在初始化 Bot 時，建立提示和對話，並將其新增到對話集。 然後，當 Bot 收到來自使用者的輸入時，對話方塊集即可解析提示的識別碼。

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

在 `DialogPromptBot` 類別中，定義對話識別碼、提示以及要在對話中追蹤的值。

```csharp
// Define identifiers for our dialogs and prompts.
private const string ReservationDialog = "reservationDialog";
private const string PartySizePrompt = "partySizePrompt";
private const string SizeRangePrompt = "sizeRangePrompt";
private const string LocationPrompt = "locationPrompt";
private const string ReservationDatePrompt = "reservationDatePrompt";

// Define keys for tracked values within the dialog.
private const string LocationKey = "location";
private const string PartySizeKey = "partySize";
```

在 Bot 的建構函式中，建立對話集、新增提示，以及新增預訂對話。 我們會在建立提示時包含自訂驗證，而且稍後會實作驗證函式。

```csharp
private readonly DialogSet _dialogSet;
private readonly DialogPromptBotAccessors _accessors;

// ...

// Initializes a new instance of the <see cref="DialogPromptBot"/> class.
public DialogPromptBot(DialogPromptBotAccessors accessors, ILoggerFactory loggerFactory)
{
    // ...

    _accessors = accessors ?? throw new System.ArgumentNullException(nameof(accessors));

    // Create the dialog set and add the prompts, including custom validation.
    _dialogSet = new DialogSet(_accessors.DialogStateAccessor);

    _dialogSet.Add(new NumberPrompt<int>(PartySizePrompt, PartySizeValidatorAsync));
    _dialogSet.Add(new NumberPrompt<int>(SizeRangePrompt, RangeValidatorAsync));
    _dialogSet.Add(new ChoicePrompt(LocationPrompt));
    _dialogSet.Add(new DateTimePrompt(ReservationDatePrompt, DateValidatorAsync));

    // Define the steps of the waterfall dialog and add it to the set.
    WaterfallStep[] steps = new WaterfallStep[]
    {
        PromptForPartySizeAsync,
        PromptForLocationAsync,
        PromptForReservationDateAsync,
        AcknowledgeReservationAsync,
    };

    _dialogSet.Add(new WaterfallDialog(ReservationDialog, steps));
}
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

在建構函式中，建立狀態存取子屬性。
接下來，建立對話集並新增提示，包括自訂驗證。
然後定義瀑布式對話的步驟，並將其新增至集合。

```javascript
constructor(conversationState) {
    // Creates our state accessor properties.
    // See https://aka.ms/about-bot-state-accessors to learn more about the bot state and state accessors.
    this.dialogStateAccessor = conversationState.createProperty(DIALOG_STATE_ACCESSOR);
    this.reservationAccessor = conversationState.createProperty(RESERVATION_ACCESSOR);
    this.conversationState = conversationState;

    // Create the dialog set and add the prompts, including custom validation.
    this.dialogSet = new DialogSet(this.dialogStateAccessor);
    this.dialogSet.add(new NumberPrompt(SIZE_RANGE_PROMPT, this.rangeValidator));
    this.dialogSet.add(new ChoicePrompt(LOCATION_PROMPT));
    this.dialogSet.add(new DateTimePrompt(RESERVATION_DATE_PROMPT, this.dateValidator));

    // Define the steps of the waterfall dialog and add it to the set.
    this.dialogSet.add(new WaterfallDialog(RESERVATION_DIALOG, [
        this.promptForPartySize.bind(this),
        this.promptForLocation.bind(this),
        this.promptForReservationDate.bind(this),
        this.acknowledgeReservation.bind(this),
    ]));
}
```

---

### <a name="implement-dialog-steps"></a>實作對話步驟

在主要 Bot 檔案中，我們會實作瀑布式對話的每個步驟。 新增提示之後，請在瀑布式對話的一個步驟中呼叫該提示，並在下列對話步驟中取得提示結果。 若要在瀑布式步驟內呼叫提示，請呼叫「瀑布式步驟內容」物件的 _prompt_ 方法。 第一個參數是要使用的提示識別碼，而第二個參數包含提示的選項，例如用來要求使用者提供輸入的文字。

這些方法會顯示：

- 如何從瀑布式步驟呼叫提示，包括如何傳遞「提示選項」。
- 如何使用「驗證」屬性對自訂驗證程式提供額外的參數。
- 如何使用「選擇」屬性提供選項提示的選項。

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

在 **DialogPromptBot.cs** 檔案中，我們會實作瀑布式對話的步驟。

在此，我們會說明前兩個瀑布式步驟，也就是 `PromptForPartySizeAsync` 和 `PromptForLocationAsync`。

```csharp
// First step of the main dialog: prompt for party size.
private async Task<DialogTurnResult> PromptForPartySizeAsync(
    WaterfallStepContext stepContext,
    CancellationToken cancellationToken = default(CancellationToken))
{
    // Prompt for the party size. The result of the prompt is returned to the next step of the waterfall.
    return await stepContext.PromptAsync(
        SizeRangePrompt,
        new PromptOptions
        {
            Prompt = MessageFactory.Text("How many people is the reservation for?"),
            RetryPrompt = MessageFactory.Text("How large is your party?"),
            Validations = new Range { Min = 3, Max = 8 },
        },
        cancellationToken);
}

// Second step of the main dialog: prompt for location.
private async Task<DialogTurnResult> PromptForLocationAsync(
    WaterfallStepContext stepContext,
    CancellationToken cancellationToken)
{
    // Record the party size information in the current dialog state.
    var size = (int)stepContext.Result;
    stepContext.Values[PartySizeKey] = size;

    // Prompt for the location.
    return await stepContext.PromptAsync(
        LocationPrompt,
        new PromptOptions
        {
            Prompt = MessageFactory.Text("Please choose a location."),
            RetryPrompt = MessageFactory.Text("Sorry, please choose a location from the list."),
            Choices = ChoiceFactory.ToChoices(new List<string> { "Redmond", "Bellevue", "Seattle" }),
        },
        cancellationToken);
}
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

在 **bot.js** 檔案中，我們會實作瀑布式對話的步驟。

在此，我們會說明前兩個瀑布式步驟，也就是 `promptForPartySize` 和 `promptForLocation`。

```javascript
async promptForPartySize(stepContext) {
    // Prompt for the party size. The result of the prompt is returned to the next step of the waterfall.
    return await stepContext.prompt(
        SIZE_RANGE_PROMPT, {
            prompt: 'How many people is the reservation for?',
            retryPrompt: 'How large is your party?',
            validations: { min: 3, max: 8 },
        });
}

async promptForLocation(stepContext) {
    // Record the party size information in the current dialog state.
    stepContext.values.size = stepContext.result;

    // Prompt for location.
    return await stepContext.prompt(LOCATION_PROMPT, {
        prompt: 'Please choose a location.',
        retryPrompt: 'Sorry, please choose a location from the list.',
        choices: ['Redmond', 'Bellevue', 'Seattle'],
    });
}
```

---

_prompt_ 方法的第二個參數會採用「提示選項」物件，其具有下列屬性。

| 屬性 | 說明 |
| :--- | :--- |
| prompt | 要傳送給使用者、要求他們輸入的初始活動。 |
| _retry prompt_ | 如果使用者的第一項輸入未經驗證，要傳送給使用者的活動。 |
| _選擇_ | 可供使用者從中選擇、搭配選擇提示使用的選擇清單。 |
| 驗證 | 要與自訂驗證程式搭配使用的其他參數。 |

一般而言，prompt 和 retry prompt 屬性都是活動，雖然其在不同程式設計語言中的處理方式有一些差異。

您應該一律指定要傳送給使用者的初始提示活動。

如果使用者的輸入因提示無法剖析其格式 (例如對數字提示輸入 "tomorrow")，或因輸入不符合驗證準則而驗證失敗，則指定 retry prompt 將有其效用。 在此情況下，如果未提供任何 retry prompt，則提示會使用初始提示活動，來重新提示使用者提供輸入。

對於選擇提示，您應該一律提供可用的選擇清單。

## <a name="custom-validation"></a>自訂驗證

您可以先驗證提示回應，再將值傳回至**瀑布**的下一個步驟。 驗證程式函式具有「提示驗證程式內容」參數並且會傳回布林值，指出輸入是否通過驗證。

提示驗證程式內容包含下列屬性：

| 屬性 | 說明 |
| :--- | :--- |
| _內容_ | Bot 目前的回合內容。 |
| _Recognized_ | 「提示辨識器結果」，其中包含使用者輸入的相關資訊 (如辨識器所處理)。 |
| _選項_ | 包含在呼叫中所提供用來啟動提示的「提示選項」。 |

提示辨識器結果具有下列屬性：

| 屬性 | 說明 |
| :--- | :--- |
| _已成功_ | 表示辨識器是否能夠剖析輸入。 |
| _值_ | 來自辨識器的傳回值。 如有必要，驗證碼可以修改此值。 |

### <a name="implement-validation-code"></a>實作驗證碼

您可在初始化階段，於 Bot 的建構函式中，建立自訂驗證與提示的關聯。

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

```csharp
// ...
_dialogSet.Add(new NumberPrompt<int>(SizeRangePrompt, RangeValidatorAsync));
// ...
_dialogSet.Add(new DateTimePrompt(ReservationDatePrompt, DateValidatorAsync));
// ...
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

```javascript
// ...
this.dialogSet.add(new NumberPrompt(SIZE_RANGE_PROMPT, this.rangeValidator));
// ...
this.dialogSet.add(new DateTimePrompt(RESERVATION_DATE_PROMPT, this.dateValidator));
// ...
```

---

**派對規模驗證程式**

我們會限制可以建立保留的合作對象大小。 若要定義有效範圍，可透過用來呼叫合作對象大小提示的「驗證」屬性來進行。

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

```csharp
// Validates whether the party size is appropriate to make a reservation.
private async Task<bool> RangeValidatorAsync(
    PromptValidatorContext<int> promptContext,
    CancellationToken cancellationToken)
{
    // Check whether the input could be recognized as an integer.
    if (!promptContext.Recognized.Succeeded)
    {
        await promptContext.Context.SendActivityAsync(
            "I'm sorry, I do not understand. Please enter the number of people in your party.",
            cancellationToken: cancellationToken);
        return false;
    }

    // Check whether the party size is appropriate.
    var size = promptContext.Recognized.Value;
    var validRange = promptContext.Options.Validations as Range;
    if (size < validRange.Min || size > validRange.Max)
    {
        await promptContext.Context.SendActivitiesAsync(
            new Activity[]
            {
                MessageFactory.Text($"Sorry, we can only take reservations for parties " +
                    $"of {validRange.Min} to {validRange.Max}."),
                promptContext.Options.RetryPrompt,
            },
            cancellationToken: cancellationToken);
        return false;
    }

    return true;
}
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

```javascript
async rangeValidator(promptContext) {
    // Check whether the input could be recognized as an integer.
    if (!promptContext.recognized.succeeded) {
        await promptContext.context.sendActivity(
            "I'm sorry, I do not understand. Please enter the number of people in your party.");
        return false;
    }
    else if (promptContext.recognized.value % 1 != 0) {
        await promptContext.context.sendActivity(
            "I'm sorry, I don't understand fractional people.");
        return false;
    }

    // Check whether the party size is appropriate.
    var size = promptContext.recognized.value;
    if (size < promptContext.options.validations.min
        || size > promptContext.options.validations.max) {
        await promptContext.context.sendActivity(
            'Sorry, we can only take reservations for parties of '
            + `${promptContext.options.validations.min} to `
            + `${promptContext.options.validations.max}.`);
        await promptContext.context.sendActivity(promptContext.options.retryPrompt);
        return false;
    }

    return true;
}
```

---

**日期時間驗證**

在預訂日期驗證程式中，我們會將預訂限制為從目前時間算起的一小時或更多小時。 我們會保留符合準則的第一個解析，並且清除其餘解析。

此驗證程式碼並不詳盡，最適合用於可剖析為日期和時間的輸入。 驗證碼會示範可供驗證日期時間提示的一些選項，而您的實作會取決於您嘗試向使用者收集哪些資訊。

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

```csharp
// Validates whether the reservation date is appropriate.
private async Task<bool> DateValidatorAsync(
    PromptValidatorContext<IList<DateTimeResolution>> promptContext,
    CancellationToken cancellationToken = default(CancellationToken))
{
    // Check whether the input could be recognized as an integer.
    if (!promptContext.Recognized.Succeeded)
    {
        await promptContext.Context.SendActivityAsync(
            "I'm sorry, I do not understand. Please enter the date or time for your reservation.",
            cancellationToken: cancellationToken);

        return false;
    }

    // Check whether any of the recognized date-times are appropriate,
    // and if so, return the first appropriate date-time.
    var earliest = DateTime.Now.AddHours(1.0);
    var value = promptContext.Recognized.Value.FirstOrDefault(v =>
        DateTime.TryParse(v.Value ?? v.Start, out var time) && DateTime.Compare(earliest, time) <= 0);

    if (value != null)
    {
        promptContext.Recognized.Value.Clear();
        promptContext.Recognized.Value.Add(value);
        return true;
    }

    await promptContext.Context.SendActivityAsync(
            "I'm sorry, we can't take reservations earlier than an hour from now.",
            cancellationToken: cancellationToken);

    return false;
}
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

```javascript
async dateValidator(promptContext) {
    // Check whether the input could be recognized as an integer.
    if (!promptContext.recognized.succeeded) {
        await promptContext.context.sendActivity(
            "I'm sorry, I do not understand. Please enter the date or time for your reservation.");
        return false;
    }

    // Check whether any of the recognized date-times are appropriate,
    // and if so, return the first appropriate date-time.
    const earliest = Date.now() + (60 * 60 * 1000);
    let value = null;
    promptContext.recognized.value.forEach(candidate => {
        // TODO: update validation to account for time vs date vs date-time vs range.
        const time = new Date(candidate.value || candidate.start);
        if (earliest < time.getTime()) {
            value = candidate;
        }
    });
    if (value) {
        promptContext.recognized.value = [value];
        return true;
    }

    await promptContext.context.sendActivity(
        "I'm sorry, we can't take reservations earlier than an hour from now.");
    return false;
}
```

---

日期時間提示會傳回符合使用者輸入的可能「日期時間解析」陣列。 例如，9:00 可能表示 9 AM 或 9 PM，星期日也模稜兩可。 此外，日期時間解析可以代表日期、時間、日期時間或範圍。 日期時間提示會字元使用 [Microsoft/Recognizers-Text](https://github.com/Microsoft/Recognizers-Text) 來剖析使用者輸入。

## <a name="update-the-turn-handler"></a>更新回合處理常式

更新 Bot 的回合處理常式以開始對話方塊，並且在完成時接受來自對話的傳回值。 我們在此假設使用者正與 Bot 互動，則 Bot 具有作用中的瀑布式對話，而對話中的下一個步驟會使用提示。

當使用者傳送訊息給 Bot 時，其會執行下列作業：

1. Bot 會擷取狀態資訊。
1. Bot 會建立對話內容
    - 如果沒有作用中的對話，其便會啟動對話，除非使用者已進行保留。
    - 如果有作用中的對話，則 Bot 會繼續該對話。 如果對話結束保留，則系統會將詳細資料記錄到狀態快取中。
1. Bot 會儲存狀態的所有變更。

當對話中的步驟呼叫該步驟內容的「提示」方法：

1. 系統會為提示建立新的執行個體、放到對話堆疊上，並加以啟動。 (主要對話則會等到提示結束後再繼續進行)。
1. 提示會傳送活動給使用者，要求他們輸入資料。

當輸入傳送至提示時：

1. 提示會嘗試根據提示類型 (例如，數字提示、選項提示等等) 來處理輸入。
1. 如果提示中包含自訂驗證，則自訂驗證程式碼會開始執行。
1. 如果輸入通過所有驗證，則提示會結束，並傳回已處理的輸入；否則，提示會再次從頭來過。

**處理提示結果**

您對提示結果所做的動作，取決於您為何向使用者要求此資訊。 選項包括：

- 使用此資訊來控制對話流程，例如使用者何時回應確認或選擇提示。
- 快取對話方塊狀態中的資訊，例如在瀑布式步驟內容的 _values_ 屬性中設定一個值，然後在對話結束時傳回所收集的資訊。
- 將資訊儲存到 Bot 狀態。 這需要您設計對話方塊，以取得 Bot 的狀態屬性存取子存取權。

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

```csharp
public async Task OnTurnAsync(
    ITurnContext turnContext,
    CancellationToken cancellationToken = default(CancellationToken))
{
    switch (turnContext.Activity.Type)
    {
        // On a message from the user:
        case ActivityTypes.Message:

            // Get the current reservation info from state.
            var reservation = await _accessors.ReservationAccessor.GetAsync(
                turnContext,
                () => null,
                cancellationToken);

            // Generate a dialog context for our dialog set.
            var dc = await _dialogSet.CreateContextAsync(turnContext, cancellationToken);

            if (dc.ActiveDialog is null)
            {
                // If there is no active dialog, check whether we have a reservation yet.
                if (reservation is null)
                {
                    // If not, start the dialog.
                    await dc.BeginDialogAsync(ReservationDialog, null, cancellationToken);
                }
                else
                {
                    // Otherwise, send a status message.
                    await turnContext.SendActivityAsync(
                        $"We'll see you on {reservation.Date}.",
                        cancellationToken: cancellationToken);
                }
            }
            else
            {
                // Continue the dialog.
                var dialogTurnResult = await dc.ContinueDialogAsync(cancellationToken);

                // If the dialog completed this turn, record the reservation info.
                if (dialogTurnResult.Status is DialogTurnStatus.Complete)
                {
                    reservation = (Reservation)dialogTurnResult.Result;
                    await _accessors.ReservationAccessor.SetAsync(
                        turnContext,
                        reservation,
                        cancellationToken);

                    // Send a confirmation message to the user.
                    await turnContext.SendActivityAsync(
                        $"Your party of {reservation.Size} is confirmed for " +
                        $"{reservation.Date} in {reservation.Location}.",
                        cancellationToken: cancellationToken);
                }
            }

            // Save the updated dialog state into the conversation state.
            await _accessors.ConversationState.SaveChangesAsync(
                turnContext, false, cancellationToken);
            break;
    }
}
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

```javascript
async onTurn(turnContext) {
    switch (turnContext.activity.type) {
        case ActivityTypes.Message:
            // Get the current reservation info from state.
            const reservation = await this.reservationAccessor.get(turnContext, null);

            // Generate a dialog context for our dialog set.
            const dc = await this.dialogSet.createContext(turnContext);

            if (!dc.activeDialog) {
                // If there is no active dialog, check whether we have a reservation yet.
                if (!reservation) {
                    // If not, start the dialog.
                    await dc.beginDialog(RESERVATION_DIALOG);
                }
                else {
                    // Otherwise, send a status message.
                    await turnContext.sendActivity(
                        `We'll see you on ${reservation.date}.`);
                }
            }
            else {
                // Continue the dialog.
                const dialogTurnResult = await dc.continueDialog();

                // If the dialog completed this turn, record the reservation info.
                if (dialogTurnResult.status === DialogTurnStatus.complete) {
                    await this.reservationAccessor.set(
                        turnContext,
                        dialogTurnResult.result);

                    // Send a confirmation message to the user.
                    await turnContext.sendActivity(
                        `Your party of ${dialogTurnResult.result.size} is ` +
                        `confirmed for ${dialogTurnResult.result.date} in ` +
                        `${dialogTurnResult.result.location}.`);
                }
            }

            // Save the updated dialog state into the conversation state.
            await this.conversationState.saveChanges(turnContext, false);
            break;
        case ActivityTypes.EndOfConversation:
        case ActivityTypes.DeleteUserData:
            break;
        default:
            break;
    }
}
```

---

您可以使用類似的技巧，來驗證任何提示類型的提示回應。

## <a name="test-your-bot"></a>測試 Bot

1. 在您的電腦本機執行範例。 如需相關指示，請參閱 [C#](https://aka.ms/dialog-prompt-cs) 或 [JS](https://aka.ms/dialog-prompt-js) 的讀我檔案。
2. 啟動模擬器，傳送如下所示的訊息來測試 Bot。

![測試對話提示範例](~/media/emulator-v4/test-dialog-prompt.png)

## <a name="additional-resources"></a>其他資源

若要直接從回合處理常式呼叫提示，請參閱採用 [C#](https://aka.ms/cs-prompt-validation-sample) 或 [JS](https://aka.ms/js-prompt-validation-sample) 的 _prompt-validations_ 範例。

此對話程式庫也包含可供取得 _OAuth 權杖_的 _OAuth 提示_，該權杖用於存取代表使用者的另一個應用程式。 如需驗證的詳細資訊，請參閱如何[新增驗證](bot-builder-authentication.md)至 Bot。

## <a name="next-steps"></a>後續步驟

> [!div class="nextstepaction"]
> [使用分支和迴圈建立進階的對話流程](bot-builder-dialog-manage-complex-conversation-flow.md)
