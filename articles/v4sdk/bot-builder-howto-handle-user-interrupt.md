---
title: 處理使用者中斷 | Microsoft Docs
description: 深入了解如何處理使用者中斷和直接對話流程。
keywords: 中斷, 切換主題, 中斷
author: ivorb
ms.author: v-ivorb
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 04/18/2019
ms.reviewer: ''
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: bd8682966dbb2e33a536a72a4016ef23e9c1fc75
ms.sourcegitcommit: f84b56beecd41debe6baf056e98332f20b646bda
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 05/03/2019
ms.locfileid: "65032624"
---
# <a name="handle-user-interruptions"></a>處理使用者中斷

[!INCLUDE[applies-to](../includes/applies-to.md)]

處理中斷是強固 Bot 很重要的層面。 使用者不一定會按照所定義的對話流程逐步進行。 其可能會嘗試詢問程序中期才會提出的問題，或想要直接取消問題而不想加以完成。 在本主題中，我們將會探索一些可供處理 Bot 使用者中斷的常見方式。

## <a name="prerequisites"></a>必要條件

- [Bot 基本概念][concept-basics]、[管理狀態][concept-state]、[對話方塊程式庫][concept-dialogs]及如何[重複使用對話方塊][component-dialogs]的知識。
- 一份核心的 Bot 範例 (使用 [**CSharp**][cs-sample] 或 [**JavaScript**][js-sample])。

## <a name="about-this-sample"></a>關於此範例

本文所使用的範例會設計一個預訂航班的 Bot，其會使用對話方塊來向使用者取得航班資訊。 使用者可以在與 Bot 對話期間，隨時發出_協助_或_取消_命令來中斷對話。 在此，我們會處理兩種中斷：

- **回合層級**：略過回合層級的處理，但讓對話方塊與所提供的資訊留在堆疊上。 在下一個回合時，從我們離開的地方接續進行。 
- **對話方塊層級**：完全取消處理，讓 Bot 可以重新來過一次。

## <a name="define-and-implement-the-interruption-logic"></a>定義和實作中斷邏輯

首先，我們要定義並實作_協助_和_取消_中斷。

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

若要使用對話方塊，請安裝 **Microsoft.Bot.Builder.Dialogs** NuGet 套件。

**Dialogs\CancelAndHelpDialog.cs**

一開始，我們會先實作 `CancelAndHelpDialog` 類別來處理使用者中斷。

[!code-csharp[Class signature](~/../botbuilder-samples/samples/csharp_dotnetcore/13.core-bot/Dialogs/CancelAndHelpDialog.cs?range=11)]

在 `CancelAndHelpDialog` 類別中，`OnBeginDialogAsync` 和 `OnContinueDialogAsync` 方法會呼叫 `InerruptAsync` 方法來檢查使用者是否已中斷一般流程。 如果流程已中斷，則會呼叫基底類別方法；，否則會從 `InterruptAsync` 傳回傳回值。

[!code-csharp[Overrides](~/../botbuilder-samples/samples/csharp_dotnetcore/13.core-bot/Dialogs/CancelAndHelpDialog.cs?range=18-27)]

如果使用者輸入「協助」，`InterrupAsync` 方法會傳送一則訊息，然後呼叫 `DialogTurnResult (DialogTurnStatus.Waiting)` 來指出最上層的對話方塊正在等候使用者回應。 如此一來，系統便只會在一個回合內中斷對話流程，到下一個回合時，我們就會從離開的地方接續進行。

如果使用者輸入「取消」，則會在其內部對話方塊內容上呼叫 `CancelAllDialogsAsync`，此方法會清除其對話方塊堆疊，並讓其以取消的狀態結束，且不會有結果值。 到 `MainDialog` (稍後會顯示) 時，其會顯示預約對話方塊已結束並傳回 Null，情況類似使用者選擇不要確認其預約時。

[!code-csharp[Interrupt](~/../botbuilder-samples/samples/csharp_dotnetcore/13.core-bot/Dialogs/CancelAndHelpDialog.cs?range=40-61&highlight=11-12,16-17)]

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

若要使用對話方塊，請安裝 **botbuilder-dialogs** npm 套件。

**dialogs/cancelAndHelpDialog.js**

一開始，我們會先實作 `CancelAndHelpDialog` 類別來處理使用者中斷。

[!code-javascript[Class signature](~/../botbuilder-samples/samples/javascript_nodejs/13.core-bot/dialogs/cancelAndHelpDialog.js?range=10)]

在 `CancelAndHelpDialog` 類別中，`onBeginDialog` 和 `onContinueDialog` 方法會呼叫 `interrupt` 方法來檢查使用者是否已中斷一般流程。 如果流程已中斷，則會呼叫基底類別方法；，否則會從 `interrupt` 傳回傳回值。

[!code-javascript[Overrides](~/../botbuilder-samples/samples/javascript_nodejs/13.core-bot/dialogs/cancelAndHelpDialog.js?range=11-25)]

如果使用者輸入「協助」，`interrupt` 方法會傳送一則訊息，然後傳回 `{ status: DialogTurnStatus.waiting }` 物件來指出最上層的對話方塊正在等候使用者回應。 如此一來，系統便只會在一個回合內中斷對話流程，到下一個回合時，我們就會從離開的地方接續進行。

如果使用者輸入「取消」，則會在其內部對話方塊內容上呼叫 `cancelAllDialogs`，此方法會清除其對話方塊堆疊，並讓其以取消的狀態結束，且不會有結果值。 到 `MainDialog` (稍後會顯示) 時，其會顯示預約對話方塊已結束並傳回 Null，情況類似使用者選擇不要確認其預約時。

[!code-javascript[Interrupt](~/../botbuilder-samples/samples/javascript_nodejs/13.core-bot/dialogs/cancelAndHelpDialog.js?range=27-40&highlight=7-8,11-12)]

---

## <a name="check-for-interruptions-each-turn"></a>每回合檢查中斷

現在，我們已說明過中斷處理類別的運作方式，接著讓我們回頭看看當 Bot 收到來自使用者的新訊息時，會發生什麼事。

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

**Dialogs\MainDialog.cs**

有新的訊息活動送達時，Bot 會執行 `MainDialog`。 `MainDialog` 會提示使用者其能提供什麼協助。 然後，其會藉由呼叫 `BeginDialogAsync` 在 `MainDialog.ActStepAsync` 方法中啟動 `BookingDialog`，如下所示。

[!code-csharp[ActStepAsync](~/../botbuilder-samples/samples/csharp_dotnetcore/13.core-bot/Dialogs/MainDialog.cs?range=54-68&highlight=13-14)]

接下來，在 `MainDialog` 類別的 `FinalStepAsync` 方法中，預約對話方塊會結束，並認為預約已完成或取消。

[!code-csharp[FinalStepAsync](~/../botbuilder-samples/samples/csharp_dotnetcore/13.core-bot/Dialogs/MainDialog.cs?range=70-91)]

這裡不會顯示 `BookingDialog` 中的程式碼，因為其與中斷處理沒有直接關聯。 其會用來提示使用者輸入預約詳細資料。 您可以在 **Dialogs\BookingDialogs.cs** 中找到該程式碼。

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

**dialogs/mainDialog.js**

有新的訊息活動送達時，Bot 會執行 `MainDialog`。 `MainDialog` 會提示使用者其能提供什麼協助。 然後，其會藉由呼叫 `beginDialog` 在 `MainDialog.actStep` 方法中啟動 `bookingDialog`，如下所示。

[!code-javascript[Act step](~/../botbuilder-samples/samples/javascript_nodejs/13.core-bot/dialogs/mainDialog.js?range=71-88&highlight=16-17)]

接下來，在 `MainDialog` 類別的 `finalStep` 方法中，預約對話方塊會結束，並認為預約已完成或取消。

[!code-javascript[Final step](~/../botbuilder-samples/samples/javascript_nodejs/13.core-bot/dialogs/mainDialog.js?range=93-110)]

這裡不會顯示 `BookingDialog` 中的程式碼，因為其與中斷處理沒有直接關聯。 其會用來提示使用者輸入預約詳細資料。 您可以在 **dialogs/bookingDialogs.js** 中找到該程式碼。

---

## <a name="handle-unexpected-errors"></a>處理未預期的錯誤

接下來，我們要處理可能會發生的任何未處理例外狀況。

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

**AdapterWithErrorHandler.cs**

在我們的範例中，配接器的 `OnTurnError` 處理常式會接收 Bot 的回合邏輯所擲回的任何例外狀況。 如果有擲回的例外狀況，處理常式就會刪除目前對話的對話狀態，防止 Bot 卡在因為狀態不良而導致的錯誤迴圈中。

[!code-csharp[AdapterWithErrorHandler](~/../botbuilder-samples/samples/csharp_dotnetcore/13.core-bot/AdapterWithErrorHandler.cs?range=12-41)]

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

**index.js**

在我們的範例中，配接器的 `onTurnError` 處理常式會接收 Bot 的回合邏輯所擲回的任何例外狀況。 如果有擲回的例外狀況，處理常式就會刪除目前對話的對話狀態，防止 Bot 卡在因為狀態不良而導致的錯誤迴圈中。

[!code-javascript[AdapterWithErrorHandler](~/../botbuilder-samples/samples/javascript_nodejs/13.core-bot/index.js?range=28-38)]

---

## <a name="register-services"></a>註冊伺服器

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

**Startup.cs**

最後，在 `Startup.cs` 中會建立暫時性的 Bot，且每一回合都會建立新的 Bot 執行個體。

[!code-csharp[Add transient bot](~/../botbuilder-samples/samples/csharp_dotnetcore/13.core-bot/Startup.cs?range=46-47)]

如需參考，以下是呼叫中用來建立上述 Bot 的類別定義。

[!code-csharp[MainDialog signature](~/../botbuilder-samples/samples/csharp_dotnetcore/13.core-bot/Dialogs/MainDialog.cs?range=15)]
[!code-csharp[DialogAndWelcomeBot signature](~/../botbuilder-samples/samples/csharp_dotnetcore/13.core-bot/Bots/DialogAndWelcomeBot.cs?range=16)]
[!code-csharp[DialogBot signature](~/../botbuilder-samples/samples/csharp_dotnetcore/13.core-bot/Bots/DialogBot.cs?range=18)]

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

**index.js**

最後，在 `index.js` 中便會建立 Bot。

[!code-javascript[Create bot](~/../botbuilder-samples/samples/javascript_nodejs/13.core-bot/index.js?range=55-56)]

如需參考，以下是呼叫中用來建立上述 Bot 的類別定義。

[!code-javascript[MainDialog signature](~/../botbuilder-samples/samples/javascript_nodejs/13.core-bot/dialogs/mainDialog.js?range=12)]
[!code-javascript[DialogAndWelcomeBot signature](~/../botbuilder-samples/samples/javascript_nodejs/13.core-bot/bots/dialogAndWelcomeBot.js?range=8)]
[!code-javascript[DialogBot signature](~/../botbuilder-samples/samples/javascript_nodejs/13.core-bot/bots/dialogBot.js?range=6)]

---

## <a name="to-test-the-bot"></a>測試 Bot

1. 如果您尚未安裝 [Bot Framework Emulator](https://aka.ms/bot-framework-emulator-readme)，請進行安裝。
1. 在您的電腦本機執行範例。
1. 啟動模擬器、連線到您的 Bot 並傳送如下所示的訊息。

<!--![test dialog prompt sample](~/media/emulator-v4/test-dialog-prompt.png)-->

## <a name="additional-information"></a>其他資訊

- [驗證範例](https://aka.ms/logout)會示範如何處理登出，此登出會使用如下所示的類似模式來處理中斷。

- 您應該傳送預設回應，而不是毫無作為，讓使用者臆測究竟發生什麼事。 預設回應應該告知使用者 Bot 可以理解哪些命令，讓使用者可以回到正軌。

- 在回合中的任何時點，回合內容的_回應_屬性都會指出 Bot 是否已在這個回合傳送訊息給使用者。 回合結束之前，Bot 應該會傳送某些訊息給使用者，即使是簡單的輸入通知也是如此。

<!-- Footnote-style links -->

[concept-basics]: bot-builder-basics.md
[concept-state]: bot-builder-concept-state.md
[concept-dialogs]: bot-builder-concept-dialog.md

[using-luis]: bot-builder-howto-v4-luis.md
[using-qna]: bot-builder-howto-qna.md

[simple-flow]: bot-builder-dialog-manage-conversation-flow.md
[prompting]: bot-builder-prompts.md
[component-dialogs]: bot-builder-compositcontrol.md

[cs-sample]: https://aka.ms/cs-core-sample
[js-sample]: https://aka.ms/js-core-sample
