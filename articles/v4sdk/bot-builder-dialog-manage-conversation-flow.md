---
title: 使用對話方塊來管理簡單對話流程 | Microsoft Docs
description: 了解如何在適用於 Node.js 的 Bot 建立器 SDK 中，使用對話方塊來管理簡單對話流程。
keywords: 簡單對話流程, 對話方塊, 提示, 瀑布, 對話方塊集合
author: v-ducvo
ms.author: v-ducvo
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 9/25/2018
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 07035c8f0dfc7473192d8c51667ed1f5cefbc555
ms.sourcegitcommit: b78fe3d8dd604c4f7233740658a229e85b8535dd
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/24/2018
ms.locfileid: "49999391"
---
# <a name="manage-simple-conversation-flow-with-dialogs"></a>使用對話方塊來管理簡單對話流程

[!INCLUDE [pre-release-label](../includes/pre-release-label.md)]

您可以使用對話方塊程式庫來管理簡單和複雜的對話流程。 在簡單對話流程中，使用者從「瀑布」的第一個步驟開始，持續進行到最後一個步驟，對話便會完成。 [複雜的對話流程](~/v4sdk/bot-builder-dialog-manage-complex-conversation-flow.md)包含分支和迴圈。

<!-- TODO: This paragraph belongs in a conceptual topic. -->

對話方塊是 Bot 中的結構，其作用類似 Bot 程式中的函式。 對話方塊會建置 Bot 所傳送的訊息，並執行所需的計算工作。 其設計目的是要以特定順序執行特定的作業。 您可以透過不同的方式叫用對話方塊，有時透過給使用者的回應來叫用，有時透過給一些外部刺激的回應來叫用，也可以由其他對話方塊來叫用。

使用對話方塊可讓 Bot 開發人員引導對話流程。 您可以建立多個對話方塊並連結在一起，以建立任何您想要讓 Bot 處理的對話流程。 Bot Builder SDK 中的 **對話方塊** 程式庫包含「提示」、「瀑布對話方塊」和「元件對話方塊」之類的內建功能，可協助您管理對話流程。 您可以使用提示來要求使用者提供不同類型的資訊。 您可以使用瀑布圖將多個步驟結合在序列中。 而且您可以使用元件對話方塊，來建立包含多個子對話方塊的模組化對話方塊系統。

在本文中，我們會使用「對話方塊集合」來建立對話流程，其中包含提示和瀑布。 我們會在來自**多回合提示**的程式碼 [[C#](https://aka.ms/cs-multi-prompts-sample)|[JS](https://aka.ms/js-multi-prompts-sample)] 範例上繪製。

如需對話方塊的概觀，請參閱[對話方塊程式庫](bot-builder-concept-dialog.md)和[對話狀態方塊](bot-builder-dialog-state.md)。

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

若要在一般情況下使用對話方塊，您需要專案或解決方案的 `Microsoft.Bot.Builder.Dialogs` NuGet 套件。

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

若要在一般情況下使用對話方塊，您需要 `botbuilder-dialogs` 程式庫，從 NPM 即可下載。

---

## <a name="using-dialogs-to-guide-the-user-through-steps"></a>使用對話引導使用者執行步驟

在此範例中，我們會建立多步驟對話方塊，以利用對話方塊集合提示使用者輸入資訊。

### <a name="create-a-dialog-with-waterfall-steps"></a>使用瀑布步驟建立對話

**WaterfallDialog** 是特定的對話方塊實作，常用來向使用者收集資訊或引導使用者完成一系列的工作。 對話的每個步驟都會實作為函式。 在每個步驟中，Bot 會[提示](bot-builder-prompts.md)使用者提供輸入、等候回應，然後將結果傳遞給下一個步驟。 第一個函式的結果將作為引數傳遞至下一個函式，依此類推。

例如，下列程式碼範例會定義委派陣列，來代表**瀑布**的步驟。 在每個提示之後，Bot 會認可使用者的輸入。 有許多方法可供您將收集到的輸入保存在對話方塊中。 請參閱[保存使用者資料](bot-builder-tutorial-persist-user-inputs.md)以了解其中的某些選項。

因為會在對話方塊中收集資訊，此範例會將資訊直接寫入到使用者的設定檔。

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

在此範例中，瀑布對話方塊會定義在 Bot 檔案內。

參考這個檔案中所使用的命名空間。

```csharp
using System;
using System.Linq;
using System.Threading;
using System.Threading.Tasks;
using Microsoft.Bot.Builder;
using Microsoft.Bot.Builder.Dialogs;
using Microsoft.Bot.Schema;
```

定義對話方塊集合的執行個體屬性。

```csharp
/// <summary>
/// The <see cref="DialogSet"/> that contains all the Dialogs that can be used at runtime.
/// </summary>
private DialogSet _dialogs;
```

在 Bot 的建構函式內建立對話方塊集合，並在集合中新增提示和瀑布對話方塊。

```csharp
/// <summary>
/// Initializes a new instance of the <see cref="MultiTurnPromptsBot"/> class.
/// </summary>
/// <param name="accessors">A class containing <see cref="IStatePropertyAccessor{T}"/> used to manage state.</param>
public MultiTurnPromptsBot(MultiTurnPromptsBotAccessors accessors)
{
    _accessors = accessors ?? throw new ArgumentNullException(nameof(accessors));

    // The DialogSet needs a DialogState accessor, it will call it when it has a turn context.
    _dialogs = new DialogSet(accessors.ConversationDialogState);

    // This array defines how the Waterfall will execute.
    var waterfallSteps = new WaterfallStep[]
    {
        NameStepAsync,
        NameConfirmStepAsync,
        AgeStepAsync,
        ConfirmStepAsync,
        SummaryStepAsync,
    };

    // Add named dialogs to the DialogSet. These names are saved in the dialog state.
    _dialogs.Add(new WaterfallDialog("details", waterfallSteps));
    _dialogs.Add(new TextPrompt("name"));
    _dialogs.Add(new NumberPrompt<int>("age"));
    _dialogs.Add(new ConfirmPrompt("confirm"));
}
```

然後，將每個步驟定義為不同的方法。 您也可以使用 lambda 運算式定義內嵌步驟。

```csharp
/// <summary>
/// One of the functions that make up the <see cref="WaterfallDialog"/>.
/// </summary>
/// <param name="stepContext">The <see cref="WaterfallStepContext"/> gives access to the executing dialog runtime.</param>
/// <param name="cancellationToken">A <see cref="CancellationToken"/>.</param>
/// <returns>A <see cref="DialogTurnResult"/> to communicate some flow control back to the containing WaterfallDialog.</returns>
private static async Task<DialogTurnResult> NameStepAsync(WaterfallStepContext stepContext, CancellationToken cancellationToken)
{
    // WaterfallStep always finishes with the end of the Waterfall or with another dialog; here it is a Prompt Dialog.
    // Running a prompt here means the next WaterfallStep will be run when the users response is received.
    return await stepContext.PromptAsync("name", new PromptOptions { Prompt = MessageFactory.Text("Please enter your name.") }, cancellationToken);
}

/// <summary>
/// One of the functions that make up the <see cref="WaterfallDialog"/>.
/// </summary>
/// <param name="stepContext">The <see cref="WaterfallStepContext"/> gives access to the executing dialog runtime.</param>
/// <param name="cancellationToken">A <see cref="CancellationToken"/>.</param>
/// <returns>A <see cref="DialogTurnResult"/> to communicate some flow control back to the containing WaterfallDialog.</returns>
private async Task<DialogTurnResult> NameConfirmStepAsync(WaterfallStepContext stepContext, CancellationToken cancellationToken)
{
    // Get the current profile object from user state.
    var userProfile = await _accessors.UserProfile.GetAsync(stepContext.Context, () => new UserProfile(), cancellationToken);

    // Update the profile.
    userProfile.Name = (string)stepContext.Result;

    // We can send messages to the user at any point in the WaterfallStep.
    await stepContext.Context.SendActivityAsync(MessageFactory.Text($"Thanks {stepContext.Result}."), cancellationToken);

    // WaterfallStep always finishes with the end of the Waterfall or with another dialog; here it is a Prompt Dialog.
    return await stepContext.PromptAsync("confirm", new PromptOptions { Prompt = MessageFactory.Text("Would you like to give your age?") }, cancellationToken);
}

/// <summary>
/// One of the functions that make up the <see cref="WaterfallDialog"/>.
/// </summary>
/// <param name="stepContext">The <see cref="WaterfallStepContext"/> gives access to the executing dialog runtime.</param>
/// <param name="cancellationToken">A <see cref="CancellationToken"/>.</param>
/// <returns>A <see cref="DialogTurnResult"/> to communicate some flow control back to the containing WaterfallDialog.</returns>
private async Task<DialogTurnResult> AgeStepAsync(WaterfallStepContext stepContext, CancellationToken cancellationToken)
{
    if ((bool)stepContext.Result)
    {
        // User said "yes" so we will be prompting for the age.

        // Get the current profile object from user state.
        var userProfile = await _accessors.UserProfile.GetAsync(stepContext.Context, () => new UserProfile(), cancellationToken);

        // WaterfallStep always finishes with the end of the Waterfall or with another dialog, here it is a Prompt Dialog.
        return await stepContext.PromptAsync("age", new PromptOptions { Prompt = MessageFactory.Text("Please enter your age.") }, cancellationToken);
    }
    else
    {
        // User said "no" so we will skip the next step. Give -1 as the age.
        return await stepContext.NextAsync(-1, cancellationToken);
    }
}

/// <summary>
/// One of the functions that make up the <see cref="WaterfallDialog"/>.
/// </summary>
/// <param name="stepContext">The <see cref="WaterfallStepContext"/> gives access to the executing dialog runtime.</param>
/// <param name="cancellationToken">A <see cref="CancellationToken"/>.</param>
/// <returns>A <see cref="DialogTurnResult"/> to communicate some flow control back to the containing WaterfallDialog.</returns>
private async Task<DialogTurnResult> ConfirmStepAsync(WaterfallStepContext stepContext, CancellationToken cancellationToken)
{
    // Get the current profile object from user state.
    var userProfile = await _accessors.UserProfile.GetAsync(stepContext.Context, () => new UserProfile(), cancellationToken);

    // Update the profile.
    userProfile.Age = (int)stepContext.Result;

    // We can send messages to the user at any point in the WaterfallStep.
    if (userProfile.Age == -1)
    {
        await stepContext.Context.SendActivityAsync(MessageFactory.Text($"No age given."), cancellationToken);
    }
    else
    {
        // We can send messages to the user at any point in the WaterfallStep.
        await stepContext.Context.SendActivityAsync(MessageFactory.Text($"I have your age as {userProfile.Age}."), cancellationToken);
    }

    // WaterfallStep always finishes with the end of the Waterfall or with another dialog, here it is a Prompt Dialog.
    return await stepContext.PromptAsync("confirm", new PromptOptions { Prompt = MessageFactory.Text("Is this ok?") }, cancellationToken);
}

/// <summary>
/// One of the functions that make up the <see cref="WaterfallDialog"/>.
/// </summary>
/// <param name="stepContext">The <see cref="WaterfallStepContext"/> gives access to the executing dialog runtime.</param>
/// <param name="cancellationToken">A <see cref="CancellationToken"/>.</param>
/// <returns>A <see cref="DialogTurnResult"/> to communicate some flow control back to the containing WaterfallDialog.</returns>
private async Task<DialogTurnResult> SummaryStepAsync(WaterfallStepContext stepContext, CancellationToken cancellationToken)
{
    if ((bool)stepContext.Result)
    {
        // Get the current profile object from user state.
        var userProfile = await _accessors.UserProfile.GetAsync(stepContext.Context, () => new UserProfile(), cancellationToken);

        // We can send messages to the user at any point in the WaterfallStep.
        if (userProfile.Age == -1)
        {
            await stepContext.Context.SendActivityAsync(MessageFactory.Text($"I have your name as {userProfile.Name}."), cancellationToken);
        }
        else
        {
            await stepContext.Context.SendActivityAsync(MessageFactory.Text($"I have your name as {userProfile.Name} and age as {userProfile.Age}."), cancellationToken);
        }
    }
    else
    {
        // We can send messages to the user at any point in the WaterfallStep.
        await stepContext.Context.SendActivityAsync(MessageFactory.Text("Thanks. Your profile will not be kept."), cancellationToken);
    }

    // WaterfallStep always finishes with the end of the Waterfall or with another dialog, here it is the end.
    return await stepContext.EndDialogAsync(cancellationToken: cancellationToken);
}
```

對話方塊會從 Bot 的 on turn 處理常式來執行，其會先建立對話方塊內容並視情況繼續 (continue) 或開始對話方塊，然後在回合結束時儲存對話和使用者狀態。

```csharp
// Run the DialogSet - let the framework identify the current state of the dialog from
// the dialog stack and figure out what (if any) is the active dialog.
var dialogContext = await _dialogs.CreateContextAsync(turnContext, cancellationToken);
var results = await dialogContext.ContinueDialogAsync(cancellationToken);

// If the DialogTurnStatus is Empty we should start a new dialog.
if (results.Status == DialogTurnStatus.Empty)
{
    await dialogContext.BeginDialogAsync("details", null, cancellationToken);
}
```

```csharp
// Save the dialog state into the conversation state.
await _accessors.ConversationState.SaveChangesAsync(turnContext, false, cancellationToken);

// Save the user profile updates into the user state.
await _accessors.UserState.SaveChangesAsync(turnContext, false, cancellationToken);
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

在此範例中，瀑布對話方塊會定義在 **bot.js** 檔案內。

匯入您需要的程式碼物件。

```javascript
const { ActivityTypes } = require('botbuilder');
const { ChoicePrompt, DialogSet, NumberPrompt, TextPrompt, WaterfallDialog } = require('botbuilder-dialogs');
```

在 Bot 的建構函式中定義並建立對話方塊集合，並在集合中新增提示和瀑布對話方塊。

```javascript
/**
*
* @param {ConversationState} conversationState A ConversationState object used to store the dialog state.
* @param {UserState} userState A UserState object used to store values specific to the user.
*/
constructor(conversationState, userState) {
    // Create a new state accessor property. See https://aka.ms/about-bot-state-accessors to learn more about bot state and state accessors.
    this.conversationState = conversationState;
    this.userState = userState;

    this.dialogState = this.conversationState.createProperty(DIALOG_STATE_PROPERTY);

    this.userProfile = this.userState.createProperty(USER_PROFILE_PROPERTY);

    this.dialogs = new DialogSet(this.dialogState);

    // Add prompts that will be used by the main dialogs.
    this.dialogs.add(new TextPrompt(NAME_PROMPT));
    this.dialogs.add(new ChoicePrompt(CONFIRM_PROMPT));
    this.dialogs.add(new NumberPrompt(AGE_PROMPT, async (prompt) => {
        if (prompt.recognized.succeeded) {
            if (prompt.recognized.value <= 0) {
                await prompt.context.sendActivity(`Your age can't be less than zero.`);
                return false;
            } else {
                return true;
            }
        }

        return false;
    }));

    // Create a dialog that asks the user for their name.
    this.dialogs.add(new WaterfallDialog(WHO_ARE_YOU, [
        this.promptForName.bind(this),
        this.confirmAgePrompt.bind(this),
        this.promptForAge.bind(this),
        this.captureAge.bind(this)
    ]));

    // Create a dialog that displays a user name after it has been collected.
    this.dialogs.add(new WaterfallDialog(HELLO_USER, [
        this.displayProfile.bind(this)
    ]));
}
```

然後，將每個步驟定義為不同的方法。 您也可以使用 lambda 運算式定義內嵌步驟。

```javascript
// This step in the dialog prompts the user for their name.
async promptForName(step) {
    return await step.prompt(NAME_PROMPT, `What is your name, human?`);
}

// This step captures the user's name, then prompts whether or not to collect an age.
async confirmAgePrompt(step) {
    const user = await this.userProfile.get(step.context, {});
    user.name = step.result;
    await this.userProfile.set(step.context, user);
    await step.prompt(CONFIRM_PROMPT, 'Do you want to give your age?', ['yes', 'no']);
}

// This step checks the user's response - if yes, the bot will proceed to prompt for age.
// Otherwise, the bot will skip the age step.
async promptForAge(step) {
    if (step.result && step.result.value === 'yes') {
        return await step.prompt(AGE_PROMPT, `What is your age?`,
            {
                retryPrompt: 'Sorry, please specify your age as a positive number or say cancel.'
            }
        );
    } else {
        return await step.next(-1);
    }
}

// This step captures the user's age.
async captureAge(step) {
    const user = await this.userProfile.get(step.context, {});
    if (step.result !== -1) {
        user.age = step.result;
        await this.userProfile.set(step.context, user);
        await step.context.sendActivity(`I will remember that you are ${ step.result } years old.`);
    } else {
        await step.context.sendActivity(`No age given.`);
    }
    return await step.endDialog();
}

// This step displays the captured information back to the user.
async displayProfile(step) {
    const user = await this.userProfile.get(step.context, {});
    if (user.age) {
        await step.context.sendActivity(`Your name is ${ user.name } and you are ${ user.age } years old.`);
    } else {
        await step.context.sendActivity(`Your name is ${ user.name } and you did not share your age.`);
    }
    return await step.endDialog();
}
```

對話方塊會從 Bot 的 on turn 處理常式來執行，其會先建立對話方塊內容並視情況繼續 (continue) 或開始對話方塊，然後在回合結束時儲存對話和使用者狀態。

```javascript
// Create a dialog context object.
const dc = await this.dialogs.createContext(turnContext);
```

```javascript
// If the bot has not yet responded, continue processing the current dialog.
await dc.continueDialog();
```

```javascript
// Start the sample dialog in response to any other input.
if (!turnContext.responded) {
    const user = await this.userProfile.get(dc.context, {});
    if (user.name) {
        await dc.beginDialog(HELLO_USER);
    } else {
        await dc.beginDialog(WHO_ARE_YOU);
    }
}
```

```javascript
// Save changes to the user state.
await this.userState.saveChanges(turnContext);

// End this turn by saving changes to the conversation state.
await this.conversationState.saveChanges(turnContext);
```

---

## <a name="dialog-context-and-waterfall-step-context-objects"></a>對話方塊內容和瀑布步驟內容物件

請使用對話方塊內容物件，從 Bot 的回合處理常式內與對話方塊集合互動。
請使用瀑布步驟內容物件，從瀑布步驟內與對話方塊集合互動。

## <a name="to-start-a-dialog"></a>開始對話方塊

若要開始對話方塊，請將您要開始的 dialogId 傳遞到對話的 beginDialog、prompt 或 replaceDialog 方法。 beginDialog 方法會將對話推上堆疊的頂端，而 replaceDialog 方法會將目前對話方塊從堆疊取出，並將取代的對話方塊推送至堆疊上。

對話方塊內容的 prompt 方法是會採用引數的協助程式方法，並且會建構適當的提示選項，然後開始提示對話。 如需提示的詳細資訊，請參閱[提示使用者進行輸入](bot-builder-prompts.md)。

## <a name="to-end-a-dialog"></a>結束對話方塊

end dialog 方法可將對話方塊從堆疊取出，並將選擇性結果傳回給父代對話，以結束對話。

最好是在對話方塊結束時明確地呼叫 endDialog 方法。

## <a name="to-clear-the-dialog-stack"></a>清除對話方塊堆疊

如果您想要將所有對話方塊從堆疊中移除，您可以藉由呼叫對話方塊內容的 cancel all dialogs 方法來清除對話方塊堆疊。

## <a name="to-repeat-a-dialog"></a>重複對話方塊

若要重複對話方塊，請使用「取代對話方塊」方法，其會將目前的對話方塊從堆疊取出，並將取代的對話方塊推送至堆疊上，開始進行該對話方塊。 這很適合用來處理[複雜對話流程](~/v4sdk/bot-builder-dialog-manage-complex-conversation-flow.md)，也是管理功能表的好方法。

## <a name="next-steps"></a>後續步驟

您現在已了解如何管理簡單對話流程，讓我們看看如何運用「取代對話方塊」方法來處理複雜對話流程。

> [!div class="nextstepaction"]
> [管理複雜交談流程](bot-builder-dialog-manage-complex-conversation-flow.md)
