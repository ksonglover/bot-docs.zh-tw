---
title: 使用基本提示管理對話流程 | Microsoft Docs
description: 了解如何在 Bot 建立器 SDK 中，使用基本提示來管理對話流程。
keywords: 對話流程, 提示
author: v-ducvo
ms.author: v-ducvo
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 7/20/2018
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 0b563197c111a37cf2f0f14fef183d52f38cca66
ms.sourcegitcommit: b78fe3d8dd604c4f7233740658a229e85b8535dd
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/24/2018
ms.locfileid: "49999158"
---
# <a name="prompt-users-for-input-using-your-own-prompts"></a>使用您自己的提示來提示使用者輸入

[!INCLUDE [pre-release-label](../includes/pre-release-label.md)]

Bot 和使用者之間的對話，通常牽涉到要求 (提示) 使用者輸入資訊、剖析使用者的回應，然後根據該資訊行動。 關於[使用對話方塊程式庫提示使用者](bot-builder-prompts.md)的主題詳細說明如何使用**對話方塊**程式庫提示使用者輸入。 除此之外，**對話方塊**程式庫會負責追蹤目前的對話及目前提出的問題。 其也提供方法來要求及驗證不同類型的資訊，例如文字、數字、日期和時間等等。

在某些情況下，內建**對話方塊**可能不適合您的 Bot；**對話方塊**對於簡單的 Bot 可能加入太多額外負荷、太死板，或是無法達到您要 Bot 執行的內容。 在這些情況下，您可以跳過程式庫，然後實作自己的提示邏輯。 本主題會示範如何設定基本的 *Echo bot*，讓您可以使用自己的提示來管理對話。

## <a name="track-prompt-states"></a>追蹤提示狀態

在提示導向的對話中，您需要追蹤目前在對話中的位置，以及目前提出的是何種問題。 在程式碼內，這會轉譯為管理幾個旗標。

例如，您可以建立幾個想要追蹤的屬性。

這些狀態會追蹤我們目前位於哪個主題和哪個提示。 若要確保這些旗標部署至雲端時能正常運作，可將其儲存在[對話狀態](bot-builder-howto-v4-state.md)中。 

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

我們會建立兩個類別來追蹤狀態。 **TopicState** 用來追蹤對話提示的進度，而 **UserProfile** 用來追蹤使用者提供的資訊。 我們會在稍後的步驟中將此資訊儲存在我們的 Bot [狀態](bot-builder-howto-v4-state.md)。

```csharp
/// <summary>
/// Contains conversation state information about the conversation in progress.
/// </summary>
public class TopicState
{
    /// <summary>
    /// Identifies the current "topic" of conversation.
    /// </summary>
    public string Topic { get; set; }

    /// <summary>
    /// Indicates whether we asked the user a question last turn, and
    /// if so, what it was.
    /// </summary>
    public string Prompt { get; set; }
}
```

```csharp
/// <summary>
/// Contains user state information for the user's profile.
/// </summary>
public class UserProfile
{
    public string UserName { get; set; }

    public int? Age { get; set; }

    public string WorkPlace { get; set; }
}
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

**app.js**

```javascript
const storage = new MemoryStorage(); // Volatile memory
const conversationState = new ConversationState(storage);
const userState = new UserState(storage);
const dialogState = conversationState.createProperty('dialogState');
const userProfile = userState.createProperty('userProfile');
```

將此程式碼置於主要的 Bot 邏輯內。

```javascript
// Pull the state of the dialog out of the conversation state manager.
const convo = await dialogState.get(context, {
    prompt: undefined,
    topic: 'profile'
});

// Pull the user profile out of the user state manager.
const userProfile = await userProfile.get(context, {  // Define the user's profile object
        "userName": undefined,
        "age": undefined,
        "workPlace": undefined
    }
);
```

---

## <a name="manage-a-topic-of-conversation"></a>管理對話的主題

在循序對話中，如收集使用者資訊的對話，您需要知道使用者輸入對話的時間和對話中使用者所在位置。 您可以設定並檢查上述定義的提示旗標，再據以行動，以便在主要的 Bot 回合處理常式中追蹤這些資訊。 下列範例會示範如何使用這些旗標，透過對話收集使用者設定檔資訊。

Bot 程式碼會顯示在這裡，並於下一節進行討論。

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

對於 ASP.NET Core，我們必須先設定 Bot 和相依性插入。

將 **EchoWithCounterBot.cs** 檔案重新命名為 **PrimitivePromptsBot.cs**，也要更新類別名稱。 這個類別保存我們的 Bot 邏輯，我們很快就會回頭更新此邏輯。

將 **EchoBotAccessors.cs** 檔案重新命名為 **BotAccessors.cs**，也要更新類別名稱。 這個類別會保存 Bot 的狀態管理物件和狀態屬性存取子。 將定義更新如下。

```csharp
using Microsoft.Bot.Builder;
using System;

/// <summary>
/// Contains the state and state property accessors for the primitive prompts bot.
/// </summary>
public class BotAccessors
{
    public const string TopicStateName = "PrimitivePrompts.TopicStateAccessor";

    public const string UserProfileName = "PrimitivePrompts.UserProfileAccessor";

    public ConversationState ConversationState { get; }

    public UserState UserState { get; }

    public IStatePropertyAccessor<TopicState> TopicStateAccessor { get; set; }

    public IStatePropertyAccessor<UserProfile> UserProfileAccessor { get; set; }

    public BotAccessors(ConversationState conversationState, UserState userState)
    {
        if (conversationState is null)
        {
            throw new ArgumentNullException(nameof(conversationState));
        }

        if (userState is null)
        {
            throw new ArgumentNullException(nameof(userState));
        }

        this.ConversationState = conversationState;
        this.UserState = userState;
    }
}
```

更新 **Startup.cs** 檔案的 `ConfigureServices` 方法，從您設定 `IStorage` 物件的地方著手。

```csharp
public void ConfigureServices(IServiceCollection services)
{
    services.AddBot<PrimitivePromptsBot>(options =>
    {
        // ...

        // The Memory Storage used here is for local bot debugging only. When the bot
        // is restarted, everything stored in memory will be gone.
        IStorage dataStore = new MemoryStorage();

        var conversationState = new ConversationState(dataStore);
        options.State.Add(conversationState);

        var userState = new UserState(dataStore);
        options.State.Add(userState);
    });

    // Create and register state accessors.
    // Accessors created here are passed into the IBot-derived class on every turn.
    services.AddSingleton<BotAccessors>(sp =>
    {
        var options = sp.GetRequiredService<IOptions<BotFrameworkOptions>>().Value;
        var conversationState = options.State.OfType<ConversationState>().FirstOrDefault();
        var userState = options.State.OfType<UserState>().FirstOrDefault();

        // Create the custom state accessor.
        // State accessors enable other components to read and write individual properties of state.
        var accessors = new BotAccessors(conversationState, userState)
        {
            TopicStateAccessor = conversationState.CreateProperty<TopicState>(BotAccessors.TopicStateName),
            UserProfileAccessor = userState.CreateProperty<UserProfile>(BotAccessors.UserProfileName),
        };

        return accessors;
    });
}
```

最後，在 **PrimitivePromptsBot.cs** 檔案中更新 Bot 邏輯。

```csharp
using System;
using System.Collections.Generic;
using System.Linq;
using System.Threading;
using System.Threading.Tasks;
using Microsoft.Bot.Builder;
using Microsoft.Bot.Schema;

public class PrimitivePromptsBot : IBot
{
    public const string ProfileTopic = "profile";

    /// <summary>
    /// Describes a field in the user profile.
    /// </summary>
    private class UserFieldInfo
    {
        /// <summary>
        /// The ID to use for this field.
        /// </summary>
        public string Key { get; set; }

        /// <summary>
        /// The prompt to use to ask for a value for this field.
        /// </summary>
        public string Prompt { get; set; }

        /// <summary>
        /// Gets the value of the corresponding field.
        /// </summary>
        public Func<UserProfile, string> GetValue { get; set; }

        /// <summary>
        /// Sets the value of the corresponding field.
        /// </summary>
        public Action<UserProfile, string> SetValue { get; set; }
    }

    /// <summary>
    /// The prompts for the user profile, indexed by field name.
    /// </summary>
    private static List<UserFieldInfo> UserFields { get; } = new List<UserFieldInfo>
    {
        new UserFieldInfo {
            Key = nameof(UserProfile.UserName),
            Prompt = "What is your name?",
            GetValue = (profile) => profile.UserName,
            SetValue = (profile, value) => profile.UserName = value,
        },
        new UserFieldInfo {
            Key = nameof(UserProfile.Age),
            Prompt = "How old are you?",
            GetValue = (profile) => profile.Age.HasValue? profile.Age.Value.ToString() : null,
            SetValue = (profile, value) =>
            {
                if (int.TryParse(value, out int age))
                {
                    profile.Age = age;
                }
            },
        },
        new UserFieldInfo {
            Key = nameof(UserProfile.WorkPlace),
            Prompt = "Where do you work?",
            GetValue = (profile) => profile.WorkPlace,
            SetValue = (profile, value) => profile.WorkPlace = value,
        },
    };

    /// <summary>
    /// The state and state accessors for the bot.
    /// </summary>
    private BotAccessors Accessors { get; }

    public PrimitivePromptsBot(BotAccessors accessors)
    {
        Accessors = accessors ?? throw new ArgumentNullException(nameof(accessors));
    }

    public async Task OnTurnAsync(ITurnContext turnContext, CancellationToken cancellationToken = default(CancellationToken))
    {
        if (turnContext.Activity.Type is ActivityTypes.Message)
        {
            // Use the state property accessors to get the topic state and user profile.
            TopicState topicState = await Accessors.TopicStateAccessor.GetAsync(
                turnContext,
                () => new TopicState { Topic = ProfileTopic, Prompt = null },
                cancellationToken);
            UserProfile userProfile = await Accessors.UserProfileAccessor.GetAsync(
                turnContext,
                () => new UserProfile(),
                cancellationToken);

            // Check whether we need more information.
            if (topicState.Topic is ProfileTopic)
            {
                // If we're expecting input, record it in the user's profile.
                if (topicState.Prompt != null)
                {
                    UserFieldInfo field = UserFields.First(f => f.Key.Equals(topicState.Prompt));
                    field.SetValue(userProfile, turnContext.Activity.Text.Trim());
                }

                // Determine which fields are not yet set.
                List<UserFieldInfo> emptyFields = UserFields.Where(f => f.GetValue(userProfile) is null).ToList();

                if (emptyFields.Any())
                {
                    // If all the fields are empty, send a welcome message.
                    if (emptyFields.Count == UserFields.Count)
                    {
                        await turnContext.SendActivityAsync("Welcome new user, please fill out your profile information.");
                    }

                    // We have at least one empty field. Prompt for the next empty field,
                    // and update the prompt flag to indicate which prompt we just sent,
                    // so that the response can be captured at the beginning of the next turn.
                    UserFieldInfo field = emptyFields.First();
                    await turnContext.SendActivityAsync(field.Prompt);
                    topicState.Prompt = field.Key;
                }
                else
                {
                    // Our user profile is complete!
                    await turnContext.SendActivityAsync($"Thank you, {userProfile.UserName}. Your profile is complete.");
                    topicState.Prompt = null;
                    topicState.Topic = null;
                }
            }
            else if (turnContext.Activity.Text.Trim().Equals("hi", StringComparison.InvariantCultureIgnoreCase))
            {
                await turnContext.SendActivityAsync($"Hi. {userProfile.UserName}.");
            }
            else
            {
                await turnContext.SendActivityAsync("Hi. I'm the Contoso cafe bot.");
            }

            // Use the state property accessors to update the topic state and user profile.
            await Accessors.TopicStateAccessor.SetAsync(turnContext, topicState, cancellationToken);
            await Accessors.UserProfileAccessor.SetAsync(turnContext, userProfile, cancellationToken);

            // Save any state changes to storage.
            await Accessors.ConversationState.SaveChangesAsync(turnContext, false, cancellationToken);
            await Accessors.UserState.SaveChangesAsync(turnContext, false, cancellationToken);
        }
    }
}
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

**app.js**

```javascript

server.post('/api/messages', (req, res) => {
    adapter.processActivity(req, res, async (context) => {
        const isMessage = (context.activity.type === ActivityTypes.Message);

        // Set up a list of fields that we need to collect from the user.
        const fields = {
            userName: "What is your name?",
            age: "How old are you?",
            workPlace: "Where do you work?"
        }

        // Pull the state of the dialog out of the conversation state manager.
        const convo = await dialogState.get(context, {
            topic: 'profile',
            prompt: undefined
        });

        // Pull the user profile out of the user state manager.
        const userProfile = await userProfile.get(context, {  // Define the user's profile object
                "userName": undefined,
                "age": undefined,
                "workPlace": undefined
            }
        );

        if (isMessage) {

            if (convo.topic === 'profile') {
                // If a prompt flag is set in the conversation state, use it to capture the incoming value
                // into the appropriate field of the user profile.
                if (convo.prompt) {
                    userProfile[convo.prompt] = context.activity.text;
                }

                 // Determine which fields are not yet set.
                 const empty_fields = [];
                 Object.keys(fields).forEach(function(key) {
                    if (!userProfile[key]) {
                        empty_fields.push({
                           key: key,
                           prompt: fields[key]
                        });
                     }
                 });

                 if (empty_fields.length) {

                    // If all the fields are empty, send a welcome message.
                    if (empty_fields.length == Object.keys(fields).length) {
                        await context.sendActivity('Welcome new user, please fill out your profile information.');
                    }

                    // We have at least one empty field. Prompt for the next empty field.
                    await context.sendActivity(empty_fields[0].prompt);

                    // update the flag to indicate which prompt we just sent
                    // so that the response can be captured at the beginning of the next turn.
                    convo.prompt = empty_fields[0].key;

                 } else {
                    // Our user profile is complete!
                    await context.sendActivity('Thank you. Your profile is complete.');
                    convo.prompt = null;
                    convo.topic = null;

                 }
            } else if (context.activity.text && context.activity.text.match(/hi/ig)) {
                // Check to see if the user said "hi" and respond with a greeting
                await context.sendActivity(`Hi ${ userProfile.userName }.`);
            } else {
                // Default message
                await context.sendActivity("Hi. I'm the Contoso bot.");
            }
        }

        // End the turn by writing state changes back to storage
        await conversationState.saveChanges(context);
        await userState.saveChanges(context);
    });
});

```

---

上面的範例程式碼會將「主題」 旗標初始化為 `profile`，以便開始設定檔集合交談。 Bot 會將 prompt 旗標更新為代表目前所提問題的值，以便繼續進行交談。 此旗標設為適當的值後，Bot 就會知道要對使用者所傳來的下一則訊息執行什麼動作，並依此處理。

最後，旗標會在 Bot 要求資訊完成時重設。 否則，Bot 會落入迴圈中，永遠不會從最後一個問題前進。

您可以擴充此模式來管理更複雜的對話流程，只要定義其他旗標或根據使用者輸入分支對話即可。

## <a name="manage-multiple-topics-of-conversations"></a>管理多個對話主題

能夠處理一個對話主題之後，您可以擴充 Bot 的邏輯來處理多個對話主題。 檢查其他條件，然後採取適當的路徑，即可處理多個主題。

您可以擴充上述範例以允許其他功能和對話主題，例如訂位或下單。 視需要將其他旗標新增至主題狀態，以便追蹤對話。

您也會發現，將程式碼分割成獨立的函式或方法很有幫助，可更輕鬆地追蹤對話流程。 常見的模式是讓 Bot 評估訊息和狀態，然後將控制權委派給實作功能詳細資料的函式。

若要協助使用者深入瀏覽對話的多個主題，請考慮提供主功能表。 例如，使用[建議的動作](bot-builder-howto-add-suggested-actions.md)，您可以讓使用者選擇可參與哪個主題，而不是猜測 Bot 可以執行哪些動作。

## <a name="validate-user-input"></a>驗證使用者輸入

**對話方塊**程式庫提供內建方式來驗證使用者的輸入，但我們也可以使用自己的提示來達成。 比方說，如果詢問使用者的年齡，我們會想得到數字，而非 "Bob" 之類的回應。

剖析數字或日期和時間是很複雜的工作，已超出本主題的範圍。 幸運的是，我們可以利用程式庫。 若要剖析此資訊，可使用 [Microsoft 的文字辨識器](https://github.com/Microsoft/Recognizers-Text)程式庫。 此套件可透過 NuGet 取得，或從存放庫下載。 (其也包含在**對話方塊**程式庫中，即使我們並未在此使用該程式庫，這點值得注意。)

此程式庫特別適合用於剖析複雜輸入，像是日期、時間或電話號碼。 在此範例中，我們會驗證預訂晚餐派對大小的號碼，但相同的概念可延伸至更複雜的驗證作業。

在以下範例中，我們只示範如何使用 `RecognizeNumber`。 可在[存放庫文件](https://github.com/Microsoft/Recognizers-Text/blob/master/README.md)中，找到如何使用程式庫中其他辨識器方法的詳細資訊。

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

若要使用 **Microsoft.Recognizers.Text.Number** 程式庫，請包含 NuGet 套件並將其新增至 using 陳述式。

```csharp
using Microsoft.Recognizers.Text.Number;
using Microsoft.Recognizers.Text;
using System.Linq;
```

然後，定義可實際執行驗證的函式。

```csharp
private async Task<bool> ValidatePartySize(ITurnContext context, string value)
{
    try
    {
        // Recognize the input as a number. This works for responses such as
        // "twelve" as well as "12"
        var result = NumberRecognizer.RecognizeNumber(value, Culture.English);

        // Attempt to convert the Recognizer result to an integer
        int.TryParse(result.First().Text, out int partySize);

        if (partySize < 6)
        {
            throw new Exception("Party size too small.");
        }
        else if (partySize > 20)
        {
            throw new Exception("Party size too big.");
        }

        // If we got through this, the number is valid
        return true;
    }
    catch (Exception)
    {
        await context.SendActivityAsync("Error with your party size. < br /> Please specify a number between 6 - 20.");
        return false;
    }
}
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

若要使用 **Recognizers** 程式庫，請在 **app.js** 中要求：

```javascript
// Required packages for this bot
var Recognizers = require('@microsoft/recognizers-text-suite');
```

然後，定義可實際執行驗證的函式。

```javascript
// Support party size between 6 and 20 only
async function validatePartySize(context, input){
    try {
        // Recognize the input as a number. This works for responses such as
        // "twelve" as well as "12"
        var result = Recognizers.recognizeNumber(input, Recognizers.Culture.English);
        var value = parseInt(results[0].resolution.value);

        if (value < 6) {
            throw new Error(`Party size too small.`);
        } else if(value > 20){
            throw new Error(`Party size too big.`);
        }
        return true; // Return the valid value
    } catch (err){
        await context.sendActivity(`${err.message} <br/>Please specify a number between 6 - 20.`);
        return false;
    }
}
```

---

在處理使用者對於提示的回應時，請先呼叫驗證函式，再繼續進行下一個提示。 如果驗證失敗，請再次提出問題。

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

```csharp
if (topicState.Prompt == "partySize")
{
    if (await ValidatePartySize(turnContext, turnContext.Activity.Text))
    {
        // Save user's response in our state, ReservationInfo, which
        // is a new class we've added to our state
        // UserFieldInfo partySize;
        partySize.SetValue(userProfile, turnContext.Activity.Text);

        // Ask next question.
        topicState.Prompt = "reserveName";
        await turnContext.SendActivityAsync("Who's name will this be under?");
    }
    else
    {
        // Ask again.
        await turnContext.SendActivityAsync("How many people are in your party?");
    }
}
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

**app.js**

```javascript
// ...
if (convo.prompt == "partySize") {
    if (await validatePartySize(context, context.activity.text)) {
        // Save user's response
        reservationInfo.partySize = context.activity.text;

        // Ask next question
        convo.prompt = "reserveName";
        await context.sendActivity("Who's name will this be under?");
    } else {
        // Ask again
        await context.sendActivity("How many people are in your party?");
    }
}
// ...
```

---

## <a name="next-step"></a>後續步驟

既然您已了解如何管理提示狀態，讓我們看看如何運用**對話方塊**程式庫來提示使用者輸入。

> [!div class="nextstepaction"]
> [使用對話方塊來提示使用者輸入](bot-builder-prompts.md)
