---
title: 重複使用對話方塊 | Microsoft Docs
description: 了解如何在 Bot Framework SDK 中使用元件對話將 Bot 邏輯模組化。
keywords: 複合控制項, 模組化 Bot 邏輯
author: v-ducvo
ms.author: v-ducvo
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 07/05/2019
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 77f1c154af5821b1e476546f307a01be27f568c0
ms.sourcegitcommit: b498649da0b44f073dc5b23c9011ea2831edb31e
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/05/2019
ms.locfileid: "67587496"
---
# <a name="reuse-dialogs"></a>重複使用對話方塊

[!INCLUDE[applies-to](../includes/applies-to.md)]

透過元件對話，您可以建立獨立的對話來處理特定案例，並將大型對話集分割成更多可管理的片段。 這些片段都有自己的對話集，而且會避免與本身之外設定的對話集發生任何名稱衝突。

## <a name="prerequisites"></a>必要條件

- [Bot 基本概念][concept-basics], the [dialogs library][concept-dialogs]，及如何[管理對話][簡單流程]的知識。
- [**CSharp**][cs-sample] or [**JavaScript**][js-sample] 中的一份多回合提示範例。

## <a name="about-the-sample"></a>關於範例

在多回合提示範例中，我們會使用瀑布式對話方塊、一些提示和元件對話來建立簡單互動，以詢問使用者一系列的問題。 程式碼會使用對話方塊來循環下列步驟：

| 步驟        | 提示類型  |
|:-------------|:-------------|
| 要求使用者的運輸模式 | 選擇提示 |
| 要求使用者的名稱 | 文字提示 |
| 詢問使用者是否要提供年齡 | 確認提示 |
| 如果他們回答 [是]，則要求年齡  | 包含驗證的數字提示，只接受大於 0 且小於 150 的年齡。 |
| 詢問是否可收集資訊 | 重複使用確認提示 |

最後，如果他們回答 [是]，則顯示所收集的資訊；否則，告訴使用者將不會保留其資訊。

## <a name="implement-the-component-dialog"></a>實作元件對話

在多回合提示範例中，我們會使用_瀑布式對話方塊_、一些_提示_和_元件對話_來建立簡單互動，以詢問使用者一系列的問題。

元件對話會封裝一個或多個對話。 元件對話有內部對話集，新增至內部對話集的對話和提示會有其本身的識別碼，只在元件對話中顯示。

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

若要使用對話方塊，請安裝 **Microsoft.Bot.Builder.Dialogs** NuGet 套件。

**Dialogs\UserProfileDialog.cs**

此處的 `UserProfileDialog` 類別衍生自 `ComponentDialog` 類別。

[!code-csharp[Class](~/../botbuilder-samples/samples/csharp_dotnetcore/05.multi-turn-prompt/Dialogs/UserProfileDialog.cs?range=13)]

在建構函式中，`AddDialog` 方法會將對話和提示新增至元件對話。 您使用此方法新增的第一個項目會設定為初始對話，但您可以藉由明確地設定 `InitialDialogId` 屬性，來加以變更。 當您啟動元件對話時，就會啟動其「初始對話」  。

[!code-csharp[Constructor](~/../botbuilder-samples/samples/csharp_dotnetcore/05.multi-turn-prompt/Dialogs/UserProfileDialog.cs?range=17-42)]

這是瀑布式對話方塊中第一個步驟的實作。

[!code-csharp[First step](~/../botbuilder-samples/samples/csharp_dotnetcore/05.multi-turn-prompt/Dialogs/UserProfileDialog.cs?range=44-54)]

如需有關如何實作瀑布式對話方塊的詳細資訊，請參閱如何[實作循序對話流程](bot-builder-dialog-manage-complex-conversation-flow.md)。

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

若要使用對話方塊，您的專案需要安裝 **botbuilder-dialogs** npm 套件。

**dialogs/userProfileDialog.js**

此處的 `UserProfileDialog` 類別會展開 `ComponentDialog`。

[!code-javascript[Class](~/../botbuilder-samples/samples/javascript_nodejs/05.multi-turn-prompt/dialogs/userProfileDialog.js?range=24)]

在建構函式中，`AddDialog` 方法會將對話和提示新增至元件對話。 您使用此方法新增的第一個項目會設定為初始對話，但您可以藉由明確地設定 `InitialDialogId` 屬性，來加以變更。 當您啟動元件對話時，就會啟動其「初始對話」  。

[!code-javascript[Constructor](~/../botbuilder-samples/samples/javascript_nodejs/05.multi-turn-prompt/dialogs/userProfileDialog.js?range=25-47)]

這是瀑布式對話方塊中第一個步驟的實作。

[!code-javascript[First step](~/../botbuilder-samples/samples/javascript_nodejs/05.multi-turn-prompt/dialogs/userProfileDialog.js?range=66-73)]

如需有關如何實作瀑布式對話方塊的詳細資訊，請參閱如何[實作循序對話流程](bot-builder-dialog-manage-complex-conversation-flow.md)。

---

在執行階段上，元件對話會維護其自己的對話堆疊。 當元件對話啟動時：

- 執行個體會建立並新增至外部對話堆疊
- 其會建立新增至其狀態的內部對話堆疊
- 其會啟動初始對話，並將該對話新增至內部對話堆疊。

在父代內容中，這看起來就像元件是使用中的對話。 在元件內部，這看起來就像初始對話是使用中的對話。

### <a name="implement-the-rest-of-the-dialog-and-add-it-to-the-bot"></a>實作對話的其餘部分，並將其新增到 Bot

在外部的對話集內 (也就是您在其中新增元件對話的對話集)，元件對話會有您建立元件時使用的識別碼。 在外部集合中，元件看起來就像單一對話，與提示相當類似。

若要使用元件對話，請將其執行個體新增至 Bot 的對話集 - 這是外部對話集。

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

**Bots\DialoBot.cs**

在範例中，這是使用從 Bot 的 `OnMessageActivityAsync` 方法呼叫的 `RunAsync` 方法所完成。

[!code-csharp[OnMessageActivityAsync](~/../botbuilder-samples/samples/csharp_dotnetcore/05.multi-turn-prompt/Bots/DialogBot.cs?range=42-48)]

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

**dialogs/userProfileDialog.js**

在範例中，我們已將 `run` 方法新增至使用者設定檔對話中。

[!code-javascript[run method](~/../botbuilder-samples/samples/javascript_nodejs/05.multi-turn-prompt/dialogs/userProfileDialog.js?range=55-64)]

**bots/dialogBot.js**

`run` 方法是從 Bot 的 `onMessage` 方法中呼叫。

[!code-javascript[onMessage](~/../botbuilder-samples/samples/javascript_nodejs/05.multi-turn-prompt/bots/dialogBot.js?range=30-37)]

---

## <a name="to-test-the-bot"></a>測試 Bot

1. 如果您尚未安裝 [Bot Framework Emulator](https://aka.ms/bot-framework-emulator-readme)，請進行安裝。
1. 在您的電腦本機執行範例。
1. 啟動模擬器、連線到您的 Bot 並傳送如下所示的訊息。

![多回合提示對話的執行範例](../media/emulator-v4/multi-turn-prompt.png)

## <a name="additional-information"></a>其他資訊

### <a name="how-cancellation-works-for-component-dialogs"></a>元件對話的取消作業如何運作

如果您從元件對話的內容呼叫_取消所有對話_，元件對話將會取消其內部堆疊上的所有對話，然後結束對話，並將控制項傳回給外部堆疊上的下一個對話。

如果您從外部內容呼叫_取消所有對話_，則元件及外部內容上其餘的對話都會遭到取消。

在 Bot 中管理巢狀元件對話時，請記住這點。

## <a name="next-steps"></a>後續步驟

您可以增強 Bot 以回應其他輸入，例如 [說明] 或 [取消] 可以中斷交談的一般流程。

> [!div class="nextstepaction"]
> [處理使用者中斷](bot-builder-howto-handle-user-interrupt.md)

<!-- Footnote-style links -->

[concept-basics]: bot-builder-basics.md
[concept-state]: bot-builder-concept-state.md
[concept-dialogs]: bot-builder-concept-dialog.md

[simple-flow]: bot-builder-dialog-manage-conversation-flow.md
[prompting]: bot-builder-prompts.md
[component-dialogs]: bot-builder-compositcontrol.md

[cs-sample]: https://aka.ms/cs-multi-prompts-sample
[js-sample]: https://aka.ms/js-multi-prompts-sample
