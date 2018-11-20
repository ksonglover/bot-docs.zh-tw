---
title: 實作循序對話流程 | Microsoft Docs
description: 了解如何在適用於 Node.js 的 Bot 建立器 SDK 中，使用對話方塊來管理簡單對話流程。
keywords: 簡單對話流程, 循序對話流程, 對話, 提示, 瀑布, 對話方塊集
author: JonathanFingold
ms.author: v-jofing
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 11/13/2018
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 06eb7d80ca8baa91c619b31dc61c7f78856a3b7c
ms.sourcegitcommit: 873361802bd1802f745544ba903aecf658cce639
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 11/13/2018
ms.locfileid: "51611045"
---
# <a name="implement-sequential-conversation-flow"></a>實作循序對話流程

[!INCLUDE [pre-release-label](../includes/pre-release-label.md)]

您可以使用對話方塊程式庫來管理簡單和複雜的對話流程。

在簡單互動中，Bot 透過一連串固定的步驟執行，然後完成對話。
在本文中，我們會使用「瀑布式對話方塊」一些「提示」和一個「對話方塊集」來建立簡單互動，以詢問使用者一系列的問題。
我們會在來自**多回合提示**的程式碼 [[C#](https://aka.ms/cs-multi-prompts-sample) | [JS](https://aka.ms/js-multi-prompts-sample)] 範例上繪製。

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

若要在一般情況下使用對話方塊，您需要專案或解決方案的 `Microsoft.Bot.Builder.Dialogs` NuGet 套件。

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

若要在一般情況下使用對話方塊，您需要 `botbuilder-dialogs` 程式庫，透過 npm 即可下載。

若要安裝此套件並將其儲存為相依性，請瀏覽至您專案的目錄並使用此命令。

```shell
npm install botbuilder-dialogs --save
```

---
下列各節反映您對大部分 Bot 實作簡單對話方塊時所會採取的步驟：

## <a name="configure-your-bot"></a>設定 Bot

我們需要已指派給對話方塊集的狀態屬性存取子，以便 Bot 用來管理[對話方塊狀態](bot-builder-dialog-state.md)。

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

我們會針對 **Startup.cs** 檔案的組態程式碼中 Bot 的對話方塊狀態，初始化狀態屬性存取子。

我們會定義 `MultiTurnPromptsBotAccessors` 類別，以保存 Bot 的狀態管理物件和狀態屬性存取子。
在此，我們只會呼叫部分的程式碼。

```csharp
public class MultiTurnPromptsBotAccessors
{
    // Initializes a new instance of the class.
    public MultiTurnPromptsBotAccessors(ConversationState conversationState, UserState userState)
    {
        ConversationState = conversationState ?? throw new ArgumentNullException(nameof(conversationState));
        UserState = userState ?? throw new ArgumentNullException(nameof(userState));
    }

    public IStatePropertyAccessor<DialogState> ConversationDialogState { get; set; }
    public IStatePropertyAccessor<UserProfile> UserProfile { get; set; }

    public ConversationState ConversationState { get; }
    public UserState UserState { get; }
}
```

我們會在 `Statup` 類別的 `ConfigureServices` 方法中註冊存取子類別。
同樣地，我們只會呼叫部分的程式碼。

```csharp
public void ConfigureServices(IServiceCollection services)
{
    // ...

    // Create and register state accessors.
    // Accessors created here are passed into the IBot-derived class on every turn.
    services.AddSingleton<MultiTurnPromptsBotAccessors>(sp =>
    {
        // We need to grab the conversationState we added on the options in the previous step
        var options = sp.GetRequiredService<IOptions<BotFrameworkOptions>>().Value;
        var conversationState = options.State.OfType<ConversationState>().FirstOrDefault();
        var userState = options.State.OfType<UserState>().FirstOrDefault();

        // Create the custom state accessor.
        // State accessors enable other components to read and write individual properties of state.
        var accessors = new MultiTurnPromptsBotAccessors(conversationState, userState)
        {
            ConversationDialogState = conversationState.CreateProperty<DialogState>("DialogState"),
            UserProfile = userState.CreateProperty<UserProfile>("UserProfile"),
        };

        return accessors;
    });
}
```

透過相依性插入，存取子可供 Bot 的建構函式程式碼使用。

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

在 **index.js** 檔案中，我們會定義狀態管理物件。
在此，我們只會呼叫部分的程式碼。

```javascript
// Import required bot services. See https://aka.ms/bot-services to learn more about the different part of a bot.
const { BotFrameworkAdapter, ConversationState, MemoryStorage, UserState } = require('botbuilder');

// Define the state store for your bot. See https://aka.ms/about-bot-state to learn more about using MemoryStorage.
// A bot requires a state storage system to persist the dialog and user state between messages.
const memoryStorage = new MemoryStorage();

// Create conversation state with in-memory storage provider.
const conversationState = new ConversationState(memoryStorage);
const userState = new UserState(memoryStorage);

// Create the main dialog, which serves as the bot's main handler.
const bot = new MultiTurnBot(conversationState, userState);
```

Bot 的建構函式會為 Bot 建立狀態屬性存取子：`this.dialogState` 和 `this.userProfile`。

---

## <a name="update-the-bot-turn-handler-to-call-the-dialog"></a>更新 Bot 回合處理常式以呼叫對話方塊

若要執行對話方塊，Bot 的回合處理常式必須針對包含 Bot 對話的對話方塊集，建立對話方塊內容。 (Bot 可以定義多個對話方塊集，但一般而言，您應該只能為您的 Bot 義一個對話方塊集。 [對話方塊程式庫](bot-builder-concept-dialog.md)描述對話方塊的重要層面。)

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

對話方塊是從 Bot 的回合處理常式執行。 處理常式會先建立 `DialogContext` 並繼續進行作用中對話方塊，或視需要開始新的對話方塊。 然後，處理常式會在回合結束時儲存對話和使用者狀態。

在 `MultiTurnPromptsBot` 類別中，我們已定義了 `_dialogs` 屬性，其中包含我們會從中產生對話內容的對話方塊集。 同樣地，我們只會在此顯示部分的回合處理常式程式碼。

```csharp
public async Task OnTurnAsync(ITurnContext turnContext, CancellationToken cancellationToken = default(CancellationToken))
{
    // ...
    if (turnContext.Activity.Type == ActivityTypes.Message)
    {
        // Run the DialogSet - let the framework identify the current state of the dialog from
        // the dialog stack and figure out what (if any) is the active dialog.
        var dialogContext = await _dialogs.CreateContextAsync(turnContext, cancellationToken);
        var results = await dialogContext.ContinueDialogAsync(cancellationToken);

        // If the DialogTurnStatus is Empty we should start a new dialog.
        if (results.Status == DialogTurnStatus.Empty)
        {
            await dialogContext.BeginDialogAsync("details", null, cancellationToken);
        }
    }

    // ...
    // Save the dialog state into the conversation state.
    await _accessors.ConversationState.SaveChangesAsync(turnContext, false, cancellationToken);

    // Save the user profile updates into the user state.
    await _accessors.UserState.SaveChangesAsync(turnContext, false, cancellationToken);
}
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

Bot 程式碼會使用對話方塊程式庫中的一些類別。

```javascript
const { ChoicePrompt, DialogSet, NumberPrompt, TextPrompt, WaterfallDialog } = require('botbuilder-dialogs');
```

對話方塊是從 Bot 的回合處理常式執行。 處理常式會先建立 `DialogContext` (`dc`) 並繼續進行作用中對話方塊，或視需要開始新的對話方塊。 然後，處理常式會在回合結束時儲存對話方塊和使用者狀態。

`MultiTurnBot` 類別會定義於 **bot.js** 檔案中。 這個類別的建構函式會新增對話方塊集的 `dialogs` 屬性，我們會從中產生對話方塊內容。 此 Bot 會收集使用者資料一次，並 使用 `WHO_ARE_YOU` 對話方塊。 一旦已填入使用者設定檔，Bot 就會使用 `HELLO_USER` 對話方塊來回應。 同樣地，我們只會在此顯示部分的回合處理常式程式碼。

```javascript
async onTurn(turnContext) {
    if (turnContext.activity.type === ActivityTypes.Message) {
        // Create a dialog context object.
        const dc = await this.dialogs.createContext(turnContext);

        const utterance = (turnContext.activity.text || '').trim().toLowerCase();

        // ...
        // If the bot has not yet responded, continue processing the current dialog.
        await dc.continueDialog();

        // Start the sample dialog in response to any other input.
        if (!turnContext.responded) {
            const user = await this.userProfile.get(dc.context, {});
            if (user.name) {
                await dc.beginDialog(HELLO_USER);
            } else {
                await dc.beginDialog(WHO_ARE_YOU);
            }
        }
    }

    // ...
    // Save changes to the user state.
    await this.userState.saveChanges(turnContext);

    // End this turn by saving changes to the conversation state.
    await this.conversationState.saveChanges(turnContext);
}
```

---

在 Bot 的回合處理常式中，我們會建立對話方塊集的對話方塊內容。 對話方塊內容會存取 Bot 的狀態快取，有效地記住對話中最後一個回合停止的地方。

如果有作用中對話方塊，則對話方塊內容的「繼續對話方塊」方法會使用觸發此回合的使用者輸入來推進對話方塊；否則 Bot 會呼叫對話方塊內容的「開始對話方塊」方法來開始對話方塊。

最後，我們會在狀態管理物件上呼叫「儲存變更」方法，以保存這一回合發生的任何變更。

### <a name="about-dialog-and-bot-state"></a>關於對話方塊和 Bot 狀態

在此 Bot 中，我們已定義兩個狀態屬性存取子：

* 一個建立於對話狀態屬性的對話方塊狀態內。 對話方塊狀態會追蹤使用者在對話方塊集中對話方塊的位置，並由對話方塊內容更新，例如當我們呼叫開始對話方塊或繼續對話方塊方法時。
* 一個建立於使用者設定檔屬性的使用者狀態內。 Bot 會使用此項來追蹤其使用者的相關資訊，而我們會在 Bot 程式碼中明確地管理此狀態。

狀態屬性存取子的 _get_ 和 _set_ 方法，會取得及設定狀態管理物件的快取中的屬性值。 快取會第一次填入回合中要求的狀態屬性值，但必須明確地保存。 為了保存這兩個狀態屬性的變更，我們會呼叫對應狀態管理物件的「儲存變更」方法。

如需詳細資訊，請參閱[對話方塊狀態](bot-builder-dialog-state.md)。

## <a name="initialize-your-bot-and-define-your-dialog"></a>初始化 Bot 並定義對話方塊

我們的簡單對話會塑造為一系列對使用者提出的問題。 C# 和 JavaScript 版本的步驟稍有不同：

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

1. 詢問他們的名稱。
1. 詢問他們是否願意提供其年齡。
1. 若是如此，詢問其年齡，否則略過此步驟。
1. 詢問所蒐集的資訊是否正確。
1. 傳送狀態訊息並結束。

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

針對 `who_are_you` 對話方塊：

1. 詢問他們的名稱。
1. 詢問他們是否願意提供其年齡。
1. 若是如此，詢問其年齡，否則略過此步驟。
1. 傳送狀態訊息並結束。

針對 `hello_user` 對話方塊：

1. 顯示 Bot 已蒐集的使用者資訊。

---

以下是在定義自己的瀑布式步驟時，所要記住的幾個事項。

* 每個 Bot 回合都會反映使用者的輸入，後面接著來自 Bot 的回應。 因此，您會在瀑布式步驟的結尾要求使用者輸入，並且在下一個瀑布式步驟中接收其答案。
* 每個提示實際上是兩個步驟的對話方塊，該對話方塊會顯示其提示並執行迴圈，直到其收到「有效」輸入為止。 (您可以依賴每種提示類型的內建驗證，也可以將自己的自訂驗證新增至提示。 如需詳細資訊，請參閱[取得使用者輸入](bot-builder-prompts.md)。)

在此範例中，對話會定義於 Bot 檔案內，並在 Bot 的建構函式中初始化。

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

定義對話方塊集合的執行個體屬性。

```csharp
// The DialogSet that contains all the Dialogs that can be used at runtime.
private DialogSet _dialogs;
```

在 Bot 的建構函式內建立對話方塊集合，並在集合中新增提示和瀑布對話方塊。

```csharp
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

在此範例中，我們將每個步驟定義為不同的方法。 您也可以使用 lambda 運算式在建構函式中定義內嵌步驟。

```csharp
private static async Task<DialogTurnResult> NameStepAsync(WaterfallStepContext stepContext, CancellationToken cancellationToken)
{
    // WaterfallStep always finishes with the end of the Waterfall or with another dialog; here it is a Prompt Dialog.
    // Running a prompt here means the next WaterfallStep will be run when the users response is received.
    return await stepContext.PromptAsync("name", new PromptOptions { Prompt = MessageFactory.Text("Please enter your name.") }, cancellationToken);
}

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

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

在此範例中，瀑布對話方塊會定義在 **bot.js** 檔案內。

定義要用於狀態屬性存取子、提示和對話方塊的識別碼。

```javascript
const DIALOG_STATE_PROPERTY = 'dialogState';
const USER_PROFILE_PROPERTY = 'user';

const WHO_ARE_YOU = 'who_are_you';
const HELLO_USER = 'hello_user';

const NAME_PROMPT = 'name_prompt';
const CONFIRM_PROMPT = 'confirm_prompt';
const AGE_PROMPT = 'age_prompt';
```

在 Bot 的建構函式中定義並建立對話方塊集，並在集合中新增提示和瀑布式對話方塊。
`NumberPrompt` 包含自訂驗證，可確保使用者輸入大於 0 的年齡。

```javascript
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

由於我們的對話方塊步驟方法會參考執行個體屬性，因此必須使用 `bind` 方法，所以 `this` 物件會在每個步驟方法內正確地解析。

在此範例中，我們將每個步驟定義為不同的方法。 您也可以使用 lambda 運算式在建構函式中定義內嵌步驟。

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

---

此範例會在對話方塊內更新使用者設定檔狀態。 這種做法適用於簡單的 Bot，但如果您想要在所有 Bot 中重複使用一個對話方塊，則不可行。

有各種選項可將對話方塊步驟和 Bot 狀態分開。 例如，一旦對話方塊蒐集完整資訊，您即可：

* 使用「結束對話方塊」方法，提供所收集的資料作為父代內容的傳回值。 這可以是 Bot 的回合處理常式，或對話方塊堆疊上先前作用中的對話方塊。 這就是提示類別的設計方式。
* 產生適當服務的要求。 如果 Bot 作為大型服務的前端，這可能效果良好。

## <a name="test-your-dialog"></a>測試對話方塊

在本機建置並執行 Bot，然後使用[模擬器](../bot-service-debug-emulator.md)與 Bot 互動。

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

1. Bot 會傳送初始問候訊息，以回應使用者加入對話的對話更新活動。
1. 輸入 `hi` 或其他輸入。 因為這回合還沒有作用中的對話方塊，所以 Bot 會開始 `details` 對話方塊。
   * Bot 會傳送對話方塊的第一個提示，並等候更多輸入。
1. 在 Bot 詢問他們時回答問題，並透過對話方塊進行。
1. 對話方塊的最後一個步驟會根據您的輸入，傳送 `Thanks` 訊息。
   * 當對話方塊結束時，其會從對話方塊堆疊中移除，而 Bot 不再有作用中的對話方塊。
1. 輸入 `hi` 或其他輸入，再度開始對話方塊。

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

1. Bot 會傳送初始問候訊息，以回應使用者加入對話方塊的對話方塊更新活動。
1. 輸入 `hi` 或其他輸入。 因為這回合還沒有作用中的對話方塊，而且尚無使用者設定檔，所以 Bot 會開始 `who_are_you` 對話方塊。
   * Bot 會傳送對話方塊的第一個提示，並等候更多輸入。
1. 在 Bot 詢問他們時回答問題，並透過對話方塊進行。
1. 對話方塊的最後一個步驟會傳送簡短的確認訊息。
1. 輸入 `hi` 或其他輸入。
   * Bot 會開始單一步驟的 `hello_user` 對話方塊，這會顯示所收集資料中的資訊，然後立即結束。

---

## <a name="next-steps"></a>後續步驟

> [!div class="nextstepaction"]
> [使用分支和迴圈建立進階的對話流程](bot-builder-dialog-manage-complex-conversation-flow.md)
