---
title: 偵錯指導方針 | Microsoft Docs
description: 了解如何對聊天機器人進行偵錯。
keywords: 對聊天機器人進行偵錯, botframework 偵錯
author: ivorb
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 07/17/2019
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: a04e1acc37c488fcd7530b7df2dd8668d4cfdcc2
ms.sourcegitcommit: a6d02ec4738e7fc90b7108934740e9077667f3c5
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/04/2019
ms.locfileid: "70299189"
---
# <a name="debugging-guidelines"></a>偵錯指導方針

[!INCLUDE[applies-to](../includes/applies-to.md)]

Bot 是將許多不同組件整合在一起運作的複雜應用程式。 和其他複雜的應用程式一樣，這種方式會導致一些有趣的錯誤，或是讓 Bot 產生出乎意料的行為。

針對聊天機器人進行偵錯有時候是很困難的工作。 每個開發人員都有自己慣用的方法來完成該工作。我們提供以下的指導方針給您做為建議，可適用於大部分的 Bot。

確認您的 Bot 如預期般運作之後，下一步就是將它連線至通道。 若要這樣做，您可以將 Bot 部署到預備伺服器，並建立自己的直接線路用戶端以供 Bot 連線。
<!--IBTODO [Direct Line client](bot-builder-howto-direct-line.md)-->

建立自己的用戶端可讓您定義通道內部的運作方式，並且可特別針對 Bot 回應特定活動交換的方式進行測試。 一旦 Bot 連線到您的用戶端後，請執行您的測試以設定 Bot 狀態並驗證功能。 如果您的 Bot 會利用如語音之類的功能，使用這些通道可提供驗證該功能的方式。

透過 Azure 入口網站在此處使用模擬器和網路聊天，可提供 Bot 在使用不同通道互動時的執行方式見解。

針對 Bot 進行偵錯的方式，與針對其他多執行緒應用程式進行偵錯的方式相似，都要使用設定中斷點或是如即時運算視窗的功能。 

Bot 遵循事件驅動程式設計架構，如果您還不熟悉，就很難進行說明。 讓您的 Bot 處於無狀態、多執行緒以及處理非同步/等候呼叫的概念，都會造成無法預期的錯誤 (Bug)。 針對 Bot 偵錯的運作方式與針對其他多執行緒應用程式的方式相似，我們將會提供一些實用的建議、工具和資源來協助您。

## <a name="understanding-bot-activities-with-the-emulator"></a>使用模擬器了解 Bot 活動

您的 Bot 除了處理一般的_訊息_活動之外，也會處理不同類型的[活動](bot-builder-basics.md#the-activity-processing-stack)。 使用[模擬器](../bot-service-debug-emulator.md)會顯示那些活動的內容、發生的時機，以及它們所包含的內容。 了解那些活動有助於您更有效率地撰寫 Bot 的程式碼，並且可讓您確認 Bot 如預期地傳送和接收活動。

## <a name="saving-and-retrieving-user-interactions-with-transcripts"></a>透過文字記錄儲存及擷取使用者互動

Azure Blob 文字記錄儲存體提供特製化資源，您可以在其中[儲存及擷取文字記錄](bot-builder-howto-v4-storage.md)，內含使用者與您的 bot 之間的互動。  

此外，一旦儲存使用者輸入互動，您即可使用 Azure 的「儲存體總管」  ，手動檢視 Blob 文字記錄存放區內所儲存的文字記錄中包含的資料。 下列範例會從 "_mynewtestblobstorage_" 的設定開啟「儲存體總管」  。 若要開啟儲存的使用者輸入選取：  Blob 容器 > ChannelId > TranscriptId > ConversationId

![Examine_stored_transcript_text](./media/examine_transcript_text_in_azure.png)

這會開啟以 JSON 格式儲存的使用者對話輸入。 使用者輸入會與索引鍵 "_text:_ " 一起保存。

## <a name="how-middleware-works"></a>中介軟體的運作方式

第一次嘗試使用[中介軟體](bot-builder-concept-middleware.md)時，可能會覺得不是非常直覺，特別是關於執行的接續或最少運算作法。 當執行傳遞給 Bot 邏輯時，中介軟體可透過呼叫 `next()` 委派命令，在回合的前緣或後緣執行。 

如果您使用以多個部分組成的中介軟體，則委派可能會將執行傳遞到中介軟體的不同部分 (若您的管線導向方式是如此的話)。 如需詳細資料，請參閱 [Bot 中介軟體管線](bot-builder-concept-middleware.md#the-bot-middleware-pipeline)以澄清概念。

如果沒有呼叫 `next()` 委派，則稱之為[最少運算路由](bot-builder-concept-middleware.md#short-circuiting)。 當中介軟體滿足目前的活動，並判斷不需要傳遞執行時，就會發生此情況。 

了解中介軟體最少運算發生的時機和原因，有助於指出在您管線中哪個中介軟體的部分應先出現。 此外，針對由 SDK 或其他開發人員提供的內建中介軟體，請務必了解它們預期會出現的內容。 在深入了解內建的中介軟體前，有些人發現先嘗試建立自己的中介軟體進行一些實驗非常有幫助。

<!-- Snip: QnA was once implemented as middleware.
For example [QnA maker](bot-builder-howto-qna.md) is designed to handle certain interactions and short-circuit the pipeline when it does, which can be confusing when first learning how to use it.
-->

## <a name="understanding-state"></a>了解狀態

持續追蹤狀態是 Bot 中很重要的一環，特別是針對複雜的工作更是如此。 一般而言，最佳做法是盡可能地快速處理活動，然後讓處理程序完成，如此狀態才會進入保存狀態。 活動可能近乎同時傳送給您的 Bot，而由於非同步架構的關係，可能造成非常令人困惑的錯誤 (Bug)。

因此，最重要的是，務必確定保存的狀態符合您預期的狀態。 根據您保存的狀態的位置，適用於 [Cosmos DB](https://docs.microsoft.com/azure/cosmos-db/local-emulator) 和 [Azure 表格儲存體](https://docs.microsoft.com/azure/storage/common/storage-use-emulator)的儲存體模擬器有助於您在使用生產環境儲存體之前先驗證該狀態。

## <a name="how-to-use-activity-handlers"></a>如何使用活動處理常式

活動處理常式可能導入另一層次的複雜度，特別是因為每個活動都在獨立的執行緒 (或是背景工作角色，視您的程式設計語言而定) 上執行。 根據處理常式執行的內容，這可能會造成目前狀態與預期狀態不符的問題。

內建狀態在回合結束時會被覆寫，不過由該回合產生的任何活動都會獨立執行，與回合管線無關。 通常，這對我們沒有影響，但如果活動處理常式變更我們需要寫入以包含該變更的狀態，則狀況又會有所不同。 在此情況下，回合管線可能會等候活動完成處理，然後才完成作業，以確保它會記錄該回合的正確狀態。

_send activity_ 方法，以及其處理常式，會造成一個特有的問題。 只需在 _on send activities_ 處理常式內呼叫 _send activity_，就會導致無限分支的執行緒。 針對該問題，有一些方法可以作為因應措施，例如將額外訊息附加到輸出資訊，或寫出到其他位置 (例如主控台)，或是寫出檔案來避免造成 Bot 當機。

## <a name="next-steps"></a>後續步驟

> [!div class="nextstepaction"]
> [如何對聊天機器人進行單元測試](unit-test-bots.md)

## <a name="additional-resources"></a>其他資源

* [Visual Studio 偵錯](https://docs.microsoft.com/visualstudio/debugger/index)
* 針對 Bot Framework 進行[偵錯、追蹤與分析](https://docs.microsoft.com/dotnet/framework/debug-trace-profile/) \(機器翻譯\)
* 針對您不想在生產環境程式碼中納入的方法使用 [ConditionalAttribute](https://docs.microsoft.com/dotnet/api/system.diagnostics.conditionalattribute?view=netcore-2.0) \(英文\)
* 使用如 [Fiddler](https://www.telerik.com/fiddler) \(英文\) 的工具查看網路流量
* [針對一般問題進行疑難排解](../bot-service-troubleshoot-bot-configuration.md)以及該區段中的其他疑難排解文章
