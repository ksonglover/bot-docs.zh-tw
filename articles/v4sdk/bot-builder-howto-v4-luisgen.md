---
title: 擷取具型別的 LUIS 結果 | Microsoft Docs
description: 了解如何使用 LUIS 搭配 Bot Builder SDK 擷取實體。
keywords: 意圖, 實體, LUISGen, 擷取
author: DeniseMak
ms.author: v-demak
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 5/16/17
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: f0e428ca0aa1b0208538e2de7fc0a293763062a1
ms.sourcegitcommit: 54ed5000c67a5b59e23b667547565dd96c7302f9
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/13/2018
ms.locfileid: "49315204"
---
# <a name="extract-intents-and-entities-using-luisgen"></a>使用 LUISGen 擷取意圖和實體

[!INCLUDE [pre-release-label](../includes/pre-release-label.md)]

除了辨識意圖，LUIS 應用程式也可以擷取實體，也就是滿足使用者要求的重要單字。 例如，在餐廳預訂的範例中，LUIS 應用程式可能可以從使用者的訊息中擷取派對人數、預訂日期或餐廳位置。 


您可以使用 [LUISGen 工具](https://aka.ms/botbuilder-tools-luisgen)來產生類別，讓您更輕鬆地在 Bot 程式碼中從 LUIS 擷取實體。

在 Node.js 命令列中，將 `luisgen` 安裝到全域路徑。
```
npm install -g luisgen
```

# <a name="ctabcs"></a>[C#](#tab/cs)

## <a name="generate-a-luis-results-class"></a>產生 LUIS 結果類別

下載 [CafeBot LUIS 範例](https://aka.ms/contosocafebot-luis)，然後在它的根資料夾中執行 LUISGen：

```
luisgen Assets\LU\models\LUIS\cafeLUISModel.json -cs ContosoCafeBot.CafeLUISModel
```

## <a name="examine-the-generated-code"></a>檢查產生的程式碼
這會產生 **cafeLUISModel.cs**，您可以將它新增至您的專案。 它提供 `cafeLuisModel` 類別，用來從 LUIS 取得強型別結果。

這個類別具有列舉，可取得 LUIS 應用程式中定義的意圖。
```cs
public enum Intent {
    Book_Table, 
    Greeting, 
    None, 
    Who_are_you_intent
};
```
它也具有 `Entities` 屬性。 因為使用者訊息中可能有多個實體，所以 `_Entities` 類別會為每個實體型別定義一個陣列。 
```cs
public class _Entities
{
    // Simple entities
    public string[] partySize;

    // Built-in entities
    public Microsoft.Bot.Builder.Ai.Luis.DateTimeSpec[] datetime;
    public double[] number;

    // Lists
    public string[][] cafeLocation;

    // Instance
    public class _Instance
    {
        public Microsoft.Bot.Builder.Ai.Luis.InstanceData[] partySize;
        public Microsoft.Bot.Builder.Ai.Luis.InstanceData[] datetime;
        public Microsoft.Bot.Builder.Ai.Luis.InstanceData[] number;
        public Microsoft.Bot.Builder.Ai.Luis.InstanceData[] cafeLocation;
    }
    [JsonProperty("$instance")]
    public _Instance _instance;
}
public _Entities Entities;
```

> [!NOTE]
> 所有實體型別都是陣列，因為 LUIS 可在使用者語句中偵測到多個具有指定型別的實體。 例如，假設使用者說「預訂明天下午 5 點和下星期六上午 9 點」，則 `datetime` 結果中會傳回「明天下午 5 點」和「下星期六上午 9 點」。
>

|實體 | 類型 | 範例 | 注意 |
|-------|-----|------|---|
|partySize| string[]| 派對人數 `four` 人| 簡單實體會辨識字串。 在此範例中，Entities.partySize[0] 是 `"four"`。
|Datetime| [DateTimeSpec](https://docs.microsoft.com/en-us/dotnet/api/microsoft.bot.builder.ai.luis.datetimespec?view=botbuilder-4.0.0-alpha)[]| 預訂 `9pm tomorrow`| 每個 **DateTimeSpec** 物件都有一個 timex 欄位，包含以 **timex** 格式指定的可能時間值。 更多有關 timex 的資訊可以在這裡找到： http://www.timeml.org/publications/timeMLdocs/timeml_1.2.1.html#timex3      更多有關用來辨識之文件庫的資訊可以在這裡找到： https://github.com/Microsoft/Recognizers-Text
|number| double[]| 派對人數 `four` 人，包括 `2` 個兒童 | `number` 會識別所有數字，不只是派對人數。 <br/> 在「派對人數 4 人，包括 2 個兒童」語句中，`Entities.number[0]` 是 4，`Entities.number[1]` 是 2。
|cafelocation| string[][] | 預訂位於 `Seattle` 位置。| cafeLocation 是清單實體，這表示它包含已辨識的清單成員。 它是陣列的陣列，已辨識實體是一個清單以上的成員。 例如，「預訂位於華盛頓」可能對應到華盛頓州清單與華盛頓特區清單。

`_Instance` 屬性提供 [InstanceData](https://docs.microsoft.com/en-us/dotnet/api/microsoft.bot.builder.ai.luis.instancedata?view=botbuilder-4.0.0-alpha)，說明每個已辨識實體的詳細資訊。

## <a name="check-intents-in-your-bot"></a>檢查 Bot 中的意圖
在 **CafeBot.cs** 中，看一下 `OnTurn` 內的程式碼。 您可以看到 Bot 呼叫 LUIS 並檢查意圖，以決定要開始哪個對話。 呼叫 [`Recognize`](https://docs.microsoft.com/en-us/dotnet/api/microsoft.bot.builder.ai.luis.luisrecognizer?view=botbuilder-4.0.0-alpha#methods) 產生的 LUIS 結果會做為引數傳遞到 `BookTable` 對話。



```cs
if(!context.Responded)
{
    // call LUIS and get results
    LuisRecognizer lRecognizer = createLUISRecognizer();
    // Use the generated class as the type parameter to Recognize()
    cafeLUISModel lResult = await lRecognizer.Recognize<cafeLUISModel>(utterance, ct);
    Dictionary<string,object> lD = new Dictionary<string,object>();
    if(lResult != null) {
        lD.Add("luisResult", lResult);
    }
    
    // top level dispatch
    switch (lResult.TopIntent().intent)
    {
        case cafeLUISModel.Intent.Greeting:
            await context.SendActivity("Hello!");
            if (userState.sendCards) await context.SendActivity(CreateCardResponse(context.Activity, createWelcomeCardAttachment()));
            break;

        case cafeLUISModel.Intent.Book_Table:
            await dc.Begin("BookTable", lD);
            break;

        case cafeLUISModel.Intent.Who_are_you_intent:
            await context.sendActivity("I'm the Contoso Cafe bot.");
            break;

        case cafeLUISModel.Intent.None:
        default:
            await getQnAResult(context);
            break;
    }
}
```

## <a name="extract-entities-in-a-dialog"></a>擷取對話中的實體

現在看一下 `Dialogs/BookTable.cs`。 `BookTable` 對話包含一系列瀑布圖步驟，每一個步驟都會檢查傳遞到 `args` 之 LUIS 結果中的實體。 如果找不到實體，對話會呼叫 `next()` 而略過提示。 如果找到，對話會出現提示，而下一個瀑布圖步驟中會收到使用者在提示下輸入的回答。

```cs
    Dialogs.Add("BookTable",
        new WaterfallStep[]
        {
            async (dc, args, next) =>
            {
                dc.ActiveDialog.State = new Dictionary<string, object>();
                IDictionary<string,object> state = dc.ActiveDialog.State;

                // add any LUIS entities to active dialog state.
                if(args.ContainsKey("luisResult")) {
                    cafeLUISModel lResult = (cafeLUISModel)args["luisResult"];
                    updateContextWithLUIS(lResult, ref state);
                }
                
                // prompt if we do not already have cafelocation
                if(state.ContainsKey("cafeLocation")) {
                    state["bookingLocation"] = state["cafeLocation"];
                    await next();
                } else {
                    await dc.Prompt("choicePrompt", "Which of our locations would you like?", promptOptions);
                }
            },
            async (dc, args, next) =>
            {
                var state = dc.ActiveDialog.State;
                if(!state.ContainsKey("cafeLocation")) {
                    var choiceResult = (FoundChoice)args["Value"];
                    state["bookingLocation"] = choiceResult.Value;
                }
                bool promptForDateTime = true;
                if(state.ContainsKey("datetime")) {
                    // validate timex
                    var inputdatetime = new string[] {(string)state["datetime"]};
                    var results = evaluateTimeX((string[])inputdatetime);
                    if(results.Count != 0) {
                        var timexResolution = results.First().TimexValue;
                        var timexProperty = new TimexProperty(timexResolution.ToString());
                        var bookingDateTime = $"{timexProperty.ToNaturalLanguage(DateTime.Now)}";
                        state["bookingDateTime"] = bookingDateTime;
                        promptForDateTime = false;
                    }
                }
                // prompt if we do not already have date and time
                if(promptForDateTime) {
                    await dc.Prompt("timexPrompt", "When would you like to arrive? (We open at 4PM.)",
                                    new PromptOptions { RetryPromptString = "We only accept reservations for the next 2 weeks and in the evenings between 4PM - 8PM" });
                } else {
                    await next();
                }                       
                
            },
            async (dc, args, next) =>
            {
                var state = dc.ActiveDialog.State;
                if(!state.ContainsKey("datetime")) { 
                    var timexResult = (TimexResult)args;
                    var timexResolution = timexResult.Resolutions.First();
                    var timexProperty = new TimexProperty(timexResolution.ToString());
                    var bookingDateTime = $"{timexProperty.ToNaturalLanguage(DateTime.Now)}";
                    state["bookingDateTime"] = bookingDateTime;
                }
                // prompt if we already do not have party size
                if(state.ContainsKey("partySize")) {
                    state["bookingGuestCount"] = state["partySize"];
                    await next();
                } else {
                    await dc.Prompt("numberPrompt", "How many in your party?");
                }
            },
            async (dc, args, next) =>
            {
                var state = dc.ActiveDialog.State;
                if(!state.ContainsKey("partySize")) {
                    state["bookingGuestCount"] = args["Value"];
                }

                await dc.Prompt("confirmationPrompt", $"Thanks, Should I go ahead and book a table for {state["bookingGuestCount"].ToString()} guests at our {state["bookingLocation"].ToString()} location for {state["bookingDateTime"].ToString()} ?");
            },
            async (dc, args, next) =>
            {
                var dialogState = dc.ActiveDialog.State;

                // TODO: Verify user said yes to confirmation prompt

                // TODO: book the table! 

                await dc.Context.SendActivity($"Thanks, I have {dialogState["bookingGuestCount"].ToString()} guests booked for our {dialogState["bookingLocation"].ToString()} location for {dialogState["bookingDateTime"].ToString()}.");
            }
        }
    );
}

// This helper method updates dialog state with any LUIS results
private void updateContextWithLUIS(cafeLUISModel lResult, ref IDictionary<string,object> dialogContext) {
    if(lResult.Entities.cafeLocation != null && lResult.Entities.cafeLocation.GetLength(0) > 0) {
        dialogContext.Add("cafeLocation", lResult.Entities.cafeLocation[0][0]);
    }
    if(lResult.Entities.partySize != null && lResult.Entities.partySize.GetLength(0) > 0) {
        dialogContext.Add("partySize", lResult.Entities.partySize[0][0]);
    } else {
        if(lResult.Entities.number != null && lResult.Entities.number.GetLength(0) > 0) {
            dialogContext.Add("partySize", lResult.Entities.number[0]);
        }
    }
    if(lResult.Entities.datetime != null && lResult.Entities.datetime.GetLength(0) > 0) {
        dialogContext.Add("datetime", lResult.Entities.datetime[0].Expressions[0]);
    }
}
```
## <a name="run-the-sample"></a>執行範例

在 Visual Studio 2017 中開啟 `ContosoCafeBot.sln`，然後執行 Bot。 使用 [Bot Framework Emulator](https://docs.microsoft.com/en-us/azure/bot-service/bot-service-debug-emulator) 以連接到範例 Bot。

在模擬器中，說出 `reserve a table` 以啟動預約對話。

![執行 Bot](media/how-to-luisgen/run-bot.png)

# <a name="typescripttabjs"></a>[TypeScript](#tab/js)

下載 [CafeBot_LUIS 範例](https://aka.ms/contosocafebot-typescript-luis-dialogs)，然後在它的根資料夾中執行 LUISGen：

```
luisgen cafeLUISModel.json -ts CafeLUISModel
```

這會產生 **CafeLUISModel.ts**，您可以將它新增至您的專案。 使用所產生檔案中的型別，可從 LUIS 辨識器取得具型別的結果。


```typescript
// call LUIS and get typed results
await luisRec.recognize(context).then(async (res : any) => 
{    
    // get a typed result
    var typedresult = res as CafeLUISModel;   
    
```

## <a name="pass-the-typed-result-to-a-dialog"></a>將具型別結果傳遞到對話

檢查 **luisbot.ts** 中的程式碼。 在 `processActivity` 處理常式中，Bot 會將具型別結果傳遞到對話。

```typescript
// Listen for incoming requests 
server.post('/api/messages', (req, res) => {
    // Route received request to adapter for processing
    adapter.processActivity(req, res, async (context) => {
        const isMessage = context.activity.type === 'message';

        // Create dialog context 
        const state = conversationState.get(context);
        const dc = dialogs.createContext(context, state);
            
        if (!isMessage) {
            await context.sendActivity(`[${context.activity.type} event detected]`);
        }

        // Check to see if anyone replied. 
        if (!context.responded) {
            await dc.continue();
            // if the dialog didn't send a response
            if (!context.responded && isMessage) {

                
                await luisRec.recognize(context).then(async (res : any) => 
                {    
                    var typedresult = res as CafeLUISModel;                
                    let topIntent = LuisRecognizer.topIntent(res);    
                    switch (topIntent)
                    {
                        case Intents.Book_Table: {
                            await context.sendActivity("Top intent is Book_Table ");                          
                            await dc.begin('reserveTable', typedresult);
                            break;
                        }
                        
                        case Intents.Greeting: {
                            await context.sendActivity("Top intent is Greeting");
                            break;
                        }
    
                        case Intents.Who_are_you_intent: {
                            await context.sendActivity("Top intent is Who_are_you_intent");
                            break;
                        }
                        default: {
                            await dc.begin('default', topIntent);
                            break;
                        }
                    }
    
                }, (err) => {
                    // there was some error
                    console.log(err);
                }
                );                                
            }
        }
    });
});
```

## <a name="check-for-existing-entities-in-a-dialog"></a>檢查對話中的現有實體

在 **luisbot.ts** 中，`reserveTable` 對話會呼叫 `SaveEntities` 協助程式函式，來檢查 LUIS 應用程式偵測到的實體。 如果找到實體，會將它們儲存到對話狀態。 對話中的每個瀑步圖步驟都會檢查實體是否儲存到對話狀態，如果沒有的話，則會提示儲存。

```typescript
dialogs.add('reserveTable', [
    async function(dc, args, next){
        var typedresult = args as CafeLUISModel;

        // Call a helper function to save the entities in the LUIS result
        // to dialog state
        await SaveEntities(dc, typedresult);

        await dc.context.sendActivity("Welcome to the reservation service.");
        
        if (dc.activeDialog.state.dateTime) {
            await next();     
        }
        else {
            await dc.prompt('dateTimePrompt', "Please provide a reservation date and time.");
        }
    },
    async function(dc, result, next){
        if (!dc.activeDialog.state.dateTime) {
            // Save the dateTimePrompt result to dialog state
            dc.activeDialog.state.dateTime = result[0].value;
        }

        // If we don't have party size, ask for it next
        if (!dc.activeDialog.state.partySize) {
            await dc.prompt('textPrompt', "How many people are in your party?");
        } else {
            await next();
        }
    },
    async function(dc, result, next){
        if (!dc.activeDialog.state.partySize) {
            dc.activeDialog.state.partySize = result;
        }
        // Ask for the reservation name next
        await dc.prompt('textPrompt', "Whose name will this be under?");
    },
    async function(dc, result){
        dc.activeDialog.state.Name = result;

        // Save data to conversation state
        var state = conversationState.get(dc.context);

        // Copy the dialog state to the conversation state
        state = dc.activeDialog.state;

        // TODO: Add in <br/>Location: ${state.cafeLocation}
        var msg = `Reservation confirmed. Reservation details:             
            <br/>Date/Time: ${state.dateTime} 
            <br/>Party size: ${state.partySize} 
            <br/>Reservation name: ${state.Name}`;
            
        await dc.context.sendActivity(msg);
        await dc.end();
    }
]);
```

`SaveEntities` 協助程式函式會檢查 `datetime` 和 `partysize` 實體。 `datetime` 實體是[預先建置的實體](https://docs.microsoft.com/en-us/azure/cognitive-services/luis/luis-reference-prebuilt-entities#builtindatetimev2)。

```typescript
// Helper function that saves any entities found in the LUIS result
// to the dialog state
async function SaveEntities( dc: DialogContext<TurnContext>, typedresult) {
    // Resolve entities returned from LUIS, and save these to state
    if (typedresult.entities)
    {
        console.log(`Entities found.`);
        let datetime = typedresult.entities.datetime;

        if (datetime) {
            // Use the first date or time found in the utterance
            if (datetime[0].timex) {
                timexValues = datetime[0].timex;
                // Datetime results from LUIS are represented in timex format:
                // http://www.timeml.org/publications/timeMLdocs/timeml_1.2.1.html#timex3                                
                // More information on the library which does the recognition can be found here: 
                // https://github.com/Microsoft/Recognizers-Text

                if (datetime[0].type === "datetime") {
                    // To parse timex, here you use the resolve and creator from
                    // @microsoft/recognizers-text-data-types-timex-expression
                    // The second parameter is an array of constraints the results must satisfy
                    var resolution = Resolver.evaluate(
                        // array of timex values to evaluate. There may be more than one: "today at 6" can be 6AM or 6PM.
                        timexValues,
                        // constrain results to times between 4pm and 8pm                        
                        [Creator.evening]);
                    if (resolution[0]) {
                        // toNaturalLanguage takes the current date into account to create a friendly string
                        dc.activeDialog.state.dateTime = resolution[0].toNaturalLanguage(new Date());
                        // You can also use resolution.toString() to format the date/time.
                    } else {
                        // time didn't satisfy constraint.
                        dc.activeDialog.state.dateTime = null;
                    }
                } 
                else  {
                    console.log(`Type ${datetime[0].type} is not yet supported. Provide both the date and the time.`);
                }
            }                                                
        }
        let partysize = typedresult.entities.partySize;
        if (partysize) {
            console.log(`partysize entity detected.${partysize}`);
            // use first partySize entity that was found in utterance
            dc.activeDialog.state.partySize = partysize[0];
        }
        let cafelocation = typedresult.entities.cafeLocation;

        if (cafelocation) {
            console.log(`location entity detected.${cafelocation}`);
            // use first cafeLocation entity that was found in utterance
            dc.activeDialog.state.cafeLocation = cafelocation[0][0];
        }
    } 
}
```

## <a name="run-the-sample"></a>執行範例

1. 如果您沒有安裝 TypeScript 編譯器，請使用此命令來安裝它：

```
npm install --global typescript
```

2. 執行 Bot 之前，請在範例的根目錄中執行 `npm install` 以安裝相依項目：

```
npm install
```

3. 從根目錄，使用 `tsc` 建置範例。 這將產生 `luisbot.js`。

4. 執行 `lib` 目錄中的 `luisbot.js`。

5. 使用 [Bot Framework Emulator](https://docs.microsoft.com/en-us/azure/bot-service/bot-service-debug-emulator) 以執行範例。

6. 在模擬器中，說出 `reserve a table` 以啟動預約對話。

![執行 Bot](media/how-to-luisgen/run-bot.png)

---


## <a name="additional-resources"></a>其他資源

如需 LUIS 的背景資訊，請參閱 [Language Understanding](./bot-builder-concept-luis.md)。


## <a name="next-steps"></a>後續步驟

> [!div class="nextstepaction"]
> [使用分派工具結合 LUIS 和 QnA Maker](./bot-builder-tutorial-dispatch.md)


