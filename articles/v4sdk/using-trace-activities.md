---
title: 將追蹤活動新增至您的 Bot | Microsoft Docs
description: 說明 Bot Framework SDK 中的追蹤活動，以及如何加以使用。
keywords: 追蹤, 活動, Bot, Bot Framework SDK
author: JonathanFingold
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 10/18/2019
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 46264241b077b1ace50b68745c9ae61a98e4d2f9
ms.sourcegitcommit: 4751c7b8ff1d3603d4596e4fa99e0071036c207c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 11/02/2019
ms.locfileid: "73443056"
---
# <a name="add-trace-activities-to-your-bot"></a>將追蹤活動新增至您的 Bot

[!INCLUDE[applies-to](../includes/applies-to.md)]

<!-- What is it and why use it -->

_追蹤活動_是指 Bot 可傳送至 Bot Framework Emulator 的活動。
您可以使用追蹤活動對 Bot 進行互動式偵錯，因為這些活動可讓您檢視 Bot 在本機執行時的資訊。

<!-- Details -->

追蹤活動只會傳送至 Emulator，而不會傳送至任何其他用戶端或通道。
Emulator 會將活動顯示在記錄中，而不是顯示在主要聊天面板。

- 透過回合內容傳送的追蹤活動會透過回合內容上註冊的_傳送活動處理常式_來傳送。
- 透過回合內容傳送的追蹤活動可藉由套用對話參考 (如果有的話)，與輸入活動建立關聯。
  若為主動式訊息，_回復至識別碼_會是新的 GUID。
- 不論用何種方式傳送，追蹤活動永遠不會設定 [已回應]  旗標。

## <a name="to-use-a-trace-activity"></a>使用追蹤活動

若要在 Emulator 中查看追蹤活動，您需要一個能讓 Bot 傳送追蹤活動的案例，例如，從配接器的回合錯誤處理常式擲回例外狀況並傳送追蹤活動。

若要從您的 Bot 傳送追蹤活動：

1. 建立新活動。
   - 將其_類型_屬性設定為 "trace"。 這是必填欄位。
   - (選擇性) 為追蹤設定適當的 [名稱]  、[標籤]  、[值]  和 [值類型]  屬性。
1. 使用_回合內容_物件的_傳送活動_方法來傳送追蹤活動。
   - 此方法會根據傳入活動，為活動的其餘必要屬性新增值。
     其中包括 [通道識別碼]  、[服務 URL]  、[寄件者]  和 [收件者]  屬性。

若要在 Emulator 中檢視追蹤活動：

1. 在您的電腦本機上執行 Bot。
1. 使用 Emulator 進行測試。
   - 與 Bot 互動，並使用您案例中的步驟來產生追蹤活動。
   - 當您的 Bot 發出追蹤活動時，追蹤活動會顯示在 Emulator 記錄中。

如果您未先設定 Bot 所依賴的 QnAMaker 知識庫，您可能會在執行核心 Bot 時看到以下追蹤活動。

![Emulator 的螢幕擷取畫面](./media/using-trace-activities.png)

## <a name="add-a-trace-activity-to-the-adapters-on-error-handler"></a>將追蹤活動新增至配接器的錯誤處理常式

配接器的_回合錯誤_處理常式會攔截 Bot 在回合期間擲回但未攔截的所有其他例外狀況。
這是追蹤活動的好處，因為您可以將使用者易懂的訊息傳送給使用者，並將有關此例外狀況的偵錯資訊傳送至 Emulator。

此範例程式碼來自**核心 Bot** 範例。 請參閱 [**C#** ](https://aka.ms/cs-core-sample) 或 [**JavaScript**](https://aka.ms/js-core-sample) 中的完整範例。

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

此範例中定義的 **SendTraceActivityAsync** 協助程式方法會將例外狀況資訊當做追蹤活動傳送至 Emulator。

**AdapterWithErrorHandler.cs**

[!code-csharp[SendTraceActivityAsync](~/../BotBuilder-Samples/samples/csharp_dotnetcore/13.core-bot/AdapterWithErrorHandler.cs?range=17-55)]

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

配接器的 **onTurnError** 處理常式會建立追蹤活動來包含例外狀況資訊，並將其傳送至 Emulator。

**index.js**

[!code-javascript[onTurnError ](~/../BotBuilder-Samples/samples/javascript_nodejs/13.core-bot/index.js?range=35-59)]

---

## <a name="additional-resources"></a>其他資源

- 如何[使用檢查中介軟體對 Bot 進行偵錯](../bot-service-debug-inspection-middleware.md)中會說明如何新增會發出追蹤活動的中介軟體。
- 若要對已部署的 Bot 進行偵錯，您可以使用 Application Insights。 如需詳細資訊，請參閱[將遙測新增至您的 Bot](bot-builder-telemetry.md)。
- 如需每個活動類型的詳細資訊，請參閱 [Bot Framework 活動結構描述](https://aka.ms/botSpecs-activitySchema)。
