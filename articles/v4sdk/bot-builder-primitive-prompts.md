---
title: 使用基本提示管理對話流程 | Microsoft Docs
description: 了解如何在 Bot 建立器 SDK 中，使用基本提示來管理對話流程。
keywords: 對話流程, 提示
author: v-ducvo
ms.author: v-ducvo
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 7/20/2018
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 1514e1bcafc87be9e8449382bca7aa156e512ed9
ms.sourcegitcommit: e8c513d3af5f0c514cadcbcd0a737a7393405afa
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/20/2018
ms.locfileid: "40228341"
---
# <a name="prompt-users-for-input-using-your-own-prompts"></a>使用您自己的提示來提示使用者輸入
[!INCLUDE [pre-release-label](../includes/pre-release-label.md)]

Bot 和使用者之間的對話，通常牽涉到要求 (提示) 使用者輸入資訊、剖析使用者的回應，然後根據該資訊行動。 在關於[使用對話方塊程式庫提示使用者](bot-builder-prompts.md)的主題中，其詳細說明如何使用**對話方塊**程式庫提示使用者輸入。 除此之外，**對話方塊**程式庫會負責追蹤目前的對話及目前提出的問題。 其也提供方法來要求不同類型的資訊，例如文字、數字、日期和時間等等。 

在某些情況下，內建**對話方塊**可能不適合您的 Bot；**對話方塊**對於簡單的 Bot 可能加入太多額外負荷、太死板，或是無法達到您要 Bot 執行的內容。 在這些情況下，您可以跳過程式庫，然後實作自己的提示邏輯。 本主題會示範如何設定基本的 *Echo bot*，讓您可以使用自己的提示來管理對話。

## <a name="track-prompt-states"></a>追蹤提示狀態

在提示導向的對話中，您需要追蹤目前在對話中的位置，以及目前提出的是何種問題。 在程式碼內，這會轉譯為管理幾個旗標。 

例如，您可以建立幾個想要追蹤的屬性。 

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

在這裡，使用者設定檔類別會以巢狀方式置於提示資訊中，可讓所有這些一起儲存在 Bot [狀態](bot-builder-howto-v4-state.md)中。

```csharp
// Where user information will be stored
public class UserProfile
{
    public string userName = null;
    public string workPlace = null;
    public int age = 0;
}

// Flags to keep track of our prompts, and the 
// user profile object for this conversation
public class PromptInformation
{
    public string topicTitle = null;
    public string prompt = null;
    public UserProfile userProfile = null;
}
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

這些狀態會追蹤我們目前位於哪個主題和哪個提示。 若要確保這些旗標部署至雲端時能正常運作，可將其儲存在[對話狀態](bot-builder-howto-v4-state.md)中。 將此程式碼置於主要的 Bot 邏輯內。

**app.js**
```javascript
// Define a topicStates object if it doesn't exist in the convoState.
if(!convo.topicStates){
    convo.topicStates = {
        "topicTitle": undefined, // Current conversation topic in progress
        "prompt": undefined      // Current prompt state in progress - indicate what question is being asked.
    };
}
```

---

## <a name="manage-a-topic-of-conversation"></a>管理對話的主題

在循序對話中，如收集使用者資訊的對話，您需要知道使用者輸入對話的時間和對話中使用者所在位置。 您可以設定並檢查上述定義的提示旗標，再據以行動，以便在主要的 Bot 回合處理常式中追蹤這些資訊。 下列範例會示範如何使用這些旗標，透過對話收集使用者設定檔資訊。

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

```csharp
// Get our current state, as defined above
var convoState = context.GetConversationState<PromptInformation>();

if (convoState.userProfile == null)
{
    await context.SendActivity("Welcome new user, please fill out your profile information.");
    convoState.topicTitle = "profileTopic"; // Start the userProfile topic
    convoState.userProfile = new UserProfile();
}

// Start or continue a conversation if we are in one
if (convoState.topicTitle == "profileTopic")
{
    if (convoState.userProfile.userName == null && convoState.prompt == null)
    {
        convoState.prompt = "askName";
        await context.SendActivity("What is your name?");
    }
    else if (convoState.prompt == "askName")
    {
        // Save the user's response
        convoState.userProfile.userName = context.Activity.Text;

        // Ask next question
        convoState.prompt = "askAge";
        await context.SendActivity("How old are you?");
    }
    else if (convoState.prompt == "askAge")
    {
        // Save user's response
        if (!Int32.TryParse(context.Activity.Text, out convoState.userProfile.age))
        {
            // Failed to convert to int
            await context.SendActivity("Failed to convert string to int");
        }
        else
        {
            // Ask next question
            convoState.prompt = "workPlace";
            await context.SendActivity("Where do you work?");
        }

    }
    else if (convoState.prompt == "workPlace")
    {
        // Save user's response
        convoState.userProfile.workPlace = context.Activity.Text;

        // Done
        convoState.topicTitle = null; // Reset conversation flag
        convoState.prompt = null;     // Reset the prompt flag
        
        await context.SendActivity("Thank you. Your profile is complete.");
    }
    else // Somehow our flags got inappropriately set
    {
        await context.SendActivity("Looks like something went wrong, let's start over");
        convoState.userProfile = null;
        convoState.prompt = null;
        convoState.topicTitle = null;
    }
}
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

**app.js**
```javascript
server.post('/api/messages', (req, res) => {
    adapter.processActivity(req, res, async (context) => {
        const isMessage = (context.activity.type === 'message');
        // State will store all of your information 
        const convo = conversationState.get(context);

        // Defined flags to manage the conversation flow and prompt states
        // convo.topicStates.topicTitle - Current conversation topic in progress
        // convo.topicStates.prompt - Current prompt state in progress - indicate what question is being asked.
        
        if (isMessage) {
            // Define a topicStates object if it doesn't exist in the convoState.
            if(!convo.topicStates){
                convo.topicStates = {
                    "topicTitle": undefined, // Current conversation topic in progress
                    "prompt": undefined      // Current prompt state in progress - indicate what question is being asked.
                };
            }
            
            // If user profile is not defined then define it.
            if(!convo.userProfile){
                
                await context.sendActivity(`Welcome new user, please fill out your profile information.`);
                convo.topicStates.topicTitle = "profileTopic"; // Start the userProfile topic
                convo.userProfile = { // Define the user's profile object
                    "userName": undefined,
                    "age": undefined,
                    "workPlace": undefined
                }; 
            }

            // Start or continue a conversation if we are in one
            if(convo.topicStates.topicTitle == "profileTopic"){
                if(!convo.userProfile.userName && !convo.topicStates.prompt){
                    convo.topicStates.prompt = "askName";
                    await context.sendActivity("What is your name?");
                }
                else if(convo.topicStates.prompt == "askName"){
                    // Save the user's response
                    convo.userProfile.userName = context.activity.text; 

                    // Ask next question
                    convo.topicStates.prompt = "askAge";
                    await context.sendActivity("How old are you?");
                }
                else if(convo.topicStates.prompt == "askAge"){
                    // Save user's response
                    convo.userProfile.age = context.activity.text;

                    // Ask next question
                    convo.topicStates.prompt = "workPlace";
                    await context.sendActivity("Where do you work?");
                }
                else if(convo.topicStates.prompt == "workPlace"){
                    // Save user's response
                    convo.userProfile.workPlace = context.activity.text;

                    // Done
                    convo.topicStates.topicTitle = undefined; // Reset conversation flag
                    convo.topicStates.prompt = undefined;     // Reset the prompt flag
                    await context.sendActivity("Thank you. Your profile is complete.");
                }
            }

            // Check for valid intents
            else if(context.activity.text && context.activity.text.match(/hi/ig)){
                await context.sendActivity(`Hi ${convo.userProfile.userName}.`);
            }
            else {
                // Default message
                await context.sendActivity("Hi. I'm the Contoso bot.");
            }
        }

    });
});

```

---

上述範例程式碼會檢查，是否在記憶體中定義使用者設定檔。 如果否，表示是新的使用者，則設定旗標自動啟動該主題。 接著，其會顯示如何將 `prompt` 旗標設定為目前所提出問題的值，以便您繼續對話。 此旗標設為適當的值後，條件式子句會在每回合捕捉使用者的回應，並依此處理。 

最後，要求資訊完成後，您必須重設這些旗標，Bot 才不會陷入迴圈。 您可以擴充此基本設定來管理 Bot 所需更複雜的對話流程，只要定義其他旗標或根據使用者輸入分支對話即可。

## <a name="manage-multiple-topics-of-conversations"></a>管理多個對話主題

能夠處理一個對話主題之後，您可以擴充 Bot 邏輯來處理多個對話主題。 就像單一對話主題，只要檢查觸發特定主題的條件，然後採取適當路徑，就可以輕鬆處理多個主題。

在上述範例中，您可以將其重構以允許其他功能和主題，例如餐廳訂位或訂購晚餐。 其他資訊可以視需要新增至使用者設定檔或主題狀態旗標，以便追蹤對話。

為了更有效地管理多個對話主題，其中一種方法是提供主要功能表。 使用[建議的動作](bot-builder-howto-add-suggested-actions.md)，您可以讓使用者選擇可參與哪些主題，然後再深入該主題。 您也會發現，將程式碼分割成獨立的函式很有幫助，可更輕鬆地追蹤對話流程。

## <a name="validate-user-input"></a>驗證使用者輸入

**對話方塊**程式庫提供內建方式來驗證使用者的輸入，但我們也可以使用自己的提示來達成。 比方說，如果詢問使用者的年齡，我們會想得到數字，而非 "Bob" 之類的回應。

剖析數字或日期和時間是很複雜的工作，已超出本主題的範圍。 幸運的是，我們可以利用程式庫。 若要正確剖析此資訊，可使用 [Microsoft 的文字辨識器](https://github.com/Microsoft/Recognizers-Text)程式庫。 此套件可透過 NuGet 取得，或從存放庫下載。 此外，**對話方塊**程式庫中也包含套件，這點值得注意，但這裡不使用該程式庫。

此程式庫特別適用於剖析一般語言，或如日期、時間或電話號碼等複雜的回應。 在此範例中，我們只驗證預訂晚餐派對大小的號碼，但相同的概念可延伸至更深入的驗證。 如果您不熟悉此程式庫，請檢閱該網站的內容，查看其所提供的功能。

在此範例中我們只示範如何使用 `RecognizeNumber`。 可在[存放庫文件](https://github.com/Microsoft/Recognizers-Text/blob/master/README.md)中，找到如何使用程式庫中其他辨識器方法的詳細資訊。

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

若要使用辨識器程式庫，將其新增至使用陳述式。

```csharp
using Microsoft.Recognizers.Text.Number;
using Microsoft.Recognizers.Text;
using System.Linq; // Used to get the first result from the recognizer
```

然後，定義可實際執行驗證的函式。

```csharp
private async Task<bool> ValidatePartySize(ITurnContext context, string value)
{
    try
    {
        // Recognize the input as a number. This works for responses such as
        // "twelve" as well as "12"
        var result = NumberRecognizer.RecognizeNumber(input, Culture.English);

        // Attempt to convert the Recognizer result to an integer
        Int32.TryParse(result.First().Text, out int partySize);
        
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
        await context.SendActivity("Error with your party size. < br /> Please specify a number between 6 - 20.");
        return false;
    }
}
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

若要使用辨識器程式庫，在 **app.js** 中要求它：

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

        if(value < 6) {
            throw new Error(`Party size too small.`);
        }
        else if(value > 20){
            throw new Error(`Party size too big.`);
        }
        return true; // Return the valid value
    }
    catch (err){
        await context.sendActivity(`${err.message} <br/>Please specify a number between 6 - 20.`);
        return false;
    }
}
```

---

在我們想要驗證的提示步驟中，呼叫驗證函式，再繼續進行下一個提示。 如果驗證失敗，請再次提出問題。

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

```csharp
if (convoState.prompt == "partySize")
{
    if (await ValidatePartySize(context, context.Activity.Text))
    {
        // Save user's response in our state, ReservationInfo, which 
        // is a new class we've added to our state
        convoState.ReservationInfo.partySize = context.Activity.Text;

        // Ask next question
        convoState.prompt = "reserveName";
        await context.SendActivity("Who's name will this be under?");
    }
    else
    {
        // Ask again
        await context.SendActivity("How many people are in your party?");
    }
}
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

**app.js**

```javascript
// ...
if(convo.topicStates.prompt == "partySize"){
    if(await validatePartySize(context, context.activity.text)){
        // Save user's response
        convo.reservationInfo.partySize = context.activity.text;
        
        // Ask next question
        topicStates.prompt = "reserveName";
        await context.sendActivity("Who's name will this be under?");
    }
    else {
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
