---
title: 將自然語言理解新增至您的 Bot | Microsoft Docs
description: 了解如何搭配 Bot Framework SDK 將 LUIS 用於自然語言理解。
keywords: Language Understanding, LUIS, intent, recognizer, entities, middleware, 意圖, 辨識器, 實體, 中介軟體
author: ivorb
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 05/23/2019
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 1641260f6673a810e7bc71ecaca1ada234286e42
ms.sourcegitcommit: 008aa6223aef800c3abccda9a7f72684959ce5e7
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/26/2019
ms.locfileid: "70026322"
---
# <a name="add-natural-language-understanding-to-your-bot"></a>將自然語言理解新增至您的 Bot

[!INCLUDE[applies-to](../includes/applies-to.md)]

讓您的 Bot 能透過對話和上下文了解使用者的意思，並不是簡單的工作，但這樣的功能可讓您的 Bot 使用起來更有自然對話的感覺。 Language Understanding (稱為 LUIS) 能讓您這麼做，使您的 Bot 可以辨識使用者訊息的意圖，這樣使用者就能使用更自然的語言，且更順利地引導交談流程。 本主題會逐步引導您將 LUIS 新增至航班預訂應用程式，以辨識使用者輸入內含的不同意圖和實體。 

## <a name="prerequisites"></a>必要條件
- [LUIS](https://www.luis.ai) 帳戶
- 本文中的程式碼是以 **Core Bot** 範例為基礎。 您需要 **[CSharp](https://aka.ms/cs-core-sample) 或 [JavaScript](https://aka.ms/js-core-sample)** 中的一份範例。 
- [Bot 基本概念](bot-builder-basics.md)、[自然語言處理](https://docs.microsoft.com/azure/cognitive-services/luis/what-is-luis)和 [管理 Bot 資源](bot-file-basics.md)的知識。

## <a name="about-this-sample"></a>關於此範例

此 Core Bot 編碼範例顯示機場航班預訂應用程式的範例。 其會使用 LUIS 服務來辨識使用者輸入，並傳回最常辨識的 LUIS 意圖。 

# <a name="ctabcsharp"></a>[C#](#tab/csharp)
在每此處理使用者輸入之後，`DialogBot` 會儲存 `UserState` 和 `ConversationState` 的目前狀態。 蒐集所有必要的資訊後，程式碼範例會建立示範航班預訂保留。 這本文中，我們將探討這的範例的 LUIS 層面。 不過，此範例的一般流程如下所示：

- 當新的使用者連線並顯示歡迎卡片時，就會呼叫 `OnMembersAddedAsync`。 
- 系統會針對每個收到的使用者輸入呼叫 `OnMessageActivityAsync`。

![LUIS 範例的邏輯流程](./media/how-to-luis/luis-logic-flow.png)

`OnMessageActivityAsync` 模組會透過 `Run` 對話擴充方法執行適當的對話。 然後，該主要對話方塊會呼叫 LUIS 協助程式，以尋找評分最高的使用者意圖。 如果使用者輸入的最高意圖傳回 "BookFlight"，則協助程式會填寫 LUIS 傳回的使用者資訊。 之後，主要對話方塊會啟動 `BookingDialog`，其會視需要向使用者取得其他資訊，例如：

- `Origin` 原始城市
- `TravelDate` 要預訂航班的日期
- `Destination` 目的地城市

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)
在每此處理使用者輸入之後，`dialogBot` 會儲存 `userState` 和 `conversationState` 的目前狀態。 蒐集所有必要的資訊後，程式碼範例會建立示範航班預訂保留。 這本文中，我們將探討這的範例的 LUIS 層面。 不過，此範例的一般流程如下所示：

- 當新的使用者連線並顯示歡迎卡片時，就會呼叫 `onMembersAdded`。 
- 系統會針對每個收到的使用者輸入呼叫 `OnMessage`。

![LUIS 範例的 javascript 邏輯流程](./media/how-to-luis/luis-logic-flow-js.png)

`onMessage` 模組會執行 `mainDialog` 以蒐集使用者輸入。
然後，該主要對話方塊會呼叫 LUIS 協助程式 `FlightBookingRecognizer`，以尋找評分最高的使用者意圖。 如果使用者輸入的最高意圖傳回 "BookFlight"，則協助程式會填寫 LUIS 傳回的使用者資訊。
回應傳來時，`mainDialog` 會保留 LUIS 所傳回的使用者資訊並啟動 `bookingDialog`。 `bookingDialog` 會視需要向使用者取得其他資訊，例如：

- `destination` 目的地城市。
- `origin` 原始城市。
- `travelDate` 要預訂航班的日期。

---

如需範例的其他層面 (例如對話或狀態) 詳細資訊，請參閱[使用對話提示蒐集使用者輸入](bot-builder-prompts.md)或[儲存使用者和交談資料](bot-builder-howto-v4-state.md)。

## <a name="create-a-luis-app-in-the-luis-portal"></a>在 LUIS 入口網站中建立 LUIS 應用程式
登入 LUIS 入口網站以建立自己的範例 LUIS 應用程式版本。 您可以在 [我的應用程式]  上建立和管理應用程式。

1. 選取 [匯入新的應用程式]  。 
1. 按一下 [選擇應用程式檔案 (JSON 格式)...]  
1. 選取 `FlightBooking.json` 檔案，該檔案位於範例的 `CognitiveModels` 資料夾中。 在 [選擇性名稱]  中，輸入 **FlightBooking**。 此檔案包含三個意圖：[預訂航班]、[取消] 和 [無]。 我們將使用這些意圖，了解使用者將訊息傳送給 Bot 時的用意。
1. [訓練](https://docs.microsoft.com/azure/cognitive-services/LUIS/luis-how-to-train)應用程式。
1. 將應用程式[發佈](https://docs.microsoft.com/azure/cognitive-services/LUIS/publishapp)到「生產」  環境。

### <a name="why-use-entities"></a>為何使用實體
LUIS 實體可讓 Bot 以智慧方式了解與標準意圖不同的特定事項或事件。 這可讓您向使用者收集額外資訊，進而讓 Bot 更聰明地回應，或可能略過要求使用者提供該資訊的某些問題。 除了三個 LUIS 意圖 [預訂航班]、[取消] 和 [無] 的定義，FlightBooking.json 檔案還包含一組實體，例如 'From.Airport' 和 'To.Airport'。 這些實體可讓 LUIS 偵測使用者原始輸入內含的其他資訊，並且在使用者要求新的旅行預約時傳回這些資訊。

如需有關如何在 LUIS 結果中顯示實體資訊的詳細資訊，請參閱[從具有意圖和實體的語句文字中擷取資料](https://docs.microsoft.com/azure/cognitive-services/luis/luis-concept-data-extraction)。

## <a name="obtain-values-to-connect-to-your-luis-app"></a>取得值以連線到您的 LUIS 應用程式
在您的 LUIS 應用程式發佈之後，您即可從 Bot 進行存取。 您必須記錄數個值，以從 Bot 存取您的 LUIS 應用程式。 您可以使用 LUIS 入口網站擷取該資訊。

### <a name="retrieve-application-information-from-the-luisai-portal"></a>從 LUIS.ai 入口網站擷取應用程式資訊
settings 檔案 (`appsettings.json` 或 `.env`) 檔案可作為將所有服務參考匯合在一起的位置。 您所擷取的資訊會新增至下一節中的這個檔案。 
1. 從 [luis.ai](https://www.luis.ai) 中選取已發佈的 LUIS 應用程式。
1. 開啟已發佈的 LUIS 應用程式後，選取 [管理]  索引標籤。![管理 LUIS 應用程式](./media/how-to-luis/manage-luis-app.png)
1. 選取左側的 [應用程式資訊]  索引標籤，將針對 [應用程式識別碼]  顯示的值記錄為 <YOUR_APP_ID>。
1. 選取左側 [金鑰和端點]  索引標籤，將針對 [撰寫金鑰]  所顯示的值記錄為 <YOUR_AUTHORING_KEY>。
1. 向下捲動到頁面結尾，將針對 [地區]  顯示的值記錄為 <YOUR_REGION>。

### <a name="update-the-settings-file"></a>更新 settings 檔案

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

將存取 LUIS 應用程式所需的資訊 (包括應用程式識別碼、撰寫金鑰和區域) 新增至 `appsettings.json` 檔案中。 這些是您先前從已發佈的 LUIS 應用程式儲存的值。 請注意，API 主機名稱應該採用 `<your region>.api.cognitive.microsoft.com` 格式。

**appsetting.json**  
[!code-json[appsettings](~/../BotBuilder-Samples/samples/csharp_dotnetcore/13.core-bot/appsettings.json?range=1-7)]

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

將存取 LUIS 應用程式所需的資訊 (包括應用程式識別碼、撰寫金鑰和區域) 新增至 `.env` 檔案中。 這些是您先前從已發佈的 LUIS 應用程式儲存的值。 請注意，API 主機名稱應該採用 `<your region>.api.cognitive.microsoft.com` 格式。

**.env**  
[!code[env](~/../BotBuilder-Samples/samples/javascript_nodejs/13.core-bot/.env?range=1-5)]

---

## <a name="configure-your-bot-to-use-your-luis-app"></a>設定 Bot 來使用 LUIS 應用程式

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

請確定您已為專案安裝 **Microsoft.Bot.Builder.AI.Luis** NuGet 套件。

若要連線到 LUIS 服務，Bot 會從 appsetting.json 檔案提取您前面新增的資訊。 `FlightBookingRecognizer` 類別包含的程式碼有來自 appsetting.json 檔案的設定，並且會藉由呼叫 `RecognizeAsync` 方法來查詢 LUIS 服務。

**FlightBookingRecognizer.cs**  

[!code-csharp[luisHelper](~/../BotBuilder-Samples/samples/csharp_dotnetcore/13.core-bot/FlightBookingRecognizer.cs?range=12-39)]

`FlightBookingEx.cs` 的邏輯會擷取 *From*、*To* 和 *TravelDate*；其會延伸在從 `MainDialog.cs` 呼叫 `FlightBookingRecognizer.RecognizeAsync<FlightBooking>` 時用來儲存 LUIS 結果的部分類別 `FlightBooking.cs`。

**CognitiveModels\FlightBookingEx.cs**  

[!code-csharp[luis helper](~/../BotBuilder-Samples/samples/csharp_dotnetcore/13.core-bot/CognitiveModels/FlightBookingEx.cs?range=8-35)]

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

若要使用 LUIS，您的專案需要安裝 **botbuilder-ai** npm 套件。

為了連線至 LUIS 服務，聊天機器人會從 `.env` 檔案使用您前面所新增的資訊。 `flightBookingRecognizer.js` 類別包含的程式碼可從 `.env` 檔案匯入您的設定，以及藉由呼叫 `recognize()` 方法來查詢 LUIS 服務。

**dialogs/flightBookingRecognizer.js**

[!code-javascript[luis helper](~/../BotBuilder-Samples/samples/javascript_nodejs/13.core-bot/dialogs/flightBookingRecognizer.js?range=6-64)]

用來擷取 From、To 和 TravelDate 的邏輯會在 `flightBookingRecognizer.js` 內實作為協助程式方法。 從 `mainDialog.js` 呼叫 `flightBookingRecognizer.executeLuisQuery()` 之後便會使用這些方法

---

現在已針對您的 Bot 設定和連線 LUIS。

## <a name="test-the-bot"></a>測試 Bot

下載並安裝最新版 [Bot Framework Emulator](https://aka.ms/bot-framework-emulator-readme)

1. 在您的電腦本機執行範例。 如需相關指示，請參閱 [C# 範例](https://aka.ms/cs-core-sample)或 [JS 範例](https://aka.ms/js-core-sample)的讀我檔案。

1. 在模擬器中，輸入訊息，例如「到巴黎旅行」或「從巴黎到柏林」。 使用在 FlightBooking.json 檔案中找到的任何語句來訓練「預訂航班」意圖。

![LUIS 預訂輸入](./media/how-to-luis/luis-user-travel-input.png)

如果 LUIS 傳回的最高意圖解析為「預訂航班」，您的 Bot 會詢問其他問題，直到儲存足夠的資訊來建立旅行預訂為止。 屆時，會將此預訂資訊傳回給您的使用者。 

![LUIS 預訂結果](./media/how-to-luis/luis-travel-result.png)

此時，程式碼 Bot 邏輯會重設，而您可以繼續建立其他預訂。 

## <a name="next-steps"></a>後續步驟

> [!div class="nextstepaction"]
> [使用 QnA Maker 回答問題](./bot-builder-howto-qna.md)
