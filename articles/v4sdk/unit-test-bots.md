---
title: 單元測試聊天機器人 | Microsoft Docs
description: 說明如何使用測試架構來對聊天機器人進行單元測試。
keywords: 聊天機器人, 測試聊天機器人, 聊天機器人測試架構
author: gabog
ms.author: ggilaber
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 07/17/2019
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 1e9d079b46c1cc4cc8c49e234b58540aeb4b2e7c
ms.sourcegitcommit: a6d02ec4738e7fc90b7108934740e9077667f3c5
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/04/2019
ms.locfileid: "70298978"
---
# <a name="how-to-unit-test-bots"></a>如何對聊天機器人進行單元測試

[!INCLUDE[applies-to](../includes/applies-to.md)]

在本主題中，我們將說明如何：

- 為聊天機器人建立單元測試
- 使用判斷提示來檢查對話方塊回合針對預期值所傳回的活動
- 使用判斷提示來檢查對話方塊所傳回的結果
- 建立不同類型的資料驅動測試
- 為不同的對話相依性建立模擬物件 (亦即 LUIS 辨識器等)

## <a name="prerequisites"></a>必要條件

## <a name="ctabcsharp"></a>[C#](#tab/csharp)

本主題所用的 [CoreBot Tests](https://aka.ms/cs-core-test-sample) 範例會參考 [Microsoft.Bot.Builder.Testing](https://www.nuget.org/packages/Microsoft.Bot.Builder.Testing/) 套件、[XUnit](https://xunit.net/) 和用來建立單元測試的 [Moq](https://github.com/moq/moq)。

## <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

本主題所用的 [CoreBot Tests](https://aka.ms/js-core-test-sample) 範例會參考 [botbuilder-testing](https://www.npmjs.com/package/botbuilder-testing) 套件、用來建立單元測試的 [Mocha](https://mochajs.org/) 以及用來在 VS Code 中視覺呈現測試結果的 [Mocha Test Explorer](https://marketplace.visualstudio.com/items?itemName=hbenl.vscode-mocha-test-adapter)。

---

## <a name="testing-dialogs"></a>測試對話方塊

在 CoreBot 範例中，對話方塊會透過 `DialogTestClient` 類別來進行單元測試，其所提供的機制可供在聊天機器人外部隔離測試對話，而不需要將程式碼部署至 Web 服務。

使用這個類別，您就可以撰寫單元測試來逐一回合地驗證回應的對話。 使用 `DialogTestClient` 類別的單元測試應該要能與使用 botbuilder 對話方塊程式庫所建立的其他對話方塊搭配運作。

下列範例示範衍生自 `DialogTestClient` 的測試：

## <a name="ctabcsharp"></a>[C#](#tab/csharp)

```csharp
var sut = new BookingDialog();
var testClient = new DialogTestClient(Channels.Msteams, sut);

var reply = await testClient.SendActivityAsync<IMessageActivity>("hi");
Assert.Equal("Where would you like to travel to?", reply.Text);

reply = await testClient.SendActivityAsync<IMessageActivity>("Seattle");
Assert.Equal("Where are you traveling from?", reply.Text);

reply = await testClient.SendActivityAsync<IMessageActivity>("New York");
Assert.Equal("When would you like to travel?", reply.Text);

reply = await testClient.SendActivityAsync<IMessageActivity>("tomorrow");
Assert.Equal("OK, I will book a flight from Seattle to New York for tomorrow, Is this Correct?", reply.Text);

reply = await testClient.SendActivityAsync<IMessageActivity>("yes");
Assert.Equal("Sure thing, wait while I finalize your reservation...", reply.Text);

reply = testClient.GetNextReply<IMessageActivity>();
Assert.Equal("All set, I have booked your flight to Seattle for tomorrow", reply.Text);
```

`DialogTestClient` 類別會定義在 `Microsoft.Bot.Builder.Testing` 命名空間中，並包含在 [Microsoft.Bot.Builder.Testing](https://www.nuget.org/packages/Microsoft.Bot.Builder.Testing/) NuGet 套件中。

## <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

```javascript
const sut = new BookingDialog();
const testClient = new DialogTestClient('msteams', sut);

let reply = await testClient.sendActivity('hi');
assert.strictEqual(reply.text, 'Where would you like to travel to?');

reply = await testClient.sendActivity('Seattle');
assert.strictEqual(reply.text, 'Where are you traveling from?');

reply = await testClient.sendActivity('New York');
assert.strictEqual(reply.text, 'When would you like to travel?');

reply = await testClient.sendActivity('tomorrow');
assert.strictEqual(reply.text, 'OK, I will book a flight from Seattle to New York for tomorrow, Is this Correct?');

reply = await testClient.sendActivity('yes');
assert.strictEqual(reply.text, 'Sure thing, wait while I finalize your reservation...');

reply = testClient.getNextReply();
assert.strictEqual(reply.text, 'All set, I have booked your flight to Seattle for tomorrow');
```

`DialogTestClient` 類別會包含在 [botbuilder-testing]() npm 套件中。

---

### <a name="dialogtestclient"></a>DialogTestClient

`DialogTestClient` 的第一個參數是目標通道。 這可以讓您根據聊天機器人的目標通道 (Teams、Slack、Cortana 等等) 來測試不同的呈現邏輯。 如果您不確定目標通道為何，則可以使用 `Emulator` 或 `Test` 通道識別碼，但請記住，某些元件可能會因為目前的通道而有不同的行為，例如，`ConfirmPrompt` 會針對 `Test` 和 `Emulator` 通道呈現不同的 [是/否] 選項。 您也可以使用這個參數，來根據通道識別碼測試對話方塊中的條件式呈現邏輯。

第二個參數是所測試對話方塊的執行個體 (注意： **"sut"** 代表 "System Under Test" (待測系統)，我們會在本文的程式碼片段中使用此縮略字)。

`DialogTestClient` 建構函式會提供其他參數，以便讓您進一步自訂用戶端行為，或在需要時將參數傳遞給所測試的對話方塊。 您可以傳遞對話方塊的初始化資料、新增自訂中介軟體，或使用您自己的 TestAdapter 和 `ConversationState` 執行個體。

### <a name="sending-and-receiving-messages"></a>傳送和接收訊息

## <a name="ctabcsharp"></a>[C#](#tab/csharp)

`SendActivityAsync<IActivity>` 方法可讓您將文字表達或 `IActivity` 傳送給對話方塊，並傳回其所收到的第一則訊息。 `<T>` 參數可用來傳回強型別的回覆執行個體，因此您不必轉換就可以對其做出判斷提示。

```csharp
var reply = await testClient.SendActivityAsync<IMessageActivity>("hi");
Assert.Equal("Where would you like to travel to?", reply.Text);
```

在某些情況下，聊天機器人可能會傳送數則訊息來回應單一活動，在這些情況下，`DialogTestClient` 會將回覆排入佇列，而您可以使用 `GetNextReply<IActivity>` 方法從回應佇列中快顯下一則訊息。

```csharp
reply = testClient.GetNextReply<IMessageActivity>();
Assert.Equal("All set, I have booked your flight to Seattle for tomorrow", reply.Text);
```

如果回應佇列中已沒有任何訊息，則 `GetNextReply<IActivity>` 會傳回 Null。

## <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

`sendActivity` 方法可讓您將文字表達或 `Activity` 傳送給對話方塊，並傳回其所收到的第一則訊息。

```javascript
let reply = await testClient.sendActivity('hi');
assert.strictEqual(reply.text, 'Where would you like to travel to?');
```

在某些情況下，聊天機器人可能會傳送數則訊息來回應單一活動，在這些情況下，`DialogTestClient` 會將回覆排入佇列，而您可以使用 `getNextReply` 方法從回應佇列中快顯下一則訊息。

```javascript
reply = testClient.getNextReply();
assert.strictEqual(reply.text, 'All set, I have booked your flight to Seattle for tomorrow');
```

如果回應佇列中已沒有任何訊息，則 `getNextReply` 會傳回 Null。

---

### <a name="asserting-activities"></a>活動的判斷提示

## <a name="ctabcsharp"></a>[C#](#tab/csharp)

CoreBot 範例中的程式碼只會對所傳回活動的 `Text` 屬性進行判斷提示。 在更複雜的聊天機器人中，您可以針對 `Speak`、`InputHint`、`ChannelData` 等其他屬性進行判斷提示。

```csharp
Assert.Equal("Sure thing, wait while I finalize your reservation...", reply.Text);
Assert.Equal("One moment please...", reply.Speak);
Assert.Equal(InputHints.IgnoringInput, reply.InputHint);
```

您可以如上所示逐一檢查每個屬性來進行此操作、您可以撰寫自己的協助程式公用程式來對活動進行判斷提示，您也可以使用 [FluentAssertions](https://fluentassertions.com/) 等其他架構來撰寫自訂判斷提示並簡化測試程式碼。

## <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

CoreBot 範例中的程式碼只會對所傳回活動的 `text` 屬性進行判斷提示。 在更複雜的聊天機器人中，您可以針對 `speak`、`inputHint`、`channelData` 等其他屬性進行判斷提示。

```javascript
assert.strictEqual(reply.text, 'Sure thing, wait while I finalize your reservation...');
assert.strictEqual(reply.speak, 'One moment please...');
assert.strictEqual(reply.inputHint, InputHints.IgnoringInput);
```

您可以如上所示逐一檢查每個屬性來進行此操作、您可以撰寫自己的協助程式公用程式來對活動進行判斷提示，您也可以使用 [Chai](https://www.chaijs.com/) 等其他程式庫來撰寫自訂判斷提示並簡化測試程式碼。

---

### <a name="passing-parameters-to-your-dialogs"></a>將參數傳遞給對話方塊

`DialogTestClient` 建構函式具有的 `initialDialogOptions` 可用來將參數傳遞給對話方塊。 例如，此範例中的 `MainDialog` 會使用從使用者的表達解析得到的實體將 LUIS 結果中的 `BookingDetails` 物件初始化，然後將此物件傳遞至呼叫中以叫用 `BookingDialog`。

您可以在測試中執行此項目，如下所示：

## <a name="ctabcsharp"></a>[C#](#tab/csharp)

```csharp
var inputDialogParams = new BookingDetails()
{
    Destination = "Seattle",
    TravelDate = $"{DateTime.UtcNow.AddDays(1):yyyy-MM-dd}"
};

var sut = new BookingDialog();
var testClient = new DialogTestClient(Channels.Msteams, sut, inputDialogParams);

```

`BookingDialog` 會接收此參數並在測試中加以存取，方式就和從 `MainDialog` 叫用時一樣。

```csharp
private async Task<DialogTurnResult> DestinationStepAsync(WaterfallStepContext stepContext, CancellationToken cancellationToken)
{
    var bookingDetails = (BookingDetails)stepContext.Options;
    ...
}
```

## <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

```javascript
const inputDialogParams = {
    destination: 'Seattle',
    travelDate: formatDate(new Date().setDate(now.getDate() + 1))
};

const sut = new BookingDialog();
const testClient = new DialogTestClient('msteams', sut, inputDialogParams);
```

`BookingDialog` 會接收此參數並可在測試中加以存取，方式就像是從 `MainDialog` 叫用一樣。

```javascript
async destinationStep(stepContext) {
    const bookingDetails = stepContext.options;
    ...
}
```

---

### <a name="asserting-dialog-turn-results"></a>對話回合結果的判斷提示

## <a name="ctabcsharp"></a>[C#](#tab/csharp)

某些對話 (如 `BookingDialog` 或 `DateResolverDialog`) 會對呼叫端對話傳回值。 `DialogTestClient` 物件會公開 `DialogTurnResult` 屬性，以供用來對對話方塊所傳回的結果進行分析和判斷提示。

例如︰

```csharp
var sut = new BookingDialog();
var testClient = new DialogTestClient(Channels.Msteams, sut);

var reply = await testClient.SendActivityAsync<IMessageActivity>("hi");
Assert.Equal("Where would you like to travel to?", reply.Text);

...

var bookingResults = (BookingDetails)testClient.DialogTurnResult.Result;
Assert.Equal("New York", bookingResults?.Origin);
Assert.Equal("Seattle", bookingResults?.Destination);
Assert.Equal("2019-06-21", bookingResults?.TravelDate);
```

`DialogTurnResult` 屬性也可用來對瀑布中的步驟所傳回的中繼結果進行檢查和判斷提示。

## <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

某些對話方塊 (如 `BookingDialog` 或 `DateResolverDialog`) 會對呼叫端對話方塊傳回值。 `DialogTestClient` 物件會公開 `dialogTurnResult` 屬性，以供用來對對話方塊所傳回的結果進行分析和判斷提示。

例如︰

```javascript
const sut = new BookingDialog();
const testClient = new DialogTestClient('msteams', sut);

let reply = await testClient.sendActivity('hi');
assert.strictEqual(reply.text, 'Where would you like to travel to?');

...

const bookingResults = client.dialogTurnResult.result;
assert.strictEqual('New York', bookingResults.destination);
assert.strictEqual('Seattle', bookingResults.origin);
assert.strictEqual('2019-06-21', bookingResults.travelDate);
```

`dialogTurnResult` 屬性也可用來對瀑布中的步驟，所傳回的中繼結果進行檢查和判斷提示。

---

### <a name="analyzing-test-output"></a>分析測試輸出

系統有時候必須讀取單元測試文字記錄以便分析測試的執行情形，而不需要對測試進行偵錯。

## <a name="ctabcsharp"></a>[C#](#tab/csharp)

[Microsoft.Bot.Builder.Testing](https://www.nuget.org/packages/Microsoft.Bot.Builder.Testing/) 套件包含 `XUnitDialogTestLogger`，其會將對話所傳送和接收的訊息記錄到主控台。

若要使用這個中介軟體，測試作業需要公開一個建構函式 (以接收 XUnit 測試執行器所提供的 `ITestOutputHelper` 物件)，並建立會透過 `middlewares` 參數傳遞至 `DialogTestClient` 的 `XUnitDialogTestLogger`。

```csharp
public class BookingDialogTests
{
    private readonly IMiddleware[] _middlewares;

    public BookingDialogTests(ITestOutputHelper output)
        : base(output)
    {
        _middlewares = new[] { new XUnitDialogTestLogger(output) };
    }

    [Fact]
    public async Task SomeBookingDialogTest()
    {
        // Arrange
        var sut = new BookingDialog();
        var testClient = new DialogTestClient(Channels.Msteams, sut, middlewares: _middlewares);

        ...
    }
}
```

以下範例指出 `XUnitDialogTestLogger` 在設定時會記錄至輸出視窗的內容：

![XUnitMiddlewareOutput](media/how-to-unit-test/cs/XUnitMiddlewareOutput.png)

如需使用 XUnit 時要如何將測試輸出傳送至主控台的其他資訊，請參閱 XUnit 文件中的[擷取輸出](https://xunit.net/docs/capturing-output.html)。

## <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

[botbuilder-testing](https://www.npmjs.com/package/botbuilder-testing) 套件包含 `DialogTestLogger`，其會將對話方塊所傳送和接收的訊息記錄到主控台。

若要使用此中介軟體，請直接透過 `middlewares` 參數將其傳遞至 `DialogTestClient` 即可。

```javascript
const client = new DialogTestClient('msteams', sut, testData.initialData, [new DialogTestLogger()]);
```

以下範例指出 `DialogTestLogger` 在設定時會記錄至輸出視窗的內容：

![DialogTestLoggerOutput](media/how-to-unit-test/js/DialogTestLoggerOutput.png)

---

在建置持續整合期間，此輸出也會記錄在組建伺服器上，並協助您分析失敗的建置。

## <a name="data-driven-tests"></a>資料驅動測試

在大部分情況下，對話方塊邏輯都不會變更，而且對話方塊中的不同執行路徑都是以使用者表達作為基礎的。 您不必針對對話中的每一種變化撰寫單一的單元測試，更輕鬆的方式是使用資料驅動測試 (也稱為參數化測試)。

例如，本文件 [概觀] 區段中的測試範例會示範如何測試一個執行流程，但如果使用者拒絕確認會怎樣？如果使用者使用不同的日期會怎樣等等。

資料驅動測試可讓我們測試上述所有情形的排列組合，而不必重新撰寫測試。

## <a name="ctabcsharp"></a>[C#](#tab/csharp)

在 CoreBot 範例中，我們從 XUnit 使用 `Theory` 測試來將測試參數化。

### <a name="theory-tests-using-inlinedata"></a>使用 InlineData 來測試理論

下列測試會確認使用者回答「取消」時，對話是否會取消。

```csharp
[Fact]
public async Task ShouldBeAbleToCancel()
{
    var sut = new TestCancelAndHelpDialog();
    var testClient = new DialogTestClient(Channels.Test, sut);

    var reply = await testClient.SendActivityAsync<IMessageActivity>("Hi");
    Assert.Equal("Hi there", reply.Text);
    Assert.Equal(DialogTurnStatus.Waiting, testClient.DialogTurnResult.Status);

    reply = await testClient.SendActivityAsync<IMessageActivity>("cancel");
    Assert.Equal("Cancelling...", reply.Text);
}
```

若要取消對話，使用者可以輸入「結束」、「沒關係」和「停下來」。 您不必針對每個可能的單字撰寫新的測試案例，而只要撰寫單一的 `Theory` 測試方法，透過 `InlineData` 值的清單接受參數，以定義每個測試案例的參數：

```csharp
[Theory]
[InlineData("cancel")]
[InlineData("quit")]
[InlineData("never mind")]
[InlineData("stop it")]
public async Task ShouldBeAbleToCancel(string cancelUtterance)
{
    var sut = new TestCancelAndHelpDialog();
    var testClient = new DialogTestClient(Channels.Test, sut, middlewares: _middlewares);

    var reply = await testClient.SendActivityAsync<IMessageActivity>("Hi");
    Assert.Equal("Hi there", reply.Text);
    Assert.Equal(DialogTurnStatus.Waiting, testClient.DialogTurnResult.Status);

    reply = await testClient.SendActivityAsync<IMessageActivity>(cancelUtterance);
    Assert.Equal("Cancelling...", reply.Text);
}
```

新測試會使用不同參數執行 4 次，而且每個案例都會在 Visual Studio 測試總管中顯示為 `ShouldBeAbleToCancel` 測試底下的子項目。 如果其中任何一次失敗了 (如下所示)，您可以對失敗的案例按一下滑鼠右鍵並進行偵錯，而不必重新執行整組測試。

![InlineDataTestResults](media/how-to-unit-test/cs/InlineDataTestResults.png)

### <a name="theory-tests-using-memberdata-and-complex-types"></a>使用 MemberData 和複雜類型的理論測試

`InlineData` 適用於會接收簡單實值型別參數 (字串、int 等) 的小型資料驅動測試。

`BookingDialog` 會接收 `BookingDetails` 物件，並傳回新的 `BookingDetails` 物件。 此對話方塊測試的非參數化版本會如下所示：

```csharp
[Fact]
public async Task DialogFlow()
{
    // Initial parameters
    var initialBookingDetails = new BookingDetails
    {
        Origin = "Seattle",
        Destination = null,
        TravelDate = null,
    };

    // Expected booking details
    var expectedBookingDetails = new BookingDetails
    {
        Origin = "Seattle",
        Destination = "New York",
        TravelDate = "2019-06-25",
    };

    var sut = new BookingDialog();
    var testClient = new DialogTestClient(Channels.Test, sut, initialBookingDetails);

    // Act/Assert
    var reply = await testClient.SendActivityAsync<IMessageActivity>("hi");
    ...

    var bookingResults = (BookingDetails)testClient.DialogTurnResult.Result;
    Assert.Equal(expectedBookingDetails.Origin, bookingResults?.Origin);
    Assert.Equal(expectedBookingDetails.Destination, bookingResults?.Destination);
    Assert.Equal(expectedBookingDetails.TravelDate, bookingResults?.TravelDate);
}
```

為了將此測試參數化，我們建立了一個包含測試案例資料的 `BookingDialogTestCase` 類別。 其包含初始的 `BookingDetails` 物件、預期的 `BookingDetails` 和字串陣列，此字串陣列包含使用者所傳來的表達，以及每一回合對話的預期回覆。

```csharp
public class BookingDialogTestCase
{
    public BookingDetails InitialBookingDetails { get; set; }

    public string[,] UtterancesAndReplies { get; set; }

    public BookingDetails ExpectedBookingDetails { get; set; }
}
```

我們也建立了一個協助程式 `BookingDialogTestsDataGenerator` 類別，其會公開 `IEnumerable<object[]> BookingFlows()` 方法，以傳回要供測試使用的測試案例集合。

為了在 Visual Studio 測試總管中分開顯示每個測試案例項目，XUnit 測試執行器會要求 `BookingDialogTestCase` 等複雜類型實作 `IXunitSerializable`，而為了簡化此程序，Bot.Builder.Testing 架構提供了 `TestDataObject` 類別，以實作此介面並可供用來包裝測試案例資料，而不必實作 `IXunitSerializable`。 

以下是 `IEnumerable<object[]> BookingFlows()` 的片段，其可說明這兩個類別的使用方式：

```csharp
public static class BookingDialogTestsDataGenerator
{
    public static IEnumerable<object[]> BookingFlows()
    {
        // Create the first test case object
        var testCaseData = new BookingDialogTestCase
        {
            InitialBookingDetails = new BookingDetails(),
            UtterancesAndReplies = new[,]
            {
                { "hi", "Where would you like to travel to?" },
                { "Seattle", "Where are you traveling from?" },
                { "New York", "When would you like to travel?" },
                { "tomorrow", $"Please confirm, I have you traveling to: Seattle from: New York on: {DateTime.Now.AddDays(1):yyyy-MM-dd}. Is this correct? (1) Yes or (2) No" },
                { "yes", null },
            },
            ExpectedBookingDetails = new BookingDetails
            {
                Destination = "Seattle",
                Origin = "New York",
                TravelDate = $"{DateTime.Now.AddDays(1):yyyy-MM-dd}",
            }, 
        };
        // wrap the test case object into TestDataObject and return it.
        yield return new object[] { new TestDataObject(testCaseData) };

        // Create the second test case object
        testCaseData = new BookingDialogTestCase
        {
            InitialBookingDetails = new BookingDetails
            {
                Destination = "Seattle",
                Origin = "New York",
                TravelDate = null,
            },
            UtterancesAndReplies = new[,]
            {
                { "hi", "When would you like to travel?" },
                { "tomorrow", $"Please confirm, I have you traveling to: Seattle from: New York on: {DateTime.Now.AddDays(1):yyyy-MM-dd}. Is this correct? (1) Yes or (2) No" },
                { "yes", null },
            },
            ExpectedBookingDetails = new BookingDetails
            {
                Destination = "Seattle",
                Origin = "New York",
                TravelDate = $"{DateTime.Now.AddDays(1):yyyy-MM-dd}",
            },
        };
        // wrap the test case object into TestDataObject and return it.
        yield return new object[] { new TestDataObject(testCaseData) };
    }
}
```

在建立了用來儲存測試資料的物件以及會公開測試案例集合的類別之後，我們會使用 XUnit 的 `MemberData` 屬性 (而非 `InlineData`) 將資料饋送至測試中，`MemberData` 的第一個參數是靜態函式的名稱，其會傳回測試案例的集合，第二個參數則是類別的型別，其會公開這個方法。

```csharp
[Theory]
[MemberData(nameof(BookingDialogTestsDataGenerator.BookingFlows), MemberType = typeof(BookingDialogTestsDataGenerator))]
public async Task DialogFlowUseCases(TestDataObject testData)
{
    // Get the test data instance from TestDataObject
    var bookingTestData = testData.GetObject<BookingDialogTestCase>();
    var sut = new BookingDialog();
    var testClient = new DialogTestClient(Channels.Test, sut, bookingTestData.InitialBookingDetails);

    // Iterate over the utterances and replies array.
    for (var i = 0; i < bookingTestData.UtterancesAndReplies.GetLength(0); i++)
    {
        var reply = await testClient.SendActivityAsync<IMessageActivity>(bookingTestData.UtterancesAndReplies[i, 0]);
        Assert.Equal(bookingTestData.UtterancesAndReplies[i, 1], reply?.Text);
    }

    // Assert the resulting BookingDetails object
    var bookingResults = (BookingDetails)testClient.DialogTurnResult.Result;
    Assert.Equal(bookingTestData.ExpectedBookingDetails?.Origin, bookingResults?.Origin);
    Assert.Equal(bookingTestData.ExpectedBookingDetails?.Destination, bookingResults?.Destination);
    Assert.Equal(bookingTestData.ExpectedBookingDetails?.TravelDate, bookingResults?.TravelDate);
}
```

以下是執行測試時，Visual Studio 測試總管中 `DialogFlowUseCases` 測試結果的範例：

![BookingDialogTests](media/how-to-unit-test/cs/BookingDialogTestsResults.png)

## <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

### <a name="simple-data-driven-tests"></a>資料驅動測試範例

下列測試會確認使用者回答「取消」時，對話是否會取消。

```javascript
describe('ShouldBeAbleToCancel', () => {
    it('Should cancel', async () => {
        const sut = new TestCancelAndHelpDialog();
        const client = new DialogTestClient('test', sut, null, [new DialogTestLogger()]);

        // Execute the test case
        let reply = await client.sendActivity('Hi');
        assert.strictEqual(reply.text, 'Hi there');
        assert.strictEqual(client.dialogTurnResult.status, 'waiting');

        reply = await client.sendActivity('cancel');
        assert.strictEqual(reply.text, 'Cancelling...');
    });
});
```

請考慮到一點，稍後我們必須能夠處理其他「取消」表達，例如「結束」、「沒關係」和「停下來」。 我們不必為每個新的表達另外撰寫 3 個重複測試，而是可以重構測試，使用表達清單來定義每個測試案例的參數：

```javascript
describe('ShouldBeAbleToCancel', () => {
    const testCases = ['cancel', 'quit', 'never mind', 'stop it'];

    testCases.map(testData => {
        it(testData, async () => {
            const sut = new TestCancelAndHelpDialog();
            const client = new DialogTestClient('test', sut, null, [new DialogTestLogger()]);

            // Execute the test case
            let reply = await client.sendActivity('Hi');
            assert.strictEqual(reply.text, 'Hi there');
            assert.strictEqual(client.dialogTurnResult.status, 'waiting');

            reply = await client.sendActivity(testData);
            assert.strictEqual(reply.text, 'Cancelling...');
        });
    });
});
```

新測試會使用不同參數執行 4 次，而且每個案例都會在 [Mocha Test Explorer](https://marketplace.visualstudio.com/items?itemName=hbenl.vscode-mocha-test-adapter) 中顯示為 `ShouldBeAbleToCancel` 測試套件底下的子項目。 如果其中任何一次失敗了 (如下所示)，您可以執行失敗的案例並對其進行偵錯，而不必重新執行整組測試。

![SimpleCancelTestResults](media/how-to-unit-test/js/SimpleCancelTestResults.png)

### <a name="data-driven-tests-with-complex-types"></a>具有複雜類型的資料驅動測試

使用簡單的表達清單適用於會接收簡單實值型別參數 (字串、int 等) 或小型物件的小型資料驅動測試。

`BookingDialog` 會接收 `BookingDetails` 物件，並傳回新的 `BookingDetails` 物件。 此對話方塊測試的非參數化版本會如下所示：

```javascript
describe('BookingDialog', () => {
    it('Returns expected booking details', async () => {
        // Initial parameters
        const initialBookingDetails = {
            origin: 'Seattle',
            destination: undefined,
            travelDate: undefined
        };

        // Expected booking details
        const expectedBookingDetails = {
            origin: 'Seattle',
            destination: 'New York',
            travelDate: '2019-06-25'
        };

        const sut = new BookingDialog('bookingDialog');
        const client = new DialogTestClient('test', sut, initialBookingDetails, [new DialogTestLogger()]);

        // Execute the test case
        const reply = await client.sendActivity('Hi');
        ...

        // Check dialog results
        const result = client.dialogTurnResult.result;
        assert.strictEqual(result.destination, expectedBookingDetails.destination);
        assert.strictEqual(result.origin, expectedBookingDetails.origin);
        assert.strictEqual(result.travelDate, expectedBookingDetails.travelDate);
    });
});
```

為了將此測試參數化，我們建立了一個會傳回測試案例資料的 `bookingDialogTestCases` 模組。 每個項目都包含測試案例名稱、初始的 'BookingDetails' 物件、預期的 'BookingDetails' 和字串陣列，此字串陣列具有使用者所傳來的表達，以及每一回合對話方塊的預期回覆。

```javascript
module.exports = [
    // Create the first test case object
    {
        name: 'Full flow',
        initialData: {},
        steps: [
            ['hi', 'To what city would you like to travel?'],
            ['Seattle', 'From what city will you be travelling?'],
            ['New York', 'On what date would you like to travel?'],
            ['tomorrow', `Please confirm, I have you traveling to: Seattle from: New York on: ${ tomorrow }. Is this correct? (1) Yes or (2) No`],
            ['yes', null]
        ],
        expectedResult: {
            destination: 'Seattle',
            origin: 'New York',
            travelDate: tomorrow
        }
    },
    // Create the second test case object
    {
        name: 'Destination and Origin provided',
        initialData: {
            destination: 'Seattle',
            origin: 'New York'
        },
        steps: [
            ['hi', 'On what date would you like to travel?'],
            ['tomorrow', `Please confirm, I have you traveling to: Seattle from: New York on: ${ tomorrow }. Is this correct? (1) Yes or (2) No`],
            ['yes', null]
        ],
        expectedStatus: 'complete',
        expectedResult: {
            destination: 'Seattle',
            origin: 'New York',
            travelDate: tomorrow
        }
    }
];
```

在建立了包含測試資料的清單後，我們可以重構測試以將此清單對應至個別的測試案例。

```javascript
describe('DialogFlowUseCases', () => {
    const testCases = require('./testData/bookingDialogTestCases.js');

    testCases.map(testData => {
        it(testData.name, async () => {
            const sut = new BookingDialog('bookingDialog');
            const client = new DialogTestClient('test', sut, testData.initialData, [new DialogTestLogger()]);

            // Execute the test case
            for (let i = 0; i < testData.steps.length; i++) {
                const reply = await client.sendActivity(testData.steps[i][0]);
                assert.strictEqual((reply ? reply.text : null), testData.steps[i][1]);
            }

            // Check dialog results
            const actualResult = client.dialogTurnResult.result;
            assert.strictEqual(actualResult.destination, testData.expectedResult.destination);
            assert.strictEqual(actualResult.origin, testData.expectedResult.origin);
            assert.strictEqual(actualResult.travelDate, testData.expectedResult.travelDate);
        });
    });
});
```

以下是執行測試套件時，Mocha Test Explorer 中 `DialogFlowUseCases` 測試套件結果的範例：

![BookingDialogTests](media/how-to-unit-test/js/BookingDialogTestsResults.png)

---

## <a name="using-mocks"></a>使用模擬

您可以將模擬元素用於目前未進行測試的項目。 做為參考，此等級通常可視為單元和整合測試。

盡可能地模擬多個元素，可以更有效地隔離您要測試的部分。 模擬元素的對象包括儲存體、配接器、中介軟體、活動管線、通道，以及任何其他非直屬於 Bot 的組件。 這也可能涉及暫時移除某些部分 (例如與您要測試之 Bot 無關的中介軟體)，以便隔離每個部分。 不過，如果您要測試中介軟體，建議可以改為模擬 Bot。

模擬元素可以採取數種形式，從以不同的已知物件取代元素，到實作最基本的 Hello World 功能。 另外可採取的形式如，假設某元素不是必要元素，則可單純移除元素，或是強制它不執行任何動作。

模擬可讓我們設定對話方塊的相依性，並確保相依性在測試執行期間處於已知狀態，而不必依賴資料庫、LUIS 模型或其他物件等外部資源。

為了讓對話方塊更容易進行測試，並減少其對外部物件的相依性，您可能需要將外部相依性插入到對話方塊建構函式中。

例如，不要在 `MainDialog` 中具現化 `BookingDialog`︰

## <a name="ctabcsharp"></a>[C#](#tab/csharp)

```csharp
public MainDialog()
    : base(nameof(MainDialog))
{
    ...
    AddDialog(new BookingDialog());
    ...
}
```

## <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

```javascript
constructor() {
    super('MainDialog');
    ...
    this.addDialog(new BookingDialog('bookingDialog'));
    ...
}
```

---

我們會以建構函式參數的形式傳遞 `BookingDialog` 的執行個體：

## <a name="ctabcsharp"></a>[C#](#tab/csharp)

```csharp
public MainDialog(BookingDialog bookingDialog)
    : base(nameof(MainDialog))
{
    ...
    AddDialog(bookingDialog);
    ...
}
```

## <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

```javascript
constructor(bookingDialog) {
    super('MainDialog');
    ...
    this.addDialog(bookingDialog);
    ...
}
```

---

這可讓我們將 `BookingDialog` 執行個體取代為模擬物件並撰寫 `MainDialog` 的單元測試，而不必呼叫實際的 `BookingDialog` 類別。

## <a name="ctabcsharp"></a>[C#](#tab/csharp)

```csharp
// Create the mock object
var mockDialog = new Mock<BookingDialog>();

// Use the mock object to instantiate MainDialog
var sut = new MainDialog(mockDialog.Object);

var testClient = new DialogTestClient(Channels.Test, sut);
```

## <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

```javascript
// Create the mock object
const mockDialog = new MockBookingDialog();

// Use the mock object to instantiate MainDialog
const sut = new MainDialog(mockDialog);

const testClient = new DialogTestClient('test', sut);
```

---

### <a name="mocking-dialogs"></a>模擬對話方塊

如上所述，`MainDialog` 會叫用 `BookingDialog` 來取得 `BookingDetails` 物件。 我們會實作並設定 `BookingDialog` 的模擬執行個體，如下所示：

## <a name="ctabcsharp"></a>[C#](#tab/csharp)

```csharp
// Create the mock object for BookingDialog.
var mockDialog = new Mock<BookingDialog>();
mockDialog
    .Setup(x => x.BeginDialogAsync(It.IsAny<DialogContext>(), It.IsAny<object>(), It.IsAny<CancellationToken>()))
    .Returns(async (DialogContext dialogContext, object options, CancellationToken cancellationToken) =>
    {
        // Send a generic activity so we can assert that the dialog was invoked.
        await dialogContext.Context.SendActivityAsync($"{mockDialogNameTypeName} mock invoked", cancellationToken: cancellationToken);

        // Create the BookingDetails instance we want the mock object to return.
        var expectedBookingDialogResult = new BookingDetails()
        {
            Destination = "Seattle",
            Origin = "New York",
            TravelDate = $"{DateTime.UtcNow.AddDays(1):yyyy-MM-dd}"
        };

        // Return the BookingDetails we need without executing the dialog logic.
        return await dialogContext.EndDialogAsync(expectedBookingDialogResult, cancellationToken);
    });

// Create the sut (System Under Test) using the mock booking dialog.
var sut = new MainDialog(mockDialog.Object);
```

在此範例中，我們使用了 [Moq](https://github.com/moq/moq) 來建立模擬對話方塊，並使用 `Setup` 和 `Returns` 方法來設定其行為。

## <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

```javascript
class MockBookingDialog extends BookingDialog {
    constructor() {
        super('bookingDialog');
    }

    async beginDialog(dc, options) {
        // Send a generic activity so we can assert that the dialog was invoked.
        await dc.context.sendActivity(`${ this.id } mock invoked`);

        // Create the BookingDetails instance we want the mock object to return.
        const bookingDetails = {
            origin: 'New York',
            destination: 'Seattle',
            travelDate: '2025-07-08'
        };

        // Return the BookingDetails we need without executing the dialog logic.
        return await dc.endDialog(bookingDetails);
    }
}
...
// Create the sut (System Under Test) using the mock booking dialog.
const sut = new MainDialog(new MockBookingDialog());
...

```

在此範例中，我們會從 `BookingDialog` 衍生並覆寫 `beginDialog` 方法以略過基礎對話邏輯，藉此來實作模擬對話方塊。

---

### <a name="mocking-luis-results"></a>模擬 LUIS 結果

在簡單案例中，您可以透過程式碼來實作模擬 LUIS 結果，如下所示：

## <a name="ctabcsharp"></a>[C#](#tab/csharp)

```csharp
var mockRecognizer = new Mock<IRecognizer>();
mockRecognizer
    .Setup(x => x.RecognizeAsync<FlightBooking>(It.IsAny<ITurnContext>(), It.IsAny<CancellationToken>()))
    .Returns(() =>
    {
        var luisResult = new FlightBooking
        {
            Intents = new Dictionary<FlightBooking.Intent, IntentScore>
            {
                { FlightBooking.Intent.BookFlight, new IntentScore() { Score = 1 } },
            },
            Entities = new FlightBooking._Entities(),
        };
        return Task.FromResult(luisResult);
    });
```

## <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

```javascript

// Create a mock class for the recognizer that overrides executeLuisQuery.
class MockFlightBookingRecognizer extends FlightBookingRecognizer {
    constructor(mockResult) {
        super();
        this.mockResult = mockResult;
    }

    async executeLuisQuery(context) {
        return this.mockResult;
    }
}
...
// Create a mock result from a string
const mockLuisResult = JSON.parse(`{"intents": {"BookFlight": {"score": 1}}, "entities": {"$instance": {}}}`);
// Use the mock result with the mock recognizer.
const mockRecognizer = new MockFlightBookingRecognizer(mockLuisResult);
...
```

---

但是 LUIS 結果可能會很複雜，如果確實如此，您可以簡化為以 json 檔案擷取想要的結果、將其作為資源新增至專案中，然後將其還原序列化為 LUIS 結果。 下列是一個範例：

## <a name="ctabcsharp"></a>[C#](#tab/csharp)

```csharp
var mockRecognizer = new Mock<IRecognizer>();
mockRecognizer
    .Setup(x => x.RecognizeAsync<FlightBooking>(It.IsAny<ITurnContext>(), It.IsAny<CancellationToken>()))
    .Returns(() =>
    {
        // Deserialize the LUIS result from embedded json file in the TestData folder.
        var bookingResult = GetEmbeddedTestData($"{GetType().Namespace}.TestData.FlightToMadrid.json");

        // Return the deserialized LUIS result.
        return Task.FromResult(bookingResult);
    });
```

## <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

```javascript
// Create a mock result from a json file
const mockLuisResult = require(`./testData/FlightToMadrid.json`);
// Use the mock result with the mock recognizer.
const mockRecognizer = new MockFlightBookingRecognizer(mockLuisResult);
```

---

## <a name="additional-information"></a>其他資訊

- [CoreBot 測試範例 (C#)](https://aka.ms/cs-core-test-sample)
- [CoreBot 測試範例 (JavaScript)](https://aka.ms/js-core-test-sample)
- [聊天機器人測試](https://github.com/microsoft/botframework-sdk/blob/master/specs/testing/testing.md)
