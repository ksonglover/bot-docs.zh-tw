---
title: 保存使用者資料 | Microsoft Docs
description: 了解如何將使用者狀態資料儲存在 Bot Framework SDK 中的儲存體。
keywords: 保存使用者資料, 儲存體, 交談資料
author: v-ducvo
ms.author: v-ducvo
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 04/19/2019
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: bb1e909de69a7690e844701e182dddcebf91cc87
ms.sourcegitcommit: aea57820b8a137047d59491b45320cf268043861
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 04/22/2019
ms.locfileid: "59904921"
---
# <a name="persist-user-data"></a>保存使用者資料

[!INCLUDE[applies-to](../includes/applies-to.md)]

當 Bot 要求使用者輸入時，您可能會想要以某些形式的儲存體保存某些資訊。 Bot Framework SDK 可讓您使用「記憶體內部儲存體」或資料庫儲存體 (例如 CosmosDB) 來儲存使用者輸入。 針對 Bot 進行測試或建立原型期間，主要會使用本機儲存體類型。 不過，生產 Bot 最適合使用永續性儲存體類型 (例如資料庫儲存體)。

本主題說明如何定義儲存體物件，以及如何將使用者輸入儲存到該儲存體物件，以便保存。 我們會使用對話方塊向使用者詢問其名稱 (如果我們還不知道的話)。 不論您選擇使用的儲存體類型為何，用於連結和保存資料的程序都相同。 本主題中的程式碼使用 `CosmosDB` 作為用來保存資料的儲存體。

## <a name="prerequisites"></a>必要條件

根據您想要使用的開發環境，您必須有某些資源。

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

* [安裝 Visual Studio 2015 或更新版本](https://www.visualstudio.com/downloads/)。
* [安裝 BotBuilder V4 範本](https://marketplace.visualstudio.com/items?itemName=BotBuilder.botbuilderv4)。

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

* [安裝 Visual Studio Code](https://www.visualstudio.com/downloads/)。
* [安裝 Node.js v8.5 或更新版本](https://nodejs.org/en/)。
* [安裝 Yeoman](http://yeoman.io/)。
* 安裝 Node.js v4 Botbuilder 範本產生器。

    ```shell
    npm install generator-botbuilder
    ```

---

此教學課程會使用下列套件。

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

我們先從基本的 EchoBot 範本開始。 如需指示，請參閱[適用於 .NET 的快速入門](~/dotnet/bot-builder-dotnet-quickstart.md)。

從 NuGet 套件管理員安裝這些額外套件。

* **Microsoft.Bot.Builder.Azure**
* **Microsoft.Bot.Builder.Dialogs**

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

我們先從基本的 EchoBot 範本開始。 如需指示，請參閱[適用於 JavaScript 的快速入門](~/javascript/bot-builder-javascript-quickstart.md)。

安裝這些額外的 npm 套件。

```cmd
npm install --save botbuilder-dialogs
npm install --save botbuilder-azure
```

---

若要測試您在本教學課程中建立的 Bot，您必須安裝 [BotFramework 模擬器](https://github.com/Microsoft/BotFramework-Emulator)。

## <a name="create-a-cosmosdb-service-and-update-your-application-settings"></a>建立 CosmosDB 服務並更新應用程式設定

若要設定 CosmosDB 服務和資料庫，請遵循[使用 CosmosDB](bot-builder-howto-v4-storage.md#using-cosmos-db) 的指示。 其步驟摘要如下︰

1. 在新的瀏覽器視窗中，登入 <a href="http://portal.azure.com/" target="_blank">Azure 入口網站</a>。
1. 按一下 [建立資源] > [資料庫] > [Azure Cosmos DB]。
1. 在 [新增帳戶] 頁面的 [識別碼] 欄位中，提供唯一的名稱。 針對 [API]，選取 [SQL]，然後提供 [訂用帳戶]、[位置] 和 [資源群組] 資訊。
1. 按一下頁面底部的 [新增] 。

然後，在該服務中新增集合以用於此 Bot。

將用來新增該集合的資料庫識別碼和集合識別碼，以及 URI 和集合中金鑰設定的主要金鑰記錄起來，以供之後用來將 Bot 連線至服務。

### <a name="update-your-application-settings"></a>更新應用程式設定

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

更新 **appsettings.json** 檔案，在其中納入 CosmosDB 的連線資訊。

```csharp
{
  // Settings for CosmosDB.
  "CosmosDB": {
    "DatabaseID": "<your-database-identifier>",
    "CollectionID": "<your-collection-identifier>",
    "EndpointUri": "<your-CosmosDB-endpoint>",
    "AuthenticationKey": "<your-primary-key>"
  }
}
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

在專案資料夾中，找出 **.env** 檔案，然後使用 Cosmos 特有資料來新增這些項目。

**.env**

```text
DB_SERVICE_ENDPOINT=<your-CosmosDB-endpoint>
AUTH_KEY=<authentication key>
DATABASE=<your-primary-key>
COLLECTION=<your-collection-identifier>
```

然後，在 Bot 的主要 **index.js** 檔案中，取代 `storage` 以使用 `CosmosDbStorage` 而非 `MemoryStorage`。 執行期間會提取環境變數，並填入這些欄位。

```javascript
const storage = new CosmosDbStorage({
    serviceEndpoint: process.env.DB_SERVICE_ENDPOINT,
    authKey: process.env.AUTH_KEY, 
    databaseId: process.env.DATABASE,
    collectionId: process.env.COLLECTION
});
```

---

## <a name="create-storage-state-manager-and-state-property-accessor-objects"></a>建立儲存體、狀態管理員和狀態屬性存取子物件

Bot 會使用狀態管理和儲存體物件來管理和保存狀態。 管理員會提供抽象層，讓您使用狀態屬性存取子存取狀態屬性，而不受基礎儲存體類型所影響。 您可以使用狀態管理員來將資料寫入至儲存體。

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

### <a name="define-a-class-for-your-user-data"></a>定義使用者資料的類別

將檔案 **CounterState.cs** 重新命名為 **UserData.cs**，並將 **CounterState** 類別重新命名為 **UserData**。

更新此類別來保存您會收集的資料。

```csharp
/// <summary>
/// Class for storing persistent user data.
/// </summary>
public class UserData
{
    public string Name { get; set; }
}
```

### <a name="define-a-class-for-your-state-and-state-property-accessor-objects"></a>為狀態和狀態屬性存取子物件定義類別

將檔案 **EchoBotAccessors.cs** 重新命名為 **BotAccessors.cs**，並將 **EchoBotAccessors** 類別重新命名為 **BotAccessors**。

更新此類別來儲存 Bot 需要用到的狀態物件和狀態屬性存取子。

```csharp
using Microsoft.Bot.Builder;
using Microsoft.Bot.Builder.Dialogs;
using System;

public class BotAccessors
{
    public UserState UserState { get; }

    public ConversationState ConversationState { get; }

    public IStatePropertyAccessor<DialogState> DialogStateAccessor { get; set; }

    public IStatePropertyAccessor<UserData> UserDataAccessor { get; set; }

    public BotAccessors(UserState userState, ConversationState conversationState)
    {
        this.UserState = userState
            ?? throw new ArgumentNullException(nameof(userState));

        this.ConversationState = conversationState
            ?? throw new ArgumentNullException(nameof(conversationState));
    }
}
```

### <a name="update-the-startup-code-for-your-bot"></a>更新 Bot 的啟動程式碼

在 **Startup.cs** 檔案中，更新 using 陳述式。

```csharp
using System;
using System.Linq;
using Microsoft.AspNetCore.Builder;
using Microsoft.AspNetCore.Hosting;
using Microsoft.Bot.Builder;
using Microsoft.Bot.Builder.Azure;
using Microsoft.Bot.Builder.Dialogs;
using Microsoft.Bot.Builder.Integration;
using Microsoft.Bot.Builder.Integration.AspNet.Core;
using Microsoft.Bot.Configuration;
using Microsoft.Bot.Connector.Authentication;
using Microsoft.Extensions.Configuration;
using Microsoft.Extensions.DependencyInjection;
using Microsoft.Extensions.Logging;
using Microsoft.Extensions.Options;
```

在 `ConfigureServices` 方法中，更新新增 Bot 呼叫 (從備用儲存體物件建立所在位置開始)，然後註冊 Bot 存取子物件。

我們需要 `DialogState` 物件的對話狀態才能追蹤對話方塊狀態。 我們會為 Bot 所會使用的對話方塊狀態屬性存取子和對話方塊集合，註冊單一資料庫。 Bot 會針對使用者狀態建立自己的狀態屬性存取子。

`BotAccessors` 存取子可有效管理 Bot 多個狀態物件的儲存體。

```cs
public void ConfigureServices(IServiceCollection services)
{
    // Register your bot.
    services.AddBot<UserDataBot>(options =>
    {
        // ...

        // Use persistent storage and create state management objects.
        var cosmosSettings = Configuration.GetSection("CosmosDB");
        IStorage storage = new CosmosDbStorage(
            new CosmosDbStorageOptions
            {
                DatabaseId = cosmosSettings["DatabaseID"],
                CollectionId = cosmosSettings["CollectionID"],
                CosmosDBEndpoint = new Uri(cosmosSettings["EndpointUri"]),
                AuthKey = cosmosSettings["AuthenticationKey"],
            });
        options.State.Add(new ConversationState(storage));
        options.State.Add(new UserState(storage));
    });

    // Register the bot's state and state property accessor objects.
    services.AddSingleton<BotAccessors>(sp =>
    {
        var options = sp.GetRequiredService<IOptions<BotFrameworkOptions>>().Value;
        var userState = options.State.OfType<UserState>().FirstOrDefault();
        var conversationState = options.State.OfType<ConversationState>().FirstOrDefault();

        return new BotAccessors(userState, conversationState)
        {
            UserDataAccessor = userState.CreateProperty<UserData>("UserDataBot.UserData"),
            DialogStateAccessor = conversationState.CreateProperty<DialogState>("UserDataBot.DialogState"),
        };
    });
}
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

### <a name="update-your-server-code"></a>更新您的伺服器程式碼

在專案的 **index.js** 檔案中，更新下列 require 陳述式。

```javascript
// Import required bot services.
const { BotFrameworkAdapter, ConversationState, UserState } = require('botbuilder');
const { CosmosDbStorage } = require('botbuilder-azure');
```

我們會使用 `UserState` 來儲存本教學課程的資料。 我們需要建立新的 `userState` 物件，並更新這行程式碼以將第二個參數傳遞至 `MainDialog` 類別。

```javascript
// Create conversation state with in-memory storage provider.
const conversationState = new ConversationState(storage);
const userState = new UserState(storage);

// Create the main dialog.
const bot = new MyBot(conversationState, userState);
```

如果發生一般錯誤，請清除對話和使用者狀態。

```javascript
// Catch-all for errors.
adapter.onTurnError = async (context, error) => {
    // This check writes out errors to console log .vs. app insights.
    console.error(`\n [onTurnError]: ${error}`);
    // Send a message to the user
    context.sendActivity(`Oops. Something went wrong!`);
    // Clear out state
    await conversationState.load(context);
    await conversationState.clear(context);
    await userState.load(context);
    await userState.clear(context);
    // Save state changes.
    await conversationState.saveChanges(context);
    await userState.saveChanges(context);
};
```

並且更新 HTTP 伺服器迴圈來呼叫我們的 Bot 物件。

```javascript
// Listen for incoming requests.
server.post('/api/messages', (req, res) => {
    adapter.processActivity(req, res, async (context) => {
        // Route to main dialog.
        await bot.onTurn(context);
    });
});
```

### <a name="update-your-bot-logic"></a>更新您的 Bot 邏輯

在 `MyBot` 類別中，需要有必要程式庫以供 Bot 操作。 在本教學課程中，我們會使用 **對話方塊**程式庫。

```javascript
// Required packages for this bot
const { ActivityTypes } = require('botbuilder');
const { DialogSet, WaterfallDialog, TextPrompt, NumberPrompt } = require('botbuilder-dialogs');

```

更新 `MyBot` 類別的建構函式以接受第二個參數 `userState`。 此外，更新建構函式以定義本教學課程所需的狀態、對話方塊和提示。 在此案例中，我們會定義有兩個步驟的瀑布，其中「步驟 1」會要求使用者輸入其名稱，而「步驟 2」則會傳回使用者的回應。 該資訊則由 Bot 的主要邏輯保存。

```javascript
constructor(conversationState, userState) {

    // creates a new state accessor property.
    this.conversationState = conversationState;
    this.userState = userState;

    this.dialogState = this.conversationState.createProperty('dialogState');
    this.userDataAccessor = this.userState.createProperty('userData');

    this.dialogs = new DialogSet(this.dialogState);

    // Add prompts
    this.dialogs.add(new TextPrompt('textPrompt'));

    // Add a waterfall dialog to collect and return the user's name.
    this.dialogs.add(new WaterfallDialog('greetings', [
        async function (step) {
            return await step.prompt('textPrompt', "What is your name?");
        },
        async function (step) {
            return await step.endDialog(step.result);
        }
    ]));
}
```

---

在儲存使用者資料時，您有一些選擇。 您可以從中選擇 SDK 所提供的幾個具有不同範圍的狀態物件。 在這裡，我們會使用對話狀態來管理對話方塊狀態物件，使用使用者狀態來管理使用者資料。

如需儲存體和狀態大致情形的詳細資訊，請參閱[儲存體](bot-builder-howto-v4-storage.md)和[如何管理對話和使用者狀態](bot-builder-howto-v4-state.md)。

## <a name="create-a-greeting-dialog"></a>建立問候語對話方塊

我們會使用對話方塊來收集使用者的名稱。 為了讓這個案例保持簡單，對話會傳回使用者的名稱，Bot 則會管理使用者資料物件和狀態。

建立 **GreetingsDialog** 類別，並包含下列程式碼。

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

```cs
using Microsoft.Bot.Builder;
using Microsoft.Bot.Builder.Dialogs;

/// <summary>Defines a dialog for collecting a user's name.</summary>
public class GreetingsDialog : DialogSet
{
    /// <summary>The ID of the main dialog.</summary>
    public const string MainDialog = "main";

    /// <summary>The ID of the text prompt to use in the dialog.</summary>
    private const string TextPrompt = "textPrompt";

    /// <summary>Creates a new instance of this dialog set.</summary>
    /// <param name="dialogState">The dialog state property accessor to use for dialog state.</param>
    public GreetingsDialog(IStatePropertyAccessor<DialogState> dialogState)
        : base(dialogState)
    {
        // Add the text prompt to the dialog set.
        Add(new TextPrompt(TextPrompt));

        // Define the main dialog and add it to the set.
        Add(new WaterfallDialog(MainDialog, new WaterfallStep[]
        {
            async (stepContext, cancellationToken) =>
            {
                // Ask the user for their name.
                return await stepContext.PromptAsync(TextPrompt, new PromptOptions
                {
                    Prompt = MessageFactory.Text("What is your name?"),
                }, cancellationToken);
            },
            async (stepContext, cancellationToken) =>
            {
                // Assume that they entered their name, and return the value.
                return await stepContext.EndDialogAsync(stepContext.Result, cancellationToken);
            },
        }));
    }
}
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

請參閱上一節，當中的對話方塊會建立在 `MainDialog` 的建構函式內。

---

如需對話的詳細資訊，請參閱[如何提示輸入](bot-builder-prompts.md)和[如何管理簡單的對話流程](bot-builder-dialog-manage-conversation-flow.md)。

## <a name="update-your-bot-to-use-the-dialog-and-user-state"></a>更新 Bot 以使用對話方塊和使用者狀態

我們將會討論 Bot 建構和如何分開管理使用者輸入。

### <a name="add-the-dialog-and-a-user-state-accessor"></a>新增對話方塊和使用者狀態存取子

我們需要追蹤對話方塊執行個體，和使用者資料的狀態屬性存取子。

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

新增程式碼以將 Bot 初始化。

```cs
using System.Linq;
using System.Threading;
using System.Threading.Tasks;
using Microsoft.Bot.Builder;
using Microsoft.Bot.Builder.Dialogs;
using Microsoft.Bot.Schema;

/// <summary>Defines the bot for the persisting user data tutorial.</summary>
public class UserDataBot : IBot
{
    /// <summary>The bot's state and state property accessor objects.</summary>
    private BotAccessors Accessors { get; }

    /// <summary>The dialog set that has the dialog to use.</summary>
    private GreetingsDialog GreetingsDialog { get; }

    /// <summary>Create a new instance of the bot.</summary>
    /// <param name="options">The options to use for our app.</param>
    /// <param name="greetingsDialog">An instance of the dialog set.</param>
    public UserDataBot(BotAccessors botAccessors)
    {
        // Retrieve the bot's state and accessors.
        Accessors = botAccessors;

        // Create the greetings dialog.
        GreetingsDialog = new GreetingsDialog(Accessors.DialogStateAccessor);
    }
}
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

請參閱上一節，當中的這些狀態存取子會定義在 `MainDialog` 的建構函式內。

---

### <a name="update-the-turn-handler"></a>更新回合處理常式

回合處理常式會在使用者第一次加入對話時歡迎他們，並在他們傳送訊息給 Bot 時加以回應。 不論何時，如果 Bot 還不知道使用者的名稱，其就會開始問候語對話，向其詢問名稱。 對話方塊完成時，我們會使用狀態屬性存取子所產生的狀態物件，將其名稱儲存至使用者狀態。 當回合結束時，存取子和狀態管理員會將物件的變更寫出到儲存體。

我們也會新增對於「刪除使用者資料」活動的支援。

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

更新 Bot 的 `OnTurnAsync` 方法。

```cs
/// <summary>Handles incoming activities to the bot.</summary>
/// <param name="turnContext">The context object for the current turn.</param>
/// <param name="cancellationToken">A cancellation token that can be used by other objects
/// or threads to receive notice of cancellation.</param>
/// <returns>A task that represents the work queued to execute.</returns>
public async Task OnTurnAsync(ITurnContext turnContext, CancellationToken cancellationToken = default(CancellationToken))
{
    // Retrieve user data from state.
    UserData userData = await Accessors.UserDataAccessor.GetAsync(turnContext, () => new UserData());

    // Establish context for our dialog from the turn context.
    DialogContext dc = await GreetingsDialog.CreateContextAsync(turnContext);

    // Handle conversation update, message, and delete user data activities.
    switch (turnContext.Activity.Type)
    {
        case ActivityTypes.ConversationUpdate:

            // Greet any user that is added to the conversation.
            IConversationUpdateActivity activity = turnContext.Activity.AsConversationUpdateActivity();
            if (activity.MembersAdded.Any(member => member.Id != activity.Recipient.Id))
            {
                if (userData.Name is null)
                {
                    // If we don't already have their name, start a dialog to collect it.
                    await turnContext.SendActivityAsync("Welcome to the User Data bot.");
                    await dc.BeginDialogAsync(GreetingsDialog.MainDialog);
                }
                else
                {
                    // Otherwise, greet them by name.
                    await turnContext.SendActivityAsync($"Hi {userData.Name}! Welcome back to the User Data bot.");
                }
            }

            break;

        case ActivityTypes.Message:

            // If there's a dialog running, continue it.
            if (dc.ActiveDialog != null)
            {
                var dialogTurnResult = await dc.ContinueDialogAsync();
                if (dialogTurnResult.Status == DialogTurnStatus.Complete
                    && dialogTurnResult.Result is string name
                    && !string.IsNullOrWhiteSpace(name))
                {
                    // If it completes successfully and returns a valid name, save the name and greet the user.
                    userData.Name = name;
                    await turnContext.SendActivityAsync($"Pleased to meet you {userData.Name}.");
                }
            }
            else if (userData.Name is null)
            {
                // Else, if we don't have the user's name yet, ask for it.
                await dc.BeginDialogAsync(GreetingsDialog.MainDialog);
            }
            else
            {
                // Else, echo the user's message text.
                await turnContext.SendActivityAsync($"{userData.Name} said, '{turnContext.Activity.Text}'.");
            }

            break;

        case ActivityTypes.DeleteUserData:

            // Delete the user's data.
            userData.Name = null;
            await turnContext.SendActivityAsync("I have deleted your user data.");

            break;
    }

    // Update the user data in the turn's state cache.
    await Accessors.UserDataAccessor.SetAsync(turnContext, userData, cancellationToken);

    // Persist any changes to storage.
    await Accessors.UserState.SaveChangesAsync(turnContext, false, cancellationToken);
    await Accessors.ConversationState.SaveChangesAsync(turnContext, false, cancellationToken);
}
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

更新的 Bot 的 `onTurn` 處理常式。

```javascript
async onTurn(turnContext) {
    const dc = await this.dialogs.createContext(turnContext); // Create dialog context
    const userData = await this.userDataAccessor.get(turnContext, {});

    switch (turnContext.activity.type) {
        case ActivityTypes.ConversationUpdate:
            if (turnContext.activity.membersAdded[0].name !== 'Bot') {
                if (userData.name) {
                    await turnContext.sendActivity(`Hi ${userData.name}! Welcome back to the User Data bot.`);
                }
                else {
                    // send a "this is what the bot does" message
                    await turnContext.sendActivity('Welcome to the User Data bot.');
                    await dc.beginDialog('greetings');
                }
            }
            break;
        case ActivityTypes.Message:
            // If there is an active dialog running, continue it
            if (dc.activeDialog) {
                var turnResult = await dc.continueDialog();
                if (turnResult.status == "complete" && turnResult.result) {
                    // If it completes successfully and returns a value, save the name and greet the user.
                    userData.name = turnResult.result;
                    await this.userDataAccessor.set(turnContext, userData);
                    await turnContext.sendActivity(`Pleased to meet you ${userData.name}.`);
                }
            }
            // Else, if we don't have the user's name yet, ask for it.
            else if (!userData.name) {
                await dc.beginDialog('greetings');
            }
            // Else, echo the user's message text.
            else {
                await turnContext.sendActivity(`${userData.name} said, ${turnContext.activity.text}.`);
            }
            break;
        case ActivityTypes.DeleteUserData:
            // Delete the user's data.
            // Note: You can use the Emulator to send this activity.
            userData.name = null;
            await this.userDataAccessor.set(turnContext, userData);
            await turnContext.sendActivity("I have deleted your user data.");
            break;
    }

    // Save changes to the conversation and user states.
    await this.conversationState.saveChanges(turnContext);
    await this.userState.saveChanges(turnContext);
}
```

---

## <a name="start-your-bot-in-visual-studio"></a>在 Visual Studio 中啟動 Bot

建置並執行應用程式。

## <a name="start-the-emulator-and-connect-your-bot"></a>啟動模擬器並且連線至您的 Bot

接下來，請啟動模擬器，然後在模擬器中連線至您的 Bot：

1. 按一下模擬器 [歡迎] 索引標籤中的 [開啟 Bot] 連結。
2. 選取位於您建立 Visual Studio 解決方案的目錄中的 .bot 檔案。

## <a name="interact-with-your-bot"></a>與您的 Bot 互動

傳送訊息給 Bot，Bot 就會以訊息回應。
![模擬器執行中](../media/emulator-v4/emulator-running.png)

## <a name="next-steps"></a>後續步驟

> [!div class="nextstepaction"]
> [管理對話和使用者狀態](bot-builder-howto-v4-state.md)
