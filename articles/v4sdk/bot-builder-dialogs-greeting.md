---
title: 實作問候語對話方塊 | Microsoft Docs
description: 使用對話方塊來歡迎使用者加入對話。
keywords: 問候語, 對話方塊, 對話流程, 對話集
author: JonathanFingold
ms.author: v-jofing
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 03/12/2019
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 451906a6187f88bd80bef7519d6cdc1aa7bd6177
ms.sourcegitcommit: f84b56beecd41debe6baf056e98332f20b646bda
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 05/03/2019
ms.locfileid: "65033872"
---
# <a name="implement-a-greeting-dialog"></a>實作問候語對話方塊

[!INCLUDE [pre-release-label](../includes/pre-release-label.md)]

您可以使用對話方塊來歡迎使用者加入對話。

如需有關歡迎使用者的詳細資訊，請參閱如何[傳送歡迎訊息給使用者][send-welcome]。

## <a name="prerequisites"></a>必要條件

- [管理狀態][concept-state]、[對話方塊程式庫][concept-dialogs]、如何[管理對話][simple-flow]及[使用對話方塊提示來收集使用者輸入][prompting]的知識。
- 一份 ??? [**CSharp**][cs-sample] 或 [**JavaScript**][js-sample] 中的範例。

## <a name="task-as-in-to-do-x-do-these-things"></a>\<工作 > [如同執行 X，執行下列項目]

<!--The key lines of code for this task.
    here are the cool lines that do that.
    just the few lines of implementation without setup.
-->

## <a name="about-the-sample-code"></a>關於範例程式碼

<!--setup & implementation & discussion of the sample code-->

- 其他所需套件 (AI.Luis、Dialogs 等)

<!--Any other key elements to get the code to work.
    Include setup for only the bits critical to the task at hand.
    don't go over all the code in the sample.
-->

## <a name="to-test-the-bot"></a>測試 Bot

1. 如果您尚未安裝 [Bot Framework Emulator](https://aka.ms/bot-framework-emulator-readme)，請進行安裝。
1. 在您的電腦本機執行範例。
1. 啟動模擬器、連線到您的 Bot 並傳送如下所示的訊息。

待辦事項：擷取新的螢幕擷取畫面。

<!--![test dialog prompt sample](~/media/emulator-v4/test-dialog-prompt.png)-->

## <a name="discussion-optional"></a>討論 [選用]

<!--Might be short and descriptive or include additional code for scenarios not covered in the samples repo
-->

## <a name="addition-information"></a>其他資訊

<!--include cross-linking other articles about the same sample.-->

- 所有範例的「關於核心 Bot」登陸頁面連結 (適用於 CoreBot 所衍生的文章)。

## <a name="next-steps"></a>後續步驟

> [!div class="nextstepaction"]
> [處理使用者中斷](bot-builder-howto-handle-user-interrupt.md)

<!-- Footnote-style links -->

[concept-basics]: bot-builder-basics.md
[concept-state]: bot-builder-concept-state.md
[concept-dialogs]: bot-builder-concept-dialog.md

[send-welcome]: bot-builder-send-welcome-message.md

[simple-flow]: bot-builder-dialog-manage-conversation-flow.md
[prompting]: bot-builder-prompts.md
[component-dialogs]: bot-builder-compositcontrol.md

[cs-sample]: ???
[js-sample]: ???
