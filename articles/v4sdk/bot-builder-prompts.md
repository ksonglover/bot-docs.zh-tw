---
title: 使用對話方塊程式庫提示使用者輸入 | Microsoft Docs
description: 了解如何在適用於 Node.js 的 Bot 建立器 SDK 中使用 Dailogs 程式庫提示使用者輸入。
keywords: 提示, 對話, AttachmentPrompt, ChoicePrompt, ConfirmPrompt, DatetimePrompt, NumberPrompt, TextPrompt, 重新提示, 驗證
author: v-ducvo
ms.author: v-ducvo
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 9/25/2018
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: bd1fe8516cddaf2b75d3c11b469e372265b59be3
ms.sourcegitcommit: b78fe3d8dd604c4f7233740658a229e85b8535dd
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/24/2018
ms.locfileid: "49997625"
---
# <a name="prompt-users-for-input-using-the-dialogs-library"></a>使用 Dialogs 程式庫提示使用者輸入

[!INCLUDE [pre-release-label](../includes/pre-release-label.md)]

張貼問題來收集資訊是 Bot 與使用者互動時的其中一種主要方式。 您可以使用[回合內容](~/v4sdk/bot-builder-basics.md#defining-a-turn)物件的「傳送活動」方法直接這麼做，然後以回應的形式來處理下一個傳入訊息。 不過，Bot Builder SDK 會提供 **對話方塊** 程式庫，其所提供的方式經過設計，可讓您更容易地問問題，以及確保回應符合特定資料類型或符合自訂驗證規則。 本主題詳細說明如何使用**提示**要求使用者輸入，來實現這一點。

本文說明如何在對話內使用提示。 如需在一般情況下使用對話方塊的相關資訊，請參閱[使用對話方塊來管理簡單對話流程](bot-builder-dialog-manage-conversation-flow.md)。

## <a name="prompt-types"></a>提示類型

對話方塊程式庫提供多種不同類型的提示，分別用於收集不同類型的回應。

| Prompt | 說明 |
|:----|:----|
| **AttachmentPrompt** | 提示使用者提供附件，例如文件或影像。 |
| **ChoicePrompt** | 提示使用者從選項集中進行選擇。 |
| **ConfirmPrompt** | 提示使用者確認其動作。 |
| **DatetimePrompt** | 提示使用者提供日期時間。 使用者可以使用「明天晚上 8 點」或「週五早上 10 點」之類的自然語言來回應。 Bot Framework SDK 會使用 LUIS `builtin.datetimeV2` 預先建置的實體。 如需詳細資訊，請參閱 [builtin.datetimev2](https://docs.microsoft.com/azure/cognitive-services/luis/luis-reference-prebuilt-entities#builtindatetimev2)。 |
| **NumberPrompt** | 提示使用者提供數字。 使用者可以使用 "10" 或「十」來回應。 舉例來說，如果回應是「十」，則提示會將回應轉換為數字，並傳回 `10` 作為結果。 |
| **TextPrompt** | 提示使用者提供文字字串。 |

## <a name="add-references-to-prompt-library"></a>將參考新增至提示庫

將 **botbuilder-dialogs** 套件新增至 Bot，即可取得**對話方塊** 程式庫。 我們將在[使用對話方塊來管理簡單對話流程](bot-builder-dialog-manage-conversation-flow.md)中討論對話方塊，但會將對話方塊用於我們的提示中。

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

從 NuGet 安裝 **Microsoft.Bot.Builder.Dialogs** 套件。

然後，納入 Bot 程式碼中程式庫的參考。

```cs
using Microsoft.Bot.Builder.Dialogs;
```

您必須透過存取子設定對話對話方塊狀態。 我們不會在此深入探討這個程式碼，但您可以在[狀態](bot-builder-howto-v4-state.md)一文中了解更多。

在 **Startup.cs** 中的 Bot 選項內，請先定義狀態物件，然後新增單一資料庫以提供存取子類別給 Bot 建構函式。 `BotAccessor` 的類別只會儲存對話與使用者狀態，以及這些項目各自的存取子。 本文結尾處所連結的範例會提供完整類別定義。 

```cs
    services.AddBot<MultiTurnPromptsBot>(options =>
    {
        InitCredentialProvider(options);

        // Create and add conversation state.
        var convoState = new ConversationState(dataStore);
        options.State.Add(convoState);

        // Create and add user state.
        var userState = new UserState(dataStore);
        options.State.Add(userState);
    });

    services.AddSingleton(sp =>
    {
        // We need to grab the conversationState we added on the options in the previous step
        var options = sp.GetRequiredService<IOptions<BotFrameworkOptions>>().Value;
        if (options == null)
        {
            throw new InvalidOperationException("BotFrameworkOptions must be configured prior to setting up the State Accessors");
        }

        var conversationState = options.State.OfType<ConversationState>().FirstOrDefault();
        if (conversationState == null)
        {
            throw new InvalidOperationException("ConversationState must be defined and added before adding conversation-scoped state accessors.");
        }

        var userState = options.State.OfType<UserState>().FirstOrDefault();
        if (userState == null)
        {
            throw new InvalidOperationException("UserState must be defined and added before adding user-scoped state accessors.");
        }

        // The dialogs will need a state store accessor. Creating it here once (on-demand) allows the dependency injection
        // to hand it to our IBot class that is create per-request.
        var accessors = new BotAccessors(conversationState, userState)
        {
            ConversationDialogState = conversationState.CreateProperty<DialogState>("DialogState"),
            UserProfile = userState.CreateProperty<UserProfile>("UserProfile"),
        };

        return accessors;
    });
```

然後，在 Bot 程式碼中定義對話方塊集合的下列物件。

```cs
    private readonly BotAccessors _accessors;

    /// <summary>
    /// The <see cref="DialogSet"/> that contains all the Dialogs that can be used at runtime.
    /// </summary>
    private DialogSet _dialogs;

    /// <summary>
    /// Initializes a new instance of the <see cref="MultiTurnPromptsBot"/> class.
    /// </summary>
    /// <param name="accessors">A class containing <see cref="IStatePropertyAccessor{T}"/> used to manage state.</param>
    public MultiTurnPromptsBot(BotAccessors accessors)
    {
        _accessors = accessors ?? throw new ArgumentNullException(nameof(accessors));

        // The DialogSet needs a DialogState accessor, it will call it when it has a turn context.
        _dialogs = new DialogSet(accessors.ConversationDialogState);

        // ...
        // other constructor items
        // ...
    }
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

使用 Echo 範本建立 JavaScript Bot。 如需詳細資訊，請參閱 [JavaScript 快速入門](../javascript/bot-builder-javascript-quickstart.md)。

從 npm 安裝對話方塊套件：

```cmd
npm install --save botbuilder-dialogs
```

若要在 Bot 中使用**對話**，請將其包含在 Bot 程式碼中。

1. 在 **bot.js** 檔案中，新增下列內容。

    ```javascript
    // Import components from the dialogs library.
    const { DialogSet, TextPrompt, WaterfallDialog } = require("botbuilder-dialogs");

    // Name for the dialog state property accessor.
    const DIALOG_STATE_PROPERTY = 'dialogState';

    // Define the names for the prompts and dialogs for the dialog set.
    const TEXT_PROMPT = 'textPrompt';
    const MAIN_DIALOG = 'mainDialog';
    ```

    「對話方塊集」會包含此 Bot 的對話方塊，而我們會使用「文字提示」要求使用者輸入資料。 我們也需要對話方塊，可用來追蹤其狀態的對話方塊狀態屬性存取子。

1. 更新您的 Bot 建構函式程式碼。 我們很快就會新增更多項目。

    ```javascript
      constructor(conversationState) {
        // Track the conversation state object.
        this.conversationState = conversationState;

        // Create a state property accessor for the dialog set.
        this.dialogState = conversationState.createProperty(DIALOG_STATE_PROPERTY);
    }
    ```

---

## <a name="prompt-the-user"></a>提示使用者

若要提示使用者輸入，請使用其中一個內建類別 (如 **TextPrompt**) 定義提示，然後將其新增至對話方塊集合，再對其指派對方塊話識別碼。

新增提示後，請將其用在有兩個步驟的瀑布對話中，「瀑布」對話方塊可用來定義一系列的步驟。 多個提示可以鏈結在一起，以建立多重步驟的對話。 如需詳細資訊，請參閱[使用對話方塊來管理簡單對話流程](bot-builder-dialog-manage-conversation-flow.md)的[使用對話方塊](bot-builder-dialog-manage-conversation-flow.md#using-dialogs-to-guide-the-user-through-steps)一節。

例如，下列對話方塊會提示使用者提供其名稱，然後使用回應向他們打招呼。 在第一回合，對話方塊會提示使用者輸入自己的名稱。 使用者的回應會作為參數傳遞至第二個步驟函式，由該函式處理輸入並傳送個人化的問候語。

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

您在對話方塊中使用的每個提示也會有指定的名稱，供對話方塊或 Bot 用來存取提示。 在此處的所有範例中，我們都會將提示識別碼公開為常數。

在 Bot 建構函式內，新增雙步驟瀑布的定義，以及供對話方塊使用的提示。 在此，我們要將其新增為獨立函式，但您也可以將其定義為內嵌 lambda (如果想要的話)。

```csharp
 public MultiTurnPromptsBot(BotAccessors accessors)
{
    _accessors = accessors ?? throw new ArgumentNullException(nameof(accessors));

    // The DialogSet needs a DialogState accessor, it will call it when it has a turn context.
    _dialogs = new DialogSet(accessors.ConversationDialogState);

    // This array defines how the Waterfall will execute.
    var waterfallSteps = new WaterfallStep[]
    {
        NameStepAsync,
        SayHiAsync,
    };

    _dialogs.Add(new WaterfallDialog("details", waterfallSteps));
    _dialogs.Add(new TextPrompt("name"));
}
```

然後，在 Bot 內定義兩個瀑布步驟。 針對文字提示，您要指定上面所定義 `TextPrompt` 的「名稱」識別碼。 請注意，方法名稱與上方 `WaterfallStep[]` 的名稱相符。 這裡的未來範例不會包含該程式碼，但您要知道在該 `WaterfallStep[]` 中必須以正確順序新增方法名稱的其他步驟。

```cs
    private static async Task<DialogTurnResult> NameStepAsync(WaterfallStepContext stepContext, CancellationToken cancellationToken)
    {
        // WaterfallStep always finishes with the end of the Waterfall or with another dialog; here it is a Prompt Dialog.
        // Running a prompt here means the next WaterfallStep will be run when the users response is received.
        return await stepContext.PromptAsync("name", new PromptOptions { Prompt = MessageFactory.Text("Please enter your name.") }, cancellationToken);
    }

    private static async Task<DialogTurnResult> SayHiAsync(WaterfallStepContext stepContext, CancellationToken cancellationToken)
    {
        await stepContext.Context.SendActivityAsync($"Hi {stepContext.Result}");

        return await stepContext.EndDialogAsync(cancellationToken);
    }
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

1. 在 Bot 的建構函式中，建立對話方塊集，並且在其中新增文字提示和瀑布對話方塊。

    ```javascript
    // Create the dialog set, and add the prompt and the waterfall dialog.
    this.dialogs = new DialogSet(this.dialogState)
        .add(new TextPrompt(TEXT_PROMPT))
        .add(new WaterfallDialog(MAIN_DIALOG, [
            async (step) => {
                // The results of this prompt will be passed to the next step.
                return await step.prompt(TEXT_PROMPT, 'What is your name?');
            },
            async (step) => {
                // The result property contains the result from the previous step.
                const userName = step.result;
                await step.context.sendActivity(`Hi ${userName}!`);
                return await step.endDialog();
            }
        ]));
    ```

1. 更新 Bot 的回合處理常式以執行對話方塊。

    ```javascript
    async onTurn(turnContext) {
        // See https://aka.ms/about-bot-activity-message to learn more about the message and other activity types.
        if (turnContext.activity.type === ActivityTypes.Message) {
            // Create a dialog context for the dialog set.
            const dc = await this.dialogs.createContext(turnContext);
            // Continue the dialog if it's active.
            await dc.continueDialog();
            if (!turnContext.responded) {
                // Otherwise, start the dialog.
                await dc.beginDialog(MAIN_DIALOG);
            }
        } else {
            // Send a default message for activity types that we don't handle.
            await turnContext.sendActivity(`[${turnContext.activity.type} event detected]`);
        }
        // Save state changes
        await this.conversationState.saveChanges(turnContext);
        }
    }
    ```

---

> [!NOTE]
> 若要開始對話方塊，請取得對話方塊內容，並使用其 _begin dialog_ 方法。 如需詳細資訊，請參閱[使用對話方塊來管理簡單對話流程](./bot-builder-dialog-manage-conversation-flow.md)。

## <a name="reusable-prompts"></a>可重複使用的提示

只要答案屬於相同類型，便可重複使用提示來提出不同問題。 例如，上述範例程式碼定義了文字提示，並將它用來要求使用者提供其名稱。 您也可以使用這個相同的提示來要求使用者提供其他文字字串，例如「您在哪裡工作？」。

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

在此範例中，文字提示的識別碼「名稱」並無法讓程式碼變得清楚。 不過，卻是理想的範例，可讓您知道提示識別碼可以是任何所選項目。

現在，我們的方法包含了要詢問使用者工作位置的第三個步驟。

```cs
    private static async Task<DialogTurnResult> NameStepAsync(WaterfallStepContext stepContext, CancellationToken cancellationToken)
    {
        // WaterfallStep always finishes with the end of the Waterfall or with another dialog; here it is a Prompt Dialog.
        // Running a prompt here means the next WaterfallStep will be run when the users response is received.
        return await stepContext.PromptAsync("name", new PromptOptions { Prompt = MessageFactory.Text("Please enter your name.") }, cancellationToken);
    }

    private static async Task<DialogTurnResult> WorkAsync(WaterfallStepContext stepContext, CancellationToken cancellationToken)
    {
        await stepContext.Context.SendActivityAsync($"Hi {stepContext.Result}!");

        return await stepContext.PromptAsync("name", new PromptOptions { Prompt = MessageFactory.Text("Where do you work?") }, cancellationToken);
    }

    private static async Task<DialogTurnResult> SayHiAsync(WaterfallStepContext stepContext, CancellationToken cancellationToken)
    {
        await stepContext.Context.SendActivityAsync($"{stepContext.Result} is a cool place!");

        return await stepContext.EndDialogAsync(cancellationToken);
    }
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

在 Bot 的建構函式中，修改瀑布以提出第二個問題。

```javascript
// Create the dialog set, and add the prompt and the waterfall dialog.
this.dialogs = new DialogSet(this.dialogState)
    .add(new TextPrompt(TEXT_PROMPT))
    .add(new WaterfallDialog(MAIN_DIALOG, [
    async (step) => {
        // Ask the user for their name.
        return await step.prompt(TEXT_PROMPT, 'What is your name?');
    },
    async (step) => {
        // Acknowledge their response and ask for their place of work.
        const userName = step.result;
        return await step.prompt(TEXT_PROMPT, `Hi ${userName}; where do you work?`);
    },
    async (step) => {
        // Acknowledge their response and exit the dialog.
        const workPlace = step.result;
        await step.context.sendActivity(`${workPlace} is a cool place!`);
        return await step.endDialog();
    }
    ]));
```

---

如果您需要使用多個不同的提示，請讓每個提示具有唯一的 dialogId。 每個新增至對話方塊集合的對話或提示都需要唯一的識別碼。 您也可以建立多個相同類型的**提示**對話方塊。 例如，您可以為上述範例建立兩個 **TextPrompt** 對話方塊：

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

```cs
_dialogs.Add(new WaterfallDialog("details", waterfallSteps));
_dialogs.Add(new TextPrompt("name"));
_dialogs.Add(new TextPrompt("workplace"));
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

比方說，您可以取代此項。

```javascript
.add(new TextPrompt(TEXT_PROMPT))
```

取代下列項目。

```javascript
.add(new TextPrompt('namePrompt'))
.add(new TextPrompt('workPlacePrompt'))
```

然後更新個別的瀑布步驟，依其各自的名稱使用這些提示。

---

為了要重複使用程式碼，為所有提示定義單一 `TextPrompt` 是可行的，因為這些提示全都應該以文字作為回應。 在您需要對提示的輸入套用不同的驗證規則時，為對話方塊命名的功能才會真的派上用場。 讓我們看看如何使用 `NumberPrompt` 來驗證提示回應。

## <a name="specify-prompt-options"></a>指定提示選項

當您在對話方塊步驟中使用提示時，您也可以提供提示選項，例如重新提示字串。

如果使用者輸入因其格式無法由提示剖析 (例如對數字提示輸入「明天」)，或因輸入不符合驗證準則，而無法滿足提示，則指定重新提示字串將有其效用。 數字提示可解譯多種不同的輸入，例如「十二」或「四分之一」，以及 "12" 和 "0.25"。

地區設定是某些提示上的選擇性參數，例如 **NumberPrompt**。 這可幫助提示更精確地剖析輸入，但並非必要項目。

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

下列程式碼會將數字提示，新增至現有的對話方塊集合 **_dialogs** 中。

```csharp
_dialogs.Add(new NumberPrompt<int>("age"));
```

在對話方塊步驟中，下列程式碼會提示使用者輸入，並提供重新提示字串以備其輸入無法解譯為數字時使用。

```csharp
return await stepContext.PromptAsync(
    "age",
    new PromptOptions {
        Prompt = MessageFactory.Text("Please enter your age."),
        RetryPrompt = MessageFactory.Text("I didn't get that. Please enter a valid age."),
    },
    cancellationToken);
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

從 dialogs 程式庫匯入 `NumberPrompt` 類別。

```javascript
const { NumberPrompt } = require("botbuilder-dialogs");
```

使用您瀑布對話方塊中的數字提示，並指定初始和重試提示字串。

```javascript
// Create the dialog set, and add the prompt and the waterfall dialog.
this.dialogs = new DialogSet(this.dialogState)
    .add(new NumberPrompt('partySize'))
    .add(new WaterfallDialog(MAIN_DIALOG, [
    async (step) => {
        // Ask the user for their party size.
        return await step.prompt('partySize', {
            prompt: 'How many people in your party?',
            retryPrompt: 'Sorry, please specify the number of people in your party.'
        });
    },
    async (step) => {
        // Acknowledge their response and exit the dialog.
        const partySize = step.result;
        await step.context.sendActivity(`That's a party of ${partySize}, thanks.`);
        return await step.endDialog();
    }
]));
```

---

選擇提示有額外的必要參數：可供使用者使用的選擇清單。

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

當我們使用 **ChoicePrompt** 要求使用者從一組選項中選擇時，我們必須在提示中附上在 **PromptOptions** 物件內提供的該組選項。 在此，我們使用 **ChoiceFactory** 將選項清單轉換為適當的格式。

```csharp
private static async Task<DialogTurnResult> FavoriteColorAsync(WaterfallStepContext stepContext, CancellationToken cancellationToken)
{
    await stepContext.Context.SendActivityAsync($"Hi {stepContext.Result}!");

    return await stepContext.PromptAsync(
        "color",
        new PromptOptions {
            Prompt = MessageFactory.Text("What's your favorite color?"),
            Choices = ChoiceFactory.ToChoices(new List<string> { "blue", "green", "red" }),
        },
        cancellationToken);
}
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

從 dialogs 程式庫匯入 `NumberPrompt` 類別。

```javascript
const { ChoicePrompt } = require("botbuilder-dialogs");
```

在您的瀑布對話方塊中使用選擇提示字元，並指定可用的選項。

```javascript
// Create the dialog set, and add the prompt and the waterfall dialog.
const list = ['green', 'blue', 'red', 'yellow'];
this.dialogs = new DialogSet(this.dialogState)
    .add(new ChoicePrompt('choicePrompt'))
    .add(new WaterfallDialog(MAIN_DIALOG, [
    async (step) => {
        // Ask the user for their party size.
        return await step.prompt('choicePrompt', {
            prompt: 'Please choose a color:',
            retryPrompt: 'Sorry, please choose a color from the list.',
            choices: list
        });
    },
    async (step) => {
        // Acknowledge their response and exit the dialog.
        const choice = step.result;
        await step.context.sendActivity(`That's ${choice.value}, thanks.`);
        return await step.endDialog();
    }
]));
```

---

## <a name="validate-a-prompt-response"></a>驗證提示回應

您可以先驗證提示回應，再將值傳回至**瀑布**的下一個步驟。 例如，若要驗證 **6** 到 **20** 數字範圍內的 **NumberPrompt**，您可以納入如下的驗證函式：

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

在提示新增至對話方塊集合時變更，以包含驗證程式函式

```cs
_dialogs.Add(new NumberPrompt<int>("partySize", PartySizeValidatorAsync));
```

驗證隨即會定義為自己的方法，根據其是否通過驗證來指出 true 或 false。 如果傳回 false，則會重新提示使用者。

```cs
private Task<bool> PartySizeValidatorAsync(PromptValidatorContext<int> promptContext, CancellationToken cancellationToken)
{
    var result = promptContext.Recognized.Value;

    if (result < 6 || result > 20)
    {
        return Task.FromResult(false);
    }

    return Task.FromResult(true);
}
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

在您建立提示字元時新增驗證方法。

```javascript
// Create the dialog set, and add the prompt and the waterfall dialog.
this.dialogs = new DialogSet(this.dialogState)
    .add(new NumberPrompt('partySizePrompt', async (promptContext) =>                                                 {
        // Check to make sure a value was recognized.
        if (promptContext.recognized.succeeded) {
            const value = promptContext.recognized.value;
            try {
                if (value < 6) {
                    throw new Error('Party size too small.');
                } else if (value > 20) {
                    throw new Error('Party size too big.')
                } else {
                    return true; // Indicate that this is a valid value.
                }
            } catch (err) {
                await promptContext.context.sendActivity(`${err.message} <br/>Please provide a valid number between 6 and 20.`);
                return false; // Indicate that this is invalid.
            }
        } else {
            return false;
        }
    }))
    .add(new WaterfallDialog(MAIN_DIALOG, [
        async (step) => {
            // Ask the user for their party size.
            return await step.prompt('partySizePrompt', {
                prompt: 'How large is your party?',
                retryPrompt: 'Sorry, please specify a size between 6 and 20.'
            });
        },
        async (step) => {
            // Acknowledge their response and exit the dialog.
            const size = step.result;
            await step.context.sendActivity(`That's a party of ${size}, thanks.`);
            return await step.endDialog();
        }
    ]));
```

---

同樣地，如果您想要對未來的日期和時間驗證 **DatetimePrompt** 回應，您可以使用如下的驗證邏輯：

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

```cs
    private Task<bool> DateTimeValidatorAsync(PromptValidatorContext<IList<DateTimeResolution>> prompt, CancellationToken cancellationToken)
    {
        if (prompt.Recognized.Succeeded)
        {
            var resolution = prompt.Recognized.Value.First();

            // Verify that the Timex received is within the desired bounds, compared to today.
            var now = DateTime.Now;
            DateTime.TryParse(resolution.Value, out var time);

            if (time < now)
            {
                return Task.FromResult(false);
            }

            return Task.FromResult(true);
        }

        return Task.FromResult(false);
    }
```

```csharp
_dialogs.Add(new DateTimePrompt("date", DateTimeValidatorAsync));
```

其他範例可在[範例存放庫](https://aka.ms/bot-samples-readme)中找到。

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

```javascript
const { DateTimePrompt } = require("botbuilder-dialogs");
```

```JavaScript
// Create the dialog set, and add the prompt and the waterfall dialog.
this.dialogs = new DialogSet(this.dialogState)
    .add(new DateTimePrompt('dateTimePrompt', async (promptContext) => {
        try {
            if (!promptContext.recognized.succeeded) { throw new Error('Value not recognized.') }
            const values = promptContext.recognized.value;
            if (!Array.isArray(values) || values.length < 0) { throw new Error('Value missing.'); }
            if ((values[0].type !== 'datetime') && (values[0].type !== 'date')) { throw new Error('Unsupported type.'); }
            const now = new Date();
            const value = new Date(values[0].value);
            if (value.getTime() < now.getTime()) { throw new Error('Value in the past.') }

            // update the return value of the prompt to be a real date object
            promptContext.recognized.value = [value];
            return true; // indicate valid
        } catch (err) {
            await promptContext.context.sendActivity(`${err} Please specify a date or a date and time in the future, like tomorrow at 9am.`);
            return false; // indicate invalid
        }
    }))
    .add(new WaterfallDialog(MAIN_DIALOG, [
        async (step) => {
            // Ask the user for their party size.
            return await step.prompt('dateTimePrompt', 'When would you like to schedule that for?');
        },
        async (step) => {
            // Acknowledge their response and exit the dialog.
            const time = step.result;
            await step.context.sendActivity(`That's ${time}, thanks.`);
            return await step.endDialog();
        }
    ]));
```

其他範例可在[範例存放庫](https://aka.ms/bot-samples-readme)中找到。

---

> [!TIP]
> 如果使用者提供了模稜兩可的答案，日期時間提示可能會解析成數個不同的日期。 根據提示的用途，您可能會想要檢查提示結果所提供的所有解析，而不只是第一個解析。

您可以使用類似的技巧，來驗證任何提示類型的提示回應。

## <a name="save-user-data"></a>儲存使用者資料

當您提示使用者輸入時，您可以選擇數種不同的方式來處理該輸入。 例如，您可以取用和捨棄輸入、將其儲存至全域變數、將其儲存至揮發性記憶體或記憶體中的儲存體容器、將其儲存至檔案，或是，將其儲存至外部資料庫。 如需如何儲存使用者資料的詳細資訊，請參閱[管理使用者資料](bot-builder-howto-v4-state.md)。

## <a name="additional-resources"></a>其他資源

如需使用了其中某些提示的完整範例，請參閱 [C#](https://aka.ms/cs-multi-prompts-sample) 或 [JavaScript](https://aka.ms/js-multi-prompts-sample) 的多回合提示 Bot。

## <a name="next-steps"></a>後續步驟

現在您已了解如何提示使用者輸入，接下來我們將透過對話方塊來管理各種對話流程，以強化 Bot 程式碼並提升使用者體驗。

> [!div class="nextstepaction"]
> [使用對話方塊來管理簡單對話流程](bot-builder-dialog-manage-conversation-flow.md)
