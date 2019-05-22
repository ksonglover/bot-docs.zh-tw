---
title: 在相同 .NET Framework 專案中遷移現有的 Bot | Microsoft Docs
description: 我們採用現有的 v3 Bot 並將其遷移至 v4 SDK，而且使用相同的專案。
keywords: bot 移轉, formflow, 對話, v3 bot
author: JonathanFingold
ms.author: v-jofing
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 02/11/2019
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: c486970a3880c95ff10e9d4bb59b50a5600c7343
ms.sourcegitcommit: 178140eb060d71803f1c6357dd5afebd7f44fe1d
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 05/17/2019
ms.locfileid: "65855882"
---
# <a name="migrate-a-net-sdk-v3-bot-to-v4"></a>將 .NET SDK v3 Bot 遷移至 v4

我們在本文中將 v3 [ContosoHelpdeskChatBot](https://github.com/Microsoft/intelligent-apps/tree/master/ContosoHelpdeskChatBot/ContosoHelpdeskChatBot) 轉換 v4 Bot，而「不需轉換專案類型」。 其仍然是 .NET Framework 專案。
此轉換可細分成下列步驟：

1. 更新和安裝 NuGet 套件
1. 更新 Global.asax.cs 檔案
1. 更新 MessagesController 類別
1. 轉換您的對話

<!--TODO: Link to the converted bot...[ContosoHelpdeskChatBot](https://github.com/EricDahlvang/intelligent-apps/tree/v4netframework/ContosoHelpdeskChatBot).-->

Bot Framework SDK v4 是以與 SDK v3 相同的基礎 REST API 作為基礎。 不過，SDK v4 是舊版 SDK 的重構，讓開發人員對於其 Bot 有更多的彈性和控制權。 SDK 中的主要變更包括：

- 透過狀態管理物件和屬性存取子管理狀態。
- 設定回合處理常式並將活動傳遞給它已變更。
- 可評分項目已不存在。 將控制權交給您的對話之前，可以檢查回合處理常式中的「全域」命令。
- 與前一版非常不同的新 Dialogs 程式庫。 您必須使用元件和瀑布式對話，以及適用於 v4 的 Formflow 對話社群實作，將舊的對話轉換至新的對話系統。

如需特定變更的詳細資訊，請參閱 [v3 和 v4 .NET SDK 之間差異](migration-about.md)。

## <a name="update-and-install-nuget-packages"></a>更新和安裝 NuGet 套件

1. 將 **Microsoft.Bot.Builder.Azure** 更新為最新的穩定版本。

    這將也會更新 **Microsoft.Bot.Builder** 和 **Microsoft.Bot.Connector** 套件，因為它們是相依項目。

1. 刪除 **Microsoft.Bot.Builder.History** 套件。 這不是 v4 SDK 的一部分。
1. 新增 **Autofac.WebApi2**

    我們將使用它來協助在 ASP.NET 中插入相依性。

1. 新增 **Bot.Builder.Community.Dialogs.Formflow**

    這是一個社群程式庫，用於從 v3 Formflow 定義檔建置 v4 對話。 它以 **Microsoft.Bot.Builder.Dialogs** 作為其相依性之一，因此系統也會為我們安裝。

如果您在此時建置，則會收到編譯器錯誤。 您可以忽略這些錯誤。 完成轉換之後，我們就會有工作程式碼。

<!--
## Add a BotDataBag class

This file will contain wrappers to add a v3-style **IBotDataBag** to make dialog conversion simpler.

```csharp
using System.Collections.Generic;

namespace ContosoHelpdeskChatBot
{
    public class BotDataBag : Dictionary<string, object>, IBotDataBag
    {
        public bool RemoveValue(string key)
        {
            return base.Remove(key);
        }

        public void SetValue<T>(string key, T value)
        {
            this[key] = value;
        }

        public bool TryGetValue<T>(string key, out T value)
        {
            if (!ContainsKey(key))
            {
                value = default(T);
                return false;
            }

            value = (T)this[key];

            return true;
        }
    }

    public interface IBotDataBag
    {
        int Count { get; }

        void Clear();

        bool ContainsKey(string key);

        bool RemoveValue(string key);

        void SetValue<T>(string key, T value);

        bool TryGetValue<T>(string key, out T value);
    }
}
```
-->

## <a name="update-your-globalasaxcs-file"></a>更新 Global.asax.cs 檔案

某些 Scaffolding 已變更，而且我們必須自行在 v4 中設定部份的[狀態管理](../bot-builder-concept-state.md)基礎結構。 例如，v4 會使用 Bot 配接器來處理驗證並將活動轉送到 Bot 程式碼，而我們會事先宣告我們的狀態屬性。

我們將建立 `DialogState` 的狀態屬性，我們現在需要該屬性才能在 v4 中支援對話。 我們將使用相依性插入來取得控制器和 Bot 程式碼的必要資訊。

在 **Global.asax.cs** 中：

1. 更新 `using` 陳述式：
    ```csharp
    using System.Configuration;
    using System.Reflection;
    using System.Web.Http;
    using Autofac;
    using Autofac.Integration.WebApi;
    using Microsoft.Bot.Builder;
    using Microsoft.Bot.Builder.Dialogs;
    using Microsoft.Bot.Connector.Authentication;
    ```
1. 從 **Application_Start** 方法中移除這幾行
    ```csharp
    BotConfig.UpdateConversationContainer();
    this.RegisterBotModules();
    ```
    此外，插入這一行。
    ```csharp
    GlobalConfiguration.Configure(BotConfig.Register);
    ```
1. 移除不再參考的 **RegisterBotModules** 方法。
1. 以此 **BotConfig.Register** 方法取代 **BotConfig.UpdateConversationContainer** 方法，我們將在其中註冊支援相依性插入所需的物件。
    > [!NOTE]
    > 此 Bot 並未使用「使用者」或「私人交談」狀態。 要包含這些項目的程式碼行會在這裡註解排除。
    ```csharp
    public static void Register(HttpConfiguration config)
    {
        ContainerBuilder builder = new ContainerBuilder();
        builder.RegisterApiControllers(Assembly.GetExecutingAssembly());

        SimpleCredentialProvider credentialProvider = new SimpleCredentialProvider(
            ConfigurationManager.AppSettings[MicrosoftAppCredentials.MicrosoftAppIdKey],
            ConfigurationManager.AppSettings[MicrosoftAppCredentials.MicrosoftAppPasswordKey]);

        builder.RegisterInstance(credentialProvider).As<ICredentialProvider>();

        // The Memory Storage used here is for local bot debugging only. When the bot
        // is restarted, everything stored in memory will be gone.
        IStorage dataStore = new MemoryStorage();

        // Create Conversation State object.
        // The Conversation State object is where we persist anything at the conversation-scope.
        ConversationState conversationState = new ConversationState(dataStore);
        builder.RegisterInstance(conversationState).As<ConversationState>();

        //var userState = new UserState(dataStore);
        //var privateConversationState = new PrivateConversationState(dataStore);

        // Create the dialog state property acccessor.
        IStatePropertyAccessor<DialogState> dialogStateAccessor
            = conversationState.CreateProperty<DialogState>(nameof(DialogState));
        builder.RegisterInstance(dialogStateAccessor).As<IStatePropertyAccessor<DialogState>>();

        IContainer container = builder.Build();
        AutofacWebApiDependencyResolver resolver = new AutofacWebApiDependencyResolver(container);
        config.DependencyResolver = resolver;
    }
    ```

## <a name="update-your-messagescontroller-class"></a>更新 MessagesController 類別

這是 v4 中發生回合處理常式的地方，因此需求大幅變更。 除了回合處理常式本身，可將大部分的內容視為重複使用文字。 在 **Controllers\MessagesController.cs** 檔案中：

1. 更新 `using` 陳述式：
    ```csharp
    using System;
    using System.Net;
    using System.Net.Http;
    using System.Threading;
    using System.Threading.Tasks;
    using System.Web.Http;
    using Bot.Builder.Community.Dialogs.FormFlow;
    using ContosoHelpdeskChatBot.Dialogs;
    using Microsoft.Bot.Builder;
    using Microsoft.Bot.Builder.Dialogs;
    using Microsoft.Bot.Connector.Authentication;
    using Microsoft.Bot.Schema;
    ```
1. 從類別中移除 `[BotAuthentication]` 屬性。 在 v4 中，Bot 的配接器會處理驗證。
1. 新增這些欄位。 **ConversationState** 會管理交談的狀態，而需要 **IStatePropertyAccessor\<DialogState>** 才能在 v4 中支援對話。
    ```csharp
    private readonly ConversationState _conversationState;
    private readonly ICredentialProvider _credentialProvider;
    private readonly IStatePropertyAccessor<DialogState> _dialogData;

    private readonly DialogSet _dialogs;
    ```
1. 將建構函式新增至：
    - 初始化執行個體欄位。
    - 在 ASP.NET 中使用相依性插入來取得參數值。 (我們在 **Global.asax.cs** 中註冊了這些類別的執行個體，以支援此功能。)
    - 建立並初始化一個對話集，以便從中建立對話內容。 (我們必須在 v4 中明確執行這項操作。)
    ```csharp
    public MessagesController(
        ConversationState conversationState,
        ICredentialProvider credentialProvider,
        IStatePropertyAccessor<DialogState> dialogData)
    {
        _conversationState = conversationState;
        _dialogData = dialogData;
        _credentialProvider = credentialProvider;

        _dialogs = new DialogSet(dialogData);
        _dialogs.Add(new RootDialog(nameof(RootDialog)));
    }
    ```
1. 取代 **Post** 方法的主體。 這是我們建立配接器並使用它來呼叫訊息迴圈的地方 (回合處理常式)。 我們在每個回合的結尾使用 `SaveChangesAsync`，以儲存 Bot 對狀態所做的任何變更。

    ```csharp
    public async Task<HttpResponseMessage> Post([FromBody]Activity activity)
    {

        var botFrameworkAdapter = new BotFrameworkAdapter(_credentialProvider);

        var invokeResponse = await botFrameworkAdapter.ProcessActivityAsync(
            Request.Headers.Authorization?.ToString(),
            activity,
            OnTurnAsync,
            default(CancellationToken));

        if (invokeResponse == null)
        {
            return Request.CreateResponse(HttpStatusCode.OK);
        }
        else
        {
            return Request.CreateResponse(invokeResponse.Status);
        }
    }
    ```
1. 新增 **OnTurnAsync** 方法，其中包含 Bot 的[回合處理常式](../bot-builder-basics.md#the-activity-processing-stack)程式碼。
    > [!NOTE]
    > 因此 v4 中不存在可評分項目。 我們會在繼續任何作用中對話之前，先檢查 Bot 的回合處理常式中來自使用者的 `cancel` 訊息。
    ```csharp
    protected async Task OnTurnAsync(ITurnContext turnContext, CancellationToken cancellationToken)
    {
        // We're only handling message activities in this bot.
        if (turnContext.Activity.Type == ActivityTypes.Message)
        {
            // Create the dialog context for our dialog set.
            DialogContext dc = await _dialogs.CreateContextAsync(turnContext, cancellationToken);

            // Globally interrupt the dialog stack if the user sent 'cancel'.
            if (turnContext.Activity.Text.Equals("cancel", StringComparison.InvariantCultureIgnoreCase))
            {
                Activity reply = turnContext.Activity.CreateReply($"Ok restarting conversation.");
                await turnContext.SendActivityAsync(reply);
                await dc.CancelAllDialogsAsync();
            }

            try
            {
                // Continue the active dialog, if any. If we just cancelled all dialog, the
                // dialog stack will be empty, and this will return DialogTurnResult.Empty.
                DialogTurnResult dialogResult = await dc.ContinueDialogAsync();
                switch (dialogResult.Status)
                {
                    case DialogTurnStatus.Empty:
                        // There was no active dialog in the dialog stack; start the root dialog.
                        await dc.BeginDialogAsync(nameof(RootDialog));
                        break;

                    case DialogTurnStatus.Complete:
                        // The last dialog on the stack completed and the stack is empty.
                        await dc.EndDialogAsync();
                        break;

                    case DialogTurnStatus.Waiting:
                    case DialogTurnStatus.Cancelled:
                        // The active dialog is waiting for a response from the user, or all
                        // dialogs were cancelled and the stack is empty. In either case, we
                        // don't need to do anything here.
                        break;
                }
            }
            catch (FormCanceledException)
            {
                // One of the dialogs threw an exception to clear the dialog stack.
                await turnContext.SendActivityAsync("Cancelled.");
                await dc.CancelAllDialogsAsync();
                await dc.BeginDialogAsync(nameof(RootDialog));
            }
        }
    }
    ```
1. 因為我們只會處理「訊息」活動，所以可刪除 **HandleSystemMessage** 方法。

### <a name="delete-the-cancelscorable-and-globalmessagehandlersbotmodule-classes"></a>刪除 CancelScorable 和 GlobalMessageHandlersBotModule 類別

因為 v4 中不存在可評分項目，而且我們已更新回合處理常式以回應 `cancel` 訊息，所以可刪除 **CancelScorable** (在**Dialogs\CancelScorable.cs**) 和**GlobalMessageHandlersBotModule** 類別。

## <a name="convert-your-dialogs"></a>轉換您的對話

我們將對原始對話進行許多變更，以將其遷移至 v4 SDK。 現在不用擔心編譯器錯誤。 待我們完成轉換後，就會解決這些錯誤。
為了不會對原始程式碼進行不必要的修改，在我們完成移轉後，仍會有一些編譯器警告。

我們所有的對話都將衍生自 `ComponentDialog`，而不會實作 v3 的 `IDialog<object>` 介面。

此 Bot 有四個需要我們轉換的對話：

| | |
|---|---|
| [RootDialog](#update-the-root-dialog) | 顯示選項並開始其他對話。 |
| [InstallAppDialog](#update-the-install-app-dialog) | 處理在電腦上安裝應用程式的要求。 |
| [LocalAdminDialog](#update-the-local-admin-dialog) | 處理本機電腦系統管理權限的要求。 |
| [ResetPasswordDialog](#update-the-reset-password-dialog) | 處理重設密碼的要求。 |

這些對話會收集輸入，但不會在您的電腦上執行上述任何作業。

### <a name="make-solution-wide-dialog-changes"></a>進行全方案的對話變更

1. 針對整個方案，以 `ComponentDialog` 取代所有出現的 `IDialog<object>`。
1. 針對整個方案，以 `DialogContext` 取代所有出現的 `IDialogContext`。
1. 針對每個對話類別，移除 `[Serializable]` 屬性。

對話內的控制流程和傳訊不再以相同的方式處理，因此我們必須在轉換每個對話時修改這個屬性。

| 作業 | v3 程式碼 | v4 程式碼 |
| :--- | :--- | :--- |
| 處理對話的開始 | 實作 `IDialog.StartAsync` | 讓此項成為瀑布式對話的第一個步驟，或實作 `Dialog.BeginDialogAsync` |
| 處理對話的接續 | 呼叫 `IDialogContext.Wait` | 在瀑布式對話中新增其他步驟，或實作 `Dialog.ContinueDialogAsync` |
| 將訊息傳送給使用者 | 呼叫 `IDialogContext.PostAsync` | 呼叫 `ITurnContext.SendActivityAsync` |
| 開始子對話 | 呼叫 `IDialogContext.Call` | 呼叫 `DialogContext.BeginDialogAsync` |
| 表示目前對話已完成的訊號 | 呼叫 `IDialogContext.Done` | 呼叫 `DialogContext.EndDialogAsync` |
| 取得使用者的輸入 | 使用 `IAwaitable<IMessageActivity>` 參數 | 使用瀑布中的提示，或使用 `ITurnContext.Activity` |

v4 程式碼的注意事項：

- 使用 `DialogContext.Context` 屬性來取得目前的回合內容。
- 瀑布步驟具有 `WaterfallStepContext` 參數，其衍生自 `DialogContext`。
- 所有具體的對話和提示類別均衍生自抽象的 `Dialog` 類別。
- 您會在建立元件對話時指派識別碼。 對話集中的每個對話都需要被指派該集合內唯一的識別碼。

### <a name="update-the-root-dialog"></a>更新根對話

在此 Bot 中，根對話會提示使用者從一組選項中進行選擇，然後根據該選擇開始進行子對話。 接著在交談存留期間執行迴圈。

- 我們可以將主要流程設定為瀑布式對話，這是 v4 SDK 中的新概念。 它會依序執行一組固定的步驟。 如需詳細資訊，請參閱[實作循序交談流程](~/v4sdk/bot-builder-dialog-manage-conversation-flow.md)。
- 提示作業現在會透過提示類別來處理，而這些類別是簡短的子對話，可提示輸入資料、執行最少的處理和驗證，並傳回一個值。 如需詳細資訊，請參閱[使用對話提示收集使用者輸入](~/v4sdk/bot-builder-prompts.md)。

在 **Dialogs/RootDialog.cs** 檔案中：

1. 更新 `using` 陳述式：
    ```csharp
    using System;
    using System.Collections.Generic;
    using System.Threading;
    using System.Threading.Tasks;
    using Microsoft.Bot.Builder;
    using Microsoft.Bot.Builder.Dialogs;
    using Microsoft.Bot.Builder.Dialogs.Choices;
    ```
1. 我們需要將 `HelpdeskOptions` 選項從字串清單轉換為選擇清單。 這將搭配選擇提示使用，其將接受選擇號碼 (清單中)、選擇值，或任何選擇的同義字作為有效的輸入。
    ```csharp
    private static List<Choice> HelpdeskOptions = new List<Choice>()
    {
        new Choice(InstallAppOption) { Synonyms = new List<string>(){ "install" } },
        new Choice(ResetPasswordOption) { Synonyms = new List<string>(){ "password" } },
        new Choice(LocalAdminOption)  { Synonyms = new List<string>(){ "admin" } }
    };
    ```
1. 新增建構函式。 此程式碼會執行以下動作：
   - 每個對話執行個體會在建立時被指派識別碼。 對話識別碼是對話要新增到其中的對話集的一部分。 還記得 Bot 有一個已在 **MessageController** 類別中初始化的對話集。 每個 `ComponentDialog` 都有自己的內部對話集，以及自己的對話識別碼集。
   - 它會新增其他對話 (包括選擇提示) 作為子對話。 在此，我們只會使用每個對話識別碼的類別名稱。
   - 它會定義包含三個步驟的瀑布式對話。 我們會立刻進行實作。
     - 對話會先提示使用者選擇要執行的工作。
     - 然後，開始與該選擇相關聯的子對話。
     - 最後，自行重新開始。
   - 瀑布的每個步驟都是一項委派，而我們接下來會實作這些步驟，並從原始對話中取得現有的程式碼。
   - 當您啟動元件對話時，就會啟動其「初始對話」。 根據預設，這是新增至元件對話的第一個子對話。 若要指派不同的子系作為初始對話，您會手動設定元件的 `InitialDialogId` 屬性。
    ```csharp
    public RootDialog(string id)
        : base(id)
    {
        AddDialog(new WaterfallDialog("choiceswaterfall", new WaterfallStep[]
            {
                PromptForOptionsAsync,
                ShowChildDialogAsync,
                ResumeAfterAsync,
            }));
        AddDialog(new InstallAppDialog(nameof(InstallAppDialog)));
        AddDialog(new LocalAdminDialog(nameof(LocalAdminDialog)));
        AddDialog(new ResetPasswordDialog(nameof(ResetPasswordDialog)));
        AddDialog(new ChoicePrompt("options"));
    }
    ```
1. 我們可以刪除 **StartAsync** 方法。 當元件對話開始時，它會自動開始它「初始」對話。 在此情況下，這就是我們在建構函式中定義的瀑布式對話。 該對話也會自動在其第一個步驟開始。
1. 我們將會刪除 **MessageReceivedAsync** 和 **ShowOptions** 方法，並以瀑布的第一個步驟取代它們。 這兩種方法會先映入使用者的眼簾，並要求他們選擇其中一個可用的選項。
   - 您可以在此看到選擇清單，而系統會提供問候和錯誤訊息作為我們的選擇提示呼叫中的選項。
   - 我們不需要指定要在對話中呼叫的下一個方法，因為瀑布會在選擇提示完成時繼續下一個步驟。
   - 選擇提示將會執行迴圈，直到它收到有效的輸入，或取消整個對話堆疊為止。
    ```csharp
    private async Task<DialogTurnResult> PromptForOptionsAsync(
        WaterfallStepContext stepContext,
        CancellationToken cancellationToken = default(CancellationToken))
    {
        // Prompt the user for a response using our choice prompt.
        return await stepContext.PromptAsync(
            "options",
            new PromptOptions()
            {
                Choices = HelpdeskOptions,
                Prompt = MessageFactory.Text(GreetMessage),
                RetryPrompt = MessageFactory.Text(ErrorMessage)
            },
            cancellationToken);
    }
    ```
1. 我們可以使用瀑布的第二個步驟取代 **OnOptionSelected**。 我們仍會根據使用者的輸入開始子對話。
   - 選擇提示會傳回 `FoundChoice` 值。 這會顯示在步驟內容的 `Result` 屬性中。 對話堆疊會將所有傳回值視為物件。 如果傳回值來自您的其中一個對話，您便知道物件值是何種類型。 如需每個提示類型傳回的內容清單，請參閱[提示類型](../bot-builder-concept-dialog.md#prompt-types)。
   - 因為選擇提示不會擲回例外狀況，所以可移除 try-catch 區塊。
   - 我們必須新增一項通過，此方法才能一律傳回適當的值。 此程式碼應該永遠不會被叫用，但如果叫用，就會讓對話「正常失敗」。
    ```csharp
    private async Task<DialogTurnResult> ShowChildDialogAsync(
        WaterfallStepContext stepContext,
        CancellationToken cancellationToken = default(CancellationToken))
    {
        // string optionSelected = await userReply;
        string optionSelected = (stepContext.Result as FoundChoice).Value;

        switch (optionSelected)
        {
            case InstallAppOption:
                //context.Call(new InstallAppDialog(), this.ResumeAfterOptionDialog);
                //break;
                return await stepContext.BeginDialogAsync(
                    nameof(InstallAppDialog),
                    cancellationToken);
            case ResetPasswordOption:
                //context.Call(new ResetPasswordDialog(), this.ResumeAfterOptionDialog);
                //break;
                return await stepContext.BeginDialogAsync(
                    nameof(ResetPasswordDialog),
                    cancellationToken);
            case LocalAdminOption:
                //context.Call(new LocalAdminDialog(), this.ResumeAfterOptionDialog);
                //break;
                return await stepContext.BeginDialogAsync(
                    nameof(LocalAdminDialog),
                    cancellationToken);
        }

        // We shouldn't get here, but fail gracefully if we do.
        await stepContext.Context.SendActivityAsync(
            "I don't recognize that option.",
            cancellationToken: cancellationToken);
        // Continue through to the next step without starting a child dialog.
        return await stepContext.NextAsync(cancellationToken: cancellationToken);
    }
    ```
1. 最後，使用瀑布的最後一個步驟取代舊的 **ResumeAfterOptionDialog** 方法。
    - 我們會將堆疊上的原始執行個體取代為本身的新執行個體，進而重新開始瀑布式對話，而不是如同我們在原始對話中一樣結束對話並傳回票證號碼。 我們可以這麼做，因為原始應用程式一律忽略傳回值 (票證號碼)，並重新開始進行根對話。
    ```csharp
    private async Task<DialogTurnResult> ResumeAfterAsync(
        WaterfallStepContext stepContext,
        CancellationToken cancellationToken = default(CancellationToken))
    {
        try
        {
            //var message = await userReply;
            var message = stepContext.Context.Activity;

            int ticketNumber = new Random().Next(0, 20000);
            //await context.PostAsync($"Thank you for using the Helpdesk Bot. Your ticket number is {ticketNumber}.");
            await stepContext.Context.SendActivityAsync(
                $"Thank you for using the Helpdesk Bot. Your ticket number is {ticketNumber}.",
                cancellationToken: cancellationToken);

            //context.Done(ticketNumber);
        }
        catch (Exception ex)
        {
            // await context.PostAsync($"Failed with message: {ex.Message}");
            await stepContext.Context.SendActivityAsync(
                $"Failed with message: {ex.Message}",
                cancellationToken: cancellationToken);

            // In general resume from task after calling a child dialog is a good place to handle exceptions
            // try catch will capture exceptions from the bot framework awaitable object which is essentially "userReply"
            logger.Error(ex);
        }

        // Replace on the stack the current instance of the waterfall with a new instance,
        // and start from the top.
        return await stepContext.ReplaceDialogAsync(
            "choiceswaterfall",
            cancellationToken: cancellationToken);
    }
    ```

### <a name="update-the-install-app-dialog"></a>更新安裝應用程式對話

安裝應用程式對話會執行一些邏輯工作，我們會將其設定為包含 4 個步驟的瀑布式對話。 如何將現有程式碼分解成瀑布步驟是每個對話的邏輯活動。 每個步驟都會註明程式碼來自的原始方法。

1. 要求使用者提供搜尋字串。
1. 查詢資料庫中可能的相符項目。
   - 如果有一項命中，請選取此項目並繼續執行。
   - 如果有多項命中，則會要求使用者選擇一項。
   - 如果沒有命中項目，就會結束對話。
1. 要求使用者提供要安裝應用程式的機器。
1. 將資訊寫入資料庫並傳送確認訊息。

在 **Dialogs/InstallAppDialog.cs** 檔案中：

1. 更新 `using` 陳述式：
    ```csharp
    using System.Collections.Generic;
    using System.Linq;
    using System.Threading;
    using System.Threading.Tasks;
    using ContosoHelpdeskChatBot.Models;
    using Microsoft.Bot.Builder;
    using Microsoft.Bot.Builder.Dialogs;
    using Microsoft.Bot.Builder.Dialogs.Choices;
    ```
1. 針對我們將用於提示和對話的識別碼，定義常數。 這可讓對話程式碼更容易維護，因為要使用的字串會定義於一個地方。
    ```csharp
    // Set up our dialog and prompt IDs as constants.
    private const string MainId = "mainDialog";
    private const string TextId = "textPrompt";
    private const string ChoiceId = "choicePrompt";
    ```
1. 針對我們將用來追蹤對話狀態的索引鍵，定義常數。
    ```csharp
    // Set up keys for managing collected information.
    private const string InstallInfo = "installInfo";
    ```
1. 新增建構函式並初始化元件的對話集。 我們此時會明確地設定 `InitialDialogId` 屬性，這表示主要瀑布式對話不必是您新增至對話集的第一個對話。 比方說，如果您想要先新增提示，這可讓您執行這項操作，而不會造成執行階段問題。
    ```csharp
    public InstallAppDialog(string id)
        : base(id)
    {
        // Initialize our dialogs and prompts.
        InitialDialogId = MainId;
        AddDialog(new WaterfallDialog(MainId, new WaterfallStep[] {
            GetSearchTermAsync,
            ResolveAppNameAsync,
            GetMachineNameAsync,
            SubmitRequestAsync,
        }));
        AddDialog(new TextPrompt(TextId));
        AddDialog(new ChoicePrompt(ChoiceId));
    }
    ```
1. 我們可以使用瀑布的第一個步驟取代 **StartAsync**。
    - 我們不必自行管理狀態，所以會追蹤對話狀態中的安裝應用程式物件。
    - 要求使用者輸入資料的訊息會變成提示呼叫中的選項。
    ```csharp
    private async Task<DialogTurnResult> GetSearchTermAsync(
        WaterfallStepContext stepContext,
        CancellationToken cancellationToken = default(CancellationToken))
    {
        // Create an object in dialog state in which to track our collected information.
        stepContext.Values[InstallInfo] = new InstallApp();

        // Ask for the search term.
        return await stepContext.PromptAsync(
            TextId,
            new PromptOptions
            {
                Prompt = MessageFactory.Text("Ok let's get started. What is the name of the application? "),
            },
            cancellationToken);
    }
    ```
1. 我們可以使用瀑布的第二個步驟取代 **appNameAsync** 和 **multipleAppsAsync**。
    - 我們現在會取得提示結果，而不只是查看使用者的最後一則訊息。
    - 資料庫查詢和 if 陳述式的組織方式與在 **appNameAsync** 中相同。 已更新 if 陳述式的每個區塊中的程式碼，可搭配 v4 對話運作。
        - 如果我們有一次命中，我們將會更新對話狀態並繼續進行下一個步驟。
        - 如果您有多項命中，我們將使用選擇提示來要求使用者從選項清單中進行選擇。 這表示我們可以刪除 **multipleAppsAsync**。
        - 如果我們沒有命中，我們會結束此對話並傳回 null 給根對話。
    ```csharp
    private async Task<DialogTurnResult> ResolveAppNameAsync(
        WaterfallStepContext stepContext,
        CancellationToken cancellationToken = default(CancellationToken))
    {
        // Get the result from the text prompt.
        var appname = stepContext.Result as string;

        // Query the database for matches.
        var names = await this.getAppsAsync(appname);

        if (names.Count == 1)
        {
            // Get our tracking information from dialog state and add the app name.
            var install = stepContext.Values[InstallInfo] as InstallApp;
            install.AppName = names.First();

            return await stepContext.NextAsync();
        }
        else if (names.Count > 1)
        {
            // Ask the user to choose from the list of matches.
            return await stepContext.PromptAsync(
                ChoiceId,
                new PromptOptions
                {
                    Prompt = MessageFactory.Text("I found the following applications. Please choose one:"),
                    Choices = ChoiceFactory.ToChoices(names),
                },
                cancellationToken);
        }
        else
        {
            // If no matches, exit this dialog.
            await stepContext.Context.SendActivityAsync(
                $"Sorry, I did not find any application with the name '{appname}'.",
                cancellationToken: cancellationToken);

            return await stepContext.EndDialogAsync(null, cancellationToken);
        }
    }
    ```
1. **appNameAsync** 也在解析查詢之後，要求使用者提供其機器名稱。 我們將在瀑布的下一個步驟中擷取邏輯的該部分。
    - 同樣地，在 v4 中，我們必須自行管理狀態。 唯一比較麻煩的事，就是我們可以透過上一個步驟中的兩個不同邏輯分支來抵達此步驟。
    - 我們會使用與之前相同的文字提示來要求使用者提供機器名稱，只是這次提供不同的選項。
    ```csharp
    private async Task<DialogTurnResult> GetMachineNameAsync(
        WaterfallStepContext stepContext,
        CancellationToken cancellationToken = default(CancellationToken))
    {
        // Get the tracking info. If we don't already have an app name,
        // Then we used the choice prompt to get it in the previous step.
        var install = stepContext.Values[InstallInfo] as InstallApp;
        if (install.AppName is null)
        {
            install.AppName = (stepContext.Result as FoundChoice).Value;
        }

        // We now need the machine name, so prompt for it.
        return await stepContext.PromptAsync(
            TextId,
            new PromptOptions
            {
                Prompt = MessageFactory.Text(
                    $"Found {install.AppName}. What is the name of the machine to install application?"),
            },
            cancellationToken);
    }
    ```
1. **machineNameAsync** 中的邏輯會包裝在瀑布的最後一個步驟中。
    - 我們可從文字提示結果中擷取機器名稱並更新對話狀態。
    - 我們正在移除呼叫以更新資料庫，因為支援的程式碼位於不同的專案中。
    - 然後我們會將成功訊息傳送給使用者並結束對話。
    ```csharp
    private async Task<DialogTurnResult> SubmitRequestAsync(
        WaterfallStepContext stepContext,
        CancellationToken cancellationToken = default(CancellationToken))
    {
        if (stepContext.Reason != DialogReason.CancelCalled)
        {
            // Get the tracking info and add the machine name.
            var install = stepContext.Values[InstallInfo] as InstallApp;
            install.MachineName = stepContext.Context.Activity.Text;

            //TODO: Save to this information to the database.
        }

        await stepContext.Context.SendActivityAsync(
            $"Great, your request to install {install.AppName} on {install.MachineName} has been scheduled.",
            cancellationToken: cancellationToken);

        return await stepContext.EndDialogAsync(null, cancellationToken);
    }
    ```
1. 為了模擬資料庫，我已經將 **getAppsAsync** 更新為查詢靜態清單，而不是查詢資料庫。
    ```csharp
    private async Task<List<string>> getAppsAsync(string Name)
    {
        List<string> names = new List<string>();

        // Simulate querying the database for applications that match.
        return (from app in AppMsis
            where app.ToLower().Contains(Name.ToLower())
            select app).ToList();
    }

    // Example list of app names in the database.
    private static readonly List<string> AppMsis = new List<string>
    {
        "µTorrent 3.5.0.44178",
        "7-Zip 17.1",
        "Ad-Aware 9.0",
        "Adobe AIR 2.5.1.17730",
        "Adobe Flash Player (IE) 28.0.0.105",
        "Adobe Flash Player (Non-IE) 27.0.0.130",
        "Adobe Reader 11.0.14",
        "Adobe Shockwave Player 12.3.1.201",
        "Advanced SystemCare Personal 11.0.3",
        "Auslogics Disk Defrag 3.6",
        "avast! 4 Home Edition 4.8.1351",
        "AVG Anti-Virus Free Edition 9.0.0.698",
        "Bonjour 3.1.0.1",
        "CCleaner 5.24.5839",
        "Chmod Calculator 20132.4",
        "CyberLink PowerDVD 17.0.2101.62",
        "DAEMON Tools Lite 4.46.1.328",
        "FileZilla Client 3.5",
        "Firefox 57.0",
        "Foxit Reader 4.1.1.805",
        "Google Chrome 66.143.49260",
        "Google Earth 7.3.0.3832",
        "Google Toolbar (IE) 7.5.8231.2252",
        "GSpot 2701.0",
        "Internet Explorer 903235.0",
        "iTunes 12.7.0.166",
        "Java Runtime Environment 6 Update 17",
        "K-Lite Codec Pack 12.1",
        "Malwarebytes Anti-Malware 2.2.1.1043",
        "Media Player Classic 6.4.9.0",
        "Microsoft Silverlight 5.1.50907",
        "Mozilla Thunderbird 57.0",
        "Nero Burning ROM 19.1.1005",
        "OpenOffice.org 3.1.1 Build 9420",
        "Opera 12.18.1873",
        "Paint.NET 4.0.19",
        "Picasa 3.9.141.259",
        "QuickTime 7.79.80.95",
        "RealPlayer SP 12.0.0.319",
        "Revo Uninstaller 1.95",
        "Skype 7.40.151",
        "Spybot - Search & Destroy 1.6.2.46",
        "SpywareBlaster 4.6",
        "TuneUp Utilities 2009 14.0.1000.353",
        "Unlocker 1.9.2",
        "VLC media player 1.1.6",
        "Winamp 5.56 Build 2512",
        "Windows Live Messenger 2009 16.4.3528.331",
        "WinPatrol 2010 31.0.2014",
        "WinRAR 5.0",
    };
    ```

### <a name="update-the-local-admin-dialog"></a>更新本機系統管理對話

在 v3 中，此對話會先映入使用者的眼簾、開始 Formflow 對話，然後將結果儲存至資料庫。 這可輕鬆地轉換成包含兩個步驟的瀑布。

1. 更新 `using` 陳述式。 請注意，此對話包含 v3 Formflow 對話。 在 v4 中，我們可以使用社群 Formflow 程式庫。
    ```csharp
    using System.Threading;
    using System.Threading.Tasks;
    using Bot.Builder.Community.Dialogs.FormFlow;
    using ContosoHelpdeskChatBot.Models;
    using Microsoft.Bot.Builder.Dialogs;
    ```
1. 我們可以移除 `LocalAdmin` 的執行個體屬性，因為可在對話狀態中取得結果。
1. 針對我們將用於對話的識別碼，定義常數。 在社群程式庫中，建構的對話一律會設定為對話所產生的類別名稱。
    ```csharp
    // Set up our dialog and prompt IDs as constants.
    private const string MainDialog = "mainDialog";
    private static string AdminDialog { get; } = nameof(LocalAdminPrompt);
    ```
1. 新增建構函式並初始化元件的對話集。 Formflow 對話會以相同的方式建立。 我們只是將它新增至建構函式中元件的對話集。
    ```csharp
    public LocalAdminDialog(string dialogId) : base(dialogId)
    {
        InitialDialogId = MainDialog;

        AddDialog(new WaterfallDialog(MainDialog, new WaterfallStep[]
        {
            BeginFormflowAsync,
            SaveResultAsync,
        }));
        AddDialog(FormDialog.FromForm(BuildLocalAdminForm, FormOptions.PromptInStart));
    }
    ```
1. 我們可以使用瀑布的第一個步驟取代 **StartAsync**。 我們已經在建構函式中建立 Formflow，而其他兩個陳述式會轉譯為此陳述式。
    ```csharp
    private async Task<DialogTurnResult> BeginFormflowAsync(
        WaterfallStepContext stepContext,
        CancellationToken cancellationToken = default(CancellationToken))
    {
        await stepContext.Context.SendActivityAsync("Great I will help you request local machine admin.");

        // Begin the Formflow dialog.
        return await stepContext.BeginDialogAsync(AdminDialog, cancellationToken: cancellationToken);
    }
    ```
1. 我們可以使用瀑布的第二個步驟取代 **ResumeAfterLocalAdminFormDialog**。 我們必須從步驟內容中取得傳回值，而不是從執行個體屬性中取得。
    ```csharp
    private async Task<DialogTurnResult> SaveResultAsync(
        WaterfallStepContext stepContext,
        CancellationToken cancellationToken = default(CancellationToken))
    {
        // Get the result from the Formflow dialog when it ends.
        if (stepContext.Reason != DialogReason.CancelCalled)
        {
            var admin = stepContext.Result as LocalAdminPrompt;

            //TODO: Save to this information to the database.
        }

        return await stepContext.EndDialogAsync(null, cancellationToken);
    }
    ```
1. **BuildLocalAdminForm** 大致維持相同，但我們沒有讓 Formflow 更新執行個體屬性。
    ```csharp
    // Nearly the same as before.
    private IForm<LocalAdminPrompt> BuildLocalAdminForm()
    {
        //here's an example of how validation can be used in form builder
        return new FormBuilder<LocalAdminPrompt>()
            .Field(nameof(LocalAdminPrompt.MachineName),
            validate: async (state, value) =>
            {
                ValidateResult result = new ValidateResult { IsValid = true, Value = value };
                //add validation here

                //this.admin.MachineName = (string)value;
                return result;
            })
            .Field(nameof(LocalAdminPrompt.AdminDuration),
            validate: async (state, value) =>
            {
                ValidateResult result = new ValidateResult { IsValid = true, Value = value };
                //add validation here

                //this.admin.AdminDuration = Convert.ToInt32((long)value) as int?;
                return result;
            })
            .Build();
    }
    ```

### <a name="update-the-reset-password-dialog"></a>更新重設密碼對話

在 v3 中，此對話會先映入使用者的眼簾、透過密碼授權給使用者、結束或開始 Formflow 對話，然後重設密碼。 這仍會轉譯成瀑布。

1. 更新 `using` 陳述式。 請注意，此對話包含 v3 Formflow 對話。 在 v4 中，我們可以使用社群 Formflow 程式庫。
    ```csharp
    using System;
    using System.Threading;
    using System.Threading.Tasks;
    using Bot.Builder.Community.Dialogs.FormFlow;
    using ContosoHelpdeskChatBot.Models;
    using Microsoft.Bot.Builder.Dialogs;
    ```
1. 針對我們將用於對話的識別碼，定義常數。 在社群程式庫中，建構的對話一律會設定為對話所產生的類別名稱。
    ```csharp
    // Set up our dialog and prompt IDs as constants.
    private const string MainDialog = "mainDialog";
    private static string ResetDialog { get; } = nameof(ResetPasswordPrompt);
    ```
1. 新增建構函式並初始化元件的對話集。 Formflow 對話會以相同的方式建立。 我們只是將它新增至建構函式中元件的對話集。
    ```csharp
    public ResetPasswordDialog(string dialogId) : base(dialogId)
    {
        InitialDialogId = MainDialog;

        AddDialog(new WaterfallDialog(MainDialog, new WaterfallStep[]
        {
            BeginFormflowAsync,
            ProcessRequestAsync,
        }));
        AddDialog(FormDialog.FromForm(BuildResetPasswordForm, FormOptions.PromptInStart));
    }
    ```
1. 我們可以使用瀑布的第一個步驟取代 **StartAsync**。 我們已經在建構函式中建立 Formflow。 否則，我們會保留相同的邏輯，只是將 v3 呼叫轉譯成 v4 對等項目。
    ```csharp
    private async Task<DialogTurnResult> BeginFormflowAsync(
        WaterfallStepContext stepContext,
        CancellationToken cancellationToken = default(CancellationToken))
    {
        await stepContext.Context.SendActivityAsync("Alright I will help you create a temp password.");

        // Check the passcode and fail out or begin the Formflow dialog.
        if (sendPassCode(stepContext))
        {
            return await stepContext.BeginDialogAsync(ResetDialog, cancellationToken: cancellationToken);
        }
        else
        {
            //here we can simply fail the current dialog because we have root dialog handling all exceptions
            throw new Exception("Failed to send SMS. Make sure email & phone number has been added to database.");
        }
    }
    ```
1. **sendPassCode** 主要留作練習。 已將原始程式碼註解排除，而此方法只會傳回 true。 此外，原始 Bot 中並未使用電子郵件地址，所以可再次予以移除。
    ```csharp
    private bool sendPassCode(DialogContext context)
    {
        //bool result = false;

        //Recipient Id varies depending on channel
        //refer ChannelAccount class https://docs.botframework.com/en-us/csharp/builder/sdkreference/dd/def/class_microsoft_1_1_bot_1_1_connector_1_1_channel_account.html#a0b89cf01fdd73cbc00a524dce9e2ad1a
        //as well as Activity class https://docs.botframework.com/en-us/csharp/builder/sdkreference/dc/d2f/class_microsoft_1_1_bot_1_1_connector_1_1_activity.html
        //int passcode = new Random().Next(1000, 9999);
        //Int64? smsNumber = 0;
        //string smsMessage = "Your Contoso Pass Code is ";
        //string countryDialPrefix = "+1";

        // TODO: save PassCode to database
        //using (var db = new ContosoHelpdeskContext())
        //{
        //    var reset = db.ResetPasswords.Where(r => r.EmailAddress == email).ToList();
        //    if (reset.Count >= 1)
        //    {
        //        reset.First().PassCode = passcode;
        //        smsNumber = reset.First().MobileNumber;
        //        result = true;
        //    }

        //    db.SaveChanges();
        //}

        // TODO: send passcode to user via SMS.
        //if (result)
        //{
        //    result = Helper.SendSms($"{countryDialPrefix}{smsNumber.ToString()}", $"{smsMessage} {passcode}");
        //}

        //return result;
        return true;
    }
    ```
1. **BuildResetPasswordForm** 沒有變更。
1. 我們可以使用瀑布的第二個步驟取代 **ResumeAfterLocalAdminFormDialog**，並將從步驟內容中取得傳回值。 我們已移除原始對話未做任何處理的電子郵件地址，並提供了虛擬結果，而非查詢資料庫。 我們會保留相同的邏輯，只是將 v3 呼叫轉譯成 v4 對等項目。
    ```csharp
    private async Task<DialogTurnResult> ProcessRequestAsync(
        WaterfallStepContext stepContext,
        CancellationToken cancellationToken = default(CancellationToken))
    {
        // Get the result from the Formflow dialog when it ends.
        if (stepContext.Reason != DialogReason.CancelCalled)
        {
            var prompt = stepContext.Result as ResetPasswordPrompt;
            int? passcode;

            // TODO: Retrieve the passcode from the database.
            passcode = 1111;

            if (prompt.PassCode == passcode)
            {
                string temppwd = "TempPwd" + new Random().Next(0, 5000);
                await stepContext.Context.SendActivityAsync(
                    $"Your temp password is {temppwd}",
                    cancellationToken: cancellationToken);
            }
        }

        return await stepContext.EndDialogAsync(null, cancellationToken);
    }
    ```

### <a name="update-models-as-necessary"></a>視需要更新模型

在某些參考 Formflow 程式庫的模型中，我們需要更新`using` 陳述式。

1. 在 `LocalAdminPrompt` 中，將它們變更如下：
    ```csharp
    using Bot.Builder.Community.Dialogs.FormFlow;
    ```
1. 在 `ResetPasswordPrompt` 中，將它們變更如下：
    ```csharp
    using Bot.Builder.Community.Dialogs.FormFlow;
    using System;
    ```

## <a name="update-webconfig"></a>更新 Web.config

註解排除 **MicrosoftAppId** 和 **MicrosoftAppPassword** 的組態金鑰。 這可讓您在本機進行 Bot 偵錯，而不需將這些值提供給模擬器。

## <a name="run-and-test-your-bot-in-the-emulator"></a>在模擬器中執行並測試您的 Bot

此時，我們應該能夠在 IIS 中本機執行 Bot，然後將它與模擬器連結。

1. 在 IIS 中執行 Bot。
1. 啟動模擬器，然後連線到 Bot 的端點 (例如：**http://localhost:3978/api/messages**)。
    - 如果這是您第一次執行 Bot，請按一下 [檔案] > [新的 Bot]，並遵循畫面上的指示。 否則，請按一下 [檔案] > [開啟 Bot] 以開啟現有的 Bot。
    - 在組態中再次檢查連接埠設定。 例如，如果 Bot 在瀏覽器中開啟至 `http://localhost:3979/`，則在模擬器中，將 Bot 的端點設定為 `http://localhost:3979/api/messages`。
1. 四個對話應該都可以運作，而且您可以在瀑布步驟中設定中斷點，以檢查哪個對話內容和對話狀態是在這些點上。

## <a name="additional-resources"></a>其他資源

v4 概念性主題：

- [Bot 的運作方式](../bot-builder-basics.md)
- [管理狀態](../bot-builder-concept-state.md)
- [對話方塊程式庫](../bot-builder-concept-dialog.md)

v4 作法主題：

- [傳送及接收文字訊息](../bot-builder-howto-send-messages.md)
- [儲存使用者和對話資料](../bot-builder-howto-v4-state.md)
- [實作循序對話流程](../bot-builder-dialog-manage-conversation-flow.md)
