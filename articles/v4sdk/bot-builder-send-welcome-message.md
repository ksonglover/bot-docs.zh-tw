---
title: 將歡迎訊息傳送給使用者 | Microsoft Docs
description: 了解如何開發 Bot，以提供親切的使用者體驗。
keywords: 概觀, 開發, 使用者體驗, 歡迎, 個人化體驗, C#, JS, 歡迎訊息, Bot, 問候, 問候語
author: DanDev33
ms.author: v-dashel
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 05/23/2019
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: f7709d6273739be6d2b3e9e3174f24ea2734f22d
ms.sourcegitcommit: ea64a56acfabc6a9c1576ebf9f17ac81e7e2a6b7
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 05/24/2019
ms.locfileid: "66215314"
---
# <a name="send-welcome-message-to-users"></a>將歡迎訊息傳送給使用者

[!INCLUDE[applies-to](../includes/applies-to.md)]

建立任何 Bot 時的主要目標就是讓您的使用者參與有意義的對話。 達成此目標的最佳方式之一就是確保從使用者第一次連線的那一刻，他們就了解您 Bot 的主要用途和功能，以及您 Bot 的建立原因。 這篇文章提供程式碼範例，協助您歡迎使用者使用 Bot。

## <a name="prerequisites"></a>必要條件
- 了解 [bot 基本概念](bot-builder-basics.md)。 
- 採用 [C# 範例](https://aka.ms/welcome-user-mvc)或 [JS 範例](https://aka.ms/bot-welcome-sample-js)的一份**歡迎使用者範例**。 此範例中的程式碼用來說明如何傳送歡迎訊息。

## <a name="about-this-sample-code"></a>關於此程式碼範例
此程式碼範例示範，如何在新的使用者一開始連線至 Bot 時就偵測到並表示歡迎。 下圖顯示此 Bot 的邏輯流程。 

### <a name="ctabcsharp"></a>[C#](#tab/csharp)
Bot 會遇到的兩個主要事件為
- `OnMembersAddedAsync`，每當有新的使用者連線至 Bot 時便會加以呼叫
- `OnMessageActivityAsync`，每次收到新的使用者輸入時便會加以呼叫。

![歡迎使用者邏輯流程](media/welcome-user-flow.png)

每當有新的使用者連線時，Bot 便會向其提供 `WelcomeMessage`、`InfoMessage` 和 `PatternMessage`。 收到新的使用者輸入時，便會檢查 WelcomeUserState 以查看 `DidBotWelcomeUser` 是否設定為 true  。 如果沒有，便會對使用者傳回初始的歡迎使用者訊息。

### <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)
Bot 會遇到的兩個主要事件為
- `onMembersAdded`，每當有新的使用者連線至 Bot 時便會加以呼叫
- `onMessage`，每次收到新的使用者輸入時便會加以呼叫。

![歡迎使用者邏輯流程](media/welcome-user-flow-js.png)

每當有新的使用者連線時，Bot 便會向其提供 `welcomeMessage`、`infoMessage` 和 `patternMessage`。 收到新的使用者輸入時，便會檢查 `welcomedUserProperty` 以查看 `didBotWelcomeUser` 是否設定為 true  。 如果沒有，便會對使用者傳回初始的歡迎使用者訊息。

---

 如果 DidBotWelcomeUser 是 _true_，便會評估使用者的輸入。 根據使用者輸入的內容，此 Bot 會執行下列其中一項：
- 針對從使用者收到的問候發出回應。
- 顯示主圖卡片來提供關於 Bot 的其他資訊。
- 重新傳送 `WelcomeMessage` 來說明此 Bot 預期的輸入。

## <a name="create-user-object"></a>建立使用者物件
### <a name="ctabcsharp"></a>[C#](#tab/csharp)
使用者狀態物件會在啟動時建立，而相依性會插入 Bot 建構函式。

**Startup.cs** [!code-csharp[ConfigureServices](~/../botBuilder-samples/samples/csharp_dotnetcore/03.welcome-user/Startup.cs?range=29-33)]

**WelcomeUserBot.cs** [!code-csharp[Class](~/../BotBuilder-Samples/samples/csharp_dotnetcore/03.welcome-user/bots/WelcomeUserBot.cs?range=41-47)]

### <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)
在啟動時，會將記憶體儲存體與使用者狀態同時定義於 index.js 中。

**Index.js** [!code-javascript[DefineUserState](~/../BotBuilder-Samples/samples/javascript_nodejs/03.welcome-users/Index.js?range=8-10,33-41)]

---

## <a name="create-property-accessors"></a>建立屬性存取子
### <a name="ctabcsharp"></a>[C#](#tab/csharp)
現在，我們會建立屬性存取子來獲得 OnMessageActivityAsync 方法內 WelcomeUserState 的控制代碼。
然後，呼叫 GetAsync 方法來取得正確範圍的金鑰。 我們隨後會使用 `SaveChangesAsync` 方法在每個使用者輸入反覆項目之後儲存使用者狀態資料。

**WelcomeUserBot.cs** [!code-csharp[OnMessageActivityAsync](~/../BotBuilder-Samples/samples/csharp_dotnetcore/03.welcome-user/bots/WelcomeUserBot.cs?range=68-71, 102-105)]

### <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)
現在，我們會建立屬性存取子來獲得 UserState 中所保存 WelcomedUserProperty 的控制代碼。

**WelcomeBot.js** [!code-javascript[DefineUserState](~/../BotBuilder-Samples/samples/javascript_nodejs/03.welcome-users/bots/welcomebot.js?range=7-10,16-22)]

---

## <a name="detect-and-greet-newly-connected-users"></a>偵測並歡迎新連線的使用者

### <a name="ctabcsharp"></a>[C#](#tab/csharp)
在 **WelcomeUserBot** 中，我們會使用 `OnMembersAddedAsync()` 檢查是否有活動更新，以了解對話中是否新增了新的使用者，然後向其傳送一組三個的初始歡迎訊息，分別是 `WelcomeMessage`、`InfoMessage` 和 `PatternMessage`。 此互動的完整程式碼如下所示。

**WelcomeUserBot.cs** [!code-csharp[WelcomeMessages](~/../BotBuilder-Samples/samples/csharp_dotnetcore/03.welcome-user/bots/WelcomeUserBot.cs?range=20-40, 55-66)]

### <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)
此 JavaScript 程式碼會在有新增的使用者時，傳送初始歡迎訊息。 其方法是檢查對話活動並確認對話中是否已新增新的成員。

**WelcomeBot.js** [!code-javascript[DefineUserState](~/../BotBuilder-Samples/samples/javascript_nodejs/03.welcome-users/bots/welcomebot.js?range=65-87)]

---

## <a name="welcome-new-user-and-discard-initial-input"></a>歡迎新的使用者並捨棄初始輸入

### <a name="ctabcsharp"></a>[C#](#tab/csharp)
此外，務必考量使用者的輸入何時實際上可能包含實用資訊，而這可能因通道而異。 若要確保使用者在所有可能的通道上都有美好的體驗，我們會檢查狀態旗標 _didBotWelcomeUser_，如果這是 "false"，我們就不會處理此初始使用者輸入。 我們會改為向使用者提供初始歡迎訊息。 布林值 _welcomedUserProperty_ 於是會設定為 "true" (儲存於 UserState 內)，我們的程式碼現在則會處理這位使用者來自所有其他訊息活動的輸入。

**WelcomeUserBot.cs** [!code-csharp[DidBotWelcomeUser](~/../BotBuilder-Samples/samples/csharp_dotnetcore/03.welcome-user/bots/WelcomeUserBot.cs?range=68-82)]

### <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)
此外，務必考量使用者的輸入何時實際上可能包含實用資訊，而這可能因通道而異。 為確保使用者在所有可能的通道上都有美好的體驗，我們會檢查 didBotWelcomedUser 屬性，如果此屬性不存在，我們便會將其設定為 "false"，並且不處理初始使用者輸入。 我們會改為向使用者提供初始歡迎訊息。 然後布林值 _didBotWelcomeUser_ 會設為 "true"，而我們的程式碼會處理來自所有其他訊息活動的使用者輸入。

**WelcomeBot.js** [!code-javascript[DidBotWelcomeUser](~/../BotBuilder-Samples/samples/javascript_nodejs/03.welcome-users/bots/welcomebot.js?range=24-38,57-59,63)]

---

## <a name="process-additional-input"></a>處理其他輸入

在歡迎新的使用者後，便會在每個訊息回合評估使用者輸入資訊，且 Bot 會根據該使用者輸入的內容來提供回應。 下列程式碼顯示用來產生該回應的決策邏輯。 

### <a name="ctabcsharp"></a>[C#](#tab/csharp)
輸入 'intro' 或 'help' 會呼叫 `SendIntroCardAsync` 函式來向使用者呈現資訊主圖卡片。 本文的下一節會檢驗該程式碼。

**WelcomeUserBot.cs** [!code-csharp[SwitchOnUtterance](~/../BotBuilder-Samples/samples/csharp_dotnetcore/03.welcome-user/bots/WelcomeUserBot.cs?range=85-100)]

### <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)
輸入 'intro' 或 'help' 會使用 CardFactory 來向使用者呈現簡介調適型卡片。 本文的下一節會檢驗該程式碼。

**WelcomeBot.js** [!code-javascript[SwitchOnUtterance](~/../BotBuilder-Samples/samples/javascript_nodejs/03.welcome-users/bots/welcomebot.js?range=40-56)]

---

## <a name="using-hero-card-greeting"></a>使用主圖卡片問候語

如先前所述，某些使用者輸入會產生「主圖卡片」  以回應其要求。 您可以在[傳送簡介卡片](./bot-builder-howto-add-media-attachments.md)中深入了解主圖卡片問候語。 必須有下列程式碼才能建立此 Bot 的主圖卡片回應。

### <a name="ctabcsharp"></a>[C#](#tab/csharp)

**WelcomeUserBot.cs** [!code-csharp[SendHeroCardGreeting](~/../BotBuilder-Samples/samples/csharp_dotnetcore/03.welcome-user/bots/WelcomeUserBot.cs?range=107-127)]

### <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

**WelcomeBot.js** [!code-javascript[SendIntroCard](~/../BotBuilder-Samples/samples/javascript_nodejs/03.welcome-users/bots/welcomebot.js?range=91-116)]

---

## <a name="test-the-bot"></a>測試 Bot

下載並安裝最新版 [Bot Framework Emulator](https://aka.ms/bot-framework-emulator-readme)

1. 在您的電腦本機執行範例。 如需相關指示，請參閱 [C# 範例](https://aka.ms/welcome-user-mvc)或 [JS 範例](https://aka.ms/bot-welcome-sample-js)的讀我檔案。
1. 使用模擬器來測試 Bot，如下所示。

![測試歡迎 Bot 範例](media/welcome-user-emulator-1.png)

測試主圖卡片問候語。

![測試歡迎 Bot 卡片](media/welcome-user-emulator-2.png)

## <a name="additional-resources"></a>其他資源

在[將媒體新增至訊息](./bot-builder-howto-add-media-attachments.md)中深入了解各種媒體回應。

## <a name="next-steps"></a>後續步驟

> [!div class="nextstepaction"]
> [收集使用者輸入](bot-builder-prompts.md)
