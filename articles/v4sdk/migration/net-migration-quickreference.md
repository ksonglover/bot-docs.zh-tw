---
title: 從 .NET v3 遷移到 v4 的快速參考 | Microsoft Docs
description: 概述 v3 和 v4 .NET Bot Framework SDK 中的主要差異。
keywords: .net, bot 移轉, 對話方塊, v3 bot
author: JonathanFingold
ms.author: v-jofing
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 05/31/2019
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: b4226e842384caf1315170354c763a44c15b0c70
ms.sourcegitcommit: 18ff5705d15b8edc85fb43001969b173625eb387
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 05/31/2019
ms.locfileid: "66453215"
---
# <a name="net-migration-quick-reference"></a>.NET 移轉快速參考

BotBuilder .NET SDK v4 導入了幾項會影響 Bot 撰寫方式的基本變更。 本指南旨在提供快速參考來強調在 v3 和 v4 SDK 中完成工作的一般差異。

- 在 Bot 與通道之間傳遞資訊的方式已變更。 在 v3 中，您使用了 _Conversation_ 物件和 _SendAsync_ 方法來處理訊息，並大量使用 Autofac 來載入各種相依性。 在 v4 中，您可以使用 _Adapter_ 和 _TurnContext_ 物件來處理訊息，而且可以使用您選擇的相依性插入文件庫。

- 此外，對話方塊和 Bot 執行個體已進一步分離。 在 v3 中，對話方塊已內建到核心 SDK 中，並且在內部處理堆疊，而子對話方塊會以 _Call_ 和 _Forward_ 方法來載入。 在 v4 中，您現在會將對話方塊當作引數來傳入 Bot 執行個體，使對話堆疊有更大的撰寫彈性和開發人員控制，而子對話方塊會使用 _BeginDialogAsync_ 和 _ReplaceDialogAsync_ 方法來載入。

- 此外，v4 提供了 `ActivityHandler` 類別，可協助自動化處理不同類型的活動，例如_訊息_、_交談更新_和_事件_活動。

這些改良會變更以 .NET 開發 Bot 的語法，特別是建立 Bot 物件、定義對話及撰寫事件處理邏輯程式碼的語法。

本主題其餘部分會比較 .NET Bot Framework SDK v3 與 v4 中對等的建構。

## <a name="to-process-incoming-messages"></a>處理傳入訊息

### <a name="v3"></a>v3

```cs
[BotAuthentication]
public class MessagesController : ApiController
{
    public async Task<HttpResponseMessage> Post([FromBody]Activity activity)
    {
        if (activity.GetActivityType() == ActivityTypes.Message)
        {
            await Conversation.SendAsync(activity, () => new Dialogs.RootDialog());
        }

        return Request.CreateResponse(HttpStatusCode.OK);
    }
}
```

### <a name="v4"></a>v4

```cs
[Route("api/messages")]
[ApiController]
public class BotController : ControllerBase
{
    private readonly IBotFrameworkHttpAdapter Adapter;
    private readonly IBot Bot;

    public BotController(IBotFrameworkHttpAdapter adapter, IBot bot)
    {
        Adapter = adapter;
        Bot = bot;
    }

    [HttpPost]
    public async Task PostAsync()
    {
        await Adapter.ProcessAsync(Request, Response, Bot);
    }
}
```

## <a name="to-send-a-message-to-a-user"></a>將訊息傳送給使用者

### <a name="v3"></a>v3

```cs
await context.PostAsync("Hello and welcome to the help desk bot.");
```

### <a name="v4"></a>v4

```cs
await turnContext.SendActivityAsync("Hello and welcome to the help desk bot.");
```

## <a name="to-load-a-root-dialog"></a>載入根對話方塊

### <a name="v3"></a>v3

```cs
await Conversation.SendAsync(activity, () => new Dialogs.RootDialog());
```

### <a name="v4"></a>v4

```cs
// Create a DialogExtensions class with a Run method.
public static class DialogExtensions
{
    public static async Task Run(
        this Dialog dialog,
        ITurnContext turnContext,
        IStatePropertyAccessor<DialogState> accessor,
        CancellationToken cancellationToken)
    {
        var dialogSet = new DialogSet(accessor);
        dialogSet.Add(dialog);

        var dialogContext = await dialogSet.CreateContextAsync(turnContext, cancellationToken);

        var results = await dialogContext.ContinueDialogAsync(cancellationToken);
        if (results.Status == DialogTurnStatus.Empty)
        {
            await dialogContext.BeginDialogAsync(dialog.Id, null, cancellationToken);
        }
    }
}

// Call it from the ActivityHandler's OnMessageActivityAsync override
protected override async Task OnMessageActivityAsync(
    ITurnContext<IMessageActivity> turnContext,
    CancellationToken cancellationToken)
{
    // Run the Dialog with the new message Activity.
    await Dialog.Run(
        turnContext,
        ConversationState.CreateProperty<DialogState>("DialogState"),
        cancellationToken);
}
```

## <a name="to-start-a-child-dialog"></a>開始子對話方塊

### <a name="v3"></a>v3

```cs
context.Call(new NextDialog(), this.ResumeAfterNextDialog);
```

或

```cs
await context.Forward(new NextDialog(), this.ResumeAfterNextDialog, message);
```

### <a name="v4"></a>v4

```cs
dialogContext.BeginDialogAsync("<child-dialog-id>", options);
```

或

```cs
dialogContext.ReplaceDialogAsync("<child-dialog-id>", options);
```

## <a name="to-end-a-dialog"></a>結束對話方塊

### <a name="v3"></a>v3

```cs
context.Done(ReturnValue);
```

### <a name="v4"></a>v4

```cs
await context.EndDialogAsync(ReturnValue);
```

## <a name="to-prompt-a-user-for-input"></a>提示使用者輸入

### <a name="v3"></a>v3

```cs
PromptDialog.Choice(
    context,
    this.OnOptionSelected,
    Options, PromptMessage,
    ErrorMessage,
    3,
    PromptStyle.PerLine);
```

### <a name="v4"></a>v4

```cs
// In the dialog's constructor, register the prompt, and waterfall steps.
AddDialog(new TextPrompt(nameof(TextPrompt)));
AddDialog(new WaterfallDialog(nameof(WaterfallDialog), new WaterfallStep[]
{
    FirstStepAsync,
    SecondStepAsync,
}));

// The initial child Dialog to run.
InitialDialogId = nameof(WaterfallDialog);

// ...

// In the first step, invoke the prompt.
private async Task<DialogTurnResult> FirstStepAsync(
    WaterfallStepContext stepContext,
    CancellationToken cancellationToken)
{
    return await stepContext.PromptAsync(
        nameof(TextPrompt),
        new PromptOptions { Prompt = MessageFactory.Text("Please enter your destination.") },
        cancellationToken);
}

// In the second step, retrieve the Result from the stepContext.
private async Task<DialogTurnResult> SecondStepAsync(
    WaterfallStepContext stepContext,
    CancellationToken cancellationToken)
{
    var destination = (string)stepContext.Result;
}
```

## <a name="to-save-information-to-dialog-state"></a>將資訊儲存到對話方塊狀態

### <a name="v3"></a>v3

所有對話方塊和其欄位都已在 V3 中自動序列化。

### <a name="v4"></a>v4

```cs
// StepContext values are auto-serialized in V4, and scoped to the dialog.
stepContext.values.destination = destination;
```

## <a name="to-write-changes-in-state-to-the-persistance-layer"></a>將狀態變更寫入持續性層

### <a name="v3"></a>v3

依預設，狀態資料會在回合結束時自動儲存。

### <a name="v4"></a>v4

```cs
// You now must explicitly save state changes before the end of the turn.
await this.conversationState.saveChanges(context, false);
await this.userState.saveChanges(context, false);
```

## <a name="to-create-and-register-state-storage"></a>建立並註冊狀態儲存體

### <a name="v3"></a>v3

```cs
// Autofac was used internally by the sdk, and state was automatic.
Conversation.UpdateContainer(
builder =>
{
    builder.RegisterModule(new AzureModule(Assembly.GetExecutingAssembly()));
    var store = new InMemoryDataStore();
    builder.Register(c => store)
        .Keyed<IBotDataStore<BotData>>(AzureModule.Key_DataStore)
        .AsSelf()
        .SingleInstance();
});
```

### <a name="v4"></a>v4

```cs
// Create the storage we'll be using for User and Conversation state.
// In-memory storage is great for testing purposes.
services.AddSingleton<IStorage, MemoryStorage>();

// Create the user state (used in this bot's Dialog implementation).
services.AddSingleton<UserState>();

// Create the conversation state (used by the Dialog system itself).
services.AddSingleton<ConversationState>();

// The dialog that will be run by the bot.
services.AddSingleton<MainDialog>();

// Create the bot as a transient. In this case the ASP.NET controller is expecting an IBot.
services.AddTransient<IBot, DialogBot>();

// In the bot's ActivityHandler implementation, call SaveChangesAsync after the OnTurnAsync completes.
public class DialogBot : ActivityHandler
{
    protected readonly Dialog Dialog;
    protected readonly BotState ConversationState;
    protected readonly BotState UserState;

    public DialogBot(ConversationState conversationState, UserState userState, Dialog dialog)
    {
        ConversationState = conversationState;
        UserState = userState;
    }

    public override async Task OnTurnAsync(ITurnContext turnContext, CancellationToken cancellationToken = default(CancellationToken))
    {
        await base.OnTurnAsync(turnContext, cancellationToken);

        // Save any state changes that might have ocurred during the turn.
        await ConversationState.SaveChangesAsync(turnContext, false, cancellationToken);
        await UserState.SaveChangesAsync(turnContext, false, cancellationToken);
    }

    protected override async Task OnMessageActivityAsync(ITurnContext<IMessageActivity> turnContext, CancellationToken cancellationToken)
    {
        // Run the dialog, passing in the message activity for this turn.
        await Dialog.Run(turnContext, ConversationState.CreateProperty<DialogState>("DialogState"), cancellationToken);
    }
}
```

## <a name="to-catch-an-error-thrown-from-a-dialog"></a>攔截從對話方塊擲出的錯誤

### <a name="v3"></a>v3

```cs
// Create a custom IPostToBot implementation to catch exceptions.
public sealed class CustomPostUnhandledExceptionToUser : IPostToBot
{
    private readonly IPostToBot inner;
    private readonly IBotToUser botToUser;
    private readonly ResourceManager resources;
    private readonly System.Diagnostics.TraceListener trace;

    public CustomPostUnhandledExceptionToUser(IPostToBot inner, IBotToUser botToUser, ResourceManager resources, System.Diagnostics.TraceListener trace)
    {
        SetField.NotNull(out this.inner, nameof(inner), inner);
        SetField.NotNull(out this.botToUser, nameof(botToUser), botToUser);
        SetField.NotNull(out this.resources, nameof(resources), resources);
        SetField.NotNull(out this.trace, nameof(trace), trace);
    }

    async Task IPostToBot.PostAsync(IActivity activity, CancellationToken token)
    {
        try
        {
            await this.inner.PostAsync(activity, token);
        }
        catch (Exception ex)
        {
            try
            {
                // Log exception and send custom error message here.
                await this.botToUser.PostAsync("custom error message");
            }
            catch (Exception inner)
            {
                this.trace.WriteLine(inner);
            }

            throw;
        }
    }
}

// Register this using AutoFac, replacing the default PostUnhandledExceptionToUser.
builder
  .RegisterType<CustomPostUnhandledExceptionToUser>()
  .Keyed<IPostToBot>(typeof(PostUnhandledExceptionToUser));
```

### <a name="v4"></a>v4

```cs
// Provide an error handler in your implementation of the BotFrameworkHttpAdapter.
public class AdapterWithErrorHandler : BotFrameworkHttpAdapter
{
    public AdapterWithErrorHandler(
        ICredentialProvider credentialProvider,
        ILogger<BotFrameworkHttpAdapter> logger,
        ConversationState conversationState = null)
        : base(credentialProvider)
    {
        OnTurnError = async (turnContext, exception) =>
        {
            // Log any leaked exception from the application.
            logger.LogError($"Exception caught : {exception.Message}");

            // Send a catch-all apology to the user.
            await turnContext.SendActivityAsync("Sorry, it looks like something went wrong.");

            if (conversationState != null)
            {
                try
                {
                    // Delete the conversation state for the current conversation, to prevent the
                    // bot from getting stuck in a error-loop caused by being in a bad state.
                    // Conversation state is similar to "cookie-state" in a web page.
                    await conversationState.DeleteAsync(turnContext);
                }
                catch (Exception e)
                {
                    logger.LogError(
                        $"Exception caught on attempting to Delete ConversationState : {e.Message}");
                }
            }
        };
    }
}
```

## <a name="to-process-different-activity-types"></a>處理不同的活動類型

### <a name="v3"></a>v3

```cs
// Within your MessageController, check the message type.
string messageType = activity.GetActivityType();
if (messageType == ActivityTypes.Message)
{
    await Conversation.SendAsync(activity, () => new Dialogs.RootDialog());
}
else if (messageType == ActivityTypes.DeleteUserData)
{
}
else if (messageType == ActivityTypes.ConversationUpdate)
{
}
else if (messageType == ActivityTypes.ContactRelationUpdate)
{
}
else if (messageType == ActivityTypes.Typing)
{
}
```

### <a name="v4"></a>v4

```cs
// In the bot's ActivityHandler implementation, override relevant methods.

protected override async Task OnMessageActivityAsync(ITurnContext<IMessageActivity> turnContext, CancellationToken cancellationToken)
{
    // Handle message activities here.
}

protected override Task OnConversationUpdateActivityAsync(ITurnContext<IConversationUpdateActivity> turnContext, CancellationToken cancellationToken)
{
    // Handle conversation update activities in general here.
}

protected override Task OnEventActivityAsync(ITurnContext<IEventActivity> turnContext, CancellationToken cancellationToken)
{
    // Handle event activities in general here.
}
```

## <a name="to-log-all-activities"></a>若要記錄所有活動

### <a name="v3"></a>v3

已使用 [IActivityLogger](https://docs.microsoft.com/en-us/dotnet/api/microsoft.bot.builder.history.iactivitylogger)。

```csharp
builder.RegisterType<ActivityLoggerImplementation>().AsImplementedInterfaces().InstancePerDependency(); 

public class ActivityLoggerImplementation : IActivityLogger
{
    async Task IActivityLogger.LogAsync(IActivity activity)
    {
        // Store the activity.
    }
}
```

### <a name="v4"></a>v4

使用 [ITranscriptLogger](https://docs.microsoft.com/en-us/dotnet/api/microsoft.bot.builder.itranscriptlogger)。

```csharp
var transcriptMiddleware = new TranscriptLoggerMiddleware(new TranscriptLoggerImplementation(Configuration.GetSection("StorageConnectionString").Value));
adapter.Use(transcriptMiddleware);

public class TranscriptLoggerImplementation : ITranscriptLogger
{
    async Task ITranscriptLogger.LogActivityAsync(IActivity activity)
    {
        // Store the activity.
    }
}
```

## <a name="to-add-bot-state-storage"></a>若要新增 bot 狀態儲存體

已變更用於存放_使用者資料_、_對話資料_和_私人對話資料_的介面。

### <a name="v3"></a>v3

狀態使用 `IBotDataStore` 實作來持續運作，並使用 Autofac 將其插入 SDK 的對話方塊狀態系統。  Microsoft 在 [Microsoft.Bot.Builder.Azure](https://github.com/Microsoft/BotBuilder-Azure/) 中提供了 `MemoryStorage`、`DocumentDbBotDataStore`、`TableBotDataStore`，以及 `SqlBotDataStore` 個類別。

[IBotDataStore<BotData>](https://docs.microsoft.com/en-us/dotnet/api/microsoft.bot.builder.dialogs.internals.ibotdatastore-1?view=botbuilder-dotnet-3.0) 用來保存資料。

```csharp
Task<bool> FlushAsync(IAddress key, CancellationToken cancellationToken);
Task<T> LoadAsync(IAddress key, BotStoreType botStoreType, CancellationToken cancellationToken);
Task SaveAsync(IAddress key, BotStoreType botStoreType, T data, CancellationToken cancellationToken);
```

```csharp
var dbPath = ConfigurationManager.AppSettings["DocDbPath"];
var dbKey = ConfigurationManager.AppSettings["DocDbKey"];
var docDbUri = new Uri(dbPath);
var storage = new DocumentDbBotDataStore(docDbUri, dbKey);
builder.Register(c => storage)
                .Keyed<IBotDataStore<BotData>>(AzureModule.Key_DataStore)
                .AsSelf()
                .SingleInstance();
```

### <a name="v4"></a>v4

儲存層使用`IStorage`介面，在為 Bot 建立每個狀態管理物件時，指定儲存層物件，例如 `UserState`、`ConversationState` 或 `PrivateConversationState`。 狀態管理物件提供到基礎儲存層的金鑰，並且也可以當做屬性管理員。 例如，使用 `IPropertyManager.CreateProperty<T>(string name)` 來建立狀態屬性存取子。  這些屬性存取子用來擷取，以及傳入和傳出 Bot 基礎儲存體儲存的值。

使用 [IStorage](https://docs.microsoft.com/en-us/dotnet/api/microsoft.bot.builder.istorage?view=botbuilder-dotnet-stable) 來保存資料。

```csharp
Task DeleteAsync(string[] keys, CancellationToken cancellationToken = default(CancellationToken));
Task<IDictionary<string, object>> ReadAsync(string[] keys, CancellationToken cancellationToken = default(CancellationToken));
Task WriteAsync(IDictionary<string, object> changes, CancellationToken cancellationToken = default(CancellationToken));
```

```csharp
var storageOptions = new CosmosDbStorageOptions()
{
    AuthKey = configuration["cosmosKey"],
    CollectionId = configuration["cosmosCollection"],
    CosmosDBEndpoint = new Uri(configuration["cosmosPath"]),
    DatabaseId = configuration["cosmosDatabase"]
};

IStorage dataStore = new CosmosDbStorage(storageOptions);
var conversationState = new ConversationState(dataStore);
services.AddSingleton(conversationState);

```

## <a name="to-use-form-flow"></a>使用表單流程

### <a name="v3"></a>v3

[Microsoft.Bot.Builder.FormFlow](https://docs.microsoft.com/en-us/dotnet/api/microsoft.bot.builder.formflow?view=botbuilder-dotnet-3.0) 已包含在核心 Bot Builder SDK 中。

### <a name="v4"></a>v4

[Bot.Builder.Community.Dialogs.FormFlow](https://www.nuget.org/packages/Bot.Builder.Community.Dialogs.FormFlow/) 現在是 Bot Builder 社群程式庫。  來源位於社群[存放庫](https://github.com/BotBuilderCommunity/botbuilder-community-dotnet/tree/develop/libraries/Bot.Builder.Community.Dialogs.FormFlow)。

## <a name="to-use-luisdialog"></a>使用 LuisDialog

### <a name="v3"></a>v3

[Microsoft.Bot.Builder.Dialogs.LuisDialog](https://docs.microsoft.com/en-us/dotnet/api/microsoft.bot.builder.dialogs.luisdialog-1?view=botbuilder-dotnet-3.0) 已包含在核心 Bot Builder SDK 中。

### <a name="v4"></a>v4

[Bot.Builder.Community.Dialogs.Luis](https://www.nuget.org/packages/Bot.Builder.Community.Dialogs.Luis/) 現在是 Bot Builder 社群程式庫。  來源位於社群[存放庫](https://github.com/BotBuilderCommunity/botbuilder-community-dotnet/tree/develop/libraries/Bot.Builder.Community.Dialogs.Luis)。