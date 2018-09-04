---
title: 使用 Dialogs 程式庫提示使用者輸入 | Microsoft Docs
description: 了解如何在適用於 Node.js 的 Bot 建立器 SDK 中使用 Dailogs 程式庫提示使用者輸入。
keywords: 提示, 對話, AttachmentPrompt, ChoicePrompt, ConfirmPrompt, DatetimePrompt, NumberPrompt, TextPrompt, 重新提示, 驗證
author: v-ducvo
ms.author: v-ducvo
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 4/10/2018
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 0b238ed510fd1d6fda82734af373f344b0dc28e3
ms.sourcegitcommit: 2dc75701b169d822c9499e393439161bc87639d2
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/24/2018
ms.locfileid: "42905362"
---
# <a name="prompt-users-for-input-using-the-dialogs-library"></a>使用 Dialogs 程式庫提示使用者輸入

[!INCLUDE [pre-release-label](../includes/pre-release-label.md)]

Bot 常會透過對使用者的提問來收集其資訊。 您可以使用[回合內容](bot-builder-concept-activity-processing.md#turn-context)物件的「傳送活動」方法直接將標準訊息傳送給使用者，以要求字串輸入；不過，Bot 建立器 SDK 提供的 **Dialogs** 程式庫則可讓您要求不同類型的資訊。 本主題將詳細說明如何使用**提示**來要求使用者輸入。

本文說明如何在對話內使用提示。 如需在一般情況下使用對話方塊的相關資訊，請參閱[使用對話方塊來管理簡單對話流程](bot-builder-dialog-manage-conversation-flow.md)。

## <a name="prompt-types"></a>提示類型

對話庫提供多種不同類型的提示，分別會要求不同類型的回應。

| Prompt | 說明 |
|:----|:----|
| **AttachmentPrompt** | 提示使用者提供附件，例如文件或影像。 |
| **ChoicePrompt** | 提示使用者從選項集中進行選擇。 |
| **ConfirmPrompt** | 提示使用者確認其動作。 |
| **DatetimePrompt** | 提示使用者提供日期時間。 使用者可以使用「明天晚上 8 點」或「週五早上 10 點」之類的自然語言來回應。 Bot Framework SDK 會使用 LUIS `builtin.datetimeV2` 預先建置的實體。 如需詳細資訊，請參閱 [builtin.datetimev2](https://docs.microsoft.com/azure/cognitive-services/luis/luis-reference-prebuilt-entities#builtindatetimev2)。 |
| **NumberPrompt** | 提示使用者提供數字。 使用者可以使用 "10" 或「十」來回應。 舉例來說，如果回應是「十」，則提示會將回應轉換為數字，並傳回 `10` 作為結果。 |
| **TextPrompt** | 提示使用者提供文字字串。 |

## <a name="add-references-to-prompt-library"></a>將參考新增至提示庫

將**對話**套件新增至 Bot，以取得**對話**庫，。 我們將在[使用對話方塊來管理簡單對話流程](bot-builder-dialog-manage-conversation-flow.md)中討論對話方塊，但會將對話方塊用於我們的提示中。

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

從 NuGet 安裝 **Microsoft.Bot.Builder.Dialogs** 套件。

然後，參考您 Bot 程式碼中的程式庫。

```cs
using Microsoft.Bot.Builder.Dialogs;
```

您可以將對話定義為類別，或在 Bot 程式碼檔案中內嵌為屬性。

本文中的程式碼，是針對定義為類別的對話而撰寫。
下列範例假設您會將程式碼，新增至對話的建構函式。

對話的主要流程是其步驟集合，且需要有指定的識別碼。 您的 Bot 會使用此識別碼來擷取對話，因此最好將其公開為常數。

```cs
public class MyDialog : DialogSet
{
    public const string Name = "mainDialog";

    public MyDialog()
    {
        // Define your dialog's prompts and steps here.
    }
}
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

從 NPM 安裝對話方塊套件：

```cmd
npm install --save botbuilder-dialogs@preview
```

若要在 Bot 中使用**對話**，請將其包含在 Bot 程式碼中。

在 app.js 檔案中，新增下列項目。

```javascript
const {DialogSet} = require("botbuilder-dialogs");
const dialogs = new DialogSet();
```

---

## <a name="prompt-the-user"></a>提示使用者

若要提示使用者輸入，您可以將提示新增至對話方塊中。 例如，您可以定義 **TextPrompt** 類型的提示，並為其指定 **textPrompt** 的對話方塊識別碼：

在新增提示對話方塊後，您可以在簡單的兩步驟瀑布圖對話方塊中使用該提示對話，或在多步驟瀑布圖中同時使用多個提示。 *瀑布圖*對話方塊是定義一系列步驟的簡易方式。 如需詳細資訊，請參閱[使用對話方塊來管理簡單對話流程](bot-builder-dialog-manage-conversation-flow.md)的[使用對話方塊](bot-builder-dialog-manage-conversation-flow.md#using-dialogs-to-guide-the-user-through-steps)一節。

在第一回合中，對話方塊會提示使用者提供其名稱，而在第二回合中，對話方塊則會將使用者輸入處理為提示的答案。

例如，下列對話方塊會提示使用者提供其名稱，然後以該名稱向他們打招呼：

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

您在對話方塊中使用的每個提示也會有指定的名稱，供對話方塊或 Bot 用來存取提示。 在此處的所有範例中，我們都會將提示識別碼公開為常數。

對於對話方塊內容**提示**或**結束**方法所做的呼叫，會指定對話方塊步驟結束的地方。 若沒有這些陳述式，對話方塊將無法正常執行。

```csharp
/// <summary>Defines a simple greeting dialog that asks for the user's name.</summary>
public class MyDialog : DialogSet
{
    /// <summary>The ID of the main dialog in the set.</summary>
    public const string Name = "mainDialog";

    /// <summary>Defines the IDs of the prompts in the set.</summary>
    public struct Inputs
    {
        /// <summary>The ID of the text prompt.</summary>
        public const string Text = "textPrompt";
    }

    /// <summary>Defines the prompts and steps of the dialog.</summary>
    public MyDialog()
    {
        Add(Inputs.Text, new TextPrompt());
        Add(Name, new WaterfallStep[]
        {
            // Each step takes in a dialog context, arguments, and the next delegate.
            async (dc, args, next) =>
            {
                // Prompt for the user's name.
                await dc.Prompt(Inputs.Text, "What is your name?");
            },
            async(dc, args, next) =>
            {
                var user = (string)args["Text"];
                await dc.Context.SendActivity($"Hi {user}!");
                await dc.End();
            }
        });
    }
}
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

```javascript
const {TextPrompt} = require("botbuilder-dialogs");
```

```javascript
// Greet user:
// Ask for the user name and then greet them by name.
dialogs.add('textPrompt', new TextPrompt());
dialogs.add('greetings', [
    async function (dc){
        await dc.prompt('textPrompt', 'What is your name?');
    },
    async function(dc, userName){
        await dc.context.sendActivity(`Hi ${userName}!`);
        await dc.end();
    }
]);
```

---

> [!NOTE]
> 若要啟動對話方塊，請取得對話方塊內容，並使用其_開始_方法。 如需詳細資訊，請參閱[使用對話方塊來管理簡單對話流程](./bot-builder-dialog-manage-conversation-flow.md)。

## <a name="reusable-prompts"></a>可重複使用的提示

您可以重複使用提示，以要求使用同類型提示的不同資訊。 例如，上述範例程式碼定義了文字提示，並將它用來要求使用者提供其名稱。 舉例來說，如果您想要，您也可以使用這個相同的提示來要求使用者提供其他文字字串，例如「您在哪裡工作？」。

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

```cs
/// <summary>Defines a simple greeting dialog that asks for the user's name and place of work.</summary>
public class MyDialog : DialogSet
{
    /// <summary>The ID of the main dialog in the set.</summary>
    public const string Name = "mainDialog";

    /// <summary>Defines the IDs of the prompts in the set.</summary>
    public struct Inputs
    {
        /// <summary>The ID of the text prompt.</summary>
        public const string Text = "textPrompt";
    }

    /// <summary>Defines the prompts and steps of the dialog.</summary>
    public MyDialog()
    {
        Add(Inputs.Text, new TextPrompt());
        Add(Name, new WaterfallStep[]
        {
            async (dc, args, next) =>
            {
                // Prompt for the user's name.
                await dc.Prompt(Inputs.Text, "What is your name?");
            },
            async(dc, args, next) =>
            {
                var user = (string)args["Text"];

                // Ask them where they work.
                await dc.Prompt(Inputs.Text, $"Hi {user}! Where do you work?");
            },
            async(dc, args, next) =>
            {
                var workplace = (string)args["Text"];

                await dc.Context.SendActivity($"{workplace} is a cool place!");
                await dc.End();
            }
        });
    }
}
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

```javascript
// Greet user:
// Ask for the user name and then greet them by name.
// Ask them where they work.
dialogs.add('textPrompt', new botbuilder_dialogs.TextPrompt());
dialogs.add('greetings',[
    async function (dc){
        await dc.prompt('textPrompt', 'What is your name?');
    },
    async function(dc, userName){
        await dc.context.sendActivity(`Hi ${userName}!`);

        // Ask them where they work.
        await dc.prompt('textPrompt', 'Where do you work?');
    },
    async function(dc, workPlace){
        await dc.context.sendActivity(`${workPlace} is a cool place!`);

        await dc.end();
    }
]);
```

---

不過，如果您想要將提示及其所要求的預期值配對，您可以為每個提示指定唯一的 *dialogId*。 對話方塊將會以唯一識別碼新增。 藉由使用不同的識別碼，您也可以建立多個相同類型的**提示**對話方塊。 例如，您可以為上述範例建立兩個 **TextPrompt** 對話方塊：

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

```cs
/// <summary>The ID of the main dialog in the set.</summary>
public const string Name = "mainDialog";

/// <summary>Defines the IDs of the prompts in the set.</summary>
public struct Inputs
{
    /// <summary>The ID of the name prompt.</summary>
    public const string Name = "namePrompt";

    /// <summary>The ID of the work prompt.</summary>
    public const string Work = "workPrompt";
}

/// <summary>Defines the prompts and steps of the dialog.</summary>
public MyDialog()
{
    Add(Inputs.Name, new TextPrompt());
    Add(Inputs.Work, new TextPrompt());
    Add(Name, new WaterfallStep[]
    {
        async (dc, args, next) =>
        {
            // Prompt for the user's name.
            await dc.Prompt(Inputs.Name, "What is your name?");
        },
        async(dc, args, next) =>
        {
            var user = (string)args["Text"];

            // Ask them where they work.
            await dc.Prompt(Inputs.Work, $"Hi {user}! Where do you work?");
        },
        async(dc, args, next) =>
        {
            var workplace = (string)args["Text"];

            await dc.Context.SendActivity($"{workplace} is a cool place!");
            await dc.End();
        }
    });
}
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

```javascript
dialogs.add('namePrompt', new TextPrompt());
dialogs.add('workPlacePrompt', new TextPrompt());
```

---

為了要重複使用程式碼，為所有提示定義單一 `textPrompt` 是可行的，因為這些提示要求以文字字串作為回應。 不過，在您需要驗證提示的輸入時，為對話方塊命名的功能才會真的派上用場。 在此情況下，各個提示可使用 **TextPrompt**，但會分別尋找一組不同的值。 讓我們看看如何使用 `NumberPrompt` 來驗證提示回應。

## <a name="specify-prompt-options"></a>指定提示選項

當您在對話方塊步驟中使用提示時，您也可以提供提示選項，例如重新提示字串。

如果使用者輸入因其格式無法由提示剖析 (例如對數字提示輸入「明天」)，或因輸入不符合驗證準則，而無法滿足提示，則指定重新提示字串將有其效用。

> [!TIP]
> 建立數字提示時，您必須指定它所將使用的輸入文化特性。 數字提示可解譯多種不同的輸入，例如「十二」或「四分之一」，以及 "12" 和 "0.25"。 輸入文化特性有助於提示更正確解譯使用者的輸入。

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

輸入文化特性會定義於個別的文件庫中。

```csharp
using Microsoft.Bot.Builder.Dialogs;
using Microsoft.Recognizers.Text;
```

下列程式碼會將數字提示新增至現有的對話方塊集 **dialogs** 中。

```csharp
dialogs.Add("numberPrompt", new NumberPrompt<int>(Culture.English));
```

在對話方塊步驟中，下列程式碼會提示使用者輸入，並提供重新提示字串以備其輸入無法解譯為數字時使用。

```csharp
await dc.Prompt("numberPrompt", "How many people are in your party?", new PromptOptions()
{
    RetryPromptString = "Sorry, please specify the number of people in your party."
});
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

```javascript
const {NumberPrompt} = require("botbuilder-dialogs");
```

```javascript
await dc.prompt('numberPrompt', 'How many people in your party?', { retryPrompt: `Sorry, please specify the number of people in your party.` })
```

```javascript
dialogs.add('numberPrompt', new NumberPrompt());
```

---

特別的是，選擇提示需要額外的資訊，即使用者可用的選擇清單。

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

此範例使用來自下列命名空間的類型。

```csharp
using Microsoft.Bot.Builder.Core.Extensions;
using Microsoft.Bot.Builder.Dialogs;
using Microsoft.Bot.Builder.Prompts.Choices;
using Microsoft.Bot.Schema;
using Microsoft.Recognizers.Text;
using System.Collections.Generic;
```


當我們使用 **ChoicePrompt** 要求使用者從一組選項中選擇時，我們必須在提示中附上在 **ChoicePromptOptions** 物件內提供的該組選項。 在此，我們使用 **ChoiceFactory** 將選項清單轉換為適當的格式。

我們也使用 **SuggestedActions** 活動作為重新提示，藉以再次向使用者提供輸入選項。


```csharp
/// <summary>Defines a dialog that asks for a choice of color.</summary>
public class MyDialog : DialogSet
{
    /// <summary>The ID of the main dialog in the set.</summary>
    public const string Name = "mainDialog";

    /// <summary>Defines the IDs of the prompts in the set.</summary>
    public struct Inputs
    {
        /// <summary>The ID of the color prompt.</summary>
        public const string Color = "colorPrompt";
    }

    /// <summary>The available colors to choose from.</summary>
    public List<string> Colors = new List<string> { "Green", "Blue" };

    /// <summary>Defines the prompts and steps of the dialog.</summary>
    public MyDialog()
    {
        Add(Inputs.Color, new ChoicePrompt(Culture.English));
        Add(Name, new WaterfallStep[]
        {
            async (dc, args, next) =>
            {
                // Prompt for a color. A choice prompt requires that you specify choice options.
                await dc.Prompt(Inputs.Color, "Please make a choice.", new ChoicePromptOptions()
                {
                    Choices = ChoiceFactory.ToChoices(Colors),
                    RetryPromptActivity =
                        MessageFactory.SuggestedActions(Colors, "Please choose a color.") as Activity
                });
            },
            async(dc, args, next) =>
            {
                var color = (FoundChoice)args["Value"];

                await dc.Context.SendActivity($"You chose {color.Value}.");
                await dc.End();
            }
        });
    }
}
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

```javascript
const {ChoicePrompt} = require("botbuilder-dialogs");
```

```javascript
dialogs.add('choicePrompt', new ChoicePrompt());
```

```javascript
// A choice prompt requires that you specify choice options.
const list = ['green', 'blue'];
await dc.prompt('choicePrompt', 'Please make a choice', list, {retryPrompt: 'Please choose a color.'});
```

---

## <a name="validate-a-prompt-response"></a>驗證提示回應

您可以先驗證提示回應，再將有效值傳回至**瀑布圖**的下一個步驟。 例如，若要驗證 **6** 到 **20** 數字範圍內的 **NumberPrompt**，您可以使用如下的驗證邏輯：

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

```cs
using Microsoft.Bot.Builder.Dialogs;
using Microsoft.Recognizers.Text;
using PromptStatus = Microsoft.Bot.Builder.Prompts.PromptStatus;
```

```cs
/// <summary>Defines a dialog that asks for the number of people in a party.</summary>
public class MyDialog : DialogSet
{
    /// <summary>The ID of the main dialog in the set.</summary>
    public const string Name = "mainDialog";

    /// <summary>Defines the IDs of the prompts in the set.</summary>
    public struct Inputs
    {
        /// <summary>The ID of the party size prompt.</summary>
        public const string Size = "parytySize";
    }

    /// <summary>Defines the prompts and steps of the dialog.</summary>
    public MyDialog()
    {
        // Include a validation function for the party size prompt.
        Add(Inputs.Size, new NumberPrompt<int>(Culture.English, async (context, result) =>
        {
            if (result.Value < 6 || result.Value > 20)
            {
                result.Status = PromptStatus.OutOfRange;
            }
        }));
        Add(Name, new WaterfallStep[]
        {
            async (dc, args, next) =>
            {
                // Prompt for the party size.
                await dc.Prompt(Inputs.Size, "How many people are in your party?", new PromptOptions()
                {
                    RetryPromptString = "Please specify party size between 6 and 20."
                });
            },
            async(dc, args, next) =>
            {
                var size = (int)args["Value"];

                await dc.Context.SendActivity($"Okay, {size} people!");
                await dc.End();
            }
        });
    }
}
```

驗證可以也封裝在其本身的私用方法內，並以該方式新增。

```cs
/// <summary>Validates input for the partySize prompt.</summary>
/// <param name="context">The context object for the current turn of the bot.</param>
/// <param name="result">The recognition result from the prompt.</param>
/// <returns>An updated recognition result.</returns>
private static async Task PartySizeValidator(ITurnContext context, Int32Result result)
{
    if (result.Value < 6 || result.Value > 20)
    {
        result.Status = PromptStatus.OutOfRange;
    }
}
```

在您的對話中，請指定要用來驗證輸入的方法。

```cs
// Include a validation function for the party size prompt.
Add(Inputs.Size, new NumberPrompt<int>(Culture.English, PartySizeValidator));
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

```javascript
// Customized prompts with validations
// A number prompt with validation for valid party size within a range.
dialogs.add('partySizePrompt', new botbuilder_dialogs.NumberPrompt( async (context, value) => {
    try {
        if(value < 6 ){
            throw new Error('Party size too small.');
        }
        else if(value > 20){
            throw new Error('Party size too big.')
        }
        else {
            return value; // Return the valid value
        }
    } catch (err) {
        await context.sendActivity(`${err.message} <br/>Please provide a valid number between 6 and 20.`);
        return undefined;
    }
}));
```

---

同樣地，如果您想要對未來的日期和時間驗證 **DatetimePrompt** 回應，您可以使用如下的驗證邏輯：

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

```cs
using Microsoft.Bot.Builder;
using Microsoft.Bot.Builder.Dialogs;
using Microsoft.Recognizers.Text;
using System;
using System.Collections.Generic;
using System.Linq;
using System.Threading.Tasks;
using DateTimeResult = Microsoft.Bot.Builder.Prompts.DateTimeResult;
using PromptStatus = Microsoft.Bot.Builder.Prompts.PromptStatus;
```

```cs
/// <summary>Validates input for the reservationTime prompt.</summary>
/// <param name="context">The context object for the current turn of the bot.</param>
/// <param name="result">The recognition result from the prompt.</param>
/// <returns>An updated recognition result.</returns>
private static async Task TimeValidator(ITurnContext context, DateTimeResult result)
{
    if (result.Resolution.Count == 0)
    {
        await context.SendActivity("Sorry, I did not recognize the time that you entered.");
        result.Status = PromptStatus.NotRecognized;
    }

    // Find any recognized time that is not in the past.
    var now = DateTime.Now;
    DateTime time = default(DateTime);
    var resolution = result.Resolution.FirstOrDefault(
        res => DateTime.TryParse(res.Value, out time) && time > now);

    if (resolution != null)
    {
        // If found, keep only that result.
        result.Resolution.Clear();
        result.Resolution.Add(resolution);
    }
    else
    {
        // Otherwise, flag the input as out of range.
        await context.SendActivity("Please enter a time in the future, such as \"tomorrow at 9am\"");
        result.Status = PromptStatus.OutOfRange;
    }
}
```

```csharp
Add(Inputs.Time, new DateTimePrompt(Culture.English, TimeValidator));
```

其他範例可在[範例存放庫](https://github.com/Microsoft/botbuilder-dotnet)中找到。

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

```JavaScript
// A date and time prompt with validation for date/time in the future.
dialogs.add('dateTimePrompt', new botbuilder_dialogs.DatetimePrompt( async (context, values) => {
    try {
        if (values.length < 0) { throw new Error('missing time') }
        if (values[0].type !== 'datetime') { throw new Error('unsupported type') }
        const value = new Date(values[0].value);
        if (value.getTime() < new Date().getTime()) { throw new Error('in the past') }
        return value;
    } catch (err) {
        await context.sendActivity(`Please enter a valid time in the future like "tomorrow at 9am".`);
        return undefined;
    }
}));
```

其他範例可在[範例存放庫](https://github.com/Microsoft/botbuilder-js)中找到。

---

> [!TIP]
> 如果使用者提供了模稜兩可的答案，日期時間提示可能會解析成數個不同的日期。 根據提示的用途，您可能會想要檢查提示結果所提供的所有解析，而不只是第一個解析。

您可以使用類似的技巧，來驗證任何提示類型的提示回應。

## <a name="save-user-data"></a>儲存使用者資料

當您提示使用者輸入時，您可以選擇數種不同的方式來處理該輸入。 例如，您可以取用和捨棄輸入、將其儲存至全域變數、將其儲存至揮發性記憶體或記憶體中的儲存體容器、將其儲存至檔案，或是，將其儲存至外部資料庫。 如需如何儲存使用者資料的詳細資訊，請參閱[管理使用者資料](bot-builder-howto-v4-state.md)。

## <a name="next-steps"></a>後續步驟

現在您已了解如何提示使用者輸入，接下來我們將透過對話方塊來管理各種對話流程，以強化 Bot 程式碼並提升使用者體驗。


