---
title: 管理對話和使用者狀態 | Microsoft Docs
description: 了解如何藉由 Bot Builder SDK for .NET 儲存及擷取資料。
keywords: 對話狀態、使用者狀態、對話流程
author: ivorb
ms.author: v-ivorb
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 09/18/18
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 30ce9c9ff5c458758a4cc9612b8f9947fa12734c
ms.sourcegitcommit: 782b3a2e788c25effd7d150a070bd2819ea92dad
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 11/01/2018
ms.locfileid: "50743652"
---
# <a name="manage-conversation-and-user-state"></a>管理對話和使用者狀態

[!INCLUDE [pre-release-label](../includes/pre-release-label.md)]

Bot 的設計是否精良，關鍵之一是必須能夠追蹤交談的上下文，讓 Bot 記住先前問題的答案這類事項。 根據 Bot 的用途，您甚至需要持續追蹤狀態，或將資料儲存超過對話存留期的時間。 Bot 必須記住*狀態*的資訊才能適當回應內送的訊息。 Bot Builder SDK 會提供兩個類別，以便用將狀態資料儲存和擷取為與使用者或對話相關聯的物件。

- **對話狀態**可協助您的 Bot 追蹤目前 Bot 與使用者進行中的對話。 如果您的 Bot 需要完成一系列的步驟或在對話主題之間進行切換，您可以使用對話屬性來按照順序管理步驟或追蹤目前的主題。 

- **使用者狀態**有許多用途，例如判斷使用者先前離開對話的位置，或單純地以名字來問候回來的使用者。 如果您儲存使用者的喜好設定，下次聊天時就可以使用該資訊來自訂對話。 例如，您可能會在有使用者感興趣的主題之新聞文章，或有可用約會時通知使用者。 

`ConversationState` 和 `UserState` 都屬於狀態類別，而且是特製化的 `BotState` 類別，具有可控制存留期和其中所儲存物件範圍的原則。 需要儲存狀態的元件會建立及註冊具有一個類型的屬性，並使用屬性存取子來存取此狀態。 Bot 狀態管理員可以使用記憶體儲存體、CosmosDB 和 Blob 儲存體。 

> [!NOTE] 
> 使用 Bot 狀態管理員來儲存喜好設定、使用者名稱或其所排序的最後一個項目，但不要來儲存重要的商務資料。 對於重要資料，請建立自己的儲存體元件，或直接寫入[儲存體](bot-builder-howto-v4-storage.md)。
> 記憶體內部的資料儲存體僅供測試之用。 這是可變更的臨時儲存體。 每次 Bot 重新啟動時，就會清除資料。

## <a name="using-conversation-state-and-user-state-to-direct-conversation-flow"></a>使用對話狀態和使用者狀態來引導對話流程
在設計對話流程時，定義狀態旗標來進行對話流程導向是很實用的做法。 旗標可以是簡單的布林值類型，也可以是包含目前主題名稱的類型。 旗標可協助您追蹤您在對話中的位置。 例如，布林值類型的旗標可以讓您知道您是否位於對話中，而主題名稱屬性則可告訴您目前位於哪個對話中。



# <a name="ctabcsharp"></a>[C#](#tab/csharp)
### <a name="conversation-and-user-state"></a>對話和使用者狀態
您可以使用 [Echo Bot 搭配計數器範例](https://aka.ms/EchoBot-With-Counter)，作為此操作說明的起點。 首先，建立 `TopicState` 類別來管理 `TopicState.cs` 中的目前對話主題，如下所示：

```csharp
public class TopicState
{
   public string Prompt { get; set; } = "askName";
}
``` 
然後在 `UserProfile.cs` 中建立 `UserProfile` 類別來管理使用者狀態。
```csharp
public class UserProfile
{
    public string UserName { get; set; }
    public string TelephoneNumber { get; set; }
}
``` 
`TopicState` 類別有一個旗標可追蹤我們在對話中的位置，並使用對話狀態來儲存。 Prompt 會初始化為 "askName" 以起始對話。 一旦 Bot 收到來自使用者的回應，Prompt 就會重新定義為 "askNumber" 以起始下一次對話。 `UserProfile` 類別會追蹤使用者名稱和電話號碼，並將其儲存在使用者狀態中。

### <a name="property-accessors"></a>屬性存取子
我們範例中的 `EchoBotAccessors` 類別在建立為單一實體，並透過相依性插入傳入 `class EchoWithCounterBot : IBot` 建構函式。 `EchoBotAccessors` 類別包含 `ConversationState`、`UserState`，以及相關聯的 `IStatePropertyAccessor`。 `conversationState` 物件會儲存主題狀態，而 `userState` 物件會儲存使用者設定檔資訊。 `ConversationState` 和 `UserState` 物件稍後會在 Startup.cs 檔案中建立。 對話和使用者狀態物件都是我們保存對話和使用者範圍內任何項目的位置。 

已更新建構函式來包含 `UserState`，如下所示：
```csharp
public EchoBotAccessors(ConversationState conversationState, UserState userState)
{
    ConversationState = conversationState ?? throw new ArgumentNullException(nameof(conversationState));
    UserState = userState ?? throw new ArgumentNullException(nameof(userState));
}
```
建立 `TopicState` 和 `UserProfile` 存取子的唯一名稱。
```csharp
public static string UserProfileName { get; } = $"{nameof(EchoBotAccessors)}.UserProfile";
public static string TopicStateName { get; } = $"{nameof(EchoBotAccessors)}.TopicState";
```
接下來，定義兩個存取子。 第一個存取子會儲存對話的主題，而第二個存取子會儲存使用者的名稱和電話號碼。
```csharp
public IStatePropertyAccessor<TopicState> TopicState { get; set; }
public IStatePropertyAccessor<UserProfile> UserProfile { get; set; }
```

已定義用以取得 ConversationState 的屬性，但您必須新增 `UserState`，如下所示：
```csharp
public ConversationState ConversationState { get; }
public UserState UserState { get; }
```
進行變更之後，請儲存檔案。 接下來，我們會更新 Startup 類別，以建立 `UserState` 物件來保存使用者範圍內的任何項目。 `ConversationState` 已經存在。 
```csharp

services.AddBot<EchoWithCounterBot>(options =>
{
    ...

    IStorage dataStore = new MemoryStorage();
    
    var conversationState = new ConversationState(dataStore);
    options.State.Add(conversationState);
        
    var userState = new UserState(dataStore);  
    options.State.Add(userState);
});
```
`options.State.Add(ConversationState);` 和 `options.State.Add(userState);` 兩行分別會新增對話狀態和使用者狀態。 接下來，建立並註冊狀態存取子。 這裡建立的存取子每次都會傳入 IBot 衍生的類別。 修改程式碼以包含使用者狀態，如下所示：
```csharp
services.AddSingleton<EchoBotAccessors>(sp =>
{
   ...
    var userState = options.State.OfType<UserState>().FirstOrDefault();
    if (userState == null)
    {
        throw new InvalidOperationException("UserState must be defined and added before adding user-scoped state accessors.");
    }
   ...
 });
```

接下來，使用 `TopicState` 和 `UserProfile` 來建立兩個存取子，並透過相依性插入將其傳入 `class EchoWithCounterBot : IBot` 類別。
```csharp
services.AddSingleton<EchoBotAccessors>(sp =>
{
   ...
    var accessors = new EchoBotAccessors(conversationState, userState)
    {
        TopicState = conversationState.CreateProperty<TopicState>(EchoBotAccessors.TopicStateName),
        UserProfile = userState.CreateProperty<UserProfile>(EchoBotAccessors.UserProfileName),
     };

     return accessors;
 });
```

對話和使用者狀態會透過 `services.AddSingleton` 程式碼區塊連結到單一實體，並透過程式碼中以 `var accessors = new EchoBotAccessor(conversationState, userState)` 開頭的狀態存放區存取子儲存。

### <a name="use-conversation-and-user-state-properties"></a>使用對話和使用者狀態屬性 
在 `EchoWithCounterBot : IBot` 類別的 `OnTurnAsync` 處理常式中，修改程式碼以提示輸入使用者名稱，然後輸入電話號碼。 為了追蹤我們在對話中的何處，我們會使用 TopicState 中定義的 Prompt 屬性。 這個屬性已初始化為 "askName"。 一旦取得使用者名稱，我們會將其設定為 "askNumber "，並將 UserName 設定為使用者輸入的名稱。 收到電話號碼之後，您可傳送確認訊息並將提示設定為「確認」，因為您位於對話的結尾。

```csharp
if (turnContext.Activity.Type == ActivityTypes.Message)
{
    // Get the conversation state from the turn context.
    var convo = await _accessors.TopicState.GetAsync(turnContext, () => new TopicState());
    
    // Get the user state from the turn context.
    var user = await _accessors.UserProfile.GetAsync(turnContext, () => new UserProfile());
    
    // Ask user name. The Prompt was initialiazed as "askName" in the TopicState.cs file.
    if (convo.Prompt == "askName")
    {
        await turnContext.SendActivityAsync("What is your name?");
        
        // Set the Prompt to ask the next question for this conversation
        convo.Prompt = "askNumber";
        
        // Set the property using the accessor
        await _accessors.TopicState.SetAsync(turnContext, convo);
        
        //Save the new prompt into the conversation state.
        await _accessors.ConversationState.SaveChangesAsync(turnContext);
    }
    else if (convo.Prompt == "askNumber")
    {
        // Set the UserName that is defined in the UserProfile class
        user.UserName = turnContext.Activity.Text;
        
        // Use the user name to prompt the user for phone number
        await turnContext.SendActivityAsync($"Hello, {user.UserName}. What's your telephone number?");
        
        // Set the Prompt now that we have collected all the data
        convo.Prompt = "confirmation";
                 
        await _accessors.TopicState.SetAsync(turnContext, convo);
        await _accessors.ConversationState.SaveChangesAsync(turnContext);

        await _accessors.UserProfile.SetAsync(turnContext, user);
        await _accessors.UserState.SaveChangesAsync(turnContext);
    }
    else if (convo.Prompt == "confirmation")
    { 
        // Set the TelephoneNumber that is defined in the UserProfile class
        user.TelephoneNumber = turnContext.Activity.Text;

        await turnContext.SendActivityAsync($"Got it, {user.UserName}. I'll call you later.");

        // reset initial prompt state
        convo.Prompt = "askName"; // Reset for a new conversation.
        
        await _accessors.TopicState.SetAsync(turnContext, convo);
        await _accessors.ConversationState.SaveChangesAsync(turnContext);
    }
}
```   

# <a name="javascripttabjs"></a>[JavaScript](#tab/js)

### <a name="conversation-and-user-state"></a>對話和使用者狀態

您可以使用 [Echo Bot 搭配計數器範例](https://aka.ms/EchoBot-With-Counter-JS)，作為此操作說明的起點。 此範例已經使用 `ConversationState` 來儲存訊息計數。 我們必須新增 `TopicStates` 物件來追蹤我們的對話狀態，以及新增 `UserState` 來追蹤 `userProfile` 物件中的使用者資訊。 

在主要 Bot 的 `index.js` 檔案中，將 `UserState` 新增至必要清單：

**index.js**

```javascript
// Import required bot services. See https://aka.ms/bot-services to learn more about the different parts of a bot.
const { BotFrameworkAdapter, MemoryStorage, ConversationState, UserState } = require('botbuilder');
```

接下來，使用 `MemoryStorage` 作為儲存體提供者來建立 `UserState`，然後將其當作第二個引數傳遞至 `MainDialog` 類別。

**index.js**

```javascript
// Create conversation state with in-memory storage provider. 
const conversationState = new ConversationState(memoryStorage);
const userState = new UserState(memoryStorage);
// Create the main bot.
const bot = new EchBot(conversationState, userState);
```

在 `bot.js` 檔案中，更新建構函式以接受 `userState` 作為第二個引數。 然後從 `conversationState` 建立 `topicState` 屬性，並從 `userState` 建立 `userProfile` 屬性。

**bot.js**

```javascript
const TOPIC_STATE = 'topic';
const USER_PROFILE = 'user';

constructor (conversationState, userState) {
    // creates a new state accessor property.see https://aka.ms/about-bot-state-accessors to learn more about the bot state and state accessors 
    this.conversationState = conversationState;
    this.topicState = this.conversationState.createProperty(TOPIC_STATE);

    // User state
    this.userState = userState;
    this.userProfile = this.userState.createProperty(USER_PROFILE);
}
```

### <a name="use-conversation-and-user-state-properties"></a>使用對話和使用者狀態屬性

在 `MainDialog` 類別的 `onTurn` 處理常式中，修改程式碼以提示輸入使用者名稱，然後輸入電話號碼。 為了追蹤我們在對話中的何處，我們會使用 `topicState` 中定義的 `prompt` 屬性。 這個屬性已初始化為 "askName"。 一旦取得使用者名稱，我們會將其設定為 "askNumber "，並將 UserName 設定為使用者輸入的名稱。 收到電話號碼之後，您可傳送確認訊息並將提示設定為 `undefined`，因為您位於對話的結尾。

**dialogs/mainDialog/index.js**

```javascript
// see https://aka.ms/about-bot-activity-message to learn more about the message and other activity types
if (turnContext.activity.type === 'message') {
    // read from state and set default object if object does not exist in storage.
    let topicState = await this.topicState.get(turnContext, {
        //Define the topic state object
        prompt: "askName"
    });
    let userProfile = await this.userProfile.get(turnContext, {  
        // Define the user's profile object
        "userName": "",
        "telephoneNumber": ""
    });

    if(topicState.prompt == "askName"){
        await turnContext.sendActivity("What is your name?");

        // Set next prompt state
        topicState.prompt = "askNumber";

        // Update state
        await this.topicState.set(turnContext, topicState);
    }
    else if(topicState.prompt == "askNumber"){
        // Set the UserName that is defined in the UserProfile class
        userProfile.userName = turnContext.activity.text;

        // Use the user name to prompt the user for phone number
        await turnContext.sendActivity(`Hello, ${userProfile.userName}. What's your telephone number?`);

        // Set next prompt state
        topicState.prompt = "confirmation";

        // Update states
        await this.topicState.set(turnContext, topicState);
        await this.userProfile.set(turnContext, userProfile);
    }
    else if(topicState.prompt == "confirmation"){
        // Set the phone number
        userProfile.telephoneNumber = turnContext.activity.text;

        // Sent confirmation
        await turnContext.sendActivity(`Got it, ${userProfile.userName}. I'll call you later.`)

        // reset initial prompt state
        topicState.prompt = "askName"; // Reset for a new conversation

        // Update states
        await this.topicState.set(turnContext, topicState);
        await this.userProfile.set(turnContext, userProfile);
    }
    
    // Save state changes to storage
    await this.conversationState.saveChanges(turnContext);
    await this.userState.saveChanges(turnContext);
    
}
else {
    await turnContext.sendActivity(`[${turnContext.activity.type} event detected]`);
}
```

---

## <a name="start-your-bot"></a>啟動 Bot
在本機執行您的 Bot。

### <a name="start-the-emulator-and-connect-your-bot"></a>啟動模擬器並且連線至您的 Bot
接下來，請啟動模擬器，然後在模擬器中連線至您的 Bot：

1. 按一下模擬器 [歡迎] 索引標籤中的 [開啟 Bot] 連結。 
2. 選取位於您建立 Visual Studio 解決方案的目錄中的 .bot 檔案。

### <a name="interact-with-your-bot"></a>與您的 Bot 互動

將 "Hi" 訊息傳送給 Bot，Bot 會詢問您的名稱和電話號碼。 在您提供該資訊之後，Bot 會傳送確認訊息。 如果您繼續執行，則 Bot 會再次經歷相同的週期。
![模擬器執行中](../media/emulator-v4/emulator-running.png)

如果您決定自行管理狀態，請參閱[使用自己的提示來管理對話流程](bot-builder-primitive-prompts.md)。 另一種方法是使用瀑布圖對話方塊。 對話會追蹤對話方塊狀態，讓您不需要建立旗標來追蹤狀態。 如需詳細資訊，請參閱[使用對話來管理簡單對話方塊](bot-builder-dialog-manage-conversation-flow.md)。

## <a name="next-steps"></a>後續步驟
您已了解如何使用狀態來協助您在儲存體中讀取和寫入 Bot 資料，接下來讓我們來了解如何直接在儲存體中讀取和寫入。

> [!div class="nextstepaction"]
> [如何直接寫入到儲存體](bot-builder-howto-v4-storage.md)。
