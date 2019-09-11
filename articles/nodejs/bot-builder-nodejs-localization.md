---
title: 支援當地語系化 | Microsoft Docs
description: 了解如何判斷使用者所在位置，並使用適用於 Node.js 的 Bot Framework SDK 啟用當地語系化功能。
author: DeniseMak
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: 68cc1134e8a26c8cbc49b0527cc598c5b409d5db
ms.sourcegitcommit: a6d02ec4738e7fc90b7108934740e9077667f3c5
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/04/2019
ms.locfileid: "70299814"
---
# <a name="support-localization"></a>支援當地語系化

[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]

Bot Builder 包含豐富的當地語系化系統，以建置可使用多種語言與使用者通訊的 Bot。 您 Bot 的所有提示都可以使用儲存在您 Bot 目錄結構中的 JSON 檔案來當地語系化。 如果您使用的系統如同執行自然語言處理的 LUIS，則您可以使用個別的模型為您 Bot 支援的每種語言設定 [LuisRecognizer][LUISRecognizer]，然後 SDK 就會自動選取符合使用者慣用地區設定的模型。

## <a name="determine-the-locale-by-prompting-the-user"></a>透過提示使用者來決定地區設定
當地語系化您 Bot 的第一個步驟，是新增識別使用者慣用語言的能力。 SDK 提供的 [session.preferredLocale()][preferredLocal] 方法可儲存並擷取每位使用者的此一偏好。 下列範例對話方塊會提示使用者輸入其偏好的語言，然後儲存其選擇。

``` javascript
bot.dialog('/localePicker', [
    function (session) {
        // Prompt the user to select their preferred locale
        builder.Prompts.choice(session, "What's your preferred language?", 'English|Español|Italiano');
    },
    function (session, results) {
        // Update preferred locale
        var locale;
        switch (results.response.entity) {
            case 'English':
                locale = 'en';
                break;
            case 'Español':
                locale = 'es';
                break;
            case 'Italiano':
                locale = 'it';
                break;
        }
        session.preferredLocale(locale, function (err) {
            if (!err) {
                // Locale files loaded
                session.endDialog(`Your preferred language is now ${results.response.entity}`);
            } else {
                // Problem loading the selected locale
                session.error(err);
            }
        });
    }
]);
```

## <a name="determine-the-locale-by-using-analytics"></a>使用分析決定地區設定
另一個決定使用者地區設定的方法，是使用類似[文字分析 API](/azure/cognitive-services/cognitive-services-text-analytics-quick-start) 的服務，根據使用者傳送訊息所用之文字自動偵測使用者的語言。

下列程式碼片段示範如何將這項服務納入您自己的 Bot。
``` javascript
var request = require('request');

bot.use({
    receive: function (event, next) {
        if (event.text && !event.textLocale) {
            var options = {
                method: 'POST',
                url: 'https://westus.api.cognitive.microsoft.com/text/analytics/v2.0/languages?numberOfLanguagesToDetect=1',
                body: { documents: [{ id: 'message', text: event.text }]},
                json: true,
                headers: {
                    'Ocp-Apim-Subscription-Key': '<YOUR API KEY>'
                }
            };
            request(options, function (error, response, body) {
                if (!error && body) {
                    if (body.documents && body.documents.length > 0) {
                        var languages = body.documents[0].detectedLanguages;
                        if (languages && languages.length > 0) {
                            event.textLocale = languages[0].iso6391Name;
                        }
                    }
                }
                next();
            });
        } else {
            next();
        }
    }
});
```

一旦您將上述程式碼片段新增至您的 Bot，呼叫 [session.preferredLocale()][preferredLocal] 就會自動傳回偵測到的語言。 `preferredLocale()` 的搜尋順序如下：
1. 呼叫 `session.preferredLocale()` 來儲存地區設定。 此值儲存在 `session.userData['BotBuilder.Data.PreferredLocale']`。
2. 偵測到的地區設定會指派給 `session.message.textLocale`。
3. Bot 的已設定預設地區設定 (例如：英文 ('en'))。

您可以使用其建構函式設定 Bot 的預設地區設定：

```javascript
var bot = new builder.UniversalBot(connector, {
    localizerSettings: { 
        defaultLocale: "es" 
    }
});
```

## <a name="localize-prompts"></a>當地語系化提示
Bot Framework SDK 的預設當地語系化系統是以檔案為基礎，允許 Bot 使用儲存在磁碟上的 JSON 檔案支援多種語言。 根據預設，當地語系化系統會在 **./locale/<IETF TAG>/index.json** 檔案中搜尋 Bot 的提示，<IETF TAG> 是有效的 [IETF 語言標記][IEFT]，代表用來尋找提示的慣用地區設定。 

下列螢幕擷取畫面顯示支援三種語言的 Bot 目錄結構：英文、義大利文和西班牙文。

![三種地區設定的目錄結構](../media/locale-dir.png)

檔案的結構是訊息識別碼和當地語系化文字字串的簡單 JSON 對應圖。 如果值為陣列而非字串，當使用 [session.localizer.gettext()][GetText] 擷取該值時會隨機選擇陣列中的一個提示。 

如果您在 [session.send()](http://docs.botframework.com/node/builder/chat-reference/classes/_botbuilder_d_.session#send) 呼叫中傳送的是訊息識別碼，而不是特定語言文字，Bot 就會自動擷取訊息的當地語系化版本：

```javascript
var bot = new builder.UniversalBot(connector, [
    function (session) {
        session.send("greeting");
        session.send("instructions");
        session.beginDialog('/localePicker');
    },
    function (session) {
        builder.Prompts.text(session, "text_prompt");
    }
]);
```

在內部，SDK 會呼叫 [`session.preferredLocale()`][preferredLocale] 以取得使用者的慣用地區設定，然後在 [`session.localizer.gettext()`][GetText] 呼叫中使用，將訊息識別碼對應至其當地語系化的文字字串。  您有時候可能需要手動呼叫當地語系化工具。 例如，傳送給 [`Prompts.choice()`][promptsChoice] 的列舉值永遠不會自動當地語系化，因此您需要先手動擷取當地語系化清單，再呼叫提示：

```javascript
var options = session.localizer.gettext(session.preferredLocale(), "choice_options");
builder.Prompts.choice(session, "choice_prompt", options);
```

預設的當地語系化工具會跨多個檔案搜尋訊息識別碼，如果它找不到識別碼 (或如果未提供當地語系化檔案)，它只會傳回識別碼的文字，以使用者不察和選擇性的方式使用當地語系化檔案。  檔案搜尋順序如下：

1. 搜尋 [`session.preferredLocale()`][preferredLocale] 所傳回之地區設定下的 **index.json**。
2. 如果地區設定包含選擇性的子標記，例如 **en-US**，則搜尋 **en** 的根標記。
3. 搜尋 Bot 的已設定預設地區設定。

## <a name="use-namespaces-to-customize-and-localize-prompts"></a>使用命名空間自訂與當地語系化提示
預設的當地語系化工具支援建立提示的命名空間，以避免訊息識別碼之間發生衝突。  您的 Bot 可以覆寫已建立命名空間的提示，以自訂或重寫另一個命名空間的提示。  您可以利用此功能自訂 SDK 的內建訊息，讓您新增其他語言的支援，或只重寫 SDK 的目前訊息。  例如，您可以變更 SDK 的預設錯誤訊息，只要將稱為 **BotBuilder.json** 的檔案新增至您 Bot 的地區設定目錄，然後新增 `default_error` 訊息識別碼項目即可：

![用於建立地區設定命名空間的 BotBuilder.json](../media/locale-namespacing.png)


## <a name="additional-resources"></a>其他資源

若要深入了解如何當地語系化辨識器，請參閱[辨識意圖](bot-builder-nodejs-recognize-intent-messages.md)。


[LUIS]: https://www.luis.ai/
[IMessage]: http://docs.botframework.com/node/builder/chat-reference/interfaces/_botbuilder_d_.imessage
[IntentRecognizerSetOptions]: https://docs.botframework.com/node/builder/chat-reference/interfaces/_botbuilder_d_.iintentrecognizersetoptions.html
[LUISRecognizer]: https://docs.botframework.com/node/builder/chat-reference/classes/_botbuilder_d_.luisrecognizer
[LUISSample]: https://aka.ms/v3-js-luisSample
[DisambiguationSample]: https://aka.ms/v3-js-onDisambiguateRoute
[preferredLocal]: https://docs.botframework.com/node/builder/chat-reference/classes/_botbuilder_d_.session#preferredlocale
[preferredLocale]: https://docs.botframework.com/node/builder/chat-reference/classes/_botbuilder_d_.session#preferredlocale
[promptsChoice]: https://docs.botframework.com/node/builder/chat-reference/interfaces/_botbuilder_d_.__global.iprompts.html#choice
[GetText]: https://docs.botframework.com/node/builder/chat-reference/interfaces/_botbuilder_d_.ilocalizer.html#gettext
[IEFT]: https://en.wikipedia.org/wiki/IETF_language_tag

