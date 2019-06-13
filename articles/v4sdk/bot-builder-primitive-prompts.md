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
ms.date: 05/23/2019
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 9b1ffd73b4b68e6ff6349110e1485eb7cbda9e25
ms.sourcegitcommit: e276008fb5dd7a37554e202ba5c37948954301f1
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/05/2019
ms.locfileid: "66693715"
---
# <a name="create-your-own-prompts-to-gather-user-input"></a>建立您自己的提示，以收集使用者輸入

[!INCLUDE[applies-to](../includes/applies-to.md)]

Bot 和使用者之間的對話，通常牽涉到要求 (提示) 使用者輸入資訊、剖析使用者的回應，然後根據該資訊行動。 Bot 應該追蹤交談的內容，以便管理其行為，以及記住先前問題的答案。 Bot 的「狀態」  是其所追蹤的資訊，以便適當地回應內送訊息。 

> [!TIP]
> 對話方塊程式庫會提供內建提示，讓使用者可使用更多的功能。 您可以在[實作循序對話流程](bot-builder-dialog-manage-conversation-flow.md)文章中找到這些提示的範例。

## <a name="prerequisites"></a>必要條件

- 本文中的程式碼是以「提示使用者輸入」範例為基礎。 您需要一份 **[C# 範例](https://aka.ms/cs-primitive-prompt-sample)或 [JavaScript 範例](https://aka.ms/js-primitive-prompt-sample)** 。
- [管理狀態](bot-builder-concept-state.md)以及如何[儲存使用者和交談資料](bot-builder-howto-v4-state.md)的知識。

## <a name="about-the-sample-code"></a>關於範例程式碼

範例 Bot 會詢問使用者一連串的問題，驗證他們的一些答案，以及儲存其輸入。 下圖顯示 Bot、使用者設定檔及對話流程類別之間的關聯性。 

## <a name="ctabcsharp"></a>[C#](#tab/csharp)
![custom-prompts](media/CustomPromptBotSample-Overview.png)

- Bot 將要收集的使用者資訊的 `UserProfile` 類別。
- `ConversationFlow` 類別會控制收集使用者資訊時的對話狀態。
- 內部 `ConversationFlow.Question` 列舉可用於追蹤我們在交談中的位置。

## <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)
![custom-prompts](media/CustomPromptBotSample-JS-Overview.png)

- Bot 將要收集的使用者資訊的 `userProfile` 類別。
- `conversationFlow` 類別會控制收集使用者資訊時的對話狀態。
- 內部 `conversationFlow.question` 列舉可用於追蹤我們在交談中的位置。

---

使用者狀態會追蹤使用者的名稱、年齡與所選日期，而對話狀態會追蹤我們剛才詢問使用者的內容。
因為我們不打算部署此 Bot，所以我們會將使用者和交談狀態設定為使用「記憶體儲存體」  。 

我們會使用 Bot 的訊息回合處理常式加上使用者和對話狀態屬性，來管理對話流程和輸入集合。 在 Bot 中，我們會記錄每個訊息回合處理常式循環期間收到的狀態屬性資訊。

## <a name="create-conversation-and-user-objects"></a>建立對話和使用者物件

## <a name="ctabcsharp"></a>[C#](#tab/csharp)

使用者和對話狀態物件會在啟動時建立，而相依性會插入 Bot 建構函式。 

**Startup.cs**  
[!code-csharp[Startup](~/../botbuilder-samples/samples/csharp_dotnetcore/44.prompt-users-for-input/Startup.cs?range=27-34)]

**Bots/CustomPromptBot.cs**  
[!code-csharp[custom prompt bot](~/../botbuilder-samples/samples/csharp_dotnetcore/44.prompt-users-for-input/Bots/CustomPromptBot.cs?range=21-28)]

## <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

在 **index.js** 中，建立狀態屬性和 Bot，然後從 `processActivity` 內呼叫 `run` Bot 方法。

[!code-javascript[custom prompt bot](~/../botbuilder-samples/samples/javascript_nodejs/44.prompt-for-user-input/index.js?range=32-35)]

[!code-javascript[custom prompt bot](~/../botbuilder-samples/samples/javascript_nodejs/44.prompt-for-user-input/index.js?range=55-58)]

---

## <a name="create-property-accessors"></a>建立屬性存取子

## <a name="ctabcsharp"></a>[C#](#tab/csharp)

我們會從建立屬性存取子開始，好讓我們有 `OnMessageActivityAsync` 方法中 `BotState` 的控制代碼。 然後，我們會呼叫 `GetAsync` 方法來取得正確範圍的金鑰：

**Bots/CustomPromptBot.cs**  
[!code-csharp[custom prompt bot](~/../botbuilder-samples/samples/csharp_dotnetcore/44.prompt-users-for-input/Bots/CustomPromptBot.cs?range=30-37)]

最後，我們會使用 `SaveChangesAsync` 方法來儲存資料。

[!code-csharp[custom prompt bot](~/../botbuilder-samples/samples/csharp_dotnetcore/44.prompt-users-for-input/Bots/CustomPromptBot.cs?range=42-43)]

## <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

在建構函式中，我們會為對話建立狀態屬性存取子，並設定狀態管理物件 (如上所建)。

**bots/customPromptBot.js**  
[!code-javascript[custom prompt bot](~/../botbuilder-samples/samples/javascript_nodejs/44.prompt-for-user-input/bots/customPromptBot.js?range=23-29)]

然後，我們會定義在主要訊息處理常式之後執行的第二個處理常式 (`onDialog`)，我們將在下一節說明。 第二個處理常式可確保我們會儲存每一回合的狀態。

[!code-javascript[custom prompt bot](~/../botbuilder-samples/samples/javascript_nodejs/44.prompt-for-user-input/bots/customPromptBot.js?range=41-48)]

---

## <a name="the-bots-message-turn-handler"></a>Bot 的訊息回合處理常式

## <a name="ctabcsharp"></a>[C#](#tab/csharp)

若要處理訊息活動，我們會先使用協助程式方法 FillOutUserProfileAsync()  ，然後再使用 SaveChangesAsync()  儲存狀態。 以下是完整程式碼。

**Bots/CustomPromptBot.cs**  
[!code-csharp[custom prompt bot](~/../botbuilder-samples/samples/csharp_dotnetcore/44.prompt-users-for-input/Bots/CustomPromptBot.cs?range=30-44)]

## <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

若要處理訊息活動，我們會設定對話和使用者資料，然後使用協助程式方法 `fillOutUserProfile()`。 以下是回合處理常式的完整程式碼。

**bots/customPromptBot.js**  
[!code-javascript[custom prompt bot](~/../botbuilder-samples/samples/javascript_nodejs/44.prompt-for-user-input/bots/customPromptBot.js?range=31-39)]
---

## <a name="filling-out-the-user-profile"></a>填寫使用者設定檔

我們會從收集資訊著手。 每個 Bot 都會提供類似的介面。

- 傳回值會指出輸入是否為此問題的有效答案。
- 如果通過驗證，則會產生要儲存的已剖析和正規化值。
- 如果驗證失敗，則會產生一則訊息，讓 Bot 再次詢問資訊。

 在下一節中，我們會定義協助程式方法來剖析及驗證使用者輸入。

## <a name="ctabcsharp"></a>[C#](#tab/csharp)

**Bots/CustomPromptBot.cs**  
[!code-csharp[custom prompt bot](~/../botbuilder-samples/samples/csharp_dotnetcore/44.prompt-users-for-input/Bots/CustomPromptBot.cs?range=46-103)]

## <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

**bots/customPromptBot.js**  
[!code-javascript[custom prompt bot](~/../botbuilder-samples/samples/javascript_nodejs/44.prompt-for-user-input/bots/customPromptBot.js?range=52-116)]

---

## <a name="parse-and-validate-input"></a>剖析及驗證輸入

我們將使用下列準則來驗證輸入。

- **名稱**必須是非空白字串。 我們的正規做法是修剪空白字元。
- **年齡**必須介於 18 與 120 之間。 我們的正規做法是傳回一個整數。
- **日期**必須是至少未來一小時的任何日期或時間。
  我們的正規做法是只傳回所剖析輸入的日期部分。

> [!NOTE]
> 對於年齡和日期輸入，我們會使用 [Microsoft/Recognizers-Text](https://github.com/Microsoft/Recognizers-Text/) 程式庫來執行初始剖析。
> 我們雖然提供範例程式碼，但不會說明文字辨識器程式庫的運作方式，而這只是剖析輸入的方法之一。
> 如需這些程式庫的詳細資訊，請參閱存放庫的**讀我**。

## <a name="ctabcsharp"></a>[C#](#tab/csharp)

將下列驗證方法新增到您的 Bot。

**Bots/CustomPromptBot.cs**  
[!code-csharp[custom prompt bot](~/../botbuilder-samples/samples/csharp_dotnetcore/44.prompt-users-for-input/Bots/CustomPromptBot.cs?range=105-203)]

## <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

**bots/customPromptBot.cs**  
[!code-javascript[custom prompt bot](~/../botbuilder-samples/samples/javascript_nodejs/44.prompt-for-user-input/bots/customPromptBot.js?range=118-189)]

---

## <a name="test-the-bot-locally"></a>在本機測試 Bot
下載並安裝 [Bot Framework Emulator](https://aka.ms/bot-framework-emulator-readme)，以在本機測試 Bot。

1. 在您的電腦本機執行範例。 如需相關指示，請參閱 [C# 範例](https://aka.ms/cs-primitive-prompt-sample)或 [JS 範例](https://aka.ms/js-primitive-prompt-sample)的讀我檔案。
1. 如下所示，使用模擬器進行測試。

![primitive-prompts](media/primitive-prompts.png)

## <a name="additional-resources"></a>其他資源

[Dialogs 程式庫](bot-builder-concept-dialog.md)會提供一些類別，以將管理交談的許多層面自動化。 

## <a name="next-step"></a>後續步驟

> [!div class="nextstepaction"]
> [實作循序對話流程](bot-builder-dialog-manage-conversation-flow.md)
