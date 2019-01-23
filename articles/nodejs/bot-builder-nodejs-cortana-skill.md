---
title: 使用 Cortana 技能建置啟用語音的 Bot | Microsoft Docs
description: 了解如何使用 Cortana 技能與適用於 Node.js 的 Bot Framework SDK 來建置啟用語音的 Bot。
author: DeniseMak
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: e00128ca82ec8b97502d8f2fbf42be10cc91ade6
ms.sourcegitcommit: b15cf37afc4f57d13ca6636d4227433809562f8b
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 01/11/2019
ms.locfileid: "54225295"
---
# <a name="build-a-speech-enabled-bot-with-cortana-skills"></a>使用 Cortana 技能建置啟用語音的 Bot

[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]

> [!div class="op_single_selector"]
> - [.NET](../dotnet/bot-builder-dotnet-cortana-skill.md)
> - [Node.js](../nodejs/bot-builder-nodejs-cortana-skill.md)

適用於 Node.js 的 Bot Framework SDK 可藉由將 Bot 連線到 Cortana 通道作為 Cortana 技能，讓您建置啟用語音的 Bot。 Cortana 技能可讓您透過 Cortana 提供功能來回應使用者的語音輸入。

> [!TIP]
> 如需有關技能是什麼及其用途的詳細資訊，請參閱 [Cortana 技能套件][CortanaGetStarted]。

使用 Bot Framework 建立 Cortana 技能不需要太多 Cortana 專屬知識，主要是建置 Bot。 與其他您可能已建立的 Bot 主要差異在於，Cortana 同時具有視覺與音訊元件。 對於視覺元件，Cortana 提供用於轉譯內容 (例如卡片) 的畫布區域。 對於音訊元件，您可以在 Bot 訊息中提供文字或 SSML，Cortana 會讀給使用者聽，讓您的 Bot 擁有語音功能。 

> [!NOTE]
> 在許多不同的裝置上皆有提供 Cortana。 部分具有螢幕，而其他 (如獨立喇叭) 可能沒有。 您應確認 Bot 能夠處理這兩者。 請參閱 [Cortana 特定實體][CortanaSpecificEntities]，以了解如何檢查裝置資訊。

## <a name="adding-speech-to-your-bot"></a>將語音新增至您的 Bot

Bot 的語音訊息會表示為語音合成標記語言 (SSML)。 Bot Framework SDK 可讓您將 SSML 包含在 Bot 的回應中，以在 Bot 顯示的內容之外控制 Bot 所說內容。

### <a name="sessionsay"></a>session.say

您的 Bot 會用 **session.say** 方法來取代 **session.send**，向使用者說話。 它包含用於傳送 SSML 輸出的選擇性參數，以及卡片等附件。 

該方法的格式如下：

```session.say(displayText: string, speechText: string, options?: object)```

| 參數 | 說明 |
|------|------|
| **displayText** | 要顯示在 Cortana UI 中的文字訊息。|
| **speechText** | Cortana 讀給使用者聽的文字或 SSML。 |
| **options** | 可包含附件或輸入提示的 [IMessage][IMessage] 物件。 輸入提示會指出 Bot 正在接受、預期或忽略輸入。 卡片附件會顯示在 Cortana 畫布中 **displayText** 資訊的下方。   |

**inputHint** 屬性有助於向 Cortana 指出 Bot 是否預期輸入。 如果您使用內建的提示，這個值會自動設為 **expectingInput** 的預設值。


| 值 | 說明 |
|------|------|
| **acceptingInput** | 您的 Bot 會被動接受輸入，但不會等候回應。 如果使用者按住 [麥克風] 按鈕，Cortana 會接受使用者的輸入。|
| **expectingInput** | 指出 Bot 正主動預期使用者的回應。 Cortana 會聆聽使用者對麥克風說的話。  |
| **ignoringInput** | Cortana 會忽略輸入。 如果您的 Bot 主動處理要求，則可能會傳送這項提示，且在完成要求之前，將會忽略來自使用者的輸入。  |


下列範例會示範 Cortana 如何讀取純文字或 SSML：

```javascript
// Have Cortana read plain text
session.say('This is the text that Cortana displays', 'This is the text that is spoken by Cortana.');

// Have Cortana read SSML
session.say('This is the text that Cortana displays', '<speak version="1.0" xmlns="http://www.w3.org/2001/10/synthesis" xml:lang="en-US">This is the text that is spoken by Cortana.</speak>');
```

此範例會示範如何讓 Cortana 知道該預期使用者輸入。 麥克風會保持開啟狀態。
```javascript
// Add an InputHint to let Cortana know to expect user input
session.say('Hi there', 'Hi, what’s your name?', {
    inputHint: builder.InputHint.expectingInput
});
```
<!-- TODO: tip about time limit and batching -->


### <a name="prompts"></a>提示

除了使用 **session.say()** 方法，您也可以使用 **speak** 和 **retrySpeak** 選項，將文字或 SSML 傳遞給內建的提示。  

```javascript
builder.Prompts.text(session, 'text based prompt', {                                    
    speak: 'Cortana reads this out initially',                                               
    retrySpeak: 'This message is repeated by Cortana after waiting a while for user input',  
    inputHint: builder.InputHint.expectingInput                                              
});
```



<!-- TODO: Link to SSML library -->

若要向使用者顯示選項清單，請使用 **Prompts.choice**。 **同義詞**選項可在辨識使用者語句時允許更多彈性空間。 在 **results.response.entity** 中會傳回**值**選項。 **動作**選項會指定 Bot 針對選擇所顯示的標籤。

**Prompts.choice** 可支援序數選擇。 這表示使用者可以說「第一個」、「第二個」、「第三個」來選擇清單中的項目。 比方說，在下列提示中，如果使用者要求 Cortana 提供「第二個選項」，則提示傳回的值為 8。

```javascript
        var choices = [
            { value: '4', action: { title: '4 Sides' }, synonyms: 'four|for|4 sided|4 sides' },
            { value: '8', action: { title: '8 Sides' }, synonyms: 'eight|ate|8 sided|8 sides' },
            { value: '12', action: { title: '12 Sides' }, synonyms: 'twelve|12 sided|12 sides' },
            { value: '20', action: { title: '20 Sides' }, synonyms: 'twenty|20 sided|20 sides' },
        ];
        builder.Prompts.choice(session, 'choose_sides', choices, { 
            speak: speak(session, 'choose_sides_ssml') // use helper function to format SSML
        });
```

在上一個範例中，提示中 **speak** 屬性的 SSML，會使用儲存在當地語系化提示檔案中的字串，以下列格式進行格式化。 

```json
{
    "choose_sides": "__Number of Sides__",
    "choose_sides_ssml": [
        "How many sides on the dice?",
        "Pick your poison.",
        "All the standard sizes are supported."
    ]
}
```


協助程式函式接著會建置語音合成標記語言 (SSML) 文件的必要根項目。 

```javascript

module.exports.speak = function (template, params, options) {
    options = options || {};
    var output = '<speak xmlns="http://www.w3.org/2001/10/synthesis" ' +
        'version="' + (options.version || '1.0') + '" ' +
        'xml:lang="' + (options.lang || 'en-US') + '">';
    output += module.exports.vsprintf(template, params);
    output += '</speak>';
    return output;
}
```

> [!TIP]
> 您可以在[骰子機範例技能](https://github.com/Microsoft/BotBuilder-Samples/tree/master/Node/demo-RollerSkill) (英文) 中找到小型的公用程式模組 (ssml.js)，用以建置 Bot 的 SSML 型回應。
> 另外還有幾個 [npm](https://www.npmjs.com/search?q=ssml) (英文) 提供的實用 SSML 程式庫，可讓您輕鬆建立格式化的 SSML。

## <a name="display-cards-in-cortana"></a>在 Cortana 中顯示卡片

除了語音回應，Cortana 也可以顯示卡片附件。 Cortana 可支援下列多媒體卡片：
* [HeroCard](https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.herocard.html)
* [ReceiptCard](https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.receiptcard.html)
* [ThumbnailCard](https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.thumbnailcard.html)

請參閱[卡片設計最佳做法][CardDesign]，以查看這些卡片在 Cortana 內的樣貌。 如需如何將複合式資訊卡 (Rich Card) 新增至 Bot 的範例，請參閱[傳送複合式資訊卡 (Rich Card)](bot-builder-nodejs-send-rich-cards.md)。 

下列程式碼會示範如何將**語音**和 **inputHint** 屬性，新增至包含 Hero 卡片的訊息中。

```javascript 

bot.dialog('HelpDialog', function (session) {
    var card = new builder.HeroCard(session)
        .title('help_title')
        .buttons([
            builder.CardAction.imBack(session, 'roll some dice', 'Roll Dice'),
            builder.CardAction.imBack(session, 'play yahtzee', 'Play Yahtzee')
        ]);
    var msg = new builder.Message(session)
        .speak(speak(session, 'I\'m roller, the dice rolling bot. You can say \'roll some dice\''))
        .addAttachment(card)
        .inputHint(builder.InputHint.acceptingInput); // Tell Cortana to accept input
    session.send(msg).endDialog();
}).triggerAction({ matches: /help/i });


/** This helper function builds the required root element of a Speech Synthesis Markup Language (SSML) document. */
module.exports.speak = function (template, params, options) {
    options = options || {};
    var output = '<speak xmlns="http://www.w3.org/2001/10/synthesis" ' +
        'version="' + (options.version || '1.0') + '" ' +
        'xml:lang="' + (options.lang || 'en-US') + '">';
    output += module.exports.vsprintf(template, params);
    output += '</speak>';
    return output;
}

```
## <a name="sample-rollerskill"></a>範例：RollerSkill
下列各節中的程式碼來自擲骰子的範例 Cortana 技能。 請從 [BotBuilder-Samples 存放庫](https://github.com/Microsoft/BotBuilder-Samples/tree/master/Node/demo-RollerSkill) (英文) 下載 Bot 的完整程式碼。

您可以向 Cortana 說出技能的[叫用名稱][InvocationNameGuidelines]以便叫用技能。 對於骰子機技能，在您[將 Bot 新增至 Cortana 通道][CortanaChannel]並將其註冊為 Cortana 技能後，可以告訴 Cortana「啟動骰子機」或「讓骰子機擲骰子」以進行叫用。

### <a name="explore-the-code"></a>探索程式碼

RollerSkill 範例一開始會開啟包含一些按鈕的卡片，來告知使用者哪些選項可供使用。

```javascript
/**
 *   Create your bot with a default message handler that receive messages from the user.
 * - This function is be called anytime the user's utterance isn't
 *   recognized by the other handlers in the bot.
 */
var bot = new builder.UniversalBot(connector, function (session) {
    // Just redirect to our 'HelpDialog'.
    session.replaceDialog('HelpDialog');
});

//...

bot.dialog('HelpDialog', function (session) {
    var card = new builder.HeroCard(session)
        .title('help_title')
        .buttons([
            builder.CardAction.imBack(session, 'roll some dice', 'Roll Dice'),
            builder.CardAction.imBack(session, 'play craps', 'Play Craps')
        ]);
    var msg = new builder.Message(session)
        .speak(speak(session, 'help_ssml'))
        .addAttachment(card)
        .inputHint(builder.InputHint.acceptingInput);
    session.send(msg).endDialog();
}).triggerAction({ matches: /help/i });
```

### <a name="prompt-the-user-for-input"></a>提示使用者輸入

以下對話方塊會設定自訂遊戲讓 Bot 玩。  它會詢問使用者想要讓骰子有幾個面，以及應該擲幾個骰子。 建置遊戲結構之後，它會將其傳遞至個別的 'PlayGameDialog'。

若要啟動對話方塊，此對話方塊上的 **triggerAction()** 處理常式可讓使用者說出「我想要擲骰子」等內容。 它會使用規則運算式來比對使用者的輸入，但您也可以使用 [LUIS 意圖](./bot-builder-nodejs-recognize-intent-luis.md)。 


```javascript


bot.dialog('CreateGameDialog', [
    function (session) {
        // Initialize game structure.
        // - dialogData gives us temporary storage of this data in between
        //   turns with the user.
        var game = session.dialogData.game = { 
            type: 'custom', 
            sides: null, 
            count: null,
            turns: 0
        };

        var choices = [
            { value: '4', action: { title: '4 Sides' }, synonyms: 'four|for|4 sided|4 sides' },
            { value: '6', action: { title: '6 Sides' }, synonyms: 'six|sex|6 sided|6 sides' },
            { value: '8', action: { title: '8 Sides' }, synonyms: 'eight|8 sided|8 sides' },
            { value: '10', action: { title: '10 Sides' }, synonyms: 'ten|10 sided|10 sides' },
            { value: '12', action: { title: '12 Sides' }, synonyms: 'twelve|12 sided|12 sides' },
            { value: '20', action: { title: '20 Sides' }, synonyms: 'twenty|20 sided|20 sides' },
        ];
        builder.Prompts.choice(session, 'choose_sides', choices, { 
            speak: speak(session, 'choose_sides_ssml') 
        });
    },
    function (session, results) {
        // Store users input
        // - The response comes back as a find result with index & entity value matched.
        var game = session.dialogData.game;
        game.sides = Number(results.response.entity);

        /**
         * Ask for number of dice.
         */
        var prompt = session.gettext('choose_count', game.sides);
        builder.Prompts.number(session, prompt, {
            speak: speak(session, 'choose_count_ssml'),
            minValue: 1,
            maxValue: 100,
            integerOnly: true
        });
    },
    function (session, results) {
        // Store users input
        // - The response is already a number.
        var game = session.dialogData.game;
        game.count = results.response;

        /**
         * Play the game we just created.
         * 
         * replaceDialog() ends the current dialog and start a new
         * one in its place. We can pass arguments to dialogs so we'll pass the
         * 'PlayGameDialog' the game we created.
         */
        session.replaceDialog('PlayGameDialog', { game: game });
    }
]).triggerAction({ matches: [
    /(roll|role|throw|shoot).*(dice|die|dye|bones)/i,
    /new game/i
 ]});
```

### <a name="render-results"></a>轉譯結果

 這個對話方塊是主要的遊戲迴圈。 Bot 會將遊戲結構儲存在 **session.conversationData** 中，因此萬一使用者說「再擲一次」，我們可以再次重擲同一組骰子。

```javascript

bot.dialog('PlayGameDialog', function (session, args) {
    // Get current or new game structure.
    var game = args.game || session.conversationData.game;
    if (game) {
        // Generate rolls
        var total = 0;
        var rolls = [];
        for (var i = 0; i < game.count; i++) {
            var roll = Math.floor(Math.random() * game.sides) + 1;
            if (roll > game.sides) {
                // Accounts for 1 in a million chance random() generated a 1.0
                roll = game.sides;
            }
            total += roll;
            rolls.push(roll);
        }

        // Format roll results
        var results = '';
        var multiLine = rolls.length > 5;
        for (var i = 0; i < rolls.length; i++) {
            if (i > 0) {
                results += ' . ';
            }
            results += rolls[i];
        }

        // Render results using a card
        var card = new builder.HeroCard(session)
            .subtitle(game.count > 1 ? 'card_subtitle_plural' : 'card_subtitle_singular', game)
            .buttons([
                builder.CardAction.imBack(session, 'roll again', 'Roll Again'),
                builder.CardAction.imBack(session, 'new game', 'New Game')
            ]);
        if (multiLine) {
            //card.title('card_title').text('\n\n' + results + '\n\n');
            card.text(results);
        } else {
            card.title(results);
        }
        var msg = new builder.Message(session).addAttachment(card);

        // Determine bots reaction for speech purposes
        var reaction = 'normal';
        var min = game.count;
        var max = game.count * game.sides;
        var score = total/max;
        if (score == 1.0) {
            reaction = 'best';
        } else if (score == 0) {
            reaction = 'worst';
        } else if (score <= 0.3) {
            reaction = 'bad';
        } else if (score >= 0.8) {
            reaction = 'good';
        }

        // Check for special craps rolls
        if (game.type == 'craps') {
            switch (total) {
                case 2:
                case 3:
                case 12:
                    reaction = 'craps_lose';
                    break;
                case 7:
                    reaction = 'craps_seven';
                    break;
                case 11:
                    reaction = 'craps_eleven';
                    break;
                default:
                    reaction = 'craps_retry';
                    break;
            }
        }

        // Build up spoken response
        var spoken = '';
        if (game.turn == 0) {
            spoken += session.gettext('start_' + game.type + '_game_ssml') + ' ';
        } 
        spoken += session.gettext(reaction + '_roll_reaction_ssml');
        msg.speak(ssml.speak(spoken));

        // Increment number of turns and store game to roll again
        game.turn++;
        session.conversationData.game = game;

        /**
         * Send card and bot's reaction to user. 
         */

        msg.inputHint(builder.InputHint.acceptingInput);
        session.send(msg).endDialog();
    } else {
        // User started session with "roll again" so let's just send them to
        // the 'CreateGameDialog'
        session.replaceDialog('CreateGameDialog');
    }
}).triggerAction({ matches: /(roll|role|throw|shoot) again/i });
```

## <a name="next-steps"></a>後續步驟
如果您的 Bot 是在本機執行或部署在雲端中，可以從 Cortana 叫用。 請參閱[測試 Cortana 技能](../bot-service-debug-cortana-skill.md)，以了解試用 Cortana 技能所需的步驟。


## <a name="additional-resources"></a>其他資源
* [Cortana 技能套件][CortanaGetStarted]
* [將語音新增至訊息](bot-builder-nodejs-text-to-speech.md)
* [SSML 參考][SSMLRef]
* [Cortana 的語音設計最佳做法][VoiceDesign]
* [Cortana 的卡片設計最佳做法][CardDesign]
* [Cortana 開發人員中心][CortanaDevCenter]
* [Cortana 的測試和偵錯最佳做法][Cortana-TestBestPractice]


[CortanaGetStarted]: /cortana/getstarted
[BFPortal]: https://dev.botframework.com/


[SSMLRef]: https://aka.ms/cortana-ssml
[IMessage]: https://docs.botframework.com/en-us/node/builder/chat-reference/interfaces/_botbuilder_d_.imessage.html
[Send]: https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.session#send
[CortanaDevCenter]: https://developer.microsoft.com/en-us/cortana

[CortanaSpecificEntities]: https://aka.ms/lgvcto
[CortanaAuth]: https://aka.ms/vsdqcj

[InvocationNameGuidelines]: https://aka.ms/cortana-invocation-guidelines
[VoiceDesign]: https://aka.ms/cortana-design-voice
[CardDesign]: https://aka.ms/cortana-design-card
[Cortana-Debug]: https://aka.ms/cortana-enable-debug
[Cortana-Publish]: https://aka.ms/cortana-publish


[CortanaTry]: https://aka.ms/try-cortana-bot
[CortanaChannel]: https://aka.ms/bot-cortana-channel
[Cortana-TestBestPractice]: https://aka.ms/cortana-test-best-practice
