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
ms.date: 11/02/2018
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: ec0cc5e942ed66c8683f8b0cc92ba7df2e36db42
ms.sourcegitcommit: 8b7bdbcbb01054f6aeb80d4a65b29177b30e1c20
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 11/14/2018
ms.locfileid: "51645668"
---
# <a name="gather-user-input-using-a-dialog-prompt"></a>使用對話方塊提示蒐集使用者輸入

[!INCLUDE [pre-release-label](../includes/pre-release-label.md)]

張貼問題來收集資訊是 Bot 與使用者互動時的其中一種主要方式。 您可以使用[回合內容](~/v4sdk/bot-builder-basics.md#defining-a-turn)物件的「傳送活動」方法直接這麼做，然後以回應的形式來處理下一個傳入訊息。 不過，Bot Builder SDK 會提供 [dialogs 程式庫](bot-builder-concept-dialog.md)，其所提供的方式經過設計，可讓您更容易地問問題，以及確保回應符合特定資料類型或符合自訂驗證規則。 本主題詳細說明如何使用提示要求使用者提供輸入，來實現這一點。

## <a name="prompt-types"></a>提示類型

在幕後，提示為兩個步驟的對話方塊。 第一步，提示會要求輸入；第二步，會傳回有效值，或利用重新提示從頂端重新開始。

對話方塊 程式庫提供數個基本提示，分別用於收集不同類型的回應。

| Prompt | 說明 | 傳回 |
|:----|:----|:----|
| _附件提示_ | 要求一或多個附件，例如文件或影像。 | 「附件」物件的集合。 |
| _選擇提示_ | 要求從一組選項中選擇。 | 「找到的選擇」物件。 |
| _確認提示_ | 要求確認。 | 布林值。 |
| _日期時間提示_ | 要求日期時間。 | 「日期時間解析」物件的集合。 |
| _數字提示_ | 要求數字。 | 數值。 |
| _文字提示_ | 要求一般文字輸入。 | 字串。 |

此程式庫也包含可供取得 _OAuth 權杖_的 _OAuth 提示_，該權杖用於存取代表使用者的另一個應用程式。 如需驗證的詳細資訊，請參閱如何[將驗證新增至 Bot](bot-builder-authentication.md)。

基本提示可解譯自然語言輸入，例如 "ten" 或 "a dozen" 是指數字，或 "tomorrow" 或 "Friday at 10am" 是指日期時間。

## <a name="using-prompts"></a>使用提示

唯有對話方塊與提示位於相同對話方塊集時，對話方塊才可以使用提示。

1. 為對話方塊狀態定義狀態屬性存取子。
1. 建立對話方塊集。
1. 建立提示，並將其新增至對話方塊集。
1. 建立將使用提示的對話方塊，並將其新增至對話方塊集。
1. 在對話方塊內，新增對提示的呼叫以及擷取提示結果。

本文討論如何建立提示，以及如何從瀑布式對話方塊呼叫。
如需對話方塊的詳細資訊，請參閱[對話方塊程式庫](bot-builder-concept-dialog.md)。
如需使用對話方塊和提示的完整 Bot 討論，請參閱如何[使用對話方塊來管理簡單對話流程](bot-builder-dialog-manage-conversation-flow.md)。

在相同對話方塊集內的一個對話方塊或多個對話方塊中，您可以在多個步驟中使用相同的提示。
不過，您可在初始化階段建立自訂驗證與提示的關聯。
因此如果您需要對同類型的提示進行不同的驗證，您則需要該提示 類型的多個執行個體，而每個執行個體都有自己的驗證碼。

### <a name="create-a-prompt"></a>建立提示

若要提示使用者提供輸入，請使用其中一個內建類別 (例如_文字提示_) 定義提示，然後將其新增至對話方塊集。

* 提示有固定的識別碼。 (識別碼在對話方塊集內必須是唯一的。)
* 提示可以有自訂驗證程式。 (請參閱[自訂驗證](#custom-validation)。)
* 對於某些提示，您可以指定「預設地區設定」。

一般而言，在初始化 Bot 時，建立提示和對話方塊，並將其新增到對話方塊集。 然後，當 Bot 收到來自使用者的輸入時，對話方塊集即可解析提示的識別碼。

例如，下列程式碼會建立兩個文字提示，並將其新增至現有的對話方塊集。 第二個文字提示會參考此處未顯示的驗證方法。

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

在此，`_dialogs` 包含現有的對話方塊集，而 `NameValidator` 是一種驗證方法。

```csharp
_dialogs.Add(new TextPrompt("nickNamePrompt"));
_dialogs.Add(new TextPrompt("namePrompt", NameValidator));
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

在此，`this.dialogs` 包含現有的對話方塊集，而 `NameValidator` 是一個驗證函式。

```javascript
this.dialogs.add(new TextPrompt('nickNamePrompt'));
this.dialogs.add(new TextPrompt('namePrompt', NameValidator));
```

---

#### <a name="locales"></a>Locales

地區設定用來決定**選擇**、**確認**、**日期時間**和**數字**提示的特定語言行為。 對於來自使用者的任何指定輸入：

* 如果通道在使用者的訊息中提供了 _locale_ 屬性，則會使用該屬性。
* 否則，如果在呼叫提示的建構函式時提供提示的「預設地區設定」，或藉由稍後設定，則會使用該預設地區設定。
* 否則，會以英文 ("en-us") 作為地區設定。

> [!NOTE]
> 地區設定是 2、3 或 4 個字元的 ISO 639 代碼，其代表某個語言或語言系列。

### <a name="call-a-prompt-from-a-waterfall-dialog"></a>從瀑布式對話方塊呼叫提示

新增提示後，請在瀑布式對話方塊的一個步驟中呼叫該提示，並在下列對話方塊步驟中取得提示結果。
若要在瀑布式步驟內呼叫提示，請呼叫「瀑布式步驟內容」物件的 _prompt_ 方法。 第一個參數是要使用的提示識別碼，而第二個參數包含提示的選項，例如用來要求使用者提供輸入的文字。

假設使用者正與 Bot 互動，則 Bot 具有作用中的瀑布式對話方塊，而對話方塊中的下一個步驟會使用提示。

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
      * 如果其輸入無效，則會重新啟動提示，而導致其重新提示輸入，而這組步驟會在下一回合重複。
      * 否則，提示會結束並將「對話方塊回合結果」物件傳回給父代對話方塊。 控制權會傳給瀑布式對話方塊的下一個步驟，而在瀑布式步驟內容的 _result_ 屬性中可取得提示結果。

<!--
> [!NOTE]
> A waterfall step delegate takes a _waterfall step context_ parameter and returns a _dialog turn result_.
> A prompt's result is contained within the prompt's return value (a dialog turn result object) when it ends.
> The waterfall dialog provides the return value in the waterfall step context parameter when it calls the next waterfall step.
-->

當提示傳回時，瀑布式步驟內容的 _result_ 屬性會設定為提示的傳回值。

這個範例顯示兩個連續瀑布式步驟的部分。 第一個步驟會使用提示來要求使用者提供其名稱。 第二個步驟會取得提示的傳回值。

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

在此，`name` 是文字提示的識別碼，而 `NameStepAsync` 和 `GreetingStepAsync` 是瀑布式對話方塊的兩個連續步驟委派。

```csharp
private static async Task<DialogTurnResult> NameStepAsync(WaterfallStepContext stepContext, CancellationToken cancellationToken)
{
    // ...

    // Prompt for the user's name.
    return await stepContext.PromptAsync(
        "name",
         new PromptOptions { Prompt = MessageFactory.Text("Please enter your name.") },
         cancellationToken);
}

private static async Task<DialogTurnResult> GreetingStepAsync(WaterfallStepContext stepContext, CancellationToken cancellationToken)
{
    // Get the user's name from the prompt result.
    string name = (string)stepContext.Result;
    await stepContext.Context.SendActivityAsync(
        MessageFactory.Text($"Pleased to meet you, {name}."),
         cancellationToken);

    // ...
}
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

在此，`name` 是文字提示的識別碼，而 `nameStep` 和 `greetingStep` 是瀑布式對話方塊的兩個連續步驟函式。

```javascript
async nameStep(step) {
    // ...

    return await step.prompt('name', 'Please enter your name.');
}

async greetingStep(step) {
    // Get the user's name from the prompt result.
    const name = step.result;
    await step.context.sendActivity(`Pleased to meet you, ${name}.`);

    // ...
}
```

---

### <a name="call-a-prompt-from-the-bots-turn-handler"></a>從 Bot 的回合處理常式呼叫提示

使用對話方塊內容的 _prompt_ 方法，即可直接從回合處理常式呼叫提示。
您必須在下一回合呼叫對話方塊內容的 _continue dialog_ 方法，並檢閱其傳回值 (_dialog turn result_ 物件)。 如需如何執行這項操作的範例，請參閱提示驗證範例 ([C#](https://aka.ms/cs-prompt-validation-sample) | [JS](https://aka.ms/js-prompt-validation-sample))，或請參閱如何[使用自己的提示來提示使用者提供輸入](bot-builder-primitive-prompts.md)以取得替代方法。

## <a name="prompt-options"></a>提示選項

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

此範例示範如何使用選擇提示，並提供三個屬性。 _favorite color_ 方法可作為瀑布式對話方塊中的步驟，而我們的對話方塊集包含瀑布式對話方塊，以及識別碼為 `colorChoice` 的選擇提示。

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

```csharp
private static async Task<DialogTurnResult> FavoriteColorAsync(WaterfallStepContext stepContext, CancellationToken cancellationToken)
{
    // ...

    return await stepContext.PromptAsync(
        "colorChoice",
        new PromptOptions {
            Prompt = MessageFactory.Text("Please choose a color."),
            RetryPrompt = MessageFactory.Text("Sorry, please choose a color from the list."),
            Choices = ChoiceFactory.ToChoices(new List<string> { "blue", "green", "red" }),
        },
        cancellationToken);
}
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

在 JavaScript SDK 中，您可以針對 `prompt` 和 `retryPrompt` 屬性提供字串。 提示會為您將這些屬性轉換為訊息活動。

```javascript
async favoriteColor(step) {
    // ...

    return await step.prompt('colorChoice', {
        prompt: 'Please choose a color:',
        retryPrompt: 'Sorry, please choose a color from the list.',
        choices: [ 'red', 'green', 'blue' ]
    });
}
```

---

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

### <a name="setup"></a>設定

我們必須先進行些許設定，再新增我們的驗證碼。

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

在 Bot 的 **.cs** 檔案中，定義預訂資訊的內部類別。

```csharp
public class Reservation
{
    public int Size { get; set; }

    public string Date { get; set; }
}
```

在 **BotAccessors.cs** 中，新增預訂資料的狀態屬性存取子。

```csharp
public class BotAccessors
{
    public BotAccessors(ConversationState conversationState)
    {
        ConversationState = conversationState ?? throw new ArgumentNullException(nameof(conversationState));
    }

    public static string DialogStateAccessorKey { get; } = "BotAccessors.DialogState";
    public static string ReservationAccessorKey { get; } = "BotAccessors.Reservation";

    public IStatePropertyAccessor<DialogState> DialogStateAccessor { get; set; }
    public IStatePropertyAccessor<ReservationBot.Reservation> ReservationAccessor { get; set; }

    public ConversationState ConversationState { get; }
}
```

在 **Startup.cs** 中，更新 `ConfigureServices` 以設定存取子。

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
            DialogStateAccessor = conversationState.CreateProperty<DialogState>(BotAccessors.DialogStateAccessorKey),
            ReservationAccessor = conversationState.CreateProperty<ReservationBot.Reservation>(BotAccessors.ReservationAccessorKey),
        };

        return accessors;
    });
}
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

不需要針對 JavaScript 變更 HTTP 服務程式碼，因此我們可讓 **index.js** 檔案保持原狀。

在 **bot.js** 中，更新 require 陳述式並新增狀態屬性存取子的識別碼。

```javascript
const { ActivityTypes } = require('botbuilder');
const { DialogSet, WaterfallDialog, NumberPrompt, DateTimePrompt, DialogTurnStatus } = require('botbuilder-dialogs');

// Define identifiers for our state property accessors.
const DIALOG_STATE_ACCESSOR = 'dialogStateAccessor';
const RESERVATION_ACCESSOR = 'reservationAccessor';
```

---

在 Bot 檔案中，新增對話方塊和提示的識別碼。

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

```csharp
// Define identifiers for our dialogs and prompts.
private const string ReservationDialog = "reservationDialog";
private const string PartySizePrompt = "partyPrompt";
private const string ReservationDatePrompt = "reservationDatePrompt";
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

```javascript
// Define identifiers for our dialogs and prompts.
const RESERVATION_DIALOG = 'reservationDialog';
const PARTY_SIZE_PROMPT = 'partySizePrompt';
const RESERVATION_DATE_PROMPT = 'reservationDatePrompt';
```

---

### <a name="define-the-prompts-and-dialogs"></a>定義提示和對話方塊

在 Bot 的建構函式程式碼中，建立對話方塊集、新增提示，以及新增預訂對話方塊。
我們會在建立提示時包含自訂驗證。 我們會接著實作驗證函式。

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

```csharp
public ReservationBot(BotAccessors accessors, ILoggerFactory loggerFactory)
{
    // ...
    _accessors = accessors ?? throw new System.ArgumentNullException(nameof(accessors));

    // Create the dialog set and add the prompts, including custom validation.
    _dialogSet = new DialogSet(_accessors.DialogStateAccessor);
    _dialogSet.Add(new NumberPrompt<int>(PartySizePrompt, PartySizeValidatorAsync));
    _dialogSet.Add(new DateTimePrompt(ReservationDatePrompt, DateValidatorAsync));

    // Define the steps of the waterfall dialog and add it to the set.
    WaterfallStep[] steps = new WaterfallStep[]
    {
        PromptForPartySizeAsync,
        PromptForReservationDateAsync,
        AcknowledgeReservationAsync,
    };
    _dialogSet.Add(new WaterfallDialog(ReservationDialog, steps));
}
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

```javascript
constructor(conversationState) {
    // Creates our state accessor properties.
    // See https://aka.ms/about-bot-state-accessors to learn more about the bot state and state accessors.
    this.dialogStateAccessor = conversationState.createProperty(DIALOG_STATE_ACCESSOR);
    this.reservationAccessor = conversationState.createProperty(RESERVATION_ACCESSOR);
    this.conversationState = conversationState;

    // Create the dialog set and add the prompts, including custom validation.
    this.dialogSet = new DialogSet(this.dialogStateAccessor);
    this.dialogSet.add(new NumberPrompt(PARTY_SIZE_PROMPT, partySizeValidator));
    this.dialogSet.add(new DateTimePrompt(RESERVATION_DATE_PROMPT, dateValidator));

    // Define the steps of the waterfall dialog and add it to the set.
    this.dialogSet.add(new WaterfallDialog(RESERVATION_DIALOG, [
        this.promptForPartySize.bind(this),
        this.promptForReservationDate.bind(this),
        this.acknowledgeReservation.bind(this),
    ]));
}
```

---

### <a name="implement-validation-code"></a>實作驗證碼

實作派對規模驗證程式。 我們會將預訂限制為 6 到 20 人的派對。

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
    // Check whether the input could be recognized as date.
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

日期時間提示會傳回符合使用者輸入的可能「日期時間解析」陣列。 例如，9:00 可能表示 9 AM 或 9 PM，星期日也模稜兩可。 此外，日期時間解析可以代表日期、時間、日期時間或範圍。 日期時間提示會字元使用 [Microsoft/Recognizers-Text](https://github.com/Microsoft/Recognizers-Text) 來剖析使用者輸入。

實作預訂日期驗證程式。 我們會將預訂限制為從目前時間算起的一小時或更多小時。 我們會保留符合準則的第一個解析，並且清除其餘解析。

此驗證碼並不詳盡。 適合用於可剖析為日期和時間的輸入。 驗證碼會示範可供驗證日期時間提示的一些選項，而您的實作會取決於您嘗試向使用者收集哪些資訊。

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

### <a name="implement-the-dialog-steps"></a>實作對話方塊步驟

使用我們新增到對話方塊集的提示。 我們在 Bot 的建構函式中建立提示時，便已新增提示的驗證。 提示第一次要求使用者輸入時，其會從所提供的選項傳送 _prompt_ 活動。 如果驗證失敗，其會傳送 _retry prompt_ 活動，要求使用者提供不同的輸入。

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

```csharp
// First step of the main dialog: prompt for party size.
private async Task<DialogTurnResult> PromptForPartySizeAsync(
    WaterfallStepContext stepContext,
    CancellationToken cancellationToken = default(CancellationToken))
{
    // Prompt for the party size. The result of the prompt is returned to the next step of the waterfall.
    return await stepContext.PromptAsync(
        PartySizePrompt,
        new PromptOptions
        {
            Prompt = MessageFactory.Text("How many people is the reservation for?"),
            RetryPrompt = MessageFactory.Text("How large is your party?"),
        },
        cancellationToken);
}

// Second step of the main dialog: record the party size and prompt for the
// reservation date.
private async Task<DialogTurnResult> PromptForReservationDateAsync(
    WaterfallStepContext stepContext,
    CancellationToken cancellationToken = default(CancellationToken))
{
    // Record the party size information in the current dialog state.
    int size = (int)stepContext.Result;
    stepContext.Values["size"] = size;

    // Prompt for the reservation date. The result of the prompt is returned to the next step of the waterfall.
    return await stepContext.PromptAsync(
        ReservationDatePrompt,
        new PromptOptions
        {
            Prompt = MessageFactory.Text("Great. When will the reservation be for?"),
            RetryPrompt = MessageFactory.Text("What time should we make your reservation for?"),
        },
        cancellationToken);
}

// Third step of the main dialog: return the collected party size and reservation date.
private async Task<DialogTurnResult> AcknowledgeReservationAsync(
    WaterfallStepContext stepContext,
    CancellationToken cancellationToken = default(CancellationToken))
{
    // Retrieve the reservation date.
    DateTimeResolution resolution = (stepContext.Result as IList<DateTimeResolution>).First();
    string time = resolution.Value ?? resolution.Start;

    // Send an acknowledgement to the user.
    await stepContext.Context.SendActivityAsync(
        "Thank you. We will confirm your reservation shortly.",
        cancellationToken: cancellationToken);

    // Return the collected information to the parent context.
    Reservation reservation = new Reservation
    {
        Date = time,
        Size = (int)stepContext.Values["size"],
    };
    return await stepContext.EndDialogAsync(reservation, cancellationToken);
}
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

```javascript
async promptForPartySize(stepContext) {
    // Prompt for the party size. The result of the prompt is returned to the next step of the waterfall.
    return await stepContext.prompt(
        PARTY_SIZE_PROMPT, {
            prompt: 'How many people is the reservation for?',
            retryPrompt: 'How large is your party?'
        });
}

async promptForReservationDate(stepContext) {
    // Record the party size information in the current dialog state.
    stepContext.values.size = stepContext.result;

    // Prompt for the reservation date. The result of the prompt is returned to the next step of the waterfall.
    return await stepContext.prompt(
        RESERVATION_DATE_PROMPT, {
            prompt: 'Great. When will the reservation be for?',
            retryPrompt: 'What time should we make your reservation for?'
        });
}

async acknowledgeReservation(stepContext) {
    // Retrieve the reservation date.
    const resolution = stepContext.result[0];
    const time = resolution.value || resolution.start;

    // Send an acknowledgement to the user.
    await stepContext.context.sendActivity(
        'Thank you. We will confirm your reservation shortly.');

    // Return the collected information to the parent context.
    return await stepContext.endDialog({ date: time, size: stepContext.values.size });
}
```

---

### <a name="update-the-turn-handler"></a>更新回合處理常式

更新 Bot 的回合處理常式以開始對話方塊，並且在完成時接受來自對話的傳回值。

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
        default:
            break;
    }
}
```

---

您可以使用類似的技巧，來驗證任何提示類型的提示回應。

## <a name="handling-prompt-results"></a>處理提示結果

您對提示結果所做的動作，取決於您為何向使用者要求此資訊。 選項包括：

* 使用此資訊來控制對話方塊流程，例如使用者何時回應確認或選擇提示。
* 快取對話方塊狀態中的資訊，例如在瀑布式步驟內容的 _values_ 屬性中設定一個值，然後在對話結束時傳回所收集的資訊。
* 將資訊儲存到 Bot 狀態。 這需要您設計對話方塊，以取得 Bot 的狀態屬性存取子存取權。

## <a name="additional-resources"></a>其他資源
若要深入了解多個提示，請參閱 [[C#](https://aka.ms/cs-multi-prompts-sample) 或 [JS](https://aka.ms/js-multi-prompts-sample)] 中的範例。

## <a name="next-steps"></a>後續步驟

> [!div class="nextstepaction"]
> [實作基本的循序對話流程](bot-builder-dialog-manage-conversation-flow.md)
