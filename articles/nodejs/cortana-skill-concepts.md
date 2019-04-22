---
title: 使用 Node.js 建置 Cortana 技能 | Microsoft Docs
description: 了解在適用於 Node.js 的 Bot Framework SDK 中建置 Cortana 技能的核心概念。
keywords: Bot Framework, Cortana 技能, 語音, Node.js, Bot 建立器, SDK, 重要概念, 核心概念
author: DeniseMak
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 02/10/2019
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: 5de773f6f8f4d46c0c1fe880588f2530c3c68f56
ms.sourcegitcommit: 721bb09f10524b0cb3961d7131966f57501734b8
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 04/18/2019
ms.locfileid: "59746062"
---
# <a name="key-concepts-for-building-a-bot-for-cortana-skills-using-nodejs"></a>使用 Node.js 針對 Cortana 技能建置 Bot 的重要概念
 
[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]

> [!NOTE]
> 本文是初步內容，將會進行更新。

本文介紹在適用於 Node.js 的 Bot Framework SDK 中建置 Cortana 技能的重要概念。 

## <a name="what-is-a-cortana-skill"></a>什麼是 Cortana 技能？
Cortana 技能是您可以使用 Cortana 用戶端叫用的 Bot，就像內建於 Windows 10 的 Bot。 使用者透過說出一些與 Bot 建立關聯的關鍵字或片語來啟動 Bot。 您可以使用 Bot Framework 入口網站，設定哪些關鍵字可用來啟動 Bot。 

Cortana 被認為是具有語音功能的通道，除了文字交談之外，還可以傳送和接收語音訊息。 發佈為 Cortana 技能的 Bot 應該是專為語音及文字而設計的。 Bot Framework 提供方法來指定語音合成標記語言 (SSML)，以定義 Bot 所傳送的語音訊息。

## <a name="acknowledge-user-utterances"></a>確認使用者語句 

<!-- Establishing conversational understanding -->
<!-- Placeholder: In this section, describe how you have to write your speech to sound natural -->


當您建立具有語音功能的 Bot 時，應該嘗試在交談中建立共同基準點並相互了解。 Bot 應該表示已聽見和了解使用者表達內容，藉以「承接」使用者的語句。

如果系統無法承接其語句，使用者會感到困惑。 例如，當 Bot 詢問「接下來要做什麼？」時，下列交談可能令人有點困惑：

> **Cortana**：您想要檢閱更多設定檔嗎？  
> **使用者**：沒有。  
> **Cortana**：後續步驟

如果 Bot 新增「好的」作為確認，這對使用者來說更加友善：

> **Cortana**：您想要檢閱更多設定檔嗎？  
> **使用者**：沒有。  
> **Cortana**：**好的**，接下來呢？

從最弱到最強的承接程度：

1. 持續注意
2. 後續的相關參與
3. 確認：最少回應或接續詞：「是啊」，「嗯嗯」、「好的」、「太棒了」
4. 示範：透過重新制定、完成來表示了解。
5. 顯示︰重複所有或部分。

### <a name="acknowledgement-and-next-relevant-contribution"></a>確認和後續的相關參與

> **使用者**：我需要在 5 月旅行。  
> **Cortana**：**好的**。 您想要在 5 月的哪一天旅行？  
> **使用者**：嗯，我必須 12 日到 15 日在那裡嗎？  
> **Cortana**：**好的**。 您要飛往哪個城市？  

### <a name="grounding-by-demonstration"></a>示範所顯示的承接

> **使用者**：我需要在 5 月旅行。  
> **Cortana**：**那麼**，您想要在 5 月的哪一天旅行？  
> **使用者**：好的，我必須 12 日到 15 日在那裡嗎？  
> **Cortana**：**那麼**您要飛往哪個城市？  
    
### <a name="closure"></a>終止

執行動作的 Bot 應呈現成功執作的辨識項。 表示失敗或了解也很重要。 

* 非語音終止：如果您按下電梯按鈕，該指示燈會亮起。  
這是兩步驟的程序：
    * 展示 (當您按下按鈕)
    * 接受 (當按鈕亮起)

## <a name="differences-in-content-presentation"></a>內容呈現的差異
請記住，Cortana 在各種裝置上受支援，其中只有一些有螢幕。 在設計啟用語音的 Bot 時，其中一項需要考慮的是，口說對話通常不會與 Bot 顯示的文字訊息相同。
<!-- If there are differences in what the bot will say, in the text vs the speak fields of a prompt or in a waterfall, for example, discuss them here.

## Speech

You bot uses the **session.say** method to speak to the user. The speak method has three overloads:
* If you pass only one parameter to **session.say**, it can be a text parameter.
* If you pass two parameters to **session.say**, it can take text and SSML.
* If you pass three parameters, the third parameter takes an options structure that specifies all the options you can pass to build an **IMessage** object.

```javascript
var bot = new builder.UniversalBot(connector, function (session) {
    session.say("Hello... I'm a decision making bot.'.", 
        ssml.speak("Hello. I can help you answer all of life's tough questions."));
    session.replaceDialog('rootMenu');
});

```
## Speech in messages

The **IMessage** object provides a **speak** property for SSML. It can be used to play a .wav file.

The **inputHint** property helps indicate to Cortana whether your bot is expecting input. If you're using a built-in prompt, this value is automatically set to the default of **expectingInput**.

The **inputHint** property can take the following values: 
* **expectingInput**: Indicates that the bot is actively expecting a response from the user. Cortana listens for the user to speak into the microphone.
* **acceptingInput**: Indicates that the bot is passively ready for input but is not waiting on a response. Cortana accepts input from the user if the user holds down the microphone button.
* **ignoringInput**: Cortana is ignoring input. Your bot may send this hint if it is actively processing a request and will ignore input from users until the request is complete.

Prompts must use the `speak:` option.

```javascript
        builder.Prompts.choice(session, "Decision Options", choices, {
            listStyle: builder.ListStyle.button,
            speak: ssml.speak("How would you like me to decide?")
        });
```

Prompts.number has *ordinal support*, meaning that you can say "the last", "the first", "the next-to-last" to choose an item in a list.

## Using synonyms

<!-- Axl Rose example -->
```javascript   
         var choices = [
            { 
                value: 'flipCoinDialog',
                action: { title: "Flip A Coin" },
                synonyms: 'toss coin|flip quarter|toss quarter'
            },
            {
                value: 'rollDiceDialog',
                action: { title: "Roll Dice" },
                synonyms: 'roll die|shoot dice|shoot die'
            },
            {
                value: 'magicBallDialog',
                action: { title: "Magic 8-Ball" },
                synonyms: 'shake ball'
            },
            {
                value: 'quit',
                action: { title: "Quit" },
                synonyms: 'exit|stop|end'
            }
        ];
        builder.Prompts.choice(session, "Decision Options", choices, {
            listStyle: builder.ListStyle.button,
            speak: ssml.speak("How would you like me to decide?")
        });
```

## <a name="configuring-your-bot"></a>設定您的 Bot

## <a name="prompts"></a>提示

## <a name="additional-resources"></a>其他資源

Cortana 文件：[Cortana 技能文件](/cortana/skills/)

Cortana SSML 參考：[語音合成標記語言 (SSML) 參考](/cortana/skills/speech-synthesis-markup-language)
