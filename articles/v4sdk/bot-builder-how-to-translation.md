---
title: 翻譯使用者輸入，讓您的 Bot 支援多國語言 | Microsoft Docs
description: 如何自動將使用者輸入翻譯成 Bot 的原生語言和翻譯回使用者的語言。
author: DeniseMak
ms.author: v-demak
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 04/06/2018
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: a6bec55010124892376904070cc9ed88c659efbd
ms.sourcegitcommit: b78fe3d8dd604c4f7233740658a229e85b8535dd
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/24/2018
ms.locfileid: "49999855"
---
# <a name="translate-from-the-users-language-to-make-your-bot-multilingual"></a>從使用者的語言翻譯，讓您的 Bot 會使用多國語言

[!INCLUDE [pre-release-label](../includes/pre-release-label.md)]

您的 Bot 可以使用 [Microsoft Translator](https://www.microsoft.com/en-us/translator/) 自動將訊息翻譯成 Bot 了解的語言，以及選擇性將 Bot 的回覆翻譯回使用者的語言。
<!-- 
- [Get a Text Services key](#get-a-text-services-key)
- [Installing Packages](#installing-packages)
- [Combine translation with LUIS or QnA Maker](#using-luis)
- [Combine translation with QnA Maker](#using-qna-maker)
-->

## <a name="get-a-text-services-key"></a>取得文字服務金鑰

首先，使用 Microsoft Translator 服務需要金鑰。 您可以在 Azure 入口網站中取得[免費試用版金鑰](https://www.microsoft.com/en-us/translator/trial.aspx#get-started)。

## <a name="installing-packages"></a>安裝套件

請確定您有將翻譯新增到 Bot 所需的套件。

# <a name="ctabcsrefs"></a>[C#](#tab/csrefs)

對以下 NuGet 套件的發行前版本[加入參考](https://docs.microsoft.com/en-us/nuget/tools/package-manager-ui) \(機器翻譯\)：

* `Microsoft.Bot.Builder.Integration.AspNet.Core`
* `Microsoft.Bot.Builder.Ai` (翻譯的必要項目)

如果您打算要結合翻譯與 Language Understanding (LUIS)，請也加入對下列項目的參考：

* `Microsoft.Bot.Builder.Luis` (LUIS 的必要項目)

# <a name="javascripttabjsrefs"></a>[JavaScript](#tab/jsrefs)

在這些服務中，任一個服務都可以使用 botbuilder-ai 套件新增到您的 Bot。 您可以透過 npm 將此套件新增到您的專案：
* `npm install --save botbuilder@preview`
* `npm install --save botbuilder-ai@preview`

---

## <a name="configure-translation"></a>設定翻譯

藉由直接將翻譯工具新增到 Bot 的中介軟體堆疊，來將 Bot 設定成針對收到的每個使用者訊息呼叫您的翻譯工具。 中介軟體會使用翻譯結果來修改使用內容物件的使用者訊息。


# <a name="ctabcssetuptranslate"></a>[C#](#tab/cssetuptranslate)

開始使用 SDK 中的 EchoBot 範例，並更新 `Startup.cs` 檔案中的 `ConfigureServices` 方法，以將 `TranslationMiddleware` 新增到 Bot。 這會設定您的 Bot 翻譯從使用者收到的每個訊息。 <!--, by simply adding it to your bot's middleware set. The middleware stores the translation results on the context object. -->

**Startup.cs**
```csharp
using System;
using System.Collections.Generic;
using Microsoft.AspNetCore.Builder;
using Microsoft.AspNetCore.Hosting;
using Microsoft.Bot.Builder.Ai;
using Microsoft.Bot.Builder.BotFramework;
using Microsoft.Bot.Builder.Core.Extensions;
using Microsoft.Bot.Builder.Integration.AspNet.Core;
using Microsoft.Extensions.Configuration;
using Microsoft.Extensions.DependencyInjection;
using Microsoft.Extensions.Logging;

// This method gets called by the runtime. Use this method to add services to the container.
public void ConfigureServices(IServiceCollection services)
{
    services.AddBot<TranslatorBot>(options =>
    {
        options.CredentialProvider = new ConfigurationCredentialProvider(Configuration);
        var middleware = options.Middleware;
        // Add translation middleware
        // The first parameter is a list of languages the bot recognizes
        middleware.Add(new TranslationMiddleware(new string[] { "en" }, "<YOUR MICROSOFT TRANSLATOR API KEY>", false));

    });
}


```

> [!TIP] 
> BotBuilder SDK 會自動根據使用者提交訊息，偵測其所使用的語言。 若要覆寫此功能，您可以提供其他回呼參數，以新增自己用來偵測及變更使用者語言的邏輯。  



看一下 `EchoBot.cs` 中的程式碼，它會送出 "You sent" 後面接著使用者說的話：

```cs
using Microsoft.Bot.Builder;
using Microsoft.Bot.Schema;
using System.Threading.Tasks;

namespace Microsoft.Bot.Samples
{
    public class EchoBot : IBot
    {
        public EchoBot() { }

        public async Task OnTurn(ITurnContext context)
        {
            switch (context.Activity.Type)
            {
                case ActivityTypes.Message:
                // echo back the user's input.
                    await context.SendActivity($"You sent '{context.Activity.Text}'");


                case ActivityTypes.ConversationUpdate:
                    foreach (var newMember in context.Activity.MembersAdded)
                    {
                        if (newMember.Id != context.Activity.Recipient.Id)
                        {
                            await context.SendActivity("Hello and welcome to the echo bot.");
                        }
                    }
                    break;
            }
        }
        
    }
}
```

當您新增翻譯中介軟體時，選擇性參數會指定是否將回覆翻譯回使用者的語言。 在 `Startup.cs` 中，我們已指定 `false`，亦即只將使用者訊息翻譯成 Bot 的語言。

```cs
        // The first parameter is a list of languages the bot recognizes
        // The second parameter is your API key
        // The third parameter specifies whether to translate the bot's replies back to the user's language
        middleware.Add(new TranslationMiddleware(new string[] { "en" }, "<YOUR MICROSOFT TRANSLATOR API KEY>", false));
```

# <a name="javascripttabjssetuptranslate"></a>[JavaScript](#tab/jssetuptranslate)

若要設定回應 Bot 的翻譯中介軟體，請將下列內容貼入 app.js。

```javascript
const { BotFrameworkAdapter, MemoryStorage, ConversationState } = require('botbuilder');
const { LanguageTranslator, LocaleConverter } = require('botbuilder-ai');
const restify = require('restify');

// Create server
let server = restify.createServer();
server.listen(process.env.port || process.env.PORT || 3978, function () {
    console.log(`${server.name} listening to ${server.url}`);
});

// Create adapter
const adapter = new BotFrameworkAdapter({ 
    appId: process.env.MICROSOFT_APP_ID, 
    appPassword: process.env.MICROSOFT_APP_PASSWORD 
});

// Add conversation state middleware
const conversationState = new ConversationState(new MemoryStorage());
adapter.use(conversationState);

// Add language translator middleware
const languageTranslator = new LanguageTranslator({
    translatorKey: "xxxxxx",
    noTranslatePatterns: new Set(),
    nativeLanguages: ['fr', 'de'] 
});
adapter.use(languageTranslator);


// Listen for incoming requests 
server.post('/api/messages', (req, res) => {
    // Route received request to adapter for processing
    adapter.processActivity(req, res, (context) => {
        if (context.activity.type === 'message') {
            const state = conversationState.get(context);
            const count = state.count === undefined ? state.count = 0 : ++state.count;
            return context.sendActivity(`${count}: You said "${context.activity.text}"`);
        } else {
            return context.sendActivity(`[${context.activity.type} event detected]`);
        }
    });
});
```

---

## <a name="run-the-bot-and-see-translated-input"></a>執行 Bot，並查看已翻譯的輸入

執行 Bot，並以其他語言輸入一些訊息。 您會看到 Bot 翻譯了使用者訊息，並在其回應中指明翻譯。

![Bot 會偵測語言並翻譯輸入](./media/how-to-bot-translate/bot-detects-language-translates-input.png)




## <a name="invoke-logic-in-the-bots-native-language"></a>以 Bot 原生語言叫用邏輯

現在，新增可檢查英文單字的邏輯。 如果使用者以其他語言說「說明」或「取消」，Bot 會將它翻譯成英文，並叫用可檢查英文單字 "help" 或 "cancel" 的邏輯。

# <a name="ctabcshelp"></a>[C#](#tab/cshelp)
```cs
        case ActivityTypes.Message:
            // check the message text before calling context.SendActivity
            var messagetext = context.Activity.Text.Trim().ToLower();
            if (string.Equals("help", messagetext))
            {
                await context.SendActivity("You asked for help.");
            }
```

# <a name="javascripttabjshelp"></a>[JavaScript](#tab/jshelp)
```javascript
if (context.activity.type === 'message') {
    // check the message text before calling context.sendActivity
    var messagetext = context.activity.text.trim().toLowerCase();
    if(messagetext === "help"){
        await context.sendActivity("you said help");
    }
}
```

---

![Bot 偵測法文的 help](./media/how-to-bot-translate/bot-detects-help-french.png)



## <a name="translate-replies-back-to-the-users-language"></a>將回覆翻譯回使用者的語言

將最後一個建構函式參數設定為 `true`，也可以將回覆翻譯回使用者的語言。

# <a name="ctabcstranslateback"></a>[C#](#tab/cstranslateback)
```cs
// Use language recognition to detect the user's language from their message, instead of providing helper callbacks.
// Last parameter indicates that we'll translate replies back to the user's language
middleware.Add(new TranslationMiddleware(new string[] { "en" }, "TRANSLATION-SUBSCRIPTION-KEY", true));
```

# <a name="javascripttabjstranslateback"></a>[JavaScript](#tab/jstranslateback)
```javascript
// Use language recognition to detect the user's language from their message, instead of providing helper callbacks.
// Last parameter indicates that we'll translate replies back to the user's language
const languageTranslator = new LanguageTranslator({
    translatorKey: "xxxxxx",
    noTranslatePatterns: new Set(),
    nativeLanguages: ['en'],
    translateBackToUserLanguage: true
});
adapter.use(languageTranslator);
```

---

## <a name="run-the-bot-to-see-replies-in-the-users-language"></a>執行 Bot，並查看以使用者語言寫的回覆

執行 Bot，並以其他語言輸入一些訊息。 您會看到它會偵測使用者的語言並翻譯回應。

![Bot 會偵測語言並翻譯回應](./media/how-to-bot-translate/bot-detects-language-translates-response.png)


## <a name="adding-logic-for-detecting-or-changing-the-user-language"></a>新增偵測或變更使用者語言的邏輯

您不要讓 Botbuilder SDK 自動偵測使用者的語言，而是提供回呼來新增自有邏輯，以供判斷使用者的語言，或判斷使用者語言何時變更。

> [!TIP] 
> 請參閱範例 Bot 的 SDK 中可實作回呼的翻譯範例，而這些回呼可供偵測及變更使用者語言。 

在下列範例中，`CheckUserChangedLanguage` 回呼會檢查特定使用者訊息以變更語言。 

# <a name="ctabcschangelanguage"></a>[C#](#tab/cschangelanguage)
```cs
// In Startup.cs
middleware.Add(new TranslationMiddleware(new string[] { "en" }, "<YOUR MICROSOFT TRANSLATOR API KEY>", null, TranslatorLocaleHelper.GetActiveLanguage, TranslatorLocaleHelper.CheckUserChangedLanguage));

// Add a file TranslatorLocalHelper.cs with the following:

    class CurrentUserState
    {
        public string Language { get; set; }
    }

    public static void SetLanguage(ITurnContext context, string language) => context.GetConversationState<CurrentUserState>().Language = language;

    public static bool IsSupportedLanguage(string language) => _supportedLanguages.Contains(language);

    public static async Task<bool> CheckUserChangedLanguage(ITurnContext context)
    {
        bool changeLang = false;
        
        // use a specific message from user to change language
        if (context.Activity.Type == ActivityTypes.Message)
        {
            var messageActivity = context.Activity.AsMessageActivity();
            if (messageActivity.Text.ToLower().StartsWith("set my language to"))
            {
                changeLang = true;
            }
            if (changeLang)
            {
                var newLang = messageActivity.Text.ToLower().Replace("set my language to", "").Trim();
                if (!string.IsNullOrWhiteSpace(newLang)
                        && IsSupportedLanguage(newLang))
                {
                    SetLanguage(context, newLang);
                    await context.SendActivity($@"Changing your language to {newLang}");
                }
                else
                {
                    await context.SendActivity($@"{newLang} is not a supported language.");
                }
                // return true to intercept message from further processesing
                return true;
            }
        }

        return false;
    }

    public static string GetActiveLanguage(ITurnContext context)
    {

        if (context.Activity.Type == ActivityTypes.Message
            && context.GetConversationState<CurrentUserState>() != null && context.GetConversationState<CurrentUserState>().Language != null)
        {
            return context.GetConversationState<CurrentUserState>().Language;
        }

        return "en";
    }

```

# <a name="javascripttabjschangelanguage"></a>[JavaScript](#tab/jschangelanguage)
```javascript
// When the user inputs 'set my language to fr'
// The bot will automatically change all text to French
async function setUserLanguage(context) {
    let state = conversationState.get(context)
    if (context.activity.text.toLowerCase().startsWith('set my language to')) {
        state.language = context.activity.text.toLowerCase().replace('set my language to', '').trim();
        await context.sendActivity(`Setting your language to ${state.language}`);
        return Promise.resolve(true);
    } else {
        return Promise.resolve(false);
    }
}

// Add language translator middleware
const languageTranslator = new LanguageTranslator({
    translatorKey: "<YOUR MICROSOFT TRANSLATOR API KEY>",
    noTranslatePatterns: new Set(),
    nativeLanguages: ['en'],
    setUserLanguage: setUserLanguage,
    translateBackToUserLanguage: true
});
adapter.use(languageTranslator);

// Listen for incoming activity 
server.post('/api/messages', (req, res) => {
    // Route received activity to adapter for processing
    adapter.processActivity(req, res, async (context) => {
        if (context.activity.type != 'message') {
            await context.sendActivity(`[${context.activity.type} event detected]`);
        } else {
            await context.sendActivity(`You said: ${context.activity.text}`)
        }
    });
});

```

---

## <a name="combining-luis-or-qna-with-translation"></a>結合 LUIS 或 QnA 與翻譯

如果您想在 Bot 中結合翻譯與其他服務 (如 LUIS 或 QnA Maker)，請先新增翻譯翻譯中介軟體，這樣訊息傳遞到預期 Bot 原生語言的其他中介軟體前就會先經過翻譯。

# <a name="ctabcslanguageluis"></a>[C#](#tab/cslanguageluis)
```cs
public void ConfigureServices(IServiceCollection services)
{
    services.AddSingleton(_ => Configuration);
    services.AddBot<AiBot>(options =>
    {
        options.CredentialProvider = new ConfigurationCredentialProvider(Configuration);
        var middleware = options.Middleware;

        // Add translation middleware
        // The first parameter is a list of languages the bot recognizes. Input in these language won't be translated.
        middleware.Add(new TranslationMiddleware(new string[] { "en" }, "<YOUR MICROSOFT TRANSLATOR API KEY>", false));

        // Add LUIS middleware after translation middleware to pass translated messages to LUIS
        middleware.Add(
            new LuisRecognizerMiddleware(
                new LuisModel("luisAppId", "subscriptionId", new Uri("luisModelBaseUrl"))));
    });
}
```

# <a name="javascripttabjslanguageluis"></a>[JavaScript](#tab/jslanguageluis)
```javascript
// Add language translator middleware
const languageTranslator = new LanguageTranslator({
    translatorKey: "xxxxxx",
    noTranslatePatterns: new Set(),
    nativeLanguages: ['en'],
    translateBackToUserLanguage: false
});
adapter.use(languageTranslator);

// Add Luis recognizer middleware
const luisRecognizer = new LuisRecognizer({
    appId: "xxxxxx",
    subscriptionKey: "xxxxxxx"
});
adapter.use(luisRecognizer);


// Listen for incoming requests 
server.post('/api/messages', (req, res) => {
    // Route received request to adapter for processing
    adapter.processActivity(req, res, async (context) => {
        if (context.activity.type != 'message') {
            await context.sendActivity(`[${context.activity.type} event detected]`);
        } else {
            let results = luisRecognizer.get(context);
            for (const intent in results.intents) {
                await context.sendActivity(`intent: ${intent.toString()} : ${results.intents[intent.toString()]}`)
            }
        }
    });
});

```

---

在您的 Bot 程式碼中，LUIS 結果是以已經翻譯為 Bot 原生語言的輸入為基礎。 請嘗試修改 Bot 程式碼以檢查 LUIS 應用程式的結果：

```cs
public async Task OnTurn(ITurnContext context)
{
    switch (context.Activity.Type)
    {
        case ActivityTypes.Message:
            // check LUIS results
            var luisResult = context.Services.Get<RecognizerResult>(LuisRecognizerMiddleware.LuisRecognizerResultKey);
            (string key, double score) topItem = luisResult.GetTopScoringIntent();
            await context.SendActivity($"The top intent was: '{topItem.key}', with score {topItem.score}");

            await context.SendActivity($"You sent '{context.Activity.Text}'");

        case ActivityTypes.ConversationUpdate:
            foreach (var newMember in context.Activity.MembersAdded)
            {
                if (newMember.Id != context.Activity.Recipient.Id)
                {
                    await context.SendActivity("Hello and welcome to the echo bot.");
                }
            }
            break;
        }
    }  
}          
```

## <a name="bypass-translation-for-specified-patterns"></a>略過指定模式的翻譯
您可能希望 Bot 不要翻譯某些特定單字，例如專有名詞。 您可以提供規則運算式來指示不應該翻譯的模式。 例如，如果使用者以 Bot 的非原生語言說「我的名字是...」，而您想要避免翻譯他們的名字，則可以使用模式來加以指定。

# <a name="ctabcsbypass"></a>[C#](#tab/csbypass)
```cs
    // Startup.cs

    // Pattern representing input to not translate
    Dictionary<string, List<string>> patterns = new Dictionary<string, List<string>>();
    // single pattern for fr language, to avoid translating what follows "my name is"
    patterns.Add("fr", new List<string> { "mon nom est (.+)" });
    
    middleware.Add(new TranslationMiddleware(new string[] { "en" }, "<YOUR API KEY>", patterns, TranslatorLocaleHelper.GetActiveLanguage, TranslatorLocaleHelper.CheckUserChangedLanguage));

```
<!-- TODO: ADD more explanation (both of these callbacks are run every time), fix image by debugging regex for l'etat -->

# <a name="javascripttabjsbypass"></a>[JavaScript](#tab/jsbypass)
```javascript
// Add language translator middleware
const languageTranslator = new LanguageTranslator({
    translatorKey: "<YOUR API KEY>",
    // single pattern for fr language, to avoid translating what follows "my name is"
    noTranslatePatterns: new Set("fr", "/mon nom est (.+)/"),
    nativeLanguages: ['en'],
    translateBackToUserLanguage: false,
    
});
adapter.use(languageTranslator);

```

---

![Bot 會略過模式的翻譯](./media/how-to-bot-translate/bot-no-translate-name-fr.png)

## <a name="localize-dates"></a>當地語系化日期

如果需要將日期當地語系化，您可以新增 `LocaleConverterMiddleware`。 例如，如果您知道 Bot 接受 `MM/DD/YYYY` 格式的日期，而其他地區設定的使用者可能輸入了 `DD/MM/YYYY` 格式的日期，則地區設定轉換器中介軟體可自動將日期轉換為 Bot 接受的格式。

> [!NOTE]
> 地區設定轉換器中介軟體的用途是只轉換日期。 它並不知道翻譯中介軟體的結果。 如果您使用翻譯中介軟體，請注意它與地區設定轉換器的結合方式。 翻譯中介軟體會將一些文字格式的日期，與其他文字輸入一起翻譯，但是它不會翻譯日期

例如，下圖顯示 Bot 將英文翻譯成法文之後回應使用者輸入。 它使用 `TranslationMiddleware` 而未使用 `LocaleConverterMiddleware`。

![Bot 翻譯日期卻不會轉換日期](./media/how-to-bot-translate/locale-date-before.png)

下面顯示如果新增 `LocaleConverterMiddleware` 的相同 Bot。

![Bot 翻譯日期卻不會轉換日期](./media/how-to-bot-translate/locale-date-after.png)

地區設定轉換器可以支援英文、法文、德文及中文的地區設定。 <!-- TODO: ADD DETAIL ABOUT SUPPORTED LOCALES -->

# <a name="ctabcstranslation"></a>[C#](#tab/cstranslation)
```cs
// Startup.cs
// Add locale converter middleware
middleware.Add(new LocaleConverterMiddleware(TranslatorLocaleHelper.GetActiveLocale, TranslatorLocaleHelper.CheckUserChangedLocale, "en-us", LocaleConverter.Converter));

// TranslatorLocaleHelper.cs
        public static async Task<bool> CheckUserChangedLocale(ITurnContext context)
        {
            bool changeLocale = false;//logic implemented by developper to make a signal for language changing 
            //use a specific message from user to change language
            if (context.Activity.Type == ActivityTypes.Message)
            {
                var messageActivity = context.Activity.AsMessageActivity();
                if (messageActivity.Text.ToLower().StartsWith("set my locale to"))
                {
                    changeLocale = true;
                }
                if (changeLocale)
                {
                    var newLocale = messageActivity.Text.ToLower().Replace("set my locale to", "").Trim(); //extracted by the user using user state 
                    if (!string.IsNullOrWhiteSpace(newLocale)
                            && IsSupportedLanguage(newLocale))
                    {
                        SetLocale(context, newLocale);
                        await context.SendActivity($@"Changing your language to {newLocale}");
                    }
                    else
                    {
                        await context.SendActivity($@"{newLocale} is not a supported locale.");
                    }
                    //intercepts message
                    return true;
                }
            }

            return false;
        }
        public static string GetActiveLocale(ITurnContext context)
        {
            if (currentLocale != null)
            {
                //the user has specified a different locale so update the bot state
                if (context.GetConversationState<CurrentUserState>() != null
                    && currentLocale != context.GetConversationState<CurrentUserState>().Locale)
                {
                    SetLocale(context, currentLocale);
                }
            }
            if (context.Activity.Type == ActivityTypes.Message
                && context.GetConversationState<CurrentUserState>() != null && context.GetConversationState<CurrentUserState>().Locale != null)
            {
                return context.GetConversationState<CurrentUserState>().Locale;
            }

            return "en-us";
        }
```

# <a name="javascripttabjstranslation"></a>[JavaScript](#tab/jstranslation)

<!-- this snippet only works if the user doesn't actually try to change their locale.  Emailed Mostafa about the issue 
It should change the locale after you type in 'set my locale to....' -->

```javascript
// Delegates for getting and setting user locale
function getUserLocale(context) {
    const state = conversationState.get(context)
    if (state.locale == undefined) {
        return 'en-us';
    } else {
        return state.locale;
    }
}

// Function to change the user's locale.
async function setUserLocale(context) {
    let state = conversationState.get(context)
    if (context.activity.text.toLowerCase().startsWith('set my locale to')) {        
        state.locale = context.activity.text.toLowerCase().replace('set my locale to', '').trim();
        await context.sendActivity(`Setting your locale to ${state.locale}`);
        return Promise.resolve(true);
    } else {
        return Promise.resolve(false);
    }
}

// Add locale converter middleware
// Will convert input to fr-fr
const localeConverter = new LocaleConverter({
    toLocale: 'fr-fr',
    setUserLocale: setUserLocale,
    getUserLocale: getUserLocale
});
adapter.use(localeConverter);


// Listen for incoming requests 
server.post('/api/messages', (req, res) => {
    // Route received request to adapter for processing
    adapter.processActivity(req, res, async (context) => {
        if (context.activity.type != 'message') {
            await context.sendActivity(`[${context.activity.type} event detected]`);
        } else {
            await context.sendActivity(`The message that you sent was:  ${context.activity.text}`)
        }
    });
});

```

---
