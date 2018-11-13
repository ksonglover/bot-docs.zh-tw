---
redirect_url: /bot-framework/bot-builder-howto-v4-state
ms.openlocfilehash: 081c7c1f3e354d4352baea029840c8175152116e
ms.sourcegitcommit: a54a70106b9fdf278fd7270b25dd51c9bd454ab1
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 11/08/2018
ms.locfileid: "51273115"
---
<a name="--"></a><!--
---
標題：建立自己的提示以收集使用者輸入 | Microsoft Docs 描述：了解如何在 Bot Builder SDK 中，使用基本提示來管理對話流程。
關鍵字：對話流程, 提示作者: v-ducvo ms.author: v-ducvo manager: kamrani ms.topic: article ms.service: bot-service ms.subservice: sdk ms.date: 10/31/2018 monikerRange: 'azure-bot-service-4.0'
---

# <a name="create-your-own-prompts-to-gather-user-input"></a>建立您自己的提示，以收集使用者輸入

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

在 **index.js** 中，更新必要陳述式以包含 `UserState`。

```javascript
const { BotFrameworkAdapter, MemoryStorage, ConversationState, UserState } = require('botbuilder');
```

接著建立使用者狀態管理物件，並且在建立 Bot 時將其傳入。

```javascript
// Create conversation and user state with in-memory storage provider.
const conversationState = new ConversationState(memoryStorage);
const userState = new UserState(memoryStorage);

// Create the bot.
const myBot = new MyBot(conversationState, userState);
```

在 **bot.js** 中，我們會針對將用來管理 Bot [狀態](bot-builder-howto-v4-state.md)的狀態屬性存取子定義識別項。 也會定義提示，以用於我們要向使用者收集的資訊。

在 `MyBot` 類別外部新增此程式碼。

```javascript
// Define identifiers for our state property accessors.
const TOPIC_STATE_PROPERTY = 'topicStateProperty';
const USER_PROFILE_PROPERTY = 'userProfileProperty';

// Define the prompts to use to ask for user profile information.
const fields = {
    userName: "What is your name?",
    age: "How old are you?",
    workPlace: "Where do you work?"
}
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

在 **bot.js** 中，更新 `MyBot` 類別定義。

我們會在 Bot 的建構函式中設定狀態屬性存取子：`topicStateAccessor` 和 `userProfileAccessor`。 主題狀態可追蹤對話的主題，而使用者設定檔可追蹤我們為使用者所收集的資訊。

```javascript
constructor(conversationState, userState) {
    // Create state property accessors.
    this.topicStateAccessor = conversationState.createProperty(TOPIC_STATE_PROPERTY);
    this.userProfileAccessor = userState.createProperty(USER_PROFILE_PROPERTY);

    // Track the conversation and user state management objects.
    this.conversationState = conversationState;
    this.userState = userState;
}
```

然後將回合處理常式更新為使用 Bot 狀態來控制對話流程，並儲存所收集的使用者資訊。

```javascript
async onTurn(turnContext) {
    // Handle only message activities from the user.
    if (turnContext.activity.type === ActivityTypes.Message) {
        // Get state properties using their accessors, providing default values.
        const topicState = await this.topicStateAccessor.get(turnContext, {
            prompt: undefined,
            topic: 'profile'
        });
        const userProfile = await this.userProfileAccessor.get(turnContext, {
            "userName": undefined,
            "age": undefined,
            "workPlace": undefined
        });

        if (topicState.topic === 'profile') {
            // If a prompt flag is set in the conversation state, use it to capture the incoming value
            // into the appropriate field of the user profile.
            if (topicState.prompt) {
                userProfile[topicState.prompt] = turnContext.activity.text;
            }

            // Determine which fields are not yet set.
            const empty_fields = [];
            Object.keys(fields).forEach(function (key) {
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
                    await turnContext.sendActivity('Welcome new user, please fill out your profile information.');
                }

                // We have at least one empty field. Prompt for the next empty field.
                await turnContext.sendActivity(empty_fields[0].prompt);

                // update the flag to indicate which prompt we just sent
                // so that the response can be captured at the beginning of the next turn.
                topicState.prompt = empty_fields[0].key;

            } else {
                // Our user profile is complete!
                await turnContext.sendActivity('Thank you. Your profile is complete.');
                topicState.prompt = null;
                topicState.topic = null;

            }
        } else if (turnContext.activity.text && turnContext.activity.text.match(/hi/ig)) {
            // Check to see if the user said "hi" and respond with a greeting
            await turnContext.sendActivity(`Hi ${userProfile.userName}.`);
        } else {
            // Default message
            await turnContext.sendActivity("Hi. I'm the Contoso bot.");
        }

        // Save state changes
        await this.conversationState.saveChanges(turnContext);
        await this.userState.saveChanges(turnContext);
    }
}
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

剖析數字或日期和時間是很複雜的工作，已超出本主題的範圍。 幸運的是，我們可以利用程式庫。 若要剖析此資訊，可使用 [Microsoft 的文字辨識器](https://github.com/Microsoft/Recognizers-Text)程式庫。 此套件可透過 NuGet 和 npm 取得。 您也可以直接從存放庫下載該套件。 (其也包含在**對話方塊**程式庫中，即使我們並未在此使用該程式庫，這點值得注意。)

此程式庫特別適合用於剖析複雜輸入，像是日期、時間或電話號碼。 在此範例中，我們會驗證預訂晚餐派對大小的號碼，但相同的概念可延伸至更複雜的驗證作業。

在以下範例中，我們只示範如何使用 `RecognizeNumber`。 可在[存放庫文件](https://github.com/Microsoft/Recognizers-Text/blob/master/README.md)中，找到如何使用程式庫中其他辨識器方法的詳細資訊。

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

若要使用 **Microsoft.Recognizers.Text.Number** 程式庫，請包含 NuGet 套件，並將這些 using 陳述式新增至 Bot 檔案。

```csharp
using System.Linq;
using Microsoft.Recognizers.Text;
using Microsoft.Recognizers.Text.Number;
```

我們有許多方式可處理驗證。 在此，我們會更新協助程式類別以包含驗證。

在 Bot 的內部 `UserFieldInfo` 類別中新增下列成員。

```csharp
/// <summary>Delegate for validating input.</summary>
/// <param name="turnContext">The current turn context. turnContext.Activity.Text contains the input to validate.</param>
/// <returns><code>true</code> if the input is valid; otherwise, <code>false</code>.</returns>
public delegate Task<bool> ValidatorDelegate(
    ITurnContext turnContext,
    CancellationToken cancellationToken = default(CancellationToken));

/// <summary>By default, evaluate all input as valid.</summary>
private static readonly ValidatorDelegate NoValidator =
    async (ITurnContext turnContext, CancellationToken cancellationToken) => true;

/// <summary>Gets or sets the validation function to use.</summary>
public ValidatorDelegate ValidateInput { get; set; } = NoValidator;
```

接著，更新 Bot `UserFields` 中的 age 項目，以定義要使用的驗證。
因為我們會在設定 age 值之前驗證輸入，所以可簡化 `SetValue` 函式並利用文字辨識器程式庫。

```csharp
private static List<UserFieldInfo> UserFields { get; } = new List<UserFieldInfo>
{
    // ...
    new UserFieldInfo {
        Key = nameof(UserProfile.Age),
        Prompt = "How old are you?",
        GetValue = (profile) => profile.Age.HasValue? profile.Age.Value.ToString() : null,
        SetValue = (profile, value) =>
        {
            // As long as the input validates, this should work.
            List<ModelResult> result = NumberRecognizer.RecognizeNumber(value, Culture.English);
            profile.Age = int.Parse(result.First().Text);
        },
        ValidateInput = async (turnContext, cancellationToken) =>
        {
            try
            {
                // Recognize the input as a number. This works for responses such as
                // "twelve" as well as "12".
                List<ModelResult> result = NumberRecognizer.RecognizeNumber(
                    turnContext.Activity.Text, Culture.English);

                // Attempt to convert the Recognizer result to an integer
                int.TryParse(result.First().Text, out int age);

                if (age < 18)
                {
                    await turnContext.SendActivityAsync(
                        "You must be 18 or older.",
                        cancellationToken: cancellationToken);
                    return false;
                }
                else if (age > 120)
                {
                    await turnContext.SendActivityAsync(
                        "You must be 120 or younger.",
                        cancellationToken: cancellationToken);
                    return false;
                }
            }
            catch
            {
                await turnContext.SendActivityAsync(
                    "I couldn't understand your input. Please enter your age in years.",
                    cancellationToken: cancellationToken);
                return false;
            }

            // If we got through this, the number is valid.
            return true;
        },
    },
    // ...
};
```

最後，我們會更新回合處理常式，在將值儲存至屬性之前，驗證所有輸入。
驗證會預設為 NoValidator 函式，並會接受任何輸入。 因此，age 提示的行為是唯一會變更的項目。 如果輸入驗證失敗，我們便不會設定此欄位，而 Bot 會提示在下一回合再次輸入此欄位。

在此，我們只會考慮需要更新的回合處理常式部分。

```csharp
public async Task OnTurnAsync(ITurnContext turnContext, CancellationToken cancellationToken = default(CancellationToken))
{
    if (turnContext.Activity.Type is ActivityTypes.Message)
    {
        // ...
        // Check whether we need more information.
        if (topicState.Topic is ProfileTopic)
        {
            // If we're expecting input, record it in the user's profile.
            if (topicState.Prompt != null)
            {
                UserFieldInfo field = UserFields.First(f => f.Key.Equals(topicState.Prompt));
                if (await field.ValidateInput(turnContext, cancellationToken))
                {
                    field.SetValue(userProfile, turnContext.Activity.Text.Trim());
                }
            }

            // ...
        }
        //...
    }
}
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

若要使用 **Recognizers** 程式庫，請新增套件並且需要 Bot 程式碼中 (在 **bot.js** 中) 使用：

```bash
npm i @microsoft/recognizers-text-suite --save
```

```javascript
// Required packages for this bot.
const Recognizers = require('@microsoft/recognizers-text-suite');
```

接著，更新 `fields` 中繼資料，以包含文字辨識和驗證程式碼：

```javascript
// Define the prompts to use to ask for user profile information.
const fields = {
    userName: { prompt: "What is your name?" },
    age: {
        prompt: "How old are you?",
        recognize: (turnContext) => {
            var result = Recognizers.recognizeNumber(
                turnContext.activity.text, Recognizers.Culture.English);
            return parseInt(result[0].resolution.value);
        },
        validate: async (turnContext) => {
            try {
                // Recognize the input as a number. This works for responses such as
                // "twelve" as well as "12".
                var result = Recognizers.recognizeNumber(
                    turnContext.activity.text, Recognizers.Culture.English);
                var age = parseInt(result[0].resolution.value);
                if (age < 18) {
                    await turnContext.sendActivity("You must be 18 or older.");
                    return false;
                }
                if (age > 120 ) {
                    await turnContext.sendActivity("You must be 120 or younger.");
                    return false;
                }
            } catch (_) {
                await turnContext.sendActivity(
                    "I couldn't understand your input. Please enter your age in years.");
                return false;
            }
            return true;
        }
    },
    workPlace: { prompt: "Where do you work?" }
}
```

在 Bot 的回合處理常式中，更新下列區塊，我們會在其中記錄使用者的輸入，以及提示使用者。 我們需要更新下列各節，以說明欄位中繼資料的變更。

```javascript
async onTurn(turnContext) {
    // Handle only message activities from the user.
    if (turnContext.activity.type === ActivityTypes.Message) {
        // ...

        if (topicState.topic === 'profile') {
            // If a prompt flag is set in the conversation state, use it to capture the incoming value
            // into the appropriate field of the user profile.
            if (topicState.prompt) {
                const field = fields[topicState.prompt];
                // If the prompt has validation, check whether the input validates.
                if (!field.validate || await field.validate(turnContext)) {
                    // Set the field, using a recognizer if one is defined.
                    userProfile[topicState.prompt] = (field.recognize)
                        ? field.recognize(turnContext)
                        : turnContext.activity.text;
                }
            }

            // ...

            if (empty_fields.length) {

                // ...

                // We have at least one empty field. Prompt for the next empty field.
                await turnContext.sendActivity(empty_fields[0].prompt.prompt);

                // ...

            } // ...
        } // ...

        // Save state changes
        await this.conversationState.saveChanges(turnContext);
        await this.userState.saveChanges(turnContext);
    }
}
```

---

## <a name="next-step"></a>後續步驟

既然您已了解如何管理提示狀態，讓我們看看如何運用**對話方塊**程式庫來提示使用者輸入。

> [!div class="nextstepaction"]
> [使用對話方塊來提示使用者輸入](bot-builder-prompts.md)

-->
