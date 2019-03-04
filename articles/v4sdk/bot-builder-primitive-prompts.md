---
title: 建立您自己的提示，以收集使用者輸入 | Microsoft Docs
description: 了解如何在 Bot Framework SDK 中，使用基本提示來管理交談流程。
keywords: 交談流程, 提示, 交談狀態, 使用者狀態, 自訂提示
author: JonathanFingold
ms.author: v-jofing
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 02/19/2019
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 4b18cc5d32d04b69fa349d22058b51fcec0e12d7
ms.sourcegitcommit: 05ddade244874b7d6e2fc91745131b99cc58b0d6
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 02/21/2019
ms.locfileid: "56591069"
---
# <a name="create-your-own-prompts-to-gather-user-input"></a>建立您自己的提示，以收集使用者輸入

[!INCLUDE [pre-release-label](../includes/pre-release-label.md)]

Bot 和使用者之間的對話，通常牽涉到要求 (提示) 使用者輸入資訊、剖析使用者的回應，然後根據該資訊行動。

Bot 應該追蹤交談的內容，以便管理其行為，以及記住先前問題的答案。 Bot 的「狀態」是其所追蹤的資訊，以便適當地回應內送訊息。

## <a name="prerequisites"></a>必要條件

- 本文中的程式碼是以**提示使用者輸入**範例為基礎。 您需要採用 [C#](https://aka.ms/cs-primitive-prompt-sample) 或 [JS](https://aka.ms/js-primitive-prompt-sample) 的一份範例。
- [管理狀態](bot-builder-concept-state.md)以及如何[儲存使用者和交談資料](bot-builder-howto-v4-state.md)的知識。
- [Bot Framework 模擬器](https://aka.ms/Emulator-wiki-getting-started)，以在本機測試 Bot。

## <a name="about-the-sample-code"></a>關於範例程式碼

在本文中，我們會詢問使用者一連串的問題，驗證他們的一些答案，以及儲存其輸入。
我們會使用 Bot 的回合處理常式及使用者和交談狀態屬性，來管理交談流程和輸入集合。

1. 定義及設定狀態
1. 使用狀態屬性來引導交談
   1. 更新 Bot 的回合處理常式。
   1. 實作協助程式方法以管理使用者資料的收集。
   1. 實作使用者輸入的驗證方法。

## <a name="define-and-configure-state"></a>定義及設定狀態

我們需要設定 Bot 以追蹤下列資訊：

- 使用者的名稱、年齡和所選日期 (我們會定義於使用者狀態中)。
- 我們剛詢問使用者的內容 (我們會定義於交談狀態中)。

因為我們不打算部署此 Bot，所以我們會將使用者和交談狀態設定為使用「記憶體儲存體」。 接下來，我們會說明組態程式碼的一些重要層面。

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

我們會定義下列類型。

- Bot 將要收集的使用者資訊的 `UserProfile` 類別。
- 用來追蹤我們在交談中所在位置資訊的 `ConversationFlow` 類別。
- 內部 `ConversationFlow.Question` 列舉可用於追蹤我們在交談中的位置。
- 在其中組合狀態管理資訊的 `CustomPromptBotAccessors` 類別。

Bot 存取子類別包含狀態管理和狀態屬性存取子物件，可透過在 ASP.NET Core 中插入相依性傳遞至 Bot。 在 Bot 中，我們會記錄您在 Bot 建立每個回合時收到的狀態屬性存取子資訊。

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

我們會建立狀態管理物件，並且在建立 Bot 時傳入這些物件。
在 Bot 中，我們會定義狀態屬性的識別碼，以便追蹤我們在交談中的位置，然後記錄狀態管理物件，並且在 Bot 的建構函式中建立狀態屬性存取子。

---

## <a name="use-state-properties-to-direct-the-conversation"></a>使用狀態屬性來引導交談

設定好狀態屬性後，我們即可在 Bot 中使用。

- 定義[回合處理常式](#the-bots-turn-handler)，以存取狀態及呼叫協助程式方法。
- 實作[協助程式方法](#filling-out-the-user-profile)，以管理使用者設定檔的收集。
- 實作[驗證方法](#parse-and-validate-input)以剖析及驗證使用者輸入。

### <a name="the-bots-turn-handler"></a>Bot 的回合處理常式

我們可使用狀態屬性存取子，從回合內容中取得狀態屬性。
如果需要填寫使用者設定檔，我們會呼叫協助程式方法，然後儲存所有狀態變更。

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

在 **CustomPromptBot.cs** 中，取得狀態屬性並呼叫 helper 方法。 (請注意，`_accessors` 執行個體屬性會設定在 Bot 的建構函式中)。

```csharp
public async Task OnTurnAsync(ITurnContext turnContext, CancellationToken cancellationToken = default(CancellationToken))
{
   if (turnContext.Activity.Type == ActivityTypes.Message)
    {
        // Get the state properties from the turn context.
        ConversationFlow flow = await _accessors.ConversationFlowAccessor.GetAsync(turnContext, () => new ConversationFlow());
        UserProfile profile = await _accessors.UserProfileAccessor.GetAsync(turnContext, () => new UserProfile());

        await FillOutUserProfileAsync(flow, profile, turnContext);

        // Update state and save changes.
        await _accessors.ConversationFlowAccessor.SetAsync(turnContext, flow);
        await _accessors.ConversationState.SaveChangesAsync(turnContext);

        await _accessors.UserProfileAccessor.SetAsync(turnContext, profile);
        await _accessors.UserState.SaveChangesAsync(turnContext);
    }
}
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

在 **bot.js** 中，取得狀態屬性並呼叫 helper 方法。 (請注意，`conversationFlow` 執行個體屬性會設定在 Bot 的建構函式中)。

```javascript
// The bot's turn handler.
async onTurn(turnContext) {
    // This bot listens for message activities.
    if (turnContext.activity.type === ActivityTypes.Message) {
        // Get the state properties from the turn context.
        const flow = await this.conversationFlow.get(turnContext, { lastQuestionAsked: question.none });
        const profile = await this.userProfile.get(turnContext, {});

        await MyBot.fillOutUserProfile(flow, profile, turnContext);

        // Update state and save changes.
        await this.conversationFlow.set(turnContext, flow);
        await this.conversationState.saveChanges(turnContext);

        await this.userProfile.set(turnContext, profile);
        await this.userState.saveChanges(turnContext);
    }
}
```

---

### <a name="filling-out-the-user-profile"></a>填寫使用者設定檔

我們會從收集資訊著手。 每個 Bot 都會提供類似的介面。

- 傳回值會指出輸入是否為此問題的有效答案。
- 如果通過驗證，則會產生要儲存的已剖析和正規化值。
- 如果驗證失敗，則會產生一則訊息，讓 Bot 再次詢問資訊。

 在[下一節](#parse-and-validate-input)中，我們會定義協助程式方法來剖析及驗證使用者輸入。

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

```csharp
private static async Task FillOutUserProfileAsync(ConversationFlow flow, UserProfile profile, ITurnContext turnContext)
{
    string input = turnContext.Activity.Text?.Trim();
    string message;
    switch (flow.LastQuestionAsked)
    {
        case ConversationFlow.Question.None:
            await turnContext.SendActivityAsync("Let's get started. What is your name?");
            flow.LastQuestionAsked = ConversationFlow.Question.Name;
            break;
        case ConversationFlow.Question.Name:
            if (ValidateName(input, out string name, out message))
            {
                profile.Name = name;
                await turnContext.SendActivityAsync($"Hi {profile.Name}.");
                await turnContext.SendActivityAsync("How old are you?");
                flow.LastQuestionAsked = ConversationFlow.Question.Age;
                break;
            }
            else
            {
                await turnContext.SendActivityAsync(message ?? "I'm sorry, I didn't understand that.");
                break;
            }
        case ConversationFlow.Question.Age:
            if (ValidateAge(input, out int age, out message))
            {
                profile.Age = age;
                await turnContext.SendActivityAsync($"I have your age as {profile.Age}.");
                await turnContext.SendActivityAsync("When is your flight?");
                flow.LastQuestionAsked = ConversationFlow.Question.Date;
                break;
            }
            else
            {
                await turnContext.SendActivityAsync(message ?? "I'm sorry, I didn't understand that.");
                break;
            }
        case ConversationFlow.Question.Date:
            if (ValidateDate(input, out string date, out message))
            {
                profile.Date = date;
                await turnContext.SendActivityAsync($"Your cab ride to the airport is scheduled for {profile.Date}.");
                await turnContext.SendActivityAsync($"Thanks for completing the booking {profile.Name}.");
                await turnContext.SendActivityAsync($"Type anything to run the bot again.");
                flow.LastQuestionAsked = ConversationFlow.Question.None;
                profile = new UserProfile();
                break;
            }
            else
            {
                await turnContext.SendActivityAsync(message ?? "I'm sorry, I didn't understand that.");
                break;
            }
    }
}


```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

```javascript
// Manages the conversation flow for filling out the user's profile.
static async fillOutUserProfile(flow, profile, turnContext) {
    const input = turnContext.activity.text;
    let result;
    switch (flow.lastQuestionAsked) {
        // If we're just starting off, we haven't asked the user for any information yet.
        // Ask the user for their name and update the conversation flag.
        case question.none:
            await turnContext.sendActivity("Let's get started. What is your name?");
            flow.lastQuestionAsked = question.name;
            break;

        // If we last asked for their name, record their response, confirm that we got it.
        // Ask them for their age and update the conversation flag.
        case question.name:
            result = this.validateName(input);
            if (result.success) {
                profile.name = result.name;
                await turnContext.sendActivity(`I have your name as ${profile.name}.`);
                await turnContext.sendActivity('How old are you?');
                flow.lastQuestionAsked = question.age;
                break;
            } else {
                // If we couldn't interpret their input, ask them for it again.
                // Don't update the conversation flag, so that we repeat this step.
                await turnContext.sendActivity(
                    result.message || "I'm sorry, I didn't understand that.");
                break;
            }

        // If we last asked for their age, record their response, confirm that we got it.
        // Ask them for their date preference and update the conversation flag.
        case question.age:
            result = this.validateAge(input);
            if (result.success) {
                profile.age = result.age;
                await turnContext.sendActivity(`I have your age as ${profile.age}.`);
                await turnContext.sendActivity('When is your flight?');
                flow.lastQuestionAsked = question.date;
                break;
            } else {
                // If we couldn't interpret their input, ask them for it again.
                // Don't update the conversation flag, so that we repeat this step.
                await turnContext.sendActivity(
                    result.message || "I'm sorry, I didn't understand that.");
                break;
            }

        // If we last asked for a date, record their response, confirm that we got it,
        // let them know the process is complete, and update the conversation flag.
        case question.date:
            result = this.validateDate(input);
            if (result.success) {
                profile.date = result.date;
                await turnContext.sendActivity(`Your cab ride to the airport is scheduled for ${profile.date}.`);
                await turnContext.sendActivity(`Thanks for completing the booking ${profile.name}.`);
                await turnContext.sendActivity('Type anything to run the bot again.');
                flow.lastQuestionAsked = question.none;
                profile = {};
                break;
            } else {
                // If we couldn't interpret their input, ask them for it again.
                // Don't update the conversation flag, so that we repeat this step.
                await turnContext.sendActivity(
                    result.message || "I'm sorry, I didn't understand that.");
                break;
            }
    }
}
```

---

### <a name="parse-and-validate-input"></a>剖析及驗證輸入

我們將使用下列準則來驗證輸入。

- **名稱**必須是非空白字串。 我們的正規做法是修剪空白字元。
- **年齡**必須介於 18 與 120 之間。 我們的正規做法是傳回一個整數。
- **日期**必須是至少未來一小時的任何日期或時間。
  我們的正規做法是只傳回所剖析輸入的日期部分。

> [!NOTE]
> 對於年齡和日期輸入，我們會使用 [Microsoft/Recognizers-Text](https://github.com/Microsoft/Recognizers-Text/) 程式庫來執行初始剖析。
> 我們雖然提供範例程式碼，但不會說明文字辨識器程式庫的運作方式，而這只是剖析輸入的方法之一。
> 如需這些程式庫的詳細資訊，請參閱存放庫的**讀我**。

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

將下列驗證方法新增到您的 Bot。

```csharp
private static bool ValidateName(string input, out string name, out string message)
{
    name = null;
    message = null;

    if (string.IsNullOrWhiteSpace(input))
    {
        message = "Please enter a name that contains at least one character.";
    }
    else
    {
        name = input.Trim();
    }

    return message is null;
}

private static bool ValidateAge(string input, out int age, out string message)
{
    age = 0;
    message = null;

    // Try to recognize the input as a number. This works for responses such as "twelve" as well as "12".
    try
    {
        // Attempt to convert the Recognizer result to an integer. This works for "a dozen", "twelve", "12", and so on.
        // The recognizer returns a list of potential recognition results, if any.
        List<ModelResult> results = NumberRecognizer.RecognizeNumber(input, Culture.English);
        foreach (var result in results)
        {
            // result.Resolution is a dictionary, where the "value" entry contains the processed string.
            if (result.Resolution.TryGetValue("value", out object value))
            {
                age = Convert.ToInt32(value);
                if (age >= 18 && age <= 120)
                {
                    return true;
                }
            }
        }

        message = "Please enter an age between 18 and 120.";
    }
    catch
    {
        message = "I'm sorry, I could not interpret that as an age. Please enter an age between 18 and 120.";
    }

    return message is null;
}

private static bool ValidateDate(string input, out string date, out string message)
{
    date = null;
    message = null;

    // Try to recognize the input as a date-time. This works for responses such as "11/14/2018", "9pm", "tomorrow", "Sunday at 5pm", and so on.
    // The recognizer returns a list of potential recognition results, if any.
    try
    {
        List<ModelResult> results = DateTimeRecognizer.RecognizeDateTime(input, Culture.English);

        // Check whether any of the recognized date-times are appropriate,
        // and if so, return the first appropriate date-time. We're checking for a value at least an hour in the future.
        DateTime earliest = DateTime.Now.AddHours(1.0);
        foreach (ModelResult result in results)
        {
            // result.Resolution is a dictionary, where the "values" entry contains the processed input.
            var resolutions = result.Resolution["values"] as List<Dictionary<string, string>>;
            foreach (var resolution in resolutions)
            {
                // The processed input contains a "value" entry if it is a date-time value, or "start" and
                // "end" entries if it is a date-time range.
                if (resolution.TryGetValue("value", out string dateString)
                    || resolution.TryGetValue("start", out dateString))
                {
                    if (DateTime.TryParse(dateString, out var candidate)
                        && earliest < candidate)
                    {
                        date = candidate.ToShortDateString();
                        return true;
                    }
                }
            }
        }

        message = "I'm sorry, please enter a date at least an hour out.";
    }
    catch
    {
        message = "I'm sorry, I could not interpret that as an appropriate date. Please enter a date at least an hour out.";
    }

    return false;
}
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

將下列驗證方法新增到您的 Bot。

```javascript
// Validates name input. Returns whether validation succeeded and either the parsed and normalized
// value or a message the bot can use to ask the user again.
static validateName(input) {
    const name = input && input.trim();
    return name != undefined
        ? { success: true, name: name }
        : { success: false, message: 'Please enter a name that contains at least one character.' };
};

// Validates age input. Returns whether validation succeeded and either the parsed and normalized
// value or a message the bot can use to ask the user again.
static validateAge(input) {

    // Try to recognize the input as a number. This works for responses such as "twelve" as well as "12".
    try {
        // Attempt to convert the Recognizer result to an integer. This works for "a dozen", "twelve", "12", and so on.
        // The recognizer returns a list of potential recognition results, if any.
        const results = Recognizers.recognizeNumber(input, Recognizers.Culture.English);
        let output;
        results.forEach(function (result) {
            // result.resolution is a dictionary, where the "value" entry contains the processed string.
            const value = result.resolution['value'];
            if (value) {
                const age = parseInt(value);
                if (!isNaN(age) && age >= 18 && age <= 120) {
                    output = { success: true, age: age };
                    return;
                }
            }
        });
        return output || { success: false, message: 'Please enter an age between 18 and 120.' };
    } catch (error) {
        return {
            success: false,
            message: "I'm sorry, I could not interpret that as an age. Please enter an age between 18 and 120."
        };
    }
}

// Validates date input. Returns whether validation succeeded and either the parsed and normalized
// value or a message the bot can use to ask the user again.
static validateDate(input) {
    // Try to recognize the input as a date-time. This works for responses such as "11/14/2018", "today at 9pm", "tomorrow", "Sunday at 5pm", and so on.
    // The recognizer returns a list of potential recognition results, if any.
    try {
        const results = Recognizers.recognizeDateTime(input, Recognizers.Culture.English);
        const now = new Date();
        const earliest = now.getTime() + (60 * 60 * 1000);
        let output;
        results.forEach(function (result) {
            // result.resolution is a dictionary, where the "values" entry contains the processed input.
            result.resolution['values'].forEach(function (resolution) {
                // The processed input contains a "value" entry if it is a date-time value, or "start" and
                // "end" entries if it is a date-time range.
                const datevalue = resolution['value'] || resolution['start'];
                // If only time is given, assume it's for today.
                const datetime = resolution['type'] === 'time'
                    ? new Date(`${now.toLocaleDateString()} ${datevalue}`)
                    : new Date(datevalue);
                if (datetime && earliest < datetime.getTime()) {
                    output = { success: true, date: datetime.toLocaleDateString() };
                    return;
                }
            });
        });
        return output || { success: false, message: "I'm sorry, please enter a date at least an hour out." };
    } catch (error) {
        return {
            success: false,
            message: "I'm sorry, I could not interpret that as an appropriate date. Please enter a date at least an hour out."
        };
    }
}
```

---

## <a name="test-the-bot-locally"></a>在本機測試 Bot
1. 在您的電腦本機執行範例。 如需相關指示，請參閱 [C#](https://aka.ms/cs-primitive-prompt-sample) 或 [JS](https://aka.ms/js-primitive-prompt-sample) 範例的讀我檔案。
1. 如下所示，使用模擬器進行測試。

![primitive-prompts](media/primitive-prompts.png)

## <a name="additional-resources"></a>其他資源

[Dialogs 程式庫](bot-builder-concept-dialog.md)會提供一些類別，以將管理交談的許多層面自動化。 

## <a name="next-step"></a>後續步驟

> [!div class="nextstepaction"]
> [實作循序對話流程](bot-builder-dialog-manage-conversation-flow.md)
