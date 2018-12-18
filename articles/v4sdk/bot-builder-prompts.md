---
title: 使用對話方塊提示收集使用者輸入 | Microsoft Docs
description: 了解如何在 Bot Builder SDK 中使用對話方塊 程式庫來提示使用者輸入。
keywords: 提示, 提示, 使用者輸入, 對話方塊, AttachmentPrompt, ChoicePrompt, ConfirmPrompt, DatetimePrompt, NumberPrompt, TextPrompt, 重新提示, 驗證
author: JonathanFingold
ms.author: v-jofing
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 11/21/2018
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: a8a0976e6f553e52e13ae13bbb719dd7bdead8f6
ms.sourcegitcommit: 91156d0866316eda8d68454a0c4cd74be5060144
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 12/07/2018
ms.locfileid: "53010536"
---
# <a name="gather-user-input-using-a-dialog-prompt"></a>使用對話方塊提示蒐集使用者輸入

[!INCLUDE [pre-release-label](../includes/pre-release-label.md)]

張貼問題來收集資訊是 Bot 與使用者互動時的其中一種主要方式。 *dialogs* 程式庫可讓您輕鬆地詢問問題，以及驗證回應以確保回應符合特定資料類型或符合自訂驗證規則。 本主題詳細說明如何從瀑布式對話建立並呼叫提示。

## <a name="prerequisites"></a>必要條件

- 本文中的程式碼是以 **DialogPromptBot** 範例為基礎。 您需要採用 [C#](https://aka.ms/dialog-prompt-cs) 或 [JS](https://aka.ms/dialog-prompt-js) 的一份範例。
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

    public string Date { get; set; }
}
```

接下來，新增預訂資料的狀態屬性存取子。

```csharp
public class DialogPromptBotAccessors
{
    public DialogPromptBotAccessors(ConversationState conversationState)
    {
        ConversationState = conversationState ?? throw new ArgumentNullException(nameof(conversationState));
    }

    public static string DialogStateAccessorKey { get; } = "DialogPromptBotAccessors.DialogState";
    public static string ReservationAccessorKey { get; } = "DialogPromptBotAccessors.Reservation";

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

    // Create and register state accesssors.
    // Acessors created here are passed into the IBot-derived class on every turn.
    services.AddSingleton<BotAccessors>(sp =>
    {
        // ...

        // Create the custom state accessor.
        // State accessors enable other components to read and write individual properties of state.
        var accessors = new BotAccessors(conversationState)
        {
            DialogStateAccessor = conversationState.CreateProperty<DialogState>(DialogPromptBotAccessors.DialogStateAccessorKey),
            ReservationAccessor = conversationState.CreateProperty<DialogPromptBot.Reservation>(DialogPromptBotAccessors.ReservationAccessorKey),
        };

        return accessors;
    });
}
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

不需要針對 JavaScript 變更 HTTP 服務程式碼，因此我們可讓 index.js 檔案保持原狀。

在 bot.js 中，我們包含了對話提示 Bot 所需的 `require` 陳述式。 
```javascript
const { ActivityTypes } = require('botbuilder');
const { DialogSet, WaterfallDialog, NumberPrompt, DateTimePrompt, ChoicePrompt, DialogTurnStatus } = require('botbuilder-dialogs');
```

新增狀態屬性存取子、對話和提示的識別碼。

```javascript
// Define identifiers for state property accessors.
const DIALOG_STATE_ACCESSOR = 'dialogStateAccessor';
const RESERVATION_ACCESSOR = 'reservationAccessor';

// Define identifiers for dialogs and prompts.
const RESERVATION_DIALOG = 'reservationDialog';
const PARTY_SIZE_PROMPT = 'partySizePrompt';
const LOCATION_PROMPT = 'locationPrompt';
const RESERVATION_DATE_PROMPT = 'reservationDatePrompt';
```

---

### <a name="create-a-dialog-set-and-prompts"></a>建立對話集和提示

一般而言，在初始化 Bot 時，建立提示和對話，並將其新增到對話集。 然後，當 Bot 收到來自使用者的輸入時，對話方塊集即可解析提示的識別碼。

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

在 `DialogPromptBot` 類別中，定義對話、提示及對話集的識別碼。
```csharp
private const string ReservationDialog = "reservationDialog";
private const string PartySizePrompt = "partyPrompt";
private const string LocationPrompt = "locationPrompt";
private const string ReservationDatePrompt = "reservationDatePrompt";

private readonly DialogSet _dialogSet;
```

在 Bot 的建構函式中，建立對話集、新增提示，以及新增預訂對話。 我們會在建立提示時包含自訂驗證，而且稍後會實作驗證函式。

```csharp
// The following code creates prompts and adds them to an existing dialog set. The DialogSet contains all the dialogs that can 
// be used at runtime. The prompts also references a validation method is not shown here.

public DialogPromptBot(DialogPromptBotAccessors accessors, ILoggerFactory loggerFactory)
{
   // ...

    // Create the dialog set and add the prompts, including custom validation.
    _dialogSet = new DialogSet(_accessors.DialogStateAccessor);
    _dialogSet.Add(new NumberPrompt<int>(PartySizePrompt, PartySizeValidatorAsync));
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

```javascript
constructor(conversationState) {
    // Creates our state accessor properties.
    this.dialogStateAccessor = conversationState.createProperty(DIALOG_STATE_ACCESSOR);
    this.reservationAccessor = conversationState.createProperty(RESERVATION_ACCESSOR);
    this.conversationState = conversationState;
    // ...
    }
```

接下來，建立對話集並新增提示，包括自訂驗證。

```javascript
    // ...
    this.dialogSet = new DialogSet(this.dialogStateAccessor);
    this.dialogSet.add(new NumberPrompt(PARTY_SIZE_PROMPT, this.partySizeValidator));
    this.dialogSet.add(new ChoicePrompt (LOCATION_PROMPT));
    this.dialogSet.add(new DateTimePrompt(RESERVATION_DATE_PROMPT, this.dateValidator));
    // ...
```

然後定義瀑布式對話的步驟，並將其新增至集合。

```javascript
    // ...
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

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

在 DialogPromptBot.cs 檔案中，我們會實作瀑布式對話的 `PromptForPartySizeAsync`、`PromptForLocationAsync`、`PromptForReservationDateAsync` 和 `AcknowledgeReservationAsync` 步驟。

在此我們只會顯示 `PromptForPartySizeAsync` 和 `PromptForLocationAsync`，這是瀑布式對話的兩個連續步驟。

```csharp
private async Task<DialogTurnResult> PromptForPartySizeAsync(WaterfallStepContext stepContext)
{
    // Prompt for the party size. The result of the prompt is returned to the next step of the waterfall.
    // If the input is not valid, the prompt is restarted, causing it to reprompt for input
    // and this set of steps is repeated next turn. Otherwise, the prompt ends and returns a _dialog turn result_ object 
    // to the parent dialog. Control passes to the next step of your waterfall dialog, with the result of the prompt 
    // available in the waterfall step context's _result_ property.
    return await stepContext.PromptAsync(
        PartySizePrompt,
        new PromptOptions
        {
            Prompt = MessageFactory.Text("How many people is the reservation for?"),
            RetryPrompt = MessageFactory.Text("How large is your party?"),
        },
        cancellationToken);
}

private async Task<DialogTurnResult> PromptForLocationAsync(WaterfallStepContext stepContext, CancellationToken cancellationToken)
{
    // Record the party size information in the current dialog state.
    int size = (int)stepContext.Result;
    stepContext.Values["size"] = size;

    return await stepContext.PromptAsync(
        "locationPrompt",
        new PromptOptions
        {
            Prompt = MessageFactory.Text("Please choose a location."),
            RetryPrompt = MessageFactory.Text("Sorry, please choose a location from the list."),
            Choices = ChoiceFactory.ToChoices(new List<string> { "Redmond", "Bellevue", "Seattle" }),
        },
        cancellationToken);
}
```

上述範例示範如何使用選擇提示，並提供三個屬性。 `PromptForLocationAsync` 方法可作為瀑布式對話中的步驟，而我們的對話集包含瀑布式對話，以及識別碼為 `locationPrompt` 的選擇提示。

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

在此，`PARTY_SIZE_PROMPT` 和 `LOCATION_PROMPT` 是提示的識別碼，而 `promptForPartySize` 和 `promptForLocation` 是瀑布式對話的兩個連續步驟函式。

```javascript
async promptForPartySize(stepContext) {
    // Prompt for the party size. The result of the prompt is returned to the next step of the waterfall.
    return await stepContext.prompt(
        PARTY_SIZE_PROMPT, {
            prompt: 'How many people is the reservation for?',
            retryPrompt: 'How large is your party?'
        });
}

async promptForLocation(stepContext) {
    // Prompt for location
    return await stepContext.prompt(
        LOCATION_PROMPT, 'Select a location.', ['Redmond', 'Bellevue', 'Seattle']
    );
}
```

---

_prompt_ 方法的第二個參數會採用「提示選項」物件，其具有下列屬性。

| 屬性 | 說明 |
| :--- | :--- |
| prompt | 要傳送給使用者、要求他們輸入的初始活動。 |
| _retry prompt_ | 如果使用者的第一項輸入未經驗證，要傳送給使用者的活動。 |
| _選擇_ | 可供使用者從中選擇、搭配選擇提示使用的選擇清單。 |

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

提示辨識器結果具有下列屬性：

| 屬性 | 說明 |
| :--- | :--- |
| _已成功_ | 表示辨識器是否能夠剖析輸入。 |
| _值_ | 來自辨識器的傳回值。 如有必要，驗證碼可以修改此值。 |

### <a name="implement-validation-code"></a>實作驗證碼

**派對規模驗證程式**

我們會將預訂限制為 6 到 20 人的派對。

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

```csharp
private async Task<bool> PartySizeValidatorAsync(
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
    int size = promptContext.Recognized.Value;
    if (size < 6 || size > 20)
    {
        await promptContext.Context.SendActivityAsync(
            "Sorry, we can only take reservations for parties of 6 to 20.",
            cancellationToken: cancellationToken);
        return false;
    }

    return true;
}
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

```javascript
async partySizeValidator(promptContext) {
    // Check whether the input could be recognized as an integer.
    if (!promptContext.recognized.succeeded) {
        await promptContext.context.sendActivity(
            "I'm sorry, I do not understand. Please enter the number of people in your party.");
        return false;
    }
    if (promptContext.recognized.value % 1 != 0) {
        await promptContext.context.sendActivity(
            "I'm sorry, I don't understand fractional people.");
        return false;
    }
    // Check whether the party size is appropriate.
    var size = promptContext.recognized.value;
    if (size < 6 || size > 20) {
        await promptContext.context.sendActivity(
            'Sorry, we can only take reservations for parties of 6 to 20.');
        return false;
    }

    return true;
}
```

---

**日期時間驗證**

在預訂日期驗證程式中，我們會將預訂限制為從目前時間算起的一小時或更多小時。 我們會保留符合準則的第一個解析，並且清除其餘解析。 以下的驗證程式碼並不詳盡，最適合用於可剖析為日期和時間的輸入。 驗證碼會示範可供驗證日期時間提示的一些選項，而您的實作會取決於您嘗試向使用者收集哪些資訊。

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

```csharp
// Validates whether the reservation date is appropriate.
// Reservations must be made at least an hour in advance.
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
    DateTime earliest = DateTime.Now.AddHours(1.0);
    DateTimeResolution value = promptContext.Recognized.Value.FirstOrDefault(v =>
        DateTime.TryParse(v.Value ?? v.Start, out DateTime time) && DateTime.Compare(earliest,time) <= 0);
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

### <a name="update-the-turn-handler"></a>更新回合處理常式

更新 Bot 的回合處理常式以開始對話方塊，並且在完成時接受來自對話的傳回值。 我們在此假設使用者正與 Bot 互動，則 Bot 具有作用中的瀑布式對話，而對話中的下一個步驟會使用提示。

1. 當使用者傳送訊息給 Bot 時，其會執行下列作業：
   1. Bot 的回合處理常式會建立對話方塊內容，並呼叫其 _continue_ 方法。
   1. 控制權會傳給作用中對話方塊的下一個步驟，在此例中是您的瀑布式對話方塊。
   1. 此步驟會呼叫其瀑布式步驟內容的 _prompt_ 方法，以要求使用者提供輸入。
   1. 瀑布式步驟內容會將提示推送到堆疊並啟動。
   1. 提示會將活動傳送給使用者，要求他們輸入。
1. 當使用者將其下一則訊息傳送給 Bot 時，其會執行下列作業：
   1. Bot 的回合處理常式會建立對話方塊內容，並呼叫其 _continue_ 方法。
   1. 控制權會傳給作用中對話方塊的下一個步驟，這是提示的第二回合。
   1. 提示會驗證使用者的輸入。

**處理提示結果**

您對提示結果所做的動作，取決於您為何向使用者要求此資訊。 選項包括：

- 使用此資訊來控制對話流程，例如使用者何時回應確認或選擇提示。
- 快取對話方塊狀態中的資訊，例如在瀑布式步驟內容的 _values_ 屬性中設定一個值，然後在對話結束時傳回所收集的資訊。
- 將資訊儲存到 Bot 狀態。 這需要您設計對話方塊，以取得 Bot 的狀態屬性存取子存取權。

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

```csharp
public async Task OnTurnAsync(ITurnContext turnContext, CancellationToken cancellationToken = default(CancellationToken))
{
    switch (turnContext.Activity.Type)
    {
        // On a message from the user:
        case ActivityTypes.Message:

            // Get the current reservation info from state.
            Reservation reservation = await _accessors.ReservationAccessor.GetAsync(
                turnContext, () => null, cancellationToken);

            // Generate a dialog context for our dialog set.
            DialogContext dc = await _dialogSet.CreateContextAsync(turnContext, cancellationToken);

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
                        $"We'll see you {reservation.Date}.",
                        cancellationToken: cancellationToken);
                }
            }
            else
            {
                // Continue the dialog.
                DialogTurnResult dialogTurnResult = await dc.ContinueDialogAsync(cancellationToken);

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
                        $"Your party of {reservation.Size} is confirmed for {reservation.Date}.",
                        cancellationToken: cancellationToken);
                }
            }

            // Save the updated dialog state into the conversation state.
            await _accessors.ConversationState.SaveChangesAsync(turnContext, false, cancellationToken);
            break;

        // Handle other incoming activity types as appropriate to your bot.
        default:
            await turnContext.SendActivityAsync($"{turnContext.Activity.Type} event detected");
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
                        `We'll see you ${reservation.date}.`);
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
                        `confirmed for ${dialogTurnResult.result.date}.`);
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
