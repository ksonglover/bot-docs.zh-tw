---
title: 儲存使用者和對話資料 | Microsoft Docs
description: 了解如何藉由 Bot Builder SDK 儲存及擷取資料。
keywords: 交談狀態, 使用者狀態, 交談, 儲存狀態, 管理 Bot 狀態
author: ivorb
ms.author: v-ivorb
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 11/26/18
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 8c3aad54a9e80e8a046a6e31a5109a1de8c61a8b
ms.sourcegitcommit: 91156d0866316eda8d68454a0c4cd74be5060144
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 12/07/2018
ms.locfileid: "53010503"
---
# <a name="save-user-and-conversation-data"></a>儲存使用者和對話資料

[!INCLUDE [pre-release-label](../includes/pre-release-label.md)]

Bot 原本就是無狀態。 部署 Bot 後，在不同的回合中，Bot 便無法在相同程序中或同部電腦上執行。 不過，Bot 可能需要追蹤交談的內容，以便管理其行為，以及記住先前問題的答案。 SDK 的狀態和儲存體功能可讓您將狀態新增至 Bot。

## <a name="prerequisites"></a>必要條件

- 需具備 [Bot 基本概念](bot-builder-basics.md)以及 Bot 如何[管理狀態](bot-builder-concept-state.md)的知識。
- 本文中的程式碼是以 **StateBot** 範例為基礎。 您需要採用 [C#](https://github.com/Microsoft/BotFramework-Samples/tree/master/SDKV4-Samples/dotnet_core/StateBot) 或 [JS]() 的一份範例。
- [Bot Framework 模擬器](https://aka.ms/Emulator-wiki-getting-started)，以在本機測試 Bot。

## <a name="about-the-sample-code"></a>關於範例程式碼

本文討論管理 Bot 狀態的設定層面。 若要新增狀態，我們會設定狀態屬性、狀態管理和儲存體，然後在 Bot 中使用這些屬性。

- 每個「狀態屬性」都包含 Bot 的狀態資訊。
- 每個狀態屬性存取子，都可讓您取得或設定相關聯的狀態屬性值。
- 每個狀態管理物件，都會自動讀取相關聯的狀態資訊，以及將其寫入至儲存體。
- 儲存層會連線到狀態的支援儲存體，例如記憶體內部 (用於測試)，或 Azure Cosmos DB 儲存體 (用於生產環境)。

我們必須設定具有狀態屬性存取子的 Bot，以便 Bot 在處理活動時，於執行階段取得及設定狀態。 使用狀態管理物件可建立狀態屬性存取子，而使用儲存層可建立狀態管理物件。 因此，我們會從儲存體層級開始，從該處逐步發展。

## <a name="configure-storage"></a>設定儲存體

因為我們不打算部署此 Bot，所以我們會使用「記憶體儲存體」在下一個步驟中設定使用者和交談狀態。

### <a name="ctabcsharp"></a>[C#](#tab/csharp)

在 **Startup.cs** 中，設定儲存層。

```csharp
public void ConfigureServices(IServiceCollection services)
{
    // ...
    IStorage storage = new MemoryStorage();
    // ...
}
```

### <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

在 **index.js** 中，設定儲存層。

```javascript
// Define state store for your bot.
const memoryStorage = new MemoryStorage();
```

---

## <a name="create-state-management-objects"></a>建立狀態管理物件

我們會追蹤「使用者」和「交談」狀態，在下一個步驟中使用，來建立「狀態屬性存取子」。

### <a name="ctabcsharp"></a>[C#](#tab/csharp)

在 **Startup.cs** 中，參考您建立狀態管理物件時的儲存層。

```csharp
public void ConfigureServices(IServiceCollection services)
{
    // ...
    ConversationState conversationState = new ConversationState(storage);
    UserState userState = new UserState(storage);
    // ...
}
```

### <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

在 **index.js** 檔案中，將 `UserState` 新增到您的 require 陳述式。

```javascript
const { BotFrameworkAdapter, MemoryStorage, ConversationState, UserState } = require('botbuilder');
```

然後，參考您建立交談和使用者狀態管理物件時的儲存層。

```javascript
// Create conversation and user state with in-memory storage provider.
const conversationState = new ConversationState(memoryStorage);
const userState = new UserState(memoryStorage);
```

---

## <a name="create-state-property-accessors"></a>建立狀態屬性存取子

若要「宣告」狀態屬性，先使用其中一個狀態管理物件，建立狀態屬性存取子。 我們會設定 Bot 以追蹤下列資訊：

- 使用者的名稱 (我們會定義於使用者狀態中)。
- 我們是否剛詢問過使用者的名稱，以及他們剛傳送訊息的一些額外資訊。

Bot 可使用存取子，從回合內容中取得狀態屬性。

### <a name="ctabcsharp"></a>[C#](#tab/csharp)

我們先定義類別，以包含我們想要在每種狀態中管理的所有資訊。

- Bot 將要收集的使用者資訊的 `UserProfile` 類別。
- `ConversationData` 類別可追蹤有關訊息抵達時間和訊息寄件者的資訊。

```csharp
// Defines a state property used to track information about the user.
public class UserProfile
{
    public string Name { get; set; }
}
```

```csharp
// Defines a state property used to track conversation data.
public class ConversationData
{
    // The time-stamp of the most recent incoming message.
    public string Timestamp { get; set; }

    // The ID of the user's channel.
    public string ChannelId { get; set; }

    // Track whether we have already asked the user's name
    public bool PromptedUserForName { get; set; } = false;
}
```

接下來，我們會定義一個類別，其中包含我們需要設定 Bot 執行個體的狀態管理資訊。

```csharp
public class StateBotAccessors
{
    public StateBotAccessors(ConversationState conversationState, UserState userState)
    {
        ConversationState = conversationState ?? throw new ArgumentNullException(nameof(conversationState));
        UserState = userState ?? throw new ArgumentNullException(nameof(userState));
    }
  
    public static string UserProfileName { get; } = "UserProfile";

    public static string ConversationDataName { get; } = "ConversationData";

    public IStatePropertyAccessor<UserProfile> UserProfileAccessor { get; set; }

    public IStatePropertyAccessor<ConversationData> ConversationDataAccessor { get; set; }
  
    public ConversationState ConversationState { get; }
  
    public UserState UserState { get; }
}
```

### <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

我們會將狀態管理物件直接傳遞至 Bot 的建構函式，並且讓 Bot 建立自己的狀態屬性存取子。

在 **index.js** 中，提供您建立 Bot 時的狀態管理物件。

```javascript
// Create the bot.
const myBot = new MyBot(conversationState, userState);
```

在 **bot.js** 中，定義您管理和追蹤狀態所需的識別碼。

```javascript
// The accessor names for the conversation data and user profile state property accessors.
const CONVERSATION_DATA_PROPERTY = 'conversationData';
const USER_PROFILE_PROPERTY = 'userProfile';
```

---

## <a name="configure-your-bot"></a>設定 Bot

我們現在已經準備好定義狀態屬性存取子，以及設定 Bot。
我們將使用交談流程狀態屬性存取子適用的交談狀態，管理物件。
我們將使用使用者設定檔狀態屬性存取子適用的使用者狀態，管理物件。

### <a name="ctabcsharp"></a>[C#](#tab/csharp)

在 **Startup.cs** 中，我們會設定 ASP.NET 以提供配套狀態屬性和管理物件。 透過 ASP.NET Core 中的相依性插入架構，從 Bot 的建構函式中擷取。

```csharp
public void ConfigureServices(IServiceCollection services)
{
    // ...
    services.AddSingleton<StateBotAccessors>(sp =>
    {
        // Create the custom state accessor.
        return new StateBotAccessors(conversationState, userState)
        {
            ConversationDataAccessor = conversationState.CreateProperty<ConversationData>(StateBotAccessors.ConversationDataName),
            UserProfileAccessor = userState.CreateProperty<UserProfile>(StateBotAccessors.UserProfileName),
        };
    });
}
```

在 Bot 的建構函式中，ASP.NET 會在建立 Bot 時提供 `CustomPromptBotAccessors` 物件。

```csharp
// Defines a bot for filling a user profile.
public class CustomPromptBot : IBot
{
    private readonly StateBotAccessors _accessors;

    public StateBot(StateBotAccessors accessors, ILoggerFactory loggerFactory)
    {
        // ...
        accessors = accessors ?? throw new System.ArgumentNullException(nameof(accessors));
    }

    // The bot's turn handler and other supporting code...
}
```

### <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

在 Bot 的建構函式 (**bot.js** 檔案) 中，我們會建立狀態屬性存取子並將其新增至 Bot。 我們也會新增狀態管理物件的參考，因為我們需要這些參考以儲存我們所做的任何狀態變更。

```javascript
constructor(conversationState, userState) {
    // Create the state property accessors for the conversation data and user profile.
    this.conversationData = conversationState.createProperty(CONVERSATION_DATA_PROPERTY);
    this.userProfile = userState.createProperty(USER_PROFILE_PROPERTY);

    // The state management objects for the conversation and user state.
    this.conversationState = conversationState;
    this.userState = userState;
}
```

---

## <a name="access-state-from-your-bot"></a>從 Bot 存取狀態

前幾節討論了初始階段步驟，以將狀態屬性存取子新增至 Bot。
現在，我們可以在執行階段使用這些存取子，來讀取和寫入狀態資訊。

1. 在我們使用狀態屬性之前，我們會使用每個存取子從儲存體載入屬性，並從狀態快取中取得該屬性。
   - 每當您透過存取子取得狀態屬性時，就應該提供預設值。 否則，您可能收到 null 值錯誤。
1. 在結束回合處理常式之前：
   1. 我們會使用存取子的 _set_ 方法，將變更推送至 Bot 狀態。
   1. 我們會使用狀態管理物件的 _save changes_ 方法，將這些變更寫入儲存體。

### <a name="ctabcsharp"></a>[C#](#tab/csharp)

```csharp
// The bot's turn handler.
public async Task OnTurnAsync(ITurnContext turnContext, CancellationToken cancellationToken = default(CancellationToken))
{
    if (turnContext.Activity.Type == ActivityTypes.Message)
    {
        // Get the state properties from the turn context.
        UserProfile userProfile =
            await _accessors.UserProfileAccessor.GetAsync(turnContext, () => new UserProfile());
        ConversationData conversationData =
            await _accessors.ConversationDataAccessor.GetAsync(turnContext, () => new ConversationData());

        if (string.IsNullOrEmpty(userProfile.Name))
        {
            // First time around this is set to false, so we will prompt user for name.
            if (conversationData.PromptedUserForName)
            {
                // Set the name to what the user provided
                userProfile.Name = turnContext.Activity.Text?.Trim();

                // Acknowledge that we got their name.
                await turnContext.SendActivityAsync($"Thanks {userProfile.Name}.");

                // Reset the flag to allow the bot to go though the cycle again.
                conversationData.PromptedUserForName = false;
            }
            else
            {
                // Prompt the user for their name.
                await turnContext.SendActivityAsync($"What is your name?");

                // Set the flag to true, so we don't prompt in the next turn.
                conversationData.PromptedUserForName = true;
            }

            // Save user state and save changes.
            await _accessors.UserProfileAccessor.SetAsync(turnContext, userProfile);
            await _accessors.UserState.SaveChangesAsync(turnContext);
        }
        else
        {
            // Add message details to the conversation data.
            conversationData.Timestamp = turnContext.Activity.Timestamp.ToString();
            conversationData.ChannelId = turnContext.Activity.ChannelId.ToString();

            // Display state data
            await turnContext.SendActivityAsync($"{userProfile.Name} sent: {turnContext.Activity.Text}");
            await turnContext.SendActivityAsync($"Message received at: {conversationData.Timestamp}");
            await turnContext.SendActivityAsync($"Message received from: {conversationData.ChannelId}");
        }

        // Update conversation state and save changes.
        await _accessors.ConversationDataAccessor.SetAsync(turnContext, conversationData);
        await _accessors.ConversationState.SaveChangesAsync(turnContext);
    }
}
```

### <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

```javascript
// The bot's turn handler.
async onTurn(turnContext) {
    if (turnContext.activity.type === ActivityTypes.Message) {
        // Get the state properties from the turn context.
        const userProfile = await this.userProfile.get(turnContext, {});
        const conversationData = await this.conversationData.get(
            turnContext, { promptedForUserName: false });

        if (!userProfile.name) {
            // First time around this is undefined, so we will prompt user for name.
            if (conversationData.promptedForUserName) {
                // Set the name to what the user provided.
                userProfile.name = turnContext.activity.text;

                // Acknowledge that we got their name.
                await turnContext.sendActivity(`Thanks ${userProfile.name}.`);

                // Reset the flag to allow the bot to go though the cycle again.
                conversationData.promptedForUserName = false;
            } else {
                // Prompt the user for their name.
                await turnContext.sendActivity('What is your name?');

                // Set the flag to true, so we don't prompt in the next turn.
                conversationData.promptedForUserName = true;
            }
            // Save user state and save changes.
            await this.userProfile.set(turnContext, userProfile);
            await this.userState.saveChanges(turnContext);
        } else {
            // Add message details to the conversation data.
            conversationData.timestamp = turnContext.activity.timestamp.toLocaleString();
            conversationData.channelId = turnContext.activity.channelId;

            // Display state data.
            await turnContext.sendActivity(`${userProfile.name} sent: ${turnContext.activity.text}`);
            await turnContext.sendActivity(`Message received at: ${conversationData.timestamp}`);
            await turnContext.sendActivity(`Message received from: ${conversationData.channelId}`);
        }
        // Update conversation state and save changes.
        await this.conversationData.set(turnContext, conversationData);
        await this.conversationState.saveChanges(turnContext);
    }
}
```

---

## <a name="test-the-bot"></a>測試 Bot

1. 在您的電腦本機執行範例。 如需相關指示，請參閱 [C#](https://github.com/Microsoft/BotFramework-Samples/tree/master/SDKV4-Samples/dotnet_core/StateBot) 或 [JS](https://github.com/Microsoft/BotFramework-Samples/tree/master/SDKV4-Samples/js/stateBot) 範例的讀我檔案。
1. 使用模擬器來測試 Bot，如下所示。

![測試狀態 Bot 範例](media/state-bot-testing-emulator.png)

## <a name="additional-resources"></a>其他資源

**隱私權：** 如果您想要儲存使用者的個人資料，請務必符合[一般資料保護規範](https://blog.botframework.com/2018/04/23/general-data-protection-regulation-gdpr)。

**狀態管理：** 所有狀態管理呼叫都是非同步的，預設會使用「最後寫入為準」。 實務上，您應該在 Bot 中儘可能密集取得、設定及儲存狀態。

**重要商務資料：** 使用 Bot 狀態來儲存喜好設定、使用者名稱或其所排序的最後一個項目，但不要來儲存重要的商務資料。 對於重要資料，請[建立自己的儲存體元件](bot-builder-custom-storage.md)，或直接寫入[儲存體](bot-builder-howto-v4-storage.md)。

**Recognizer-Text：** 此範例會使用 Microsoft/Recognizers-Text 程式庫來剖析及驗證使用者輸入。 如需詳細資訊，請參閱[概觀](https://github.com/Microsoft/Recognizers-Text#microsoft-recognizers-text-overview)頁面。

## <a name="next-step"></a>後續步驟

您現在已知道如何設定狀態來協助您讀取 Bot 資料以及將其寫入儲存體，讓我們說明如何詢問使用者一連串的問題、驗證其答案，以及儲存其輸入。

> [!div class="nextstepaction"]
> [建立您自己的提示，以收集使用者輸入](bot-builder-primitive-prompts.md)。
