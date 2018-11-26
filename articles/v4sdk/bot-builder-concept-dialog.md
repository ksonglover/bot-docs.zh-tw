---
title: Bot Builder SDK 內的對話方塊 | Microsoft Docs
description: 說明什麼是對話方塊，以及對話方塊在 Bot Builder SDK 中的運作方是。
keywords: 交談流程, 提示, 對話狀態, 辨識意圖, 單一回合, 多個回合, Bot 交談, 對話, 提示, 瀑布, 對話集
author: johnataylor
ms.author: johtaylo
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 11/22/2018
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 52a2867f4d62be4969ed77d8d83e1e752edd7f92
ms.sourcegitcommit: 6cb37f43947273a58b2b7624579852b72b0e13ea
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 11/22/2018
ms.locfileid: "52288837"
---
# <a name="dialogs-library"></a>對話方塊程式庫

[!INCLUDE [pre-release-label](../includes/pre-release-label.md)]

SDK 的核心是透過「對話方塊」的概念管理對話。 對話方塊物件可處理輸入活動，並產生輸出回應。 Bot 的商務邏輯會在對話方塊類別中直接或間接地執行。

在執行階段，對話方塊執行個體會在堆疊中排列。 堆疊頂端的對話方塊稱為 ActiveDialog。 目前使用中的對話方塊會處理輸入活動。 堆疊會在每回合對話之間保存下來 (不受時間限制，且可能會橫跨好幾天)。 

## <a name="dialog-lifecycle"></a>對話方塊生命週期

對話方塊會實作三項主要功能：
- BeginDialog
- ContinueDialog
- ResumeDialog

在執行階段，對話方塊和 DialogContext 類別會一起運作，以選擇適當的對話方塊來處理活動。 DialogContext 類別會繫結持續性的對話方塊堆疊、輸入 Activity 和 DialogSet 類別。 DialogSet 包含 Bot 可以呼叫的對話方塊。

DialogContext 的介面會反映對話方塊關於開始和繼續 (continue) 的基礎觀念。 應用程式的一般模式一律會先呼叫 ContinueDialog。 如果沒有堆疊，因此也沒有 ActiveDialog，應用程式應該會藉由在 DialogContext 上呼叫 BeginDialog 來開始進行其選擇的對話方塊。 這會導致系統將 DialogSet 中的對應對話方塊項目推送至堆疊 (就技術上而言，新增至堆疊的會是對話方塊的識別碼)，然後此項目會將呼叫委派給特定對話方塊物件上的 BeginDialog。 如果已有 ActiveDialog，其會直接將呼叫委派給該對話方塊的 ContinueDialog，而在處理時，會將任何相關聯的持續性屬性提供給該對話方塊。

請注意，**對話方塊的 BeginDialog** 是初始化程式碼，並會採用初始化屬性 (在程式碼中稱為「選項」)，至於 **對話方塊的 ContinueDialog** 程式碼則可在執行後，繼續 (continue) 在遵循持續性的活動抵達時執行。 例如，假設有一個對話方塊問使用者問題，則問題會在 BeginDialog 中詢問，且應該會在 ContinueDialog 中回答。

為了支援巢狀對話方塊 (也就是該對話具有子對話)，另有一種繼續 (continuation) 類型，稱為繼續 (resumption)。 當子對話方塊完成時，DialogContext 會在父對話方塊上呼叫 ResumeDialog 方法。

提示和瀑布都是 SDK 所提供的具體對話方塊範例。 許多案例是藉由撰寫這些抽象概念來加以建置的，但實際上，所執行的邏輯一律是相同的開始，也就是此處所述的繼續 (continue) 和繼續 (resume) 模式。 

Bot Builder SDK 中的 **對話方塊** 程式庫包含「提示」、「瀑布對話方塊」和「元件對話方塊」之類的內建功能，可協助您管理 Bot 的對話。 您可以使用提示向使用者詢問不同類型的資訊，使用瀑布將多個步驟結合成順序，以及使用元件對話將對話邏輯封裝到不同類別，進而整合到其他 Bot。
## <a name="waterfall-dialogs-and-prompts"></a>瀑布對話方塊和提示

**對話方塊** 程式庫隨附一組提示類型，供您用來收集各種類型的使用者輸入。 例如，若要要求使用者輸入文字，您可以使用 **TextPrompt**；若要要求使用者輸入數字，您可以使用 **NumberPrompt**；若要要求輸入日期和時間，則可以使用 **DateTimePrompt**。 提示是特定類型的對話。 若要從瀑布對話方塊使用提示，請在相同的對話方塊集合內新增瀑布和提示。 

因為提示和回應的互動天性，若要實作提示，瀑布對話方塊中至少需要兩個步驟，一個用來傳送提示，一個用來擷取和處理回應。  如果您有額外的提示，有時您可以將兩者結合，使用單一函式先處理使用者的回應，然後再啟動下一個提示。

`WaterfallDialog` 是特定的對話方塊實作，可用來向使用者收集資訊或引導使用者完成一系列的工作。 這些工作會實作為函式陣列，其中第一個函式的結果將作為引數傳遞至下一個函式，依此類推。 每個函式通常代表整個程序的其中一個步驟。 在每個步驟中，Bot 會提示使用者提供輸入、等候回應，然後將結果傳遞給下一個步驟。 

提示和瀑布都是對話方塊，如下列的類別階層所示。 

![對話方塊類別](media/bot-builder-dialog-classes.png)

瀑布對話方塊是由一系列的瀑布步驟所組成的。 每個步驟都是非同步的委派，會採用「瀑布步驟內容」(`step`) 參數。 模式是指瀑布步驟中所完成的最後一件工作，是要開始子對話方塊 (通常是提示)，還是結束瀑布本身。 下圖顯示瀑布步驟序列和所發生的堆疊作業。

![對話方塊概念](media/bot-builder-dialog-concept.png)

您所能處理的傳回值，可來自對話方塊中瀑布步驟內的對話方塊，或來自 Bot 的 on turn 處理常式。
在瀑布步驟內，對話方塊會在瀑布步驟內容的「結果」屬性中提供傳回值。
您通常只需要檢查 Bot 回合邏輯的對話方塊回合結果狀態。

## <a name="dialog-state"></a>對話方塊狀態

對話是一種實作多回合交談邏輯的方法，因此，這是 SDK 中依賴多個回合中保存狀態的功能範例。 對話中若無狀態，您的 Bot 便無法得知其位於對話集中的何處，或其已經蒐集的資訊。

對話式 Bot 通常會保留對話集合做為其 Bot 實作中的成員變數。 該對話集會透過存取子物件 (可供存取保存狀態) 的控點來建立。 如需 Bot 內狀態的背景，請參閱[管理狀態](bot-builder-concept-state.md)。 

![對話方塊狀態](media/bot-builder-dialog-state.png)

當 Bot 的 on turn 處理常式受到呼叫時，Bot 會藉由在對話集上呼叫 *create context* 來初始化對話子系統，以傳回「對話內容」。 建立對話內容時需要狀態，透過建立對話集時所提供的存取子即可存取狀態。 對話集可以透過存取子，取得適當的對話狀態 JSON。 該對話內容包含對話所需的必要資訊。

在[儲存交談和使用者資料](bot-builder-howto-v4-state.md)中可找到狀態存取子的詳細資料。

## <a name="repeating-a-dialog"></a>重複對話方塊

若要重複對話方塊，請使用「取代對話方塊」方法。 對話方塊內容的「取代對話方塊」方法會將目前的對話方塊從堆疊取出，並將取代的對話方塊推送至堆疊上，開始進行該對話方塊。 藉由以對話方塊本身來取代對話，您可以使用這個方法來建立迴圈。 請注意，如果您需要保存目前對話方塊的內部狀態，則必須將資訊傳遞給「取代對話方塊」方法呼叫中的新對話方塊執行個體，然後適當地將對話方塊初始化。 傳遞至新對話方塊的選項，可透過步驟內容在任何對話方塊步驟中的「選項」屬性來存取。 這除了很適合用來處理複雜對話流程，也能管理功能表。

## <a name="branch-a-conversation"></a>產生對話分支

對話方塊內容會維持_對話方塊堆疊_，對於堆疊中的每個對話方塊，追蹤哪個步驟是下一步。 「開始對話方塊」方法會將對話方塊推送至堆疊頂端，而其「結束對話方塊」方法則會將頂端的對話方塊從堆疊中移除。

對話方塊可以藉由呼叫對話方塊內容的「開始對話方塊」方法，並提供新對話方塊的識別碼，在同一對話集合內開始新的對話方塊，然後讓新的對話成為目前使用中的對話方塊。 原始對話方塊仍在堆疊中，但呼叫對話方塊內容的「繼續 (continue) 對話方塊」方法只會傳送到堆疊頂端的對話方塊，也就是「使用中的對話方塊」。 當對話方塊從堆疊中移除時，對話方塊內容會從原始對話方塊停止的地方，繼續 (resume) 使用堆疊中瀑布的下一個步驟。

因此，藉由包含在對話方塊中，可以有條件地從一組可用的對話方塊中選擇對話方塊來開始的步驟，您即可在對話流程內建立分支。

## <a name="component-dialog"></a>元件對話方塊
有時您會想要撰寫可重複使用的對話方塊，以便用於不同案例。 可能的範例包括地址 對話方塊，以便要求使用者提供街道、城市和郵遞區號的值。 

ComponentDialog 具有個別的 DialogSet，因此可提供某種程度的隔離。 其可透過擁有個別的 DialogSet，來避免名稱與包含對話方塊 的父代發生衝突，並且其也會建立自己的獨立內部對話方塊 執行階段 (藉由建立自己的 DialogContext)，並對該執行階段分派活動。 這個第二次分派表示其有機會攔截活動。 如果您想要實作「說明」和「取消」等功能，這會非常有用。  請參閱 [Enterprise Bot 範本](https://aka.ms/abs/templates/cabot)範例。 

## <a name="next-steps"></a>後續步驟

> [!div class="nextstepaction"]
> [使用對話方塊程式庫來收集使用者輸入](bot-builder-prompts.md)
