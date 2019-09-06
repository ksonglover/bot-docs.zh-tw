---
title: 將主動式通知傳送給使用者 | Microsoft Docs
description: 了解如何傳送通知訊息
keywords: 主動式訊息, 通知訊息, bot 通知,
author: jonathanfingold
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 05/23/2019
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 2a1f36e548f92e9c057947f0c5a82e426a8d0d23
ms.sourcegitcommit: eacf1522d648338eebefe2cc5686c1f7866ec6a2
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/30/2019
ms.locfileid: "70167381"
---
# <a name="send-proactive-notifications-to-users"></a>將主動式通知傳送給使用者

[!INCLUDE[applies-to](../includes/applies-to.md)]

一般而言，Bot 直接傳送給使用者的每個訊息都與使用者先前的輸入相關。
在某些情況下，Bot 可能需要將與目前對話主題或使用者最後傳送的訊息不直接相關的訊息傳送給使用者。 這些類型的訊息稱為_主動訊息_。

主動訊息可用於各種情況。 比方說，如果使用者先前已要求 Bot 監視產品的價格，則 Bot 可以在產品的價格下降了 20% 時對使用者發出警示。 或者，如果 Bot 需要一些時間來編譯對使用者問題的回應，它可能會通知使用者已延遲，並且在此同時允許交談繼續進行。 當 Bot 完成問題回應的編譯時，它會與使用者分享該資訊。

在您的 Bot 中實作主動式訊息時，請勿在短時間內傳送數個主動式訊息。 某些通道會強制限制 Bot 將訊息傳送給使用者的頻率，如果違反了這些限制，將會停用 Bot。

臨機操作主動式訊息是最簡單的主動式訊息類型。 Bot 只會在每次觸發時將訊息插入對話，不會顧及使用者目前是否與 Bot 在其他對話主題中，也不會嘗試以任何方式變更對話。

若要更順利地處理通知，請考慮使用其他方式將通知整合到對話流程中，例如在對話狀態中設定旗標，或將通知新增至佇列。

## <a name="prerequisites"></a>必要條件

- 了解 [bot 基本概念](bot-builder-basics.md)。
- 採用 **[C#](https://aka.ms/proactive-sample-cs) 或 [JavaScript](https://aka.ms/proactive-sample-js)** 的主動式訊息範例複本。 這個範例用來說明本文中的主動式傳訊。

## <a name="about-the-proactive-sample"></a>關於主動式範例

此範例有 Bot，以及用來將主動訊息傳送到 Bot 的額外控制器，如下圖所示。

![主動式 Bot](media/proactive-sample-bot.png)

## <a name="retrieve-and-store-conversation-reference"></a>擷取與儲存對話參考

當模擬器連線到 Bot 時，Bot 會收到兩個對話更新活動。 在 Bot 的對話更新活動處理常式中，Bot 會擷取對話參考並將其儲存在字典中，如下所示。

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

**Bots\ProactiveBot.cs**

[!code-csharp[OnConversationUpdateActivityAsync](~/../botbuilder-samples/samples/csharp_dotnetcore/16.proactive-messages/Bots/ProactiveBot.cs?range=26-37&highlight=3-4,9)]

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

**bots/proactiveBot.js**

[!code-javascript[onConversationUpdateActivity](~/../botbuilder-samples/samples/javascript_nodejs/16.proactive-messages/bots/proactiveBot.js?range=13-17&highlight=2)]

[!code-javascript[onConversationUpdateActivity](~/../botbuilder-samples/samples/javascript_nodejs/16.proactive-messages/bots/proactiveBot.js?range=41-44&highlight=2-3)]

---

注意：在實際案例中，您會將對話參考保存在資料庫中，而不是使用記憶體中的物件。

對話參考具有_對話_屬性，用來描述其中有活動的對話。 對話具有_使用者_屬性，該屬性會列出參與對話的使用者，對話還具有_服務 URL_屬性，通道會用此屬性來表示可能傳送目前活動回覆的 URL。 傳送主動訊息給使用者需要有效的對話參考。

## <a name="send-proactive-message"></a>傳送主動訊息

第二個控制器 (_通知_控制器) 會負責將主動訊息傳送至 Bot。 請使用下列步驟來產生主動訊息。

1. 擷取對話參考，以對其傳送主動訊息。
1. 呼叫配接器的_繼續對話_方法，並提供要使用的對話參考和回合處理常式委派。 繼續對話方法會為參考對話產生回合內容，然後呼叫指定的回合處理常式委派。
1. 在委派中，使用回合內容來傳送主動訊息。

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

**Controllers\NotifyController .cs**

每當要求 Bot 的通知頁面時，通知控制器就會從字典中擷取對話參考。
接著，控制器會使用 `ContinueConversationAsync` 和 `BotCallback` 方法來傳送主動訊息。

[!code-csharp[Notify logic](~/../botbuilder-samples/samples/csharp_dotnetcore/16.proactive-messages/Controllers/NotifyController.cs?range=17-60&highlight=28,40-43)]

若要傳送主動訊息，配接器會需要 Bot 的應用程式識別碼。 在生產環境中，您可以使用 Bot 的應用程式識別碼。 在本機測試環境中，您可以使用任何 GUID。 如果 Bot 目前尚未獲派應用程式識別碼，通知控制器會自行產生用於呼叫的預留位置識別碼。

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

**index.js**

每當要求伺服器的 `/api/notify` 頁面時，伺服器就會從字典中擷取對話參考。
接著，伺服器會使用 `continueConversation` 方法來傳送主動訊息。
`continueConversation` 的參數是一個函式，作為 Bot 在這一回合的回合處理常式。

[!code-javascript[Notify logic](~/../botbuilder-samples/samples/javascript_nodejs/16.proactive-messages/index.js?range=56-68&highlight=4-5)]

---

## <a name="test-your-bot"></a>測試 Bot

1. 如果您尚未安裝 [Bot Framework Emulator](https://aka.ms/bot-framework-emulator-readme)，請進行安裝。
1. 在您的電腦本機執行範例。
1. 啟動模擬器並連線至您的 Bot。
1. 載入至您 Bot 的 api/notify 頁面。 這會在模擬器中產生主動訊息。

## <a name="additional-information"></a>其他資訊

除了本文使用的範例之外，您也可以在 [GitHub](https://github.com/Microsoft/BotBuilder-Samples/) 上取得 C# 和 JS 的其他範例。

### <a name="avoiding-401-unauthorized-errors"></a>避免 401「未經授權」錯誤 

根據預設，如果傳入的要求已經過 BotAuthentication 的驗證，則 BotBuilder SDK 會將 `serviceUrl` 加入受信任的主機名稱清單。 這些名稱會保留在記憶體內部快取中。 如果 Bot 重新啟動，正在等待主動訊息的使用者會無法接收該訊息，必須在 Bot 重新啟動後，再次對 Bot 傳送訊息才能解決此問題。 

若要避免這個問題，您必須以手動方式將 `serviceUrl` 加入受信任的主機名稱清單，方式如下： 

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

```csharp 
MicrosoftAppCredentials.TrustServiceUrl(serviceUrl); 
``` 

針對主動式傳訊，`serviceUrl` 是主動訊息接收者使用的通道 URL，您可以在 `Activity.ServiceUrl` 中找到此項目。 

您可以直接將上述程式碼加在傳送主動訊息的程式碼之前。 在[主動式訊息範例](https://github.com/microsoft/BotBuilder-Samples/tree/master/samples/csharp_dotnetcore/16.proactive-messages)中，您可以將其放在 `NotifyController.cs` 中，就在 `await turnContext.SendActivityAsync("proactive hello");` 的前面。

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

```js
MicrosoftAppCredentials.trustServiceUrl(serviceUrl);
```

針對主動式傳訊，`serviceUrl` 是主動訊息接收者使用的通道 URL，您可以在 `activity.serviceUrl` 中找到此項目。

您可以直接將上述程式碼加在傳送主動訊息的程式碼之前。 在[主動式訊息範例](https://github.com/microsoft/BotBuilder-Samples/tree/master/samples/javascript_nodejs/16.proactive-messages)中，您可以將其放在 `index.js` 中，就在 `await turnContext.sendActivity('proactive hello');` 的前面。

---

## <a name="next-steps"></a>後續步驟

> [!div class="nextstepaction"]
> [實作循序對話流程](bot-builder-dialog-manage-conversation-flow.md)
