---
title: 使用分支和迴圈建立進階交談流程 | Microsoft Docs
description: 了解如何在 Bot Framework SDK 中透過對話來管理複雜的交談流程。
keywords: 複雜對話流程, 重複, 迴圈, 選單, 對話方塊, 提示, 瀑布, 對話方塊集
author: JonathanFingold
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 11/06/2019
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: ec56a36feb747160e1a82f9831aa323074d46801
ms.sourcegitcommit: 312a4593177840433dfee405335100ce59aac347
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 11/12/2019
ms.locfileid: "73933616"
---
# <a name="create-advanced-conversation-flow-using-branches-and-loops"></a>使用分支和迴圈建立進階的交談流程

[!INCLUDE[applies-to](../includes/applies-to.md)]

您可以使用對話方塊程式庫來管理簡單和複雜的對話流程。
在本文中，我們會說明如何管理可進行分支和迴圈處理的複雜交談。
我們也會說明如何在對話塊的不同部分之間傳遞引數。

## <a name="prerequisites"></a>必要條件

- [Bot 基本概念][concept-basics]、[管理狀態][concept-state]、[對話方塊程式庫][concept-dialogs]及如何[實作循序對話流程][simple-dialog]的知識。
- 採用 [**C#** ][cs-sample] 或 [**JavaScript**][js-sample] 的複雜對話方塊範例複本。

## <a name="about-this-sample"></a>關於此範例

此範例所代表的 Bot 可讓使用者註冊，以評論清單中最多兩家公司。

`DialogAndWelcomeBot` 會展開 `DialogBot`，以定義不同活動的處理常式及 Bot 的回合處理常式。 `DialogBot` 會執行對話方塊：

- `DialogBot` 會使用 _run_ 方法來開始對話方塊。
- `MainDialog` 是其他兩個對話方塊的父項，只在對話方塊中的特定時間進行呼叫。 本文會提供這些對話方塊的詳細資料。

對話方塊分成 `MainDialog`、`TopLevelDialog` 和 `ReviewSelectionDialog` 元件對話，並一起執行下列動作：

- 這些對話方塊會要求使用者的名稱和年齡，然後根據使用者的年齡進行_分支_。
  - 如果使用者太年輕，則不會要求使用者評論任何公司。
  - 如果使用者年紀夠大，就會開始收集使用者的評論喜好。
    - 其可讓使用者選取要評論的公司。
    - 如果使用者選擇了一家公司，這些對話方塊就會以_迴圈_來允許選取第二間公司。
- 並在最後感謝使用者的參與。

其使用兩個瀑布式對話，和一些提示來管理複雜對話。

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

![複雜的 Bot 流程](./media/complex-conversation-flow.png)

若要使用對話方塊，您的專案必須安裝 **Microsoft.Bot.Builder.Dialogs** NuGet 套件。

**Startup.cs**

我們會在 `Startup` 中為 Bot 註冊服務。 這些服務可透過相依性插入來提供給程式碼的其他部分。

- Bot 的基本服務：認證提供者、配接器及 Bot 實作。
- 用於管理狀態的服務：儲存體、使用者狀態及交談狀態。
- Bot 會使用的對話方塊。

[!code-csharp[ConfigureServices](~/../botbuilder-samples/samples/csharp_dotnetcore/43.complex-dialog/Startup.cs?range=22-36)]

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

![複雜的 Bot 流程](./media/complex-conversation-flow-js.png)

若要使用對話方塊，您的專案需要安裝 **botbuilder-dialogs** npm 套件。

**index.js**

我們會為 Bot 建立其他程式碼部分需要的服務。

- Bot 的基本服務：配接器及 Bot 實作。
- 用於管理狀態的服務：儲存體、使用者狀態及對話狀態。
- Bot 會使用的對話方塊。

[!code-javascript[ConfigureServices](~/../botbuilder-samples/samples/javascript_nodejs/43.complex-dialog/index.js?range=26-65)]

---

> [!NOTE]
> 記憶體儲存體僅供測試用途，而不適用於生產環境。
> 針對生產 Bot，請務必使用永續性的儲存體類型。

## <a name="define-a-class-in-which-to-store-the-collected-information"></a>定義用來儲存所收集資訊的類別

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

**UserProfile.cs**

[!code-csharp[UserProfile class](~/../botbuilder-samples/samples/csharp_dotnetcore/43.complex-dialog/UserProfile.cs?range=8-16)]

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

**userProfile.js**

[!code-javascript[UserProfile class](~/../botbuilder-samples/samples/javascript_nodejs/43.complex-dialog/userProfile.js?range=4-12)]

---

## <a name="create-the-dialogs-to-use"></a>建立要使用的對話方塊

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

**Dialogs\MainDialog.cs**

我們已經定義了元件對話：`MainDialog`，其包含幾個主要步驟，並指示了對話方塊和提示。 第一個步驟是呼叫 `TopLevelDialog` (下面會說明)。

[!code-csharp[step implementations](~/../botbuilder-samples/samples/csharp_dotnetcore/43.complex-dialog/Dialogs/MainDialog.cs?range=31-50&highlight=3)]

**Dialogs\TopLevelDialog.cs**

初始、最上層的對話有四個步驟：

1. 詢問使用者的名稱。
1. 詢問使用者的年齡。
1. 根據使用者的年齡分支。
1. 最後，感謝使用者的參與並傳回所收集的資訊。

在第一個步驟中，我們會清除使用者設定檔，以便對話方塊每次都能從空的設定檔開始。 由於最後一個步驟會在結束時傳回資訊，因此 `AcknowledgementStepAsync` 會在結束時將資訊儲存至使用者狀態，然後將該資訊傳回主要對話方塊，以在最後一個步驟中使用。

[!code-csharp[step implementations](~/../botbuilder-samples/samples/csharp_dotnetcore/43.complex-dialog/Dialogs/TopLevelDialog.cs?range=39-96&highlight=3-4,47-49,56-57)]

**Dialogs\ReviewSelectionDialog.cs**

review-selection 對話方塊會從最上層對話方塊的 `StartSelectionStepAsync` 開始，且具有兩個步驟：

1. 要求使用者選擇要評論的公司，或選擇 `done` 表示完成。
1. 視情況重複此對話或結束。

在此設計中，最上層的對話一律優先於堆疊上的 review-selection 對話，而 review-selection 對話可以是最上層對話的子系。

[!code-csharp[step implementations](~/../botbuilder-samples/samples/csharp_dotnetcore/43.complex-dialog/Dialogs/ReviewSelectionDialog.cs?range=42-106)]

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

**dialogs/mainDialog.js**

我們已經定義了元件對話：`MainDialog`，其包含幾個主要步驟，並指示了對話方塊和提示。 第一個步驟是呼叫 `TopLevelDialog` (下面會說明)。

[!code-javascript[step implementations](~/../botbuilder-samples/samples/javascript_nodejs/43.complex-dialog/dialogs/mainDialog.js?range=43-55&highlight=2)]

**dialogs/topLevelDialog.js**

初始、最上層的對話有四個步驟：

1. 詢問使用者的名稱。
1. 詢問使用者的年齡。
1. 根據使用者的年齡分支。
1. 最後，感謝使用者的參與並傳回所收集的資訊。

在第一個步驟中，我們會清除使用者設定檔，以便對話方塊每次都能從空的設定檔開始。 由於最後一個步驟會在結束時傳回資訊，因此 `acknowledgementStep` 會在結束時將資訊儲存至使用者狀態，然後將該資訊傳回主要對話方塊，以在最後一個步驟中使用。

[!code-javascript[step implementations](~/../botbuilder-samples/samples/javascript_nodejs/43.complex-dialog/dialogs/topLevelDialog.js?range=32-76&highlight=2-3,37-39,43-44)]

**dialogs/reviewSelectionDialog.js**

review-selection 對話方塊會從最上層對話方塊的 `startSelectionStep` 開始，且具有兩個步驟：

1. 要求使用者選擇要評論的公司，或選擇 `done` 表示完成。
1. 視情況重複此對話或結束。

在此設計中，最上層的對話一律優先於堆疊上的 review-selection 對話，而 review-selection 對話可以是最上層對話的子系。

[!code-javascript[step implementations](~/../botbuilder-samples/samples/javascript_nodejs/43.complex-dialog/dialogs/reviewSelectionDialog.js?range=33-78)]

---

## <a name="implement-the-code-to-manage-the-dialog"></a>實作程式碼來管理對話方塊

Bot 的回合處理常式會重複這些對話所定義的一個交談流程。
當我們收到來自使用者的訊息時：

1. 繼續進行作用中的對話 (如果有的話)。
   - 如果沒有作用中的對話，我們會清除使用者設定檔並開始最上層的對話。
   - 如果作用中對話已完成，我們會收集和儲存所傳回的資訊並顯示狀態訊息。
   - 否則，作用中對話仍為中間程序，而我們目前不需要採取任何其他動作。
1. 儲存交談狀態，以便保存對話狀態的所有更新。

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

<!-- **DialogExtensions.cs**

In this sample, we've defined a `Run` helper method that we will use to create and access the dialog context.
Since component dialog defines an inner dialog set, we have to create an outer dialog set that's visible to the message handler code, and use that to create a dialog context.

- `dialog` is the main component dialog for the bot.
- `turnContext` is the current turn context for the bot.

[!code-csharp[Run method](~/../botbuilder-samples/samples/csharp_dotnetcore/43.complex-dialog/DialogExtensions.cs?range=13-24)]

-->

**Bots\DialogBot.cs**

訊息處理常式會呼叫 `RunAsync` 方法來管理對話方塊，而我們已經覆寫回合處理常式來將任何變更儲存至對話和使用者狀態，這可能會在回合期間發生。 基礎 `OnTurnAsync` 會呼叫 `OnMessageActivityAsync` 方法，確保儲存呼叫會在回合結束時執行。

[!code-csharp[Overrides](~/../botbuilder-samples/samples/csharp_dotnetcore/43.complex-dialog/Bots/DialogBot.cs?range=33-48&highlight=5-7)]

**Bots\DialogAndWelcome.cs**

`DialogAndWelcomeBot` 擴充了上述的 `DialogBot`，可在使用者加入對話時提供歡迎訊息，且是 `Startup.cs` 中所建立的項目。

[!code-csharp[On members added](~/../botbuilder-samples/samples/csharp_dotnetcore/43.complex-dialog/Bots/DialogAndWelcome.cs?range=21-38)]

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

**dialogs/mainDialog.js**

在此範例中，我們定義了 `run` 方法，我們將用此方法來建立及存取對話方塊內容。
由於元件對話會定義內部對話集，因此我們必須建立可讓訊息處理常式程式碼看見的外部對話集，並使用其建立對話方塊內容。

- `turnContext` 是 Bot 目前的回合內容。
- `accessor` 是我們建立的存取子，用來管理對話方塊狀態。

[!code-javascript[run method](~/../botbuilder-samples/samples/javascript_nodejs/43.complex-dialog/dialogs/mainDialog.js?range=32-41)]

**bots/dialogBot.js**

訊息處理常式會呼叫 `run` 協助程式方法來管理對話方塊，我們會實作回合處理常式，來將任何變更儲存至對話和使用者狀態，而這可能會在回合期間發生。 呼叫 `next` 會讓基礎實作呼叫 `onDialog` 方法，確保儲存呼叫會在回合結束時執行。

[!code-javascript[Overrides](~/../botbuilder-samples/samples/javascript_nodejs/43.complex-dialog/bots/dialogBot.js?range=24-41)]

**bots/dialogAndWelcomeBot.js**

`DialogAndWelcomeBot` 擴充了上述的 `DialogBot`，可在使用者加入對話時提供歡迎訊息，且是 `index.js` 中所建立的項目。

[!code-javascript[On members added](~/../botbuilder-samples/samples/javascript_nodejs/43.complex-dialog/bots/dialogAndWelcomeBot.js?range=10-21)]

---

## <a name="branch-and-loop"></a>分支和迴圈

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

**Dialogs\TopLevelDialog.cs**

以下是分支邏輯範例，來自_最上層_對話方塊的步驟：

[!code-csharp[branching logic](~/../botbuilder-samples/samples/csharp_dotnetcore/43.complex-dialog/Dialogs/TopLevelDialog.cs?range=68-80)]

**Dialogs\ReviewSelectionDialog.cs**

以下是迴圈邏輯範例，來自_檢閱選取_對話方塊的步驟：

[!code-csharp[looping logic](~/../botbuilder-samples/samples/csharp_dotnetcore/43.complex-dialog/Dialogs/ReviewSelectionDialog.cs?range=96-105)]

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

**dialogs/topLevelDialog.js**

以下是分支邏輯範例，來自_最上層_對話方塊的步驟：

[!code-javascript[branching logic](~/../botbuilder-samples/samples/javascript_nodejs/43.complex-dialog/dialogs/topLevelDialog.js?range=56-64)]

**dialogs/reviewSelectionDialog.js**

以下是迴圈邏輯範例，來自_檢閱選取_對話方塊的步驟：

[!code-javascript[looping logic](~/../botbuilder-samples/samples/javascript_nodejs/43.complex-dialog/dialogs/reviewSelectionDialog.js?range=71-77)]

---

## <a name="to-test-the-bot"></a>測試 Bot

1. 如果您尚未安裝 [Bot Framework Emulator](https://aka.ms/bot-framework-emulator-readme)，請進行安裝。
1. 在您的電腦本機執行範例。
1. 啟動模擬器、連線到您的 Bot 並傳送如下所示的訊息。

![測試複雜對話範例](~/media/emulator-v4/test-complex-dialog.png)

## <a name="additional-resources"></a>其他資源

如需實作對話方式的簡介，請參閱[實作循序交談流程][simple-dialog]，該流程使用單一瀑布式對話和一些提示來建立簡單的互動，以詢問使用者一連串的問題。

Dialogs 程式庫包含提示的基本驗證。 您也可以新增自訂憑證。 如需詳細資訊，請參閱[使用對話提示收集使用者輸入][dialog-prompts]。

若要簡化對話程式碼並重複使用於多個 Bot，您可以將部分的對話集定義為個別的類別。
如需詳細資訊，請參閱[重複使用對話][component-dialogs]。

## <a name="next-steps"></a>後續步驟

> [!div class="nextstepaction"]
> [重複使用對話方塊](bot-builder-compositcontrol.md)

<!-- Footnote-style links -->

[concept-basics]: bot-builder-basics.md
[concept-state]: bot-builder-concept-state.md
[concept-dialogs]: bot-builder-concept-dialog.md

[simple-dialog]: bot-builder-dialog-manage-conversation-flow.md
[dialog-prompts]: bot-builder-prompts.md
[component-dialogs]: bot-builder-compositcontrol.md

[cs-sample]: https://aka.ms/cs-complex-dialog-sample
[js-sample]: https://aka.ms/js-complex-dialog-sample
