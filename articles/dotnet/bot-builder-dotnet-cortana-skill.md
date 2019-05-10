---
title: 使用 .NET 建置 Cortana 技能 | Microsoft Docs
description: 了解在適用於 .NET 的 Bot Framework SDK 中建置 Cortana 技能的核心概念。
keywords: Bot Framework, Cortana 技能, 語音, .NET, SDK, 重要概念, 核心概念
author: DeniseMak
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: fd7d20b71c8f6c3013e7af5c7c80623089f0dce0
ms.sourcegitcommit: f84b56beecd41debe6baf056e98332f20b646bda
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 05/03/2019
ms.locfileid: "65032901"
---
# <a name="build-a-speech-enabled-bot-with-cortana-skills"></a>使用 Cortana 技能建置啟用語音的 Bot

[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]

> [!div class="op_single_selector"]
> - [.NET](../dotnet/bot-builder-dotnet-cortana-skill.md)
> - [Node.js](../nodejs/bot-builder-nodejs-cortana-skill.md)


適用於 .NET 的 Bot Framework SDK 可藉由將 Bot 連線到 Cortana 通道作為 Cortana 技能，讓您建置啟用語音的 Bot。 


> [!TIP]
> 如需有關技能是什麼及其用途的詳細資訊，請參閱 [Cortana 技能套件][CortanaGetStarted]。

使用 Bot Framework 建立 Cortana 技能不需要太多 Cortana 專屬知識，主要是建置 Bot。 與其他您在過去可能已建立的 Bot 可能主要差異在於，Cortana 同時具有視覺與音訊元件。 對於視覺元件，Cortana 提供用於轉譯內容 (例如卡片) 的畫布區域。 對於音訊元件，您可以在 Bot 訊息中提供文字或 SSML，Cortana 會讀給使用者聽，讓您的 Bot 擁有語音功能。 

> [!NOTE]
> 在許多不同的裝置上皆有提供 Cortana。 部分具有螢幕，而其他 (如獨立喇叭) 可能沒有。 您應確認 Bot 能夠處理這兩者。 請參閱 [Cortana 特定實體][CortanaSpecificEntities]，以了解如何檢查裝置資訊。

## <a name="adding-speech-to-your-bot"></a>將語音新增至您的 Bot

Bot 的語音訊息會表示為語音合成標記語言 (SSML)。 Bot Framework SDK 可讓您將 SSML 包含在 Bot 的回應中，以在 Bot 顯示的內容之外控制 Bot 所說內容。  您也可以指定 Bot 是否接受、預期還是略過使用者輸入，來控制 Cortana 的麥克風狀態。

設定 `IMessageActivity` 物件的 `Speak` 屬性，可指定 Cortana 說出的訊息。 如果您指定純文字，Cortana 會決定單字發音的方式。 

```cs
Activity reply = activity.CreateReply("This is the text that Cortana displays."); 
reply.Speak = "This is the text that Cortana will say.";
```

如果您想要更充分掌控音調、語氣及強調，請將 `Speak` 屬性格式化為 [語音合成標記語言 (SSML)](http://www.w3.org/TR/speech-synthesis/) (英文)。  

下列程式碼範例會指定應該以適量的強調說出 "text" 這個字：
```cs
Activity reply = activity.CreateReply("This is the text that will be displayed.");
reply.Speak = "<speak version=\"1.0\" xmlns=\"http://www.w3.org/2001/10/synthesis\" xml:lang=\"en-US\">This is the <emphasis level=\"moderate\">text</emphasis> that will be spoken.</speak>";
```


**inputHint** 屬性有助於向 Cortana 指出 Bot 是否預期輸入。 提示的預設值是 **ExpectingInput**，而其他類型回應的預設值為 **AcceptingInput**。


| 值 | 說明 |
|------|------|
| **AcceptingInput** | 您的 Bot 會被動接受輸入，但不會等候回應。 如果使用者按住 [麥克風] 按鈕，Cortana 會接受使用者的輸入。|
| **ExpectingInput** | 指出 Bot 正主動預期使用者的回應。 Cortana 會聆聽使用者對麥克風說的話。  |
| **IgnoringInput** | Cortana 會忽略輸入。 如果您的 Bot 主動處理要求，則可能會傳送這項提示，且在完成要求之前，將會忽略來自使用者的輸入。  |

<!-- TODO: tip about time limit and batching -->

此範例會示範如何讓 Cortana 知道該預期使用者輸入。 麥克風會保持開啟狀態。
```cs
// Add an InputHint to let Cortana know to expect user input
Activity reply = activity.CreateReply("This is the text that will be displayed."); 
reply.Speak = "This is the text that will be spoken.";
reply.InputHint = InputHints.ExpectingInput;
```



## <a name="display-cards-in-cortana"></a>在 Cortana 中顯示卡片

除了語音回應，Cortana 也可以顯示卡片附件。 Cortana 可支援下列複合式資訊卡 (Rich Card)：

| 卡片類型 | 說明 |
|----|----|
| [HeroCard][heroCard] | 通常包含單一大型影像、一或多個按鈕和文字的卡片。 |
| [ThumbnailCard][thumbnailCard] | 通常包含單一縮圖影像、一或多個按鈕和文字的卡片。 |
| [ReceiptCard][receiptCard] | 讓 Bot 向使用者提供收據的卡片。 通常包含收據上的項目清單 (稅金和總金額資訊) 和其他文字。 |
| [SignInCard][signinCard] | 可讓 Bot 要求使用者登入的卡片。 它通常包含文字，以及一或多個按鈕，使用者可以按一下按鈕來起始登入流程。 |


請參閱[卡片設計最佳做法][CardDesign]，以查看這些卡片在 Cortana 內的樣貌。 如需如何在 Bot 中使用複合式資訊卡 (Rich Card) 的範例，請參閱[將多媒體卡片附件新增到郵件](bot-builder-dotnet-add-rich-card-attachments.md)。 

<!--
The following code demonstrates how to add the `Speak` and `InputHint` properties to a message containing a `HeroCard`.
-->


## <a name="sample-rollerskill"></a>範例：RollerSkill
下列各節中的程式碼來自擲骰子的範例 Cortana 技能。 請從 [BotBuilder-Samples 存放庫](https://github.com/Microsoft/BotBuilder-Samples/) (英文) 下載 Bot 的完整程式碼。

您可以向 Cortana 說出技能的[叫用名稱][InvocationNameGuidelines]以便叫用技能。 對於骰子機技能，在您[將 Bot 新增至 Cortana 通道][CortanaChannel]並將其註冊為 Cortana 技能後，可以告訴 Cortana「啟動骰子機」或「讓骰子機擲骰子」以進行叫用。

### <a name="explore-the-code"></a>探索程式碼



為了叫用適當的對話方塊，`RootDispatchDialog.cs` 中定義的活動處理常式會使用規則運算式，來比對使用者的輸入。 例如，如果使用者說了像是「我想要擲骰子」這樣的語句，就會觸發下列範例中的處理常式。 同義字包含在規則運算式中，因此類似的語句將會觸發對話方塊。
```cs
        [RegexPattern("(roll|role|throw|shoot).*(dice|die|dye|bones)")]
        [RegexPattern("new game")]
        [ScorableGroup(1)]
        public async Task NewGame(IDialogContext context, IActivity activity)
        {
            context.Call(new CreateGameDialog(), AfterGameCreated);
        }
```

`CreateGameDialog` 對話方塊會設定自訂遊戲讓 Bot 玩。 它會使用 `PromptDialog` 來詢問使用者想要讓骰子有幾個面，以及應該擲幾個骰子。 請注意，用來初始化提示的 `PromptOptions` 物件包含 `speak` 屬性，可供語音版本的提示使用。

```cs
    [Serializable]
    public class CreateGameDialog : IDialog<GameData>
    {
        public async Task StartAsync(IDialogContext context)
        {
            context.UserData.SetValue<GameData>(Utils.GameDataKey, new GameData());

            var descriptions = new List<string>() { "4 Sides", "6 Sides", "8 Sides", "10 Sides", "12 Sides", "20 Sides" };
            var choices = new Dictionary<string, IReadOnlyList<string>>()
             {
                { "4", new List<string> { "four", "for", "4 sided", "4 sides" } },
                { "6", new List<string> { "six", "sex", "6 sided", "6 sides" } },
                { "8", new List<string> { "eight", "8 sided", "8 sides" } },
                { "10", new List<string> { "ten", "10 sided", "10 sides" } },
                { "12", new List<string> { "twelve", "12 sided", "12 sides" } },
                { "20", new List<string> { "twenty", "20 sided", "20 sides" } }
            };

            var promptOptions = new PromptOptions<string>(
                Resources.ChooseSides,
                choices: choices,
                descriptions: descriptions,
                speak: SSMLHelper.Speak(Utils.RandomPick(Resources.ChooseSidesSSML))); // spoken prompt

            PromptDialog.Choice(context, this.DiceChoiceReceivedAsync, promptOptions);
        }

        private async Task DiceChoiceReceivedAsync(IDialogContext context, IAwaitable<string> result)
        {
            GameData game;
            if (context.UserData.TryGetValue<GameData>(Utils.GameDataKey, out game))
            {
                int sides;
                if (int.TryParse(await result, out sides))
                {
                    game.Sides = sides;
                    context.UserData.SetValue<GameData>(Utils.GameDataKey, game);
                }

                var promptText = string.Format(Resources.ChooseCount, sides);

                var promptOption = new PromptOptions<long>(promptText, choices: null, speak: SSMLHelper.Speak(Utils.RandomPick(Resources.ChooseCountSSML)));

                var prompt = new PromptDialog.PromptInt64(promptOption);
                context.Call<long>(prompt, this.DiceNumberReceivedAsync);
            }
        }

        private async Task DiceNumberReceivedAsync(IDialogContext context, IAwaitable<long> result)
        {
            GameData game;
            if (context.UserData.TryGetValue<GameData>(Utils.GameDataKey, out game))
            {
                game.Count = await result;
                context.UserData.SetValue<GameData>(Utils.GameDataKey, game);
            }

            context.Done(game);
        }
    }
```

`PlayGameDialog` 會藉由在 `HeroCard` 中顯示結果、並使用 `Speak` 方法建置要說出的訊息，來呈現這兩個結果。

```cs
   [Serializable]
    public class PlayGameDialog : IDialog<object>
    {
        private const string RollAgainOptionValue = "roll again";

        private const string NewGameOptionValue = "new game";

        private GameData gameData;

        public PlayGameDialog(GameData gameData)
        {
            this.gameData = gameData;
        }

        public async Task StartAsync(IDialogContext context)
        {
            if (this.gameData == null)
            {
                if (!context.UserData.TryGetValue<GameData>(Utils.GameDataKey, out this.gameData))
                {
                    // User started session with "roll again" so let's just send them to
                    // the 'CreateGameDialog'
                    context.Done<object>(null);
                }
            }

            int total = 0;
            var randomGenerator = new Random();
            var rolls = new List<int>();

            // Generate Rolls
            for (int i = 0; i < this.gameData.Count; i++)
            {
                var roll = randomGenerator.Next(1, this.gameData.Sides);
                total += roll;
                rolls.Add(roll);
            }

            // Format rolls results
            var result = string.Join(" . ", rolls.ToArray());
            bool multiLine = rolls.Count > 5;

            var card = new HeroCard()
            {
                Subtitle = string.Format(
                    this.gameData.Count > 1 ? Resources.CardSubtitlePlural : Resources.CardSubtitleSingular,
                    this.gameData.Count,
                    this.gameData.Sides),
                Buttons = new List<CardAction>()
                {
                    new CardAction(ActionTypes.ImBack, "Roll Again", value: RollAgainOptionValue),
                    new CardAction(ActionTypes.ImBack, "New Game", value: NewGameOptionValue)
                }
            };

            if (multiLine)
            {
                card.Text = result;
            }
            else
            {
                card.Title = result;
            }

            var message = context.MakeMessage();
            message.Attachments = new List<Attachment>()
            {
                card.ToAttachment()
            };

            // Determine bots reaction for speech purposes
            string reaction = "normal";

            var min = this.gameData.Count;
            var max = this.gameData.Count * this.gameData.Sides;
            var score = total / max;
            if (score == 1)
            {
                reaction = "Best";
            }
            else if (score == 0)
            {
                reaction = "Worst";
            }
            else if (score <= 0.3)
            {
                reaction = "Bad";
            }
            else if (score >= 0.8)
            {
                reaction = "Good";
            }

            // Check for special craps rolls
            if (this.gameData.Type == "Craps")
            {
                switch (total)
                {
                    case 2:
                    case 3:
                    case 12:
                        reaction = "CrapsLose";
                        break;
                    case 7:
                        reaction = "CrapsSeven";
                        break;
                    case 11:
                        reaction = "CrapsEleven";
                        break;
                    default:
                        reaction = "CrapsRetry";
                        break;
                }
            }

            // Build up spoken response
            var spoken = string.Empty;
            if (this.gameData.Turns == 0)
            {
                spoken += Utils.RandomPick(Resources.ResourceManager.GetString($"Start{this.gameData.Type}GameSSML"));
            }

            spoken += Utils.RandomPick(Resources.ResourceManager.GetString($"{reaction}RollReactionSSML"));

            message.Speak = SSMLHelper.Speak(spoken);

            // Increment number of turns and store game to roll again
            this.gameData.Turns++;
            context.UserData.SetValue<GameData>(Utils.GameDataKey, this.gameData);

            // Send card and bots reaction to user.
            message.InputHint = InputHints.AcceptingInput;
            await context.PostAsync(message);

            context.Done<object>(null);
        }
    }
```
## <a name="next-steps"></a>後續步驟

如果您的 Bot 是在本機執行或部署在雲端中，可以從 Cortana 叫用。 請參閱[測試 Cortana 技能](../bot-service-debug-cortana-skill.md)，以了解試用 Cortana 技能所需的步驟。


## <a name="additional-resources"></a>其他資源
* [Cortana 技能套件][CortanaGetStarted]
* [將語音新增至訊息](bot-builder-dotnet-text-to-speech.md)
* [SSML 參考][SSMLRef]
* [Cortana 的語音設計最佳做法][VoiceDesign]
* [Cortana 的卡片設計最佳做法][CardDesign]
* [Cortana 開發人員中心][CortanaDevCenter]
* [Cortana 的測試和偵錯最佳做法][Cortana-TestBestPractice]
* <a href="/dotnet/api/?view=botbuilder-3.11.0" target="_blank">適用於 .NET 的 Bot Framework SDK 參考</a>

[CortanaGetStarted]: /cortana/getstarted
[BFPortal]: https://dev.botframework.com/

[SSMLRef]: https://aka.ms/cortana-ssml
[CortanaDevCenter]: https://developer.microsoft.com/en-us/cortana

[CortanaSpecificEntities]: https://aka.ms/lgvcto
[CortanaAuth]: https://aka.ms/vsdqcj

[InvocationNameGuidelines]: https://aka.ms/cortana-invocation-guidelines  
[VoiceDesign]: https://aka.ms/cortana-design-voice
[CardDesign]: https://aka.ms/cortana-design-card
[Cortana-Debug]: https://aka.ms/cortana-enable-debug
[Cortana-Publish]: https://aka.ms/cortana-publish


[CortanaChannel]: https://aka.ms/bot-cortana-channel
[Cortana-TestBestPractice]: https://aka.ms/cortana-test-best-practice

[heroCard]: /dotnet/api/microsoft.bot.connector.herocard

[thumbnailCard]: /dotnet/api/microsoft.bot.connector.thumbnailcard 

[receiptCard]: /dotnet/api/microsoft.bot.connector.receiptcard 

[signinCard]: /dotnet/api/microsoft.bot.connector.signincard 


