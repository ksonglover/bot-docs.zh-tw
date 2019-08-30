---
title: Bot Framework SDK 移轉概觀 | Microsoft Docs
description: 提供 SDK 變更以及如何從 v3 遷移至 v4 的概觀。
keywords: 聊天機器人移轉
author: mmiele
ms.author: v-mimiel
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 06/11/2019
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 576947edf99705e5d0d8850837b3469f13381d06
ms.sourcegitcommit: 008aa6223aef800c3abccda9a7f72684959ce5e7
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/26/2019
ms.locfileid: "70026412"
---
# <a name="migration-overview"></a>移轉概觀

Bot Framework SDK v4 是根據客戶的意見反應以及從舊版 SDK 學到的經驗來建置而成的。 新版本導入了正確的抽象層級，同時又讓聊天機器人元件擁有彈性架構。 舉例來說，新版本可讓您建立簡單的聊天機器人，然後使用 Bot Framework SDK v4 的模組化和擴充性讓這個聊天機器人變得成熟。

> [!NOTE]
> Bot Framework SDK v4 致力於讓簡單的事情保持簡單，並讓複雜的事情得以實現。

由於採用了開放式方法，Bot Framework SDK v4 是透過與社群合作而建置起來的。 首次提交提取要求時，[參與者授權合約](https://cla.opensource.microsoft.com/) (CLA) 會自動判斷您是否需要授權。 所有存放庫只須進行此程序一次即可。 一般而言，建立一組要實現的目標需要一些時間。

## <a name="what-happens-to-bots-built-using-sdk-v3"></a>使用 SDK v3 所建置的聊天機器人會發生什麼事

Bot Framework SDK v3 將會淘汰，但現有的 V3 聊天機器人工作負載會繼續執行而不會中斷。 如需詳細資訊，請參閱[Bot Framework SDK 第 3 版存留期支援](https://docs.microsoft.com/en-us/azure/bot-service/bot-service-resources-bot-framework-faq?view=azure-bot-service-4.0#bot-framework-sdk-version-3-lifetime-support)。

強烈建議您開始將 V3 聊天機器人遷移至 V4。 為了支援此移轉，我們已製作相關文件，且會透過標準管道對移轉計劃提供延伸支援。

## <a name="advantages"></a>優點

- 更豐富、具彈性且開放的架構：可實現更有彈性的對話設計
- 可用性：導入其他有新管道功能的案例
- 在開發週期擴充了主題專家 (SME) 人員：新的 GUI 設計工具可讓非開發人員針對對話設計進行共同作業
- 開發速度：新的偵錯和測試開發人員工具
- 效能深入解析：新的遙測功能，可評估及改善對話品質
- 智慧：改進了認知服務功能

## <a name="why-migrate"></a>為何要遷移
<!-- [!] The declarative model introduced with Adaptive Dialogs would go great here (when ready).  -->
- 對話管理有彈性且已經過改善
  - 用於處理活動的聊天機器人配接器
  - 狀態管理已重構
  - 新增對話程式庫
  - 有中介軟體來實現可撰寫且可延伸的設計：乾淨且一致而可自訂行為的勾點
- 針對 .NET Core 來建置
  - 提升效能
  - 跨平台相容性 (Windows/Mac/Linux)
- 跨多種程式設計語言的一致程式設計模型
- 經過改善的文件
- Bot Inspector 會提供已擴充的偵錯功能
- 虛擬小幫手
  - 全方位的解決方案，可讓您更容易地建立有基本對話式意圖、分派整合、QnA Maker、Application Insights 和自動化部署的聊天機器人。
  - 可延伸的技能。 藉由將可重複使用的交談式功能 (稱為技能) 拼接在一起來撰寫交談式體驗。
- 測試架構：現成可用的測試功能，具有新的傳輸獨立配接器架構
- 遙測：可使用對話式 AI 分析來重點了解聊天機器人的健康情況和行為
- 即將推出 (預覽)
  - 自適性對話方塊：建置可隨著對話的進展而動態變化的對話
  - 語言產生：對片語定義多個變化
- 未來
  - 宣告式設計允許設計工具有抽象層級
  - GUI 對話設計工具
- Azure Bot 服務 
  - Direct Line Speech 通道。 結合 Bot Framework 和 Microsoft 的語音服務。 這可提供通道，以便能在用戶端與聊天機器人應用程式之間雙向串流語音和文字

## <a name="whats-changed"></a>變更的項目

Bot Framework SDK v4 支援與 v3 相同的基礎 Bot Framework Service。 不過，v4 是舊版 SDK 的重構，目的是為了提升聊天機器人的建立彈性和控制力。 其包含下列工具：

- 導入了聊天機器人配接器
  - 此配接器是活動處理堆疊的一部分
  - 其會處理 Bot Framework 驗證，並為每個回合初始化內容
  - 其會管理通道和聊天機器人回合處理常式之間的傳入和傳出流量，並封裝對 Bot Framework 連接器服務的呼叫
  - 如需詳細資料，請參閱[聊天機器人的運作方式](../bot-builder-basics.md)
- 狀態管理已重構
  - 狀態資料不再於 Bot 內自動提供使用
  - 現在會透過狀態管理物件和屬性存取子管理狀態
  - 如需詳細資料，請參閱[管理狀態](../bot-builder-concept-state.md)
- 導入了新的對話程式庫
  - v3 對話需要針對新的對話程式庫重新撰寫
  - 如需詳細資料，請參閱[對話程式庫](../bot-builder-concept-dialog.md)

## <a name="whats-involved-in-migration-work"></a>移轉工作牽涉哪些步驟

- 更新設定邏輯
- 植入任何重要的使用者狀態
  - 附註：敏感的使用者狀態不能留在聊天機器人的狀態中，請改為儲存在受您控制的個別儲存體中
- 植入聊天機器人和對話方塊邏輯 (如需詳細資訊，請參閱語言專屬主題)

### <a name="migration-estimation-worksheet"></a>移轉估計工作表

下列工作表可引導您評估移轉工作負載。 在 [發生次數]  資料行中，將 *count* 取代為實際數值。 在 [T 恤]  資料行中，輸入如下的值：*小*、*中*、*大*(視您的評估而定)。

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

| 步驟 | V3 | V4 | 發生次數 | 複雜度 | T 恤 |
| -- | -- | -- | -- | -- | -- |
若要取得連入活動 | IDialogContext.Activity | ITurnContext.Activity | count | 小型  
若要建立活動並傳送給使用者 | activity.CreateReply(“text”) IDialogContext.PostAsync | MessageFactory.Text(“text”) ITurnContext.SendActivityAsync | count | 小型 |
狀態管理 | UserData、ConversationData 和 PrivateConversationData context.UserData.SetValue context.UserData.TryGetValue botDataStore.LoadAsyn | UserState、ConversationState 和 PrivateConversationState (含屬性存取子) | context.UserData.SetValue - count context.UserData.TryGetValue - count botDataStore.LoadAsyn - count | 中至大 (請參閱可用的[使用者狀態管理](https://docs.microsoft.com/en-us/azure/bot-service/bot-builder-concept-state?view=azure-bot-service-4.0#state-management)) |
處理對話的開始 | Implement IDialog.StartAsync | 讓此項成為瀑布式對話的第一個步驟。 | count | 小型 |  
傳送活動 | IDialogContext.PostAsync. | Call ITurnContext.SendActivityAsync. | count | 小型 |  
等候使用者回應 | 使用 IAwaitable<IMessageActivity>parameter 參數並呼叫 IDialogContext.Wait | 傳回等候 ITurnContext.PromptAsync 以開始提示對話方塊。 然後在瀑布的下一個步驟中擷取結果。 | count | 中 (視流程而定) |  
處理對話的接續 | IDialogContext.Wait | 在瀑布式對話方塊中新增其他步驟，或實作 Dialog.ContinueDialogAsync | count | 大型 |  
表示在使用者的下一則訊息前處理結束 | IDialogContext.Wait | 傳回 Dialog.EndOfTurn。 | count | 中 |  
開始子對話 | IDialogContext.Call | 傳回等候步驟內容的 BeginDialogAsyncmethod。 如果子對話方塊傳回值，該值可透過步驟內容的 Resultproperty 用於瀑布的下一個步驟。 | count | 中 |  
以新對話取代目前的對話 | IDialogContext.Forward | 傳回等候 ITurnContext.ReplaceDialogAsync。 | count | 大型 |  
表示目前對話已完成的訊號 | IDialogContext.Done | 傳回等候步驟內容的 EndDialogAsync 方法。 | count | 中 |  
對話失敗。 | IDialogContext.Fail | 擲回要在聊天機器人的另一個層級攔截的例外狀況、結束狀態為「已取消」的步驟，或呼叫該步驟或對話內容的 CancelAllDialogsAsync。 | count | 小型 |  

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

| 步驟 | V3 | V4 | 發生次數 | 複雜度 | T 恤 |
| -- | -- | -- | -- | -- | -- |
若要取得連入活動 | IMessage | TurnContext.activity | count | 小型  
若要建立活動並傳送給使用者 | 呼叫 Session.send('message') | 呼叫 TurnContext.sendActivity | count | 小型 |
狀態管理 | UserState 和 ConversationState UserState.get()、UserState.saveChanges()、ConversationState.get()、ConversationState.saveChanges() | UserState 和 ConversationState (含屬性存取子) | count | 中至大 (請參閱可用的[使用者狀態管理](https://docs.microsoft.com/en-us/azure/bot-service/bot-builder-concept-state?view=azure-bot-service-4.0#state-management)) |
處理對話的開始 | 呼叫 session.beginDialog，並傳入對話方塊的識別碼 | 呼叫 DialogContext.beginDialog | count | 小型 |  
傳送活動 | 呼叫 Session.send | 呼叫 TurnContext.sendActivity | count | 小型 |  
等候使用者回應 | 從瀑布式步驟內呼叫提示，例如：builder.Prompts.text(session, 'Please enter your destination')。 在下一個步驟擷取回應。 | 傳回等候  TurnContext.prompt 開始進行提示對話方塊。 然後在瀑布的下一個步驟中擷取結果。 | count | 中 (視流程而定) |  
處理對話的接續 | 自動 | 在瀑布式對話方塊中新增其他步驟，或實作 Dialog.continueDialog | count | 大型 |  
表示在使用者的下一則訊息前處理結束 | Session.endDialog | 傳回 Dialog.EndOfTurn | count | 中 |  
開始子對話 | Session.beginDialog | 傳回等候步驟內容的 beginDialog 方法。 如果子對話方塊傳回值，該值將可透過步驟內容的結果屬性用於瀑布的下一個步驟。 | count | 中 |  
以新對話取代目前的對話 | Session.replaceDialog | ITurnContext.replaceDialog | count | 大型 |  
表示目前對話已完成的訊號 | Session.endDialog | 傳回等候步驟內容的 endDialog 方法。 | count | 中 |  
對話失敗。 | Session.pruneDialogStack | 擲回要在 Bot 的另一個層級攔截的例外狀況、結束狀態為「已取消」的步驟，或呼叫該步驟或對話內容的 cancelAllDialogs。 | count | 小型 |  

---

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

Bot Framework SDK v4 是以與 v3 相同的基礎 REST API 作為基礎。 不過，v4 是舊版 SDK 的重構，目的是為了提升聊天機器人的彈性和控制力。

建議您遷移至 .NET Core，其效能已有極大改善。 不過，某些現有的 V3 聊天機器人會使用沒有對等 .NET Core 的外部程式庫。 在此情況下，Bot Framework SDK v4 可與 .NET Framework 4.6.1 版或更高版本搭配使用。 您可以在 [corebot](https://github.com/microsoft/BotBuilder-Samples/tree/master/samples/csharp_webapi) 位置中找到範例。

將專案從 v3 遷移至 v4 時，您可以選擇下列其中一個選項：就地轉換為 **.NET Framework** 或植入到適用於 **.NET Core** 的新專案。

#### <a name="net-framework"></a>.NET Framework

- 更新和安裝 NuGet 套件
- 更新 Global.asax.cs 檔案
- 更新 MessagesController 類別
- 轉換您的對話

如需詳細資訊，請參閱[將 .NET v3 聊天機器人遷移至 .NET Framework v4 聊天機器人](conversion-framework.md)。

#### <a name="net-core"></a>.NET Core

- 使用範本建立新專案
- 視需要安裝其他 NuGet 套件
- 將聊天機器人個人化、更新 Startup.cs 檔案，然後更新控制器類別
- 更新聊天機器人類別
- 複製並更新對話方塊和模型

如需詳細資訊，請參閱[將 .NET v3 聊天機器人遷移至 .NET Core v4 聊天機器人](conversion-core.md)。

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

**Bot Framework JavaScript SDK v4** 導入了幾項針對聊天機器人撰寫方式的基本變更。 這些變更會影響以 Javascript 開發聊天機器人的語法，特別是建立聊天機器人物件、定義對話方塊及撰寫事件處理邏輯程式碼的語法。 Bot Framework SDK v4 是以與 v3 相同的基礎 REST API 作為基礎。 不過，v4 是舊版 SDK 的重構，目的是為了提升聊天機器人的彈性和控制力，尤其是：

- 對話方塊和聊天機器人執行個體已進一步分離。 在 v3 中，對話方塊已直接註冊在 Bot 建構函式中。 在 v4 中，您現在會將對話方塊當作引數來傳入 Bot 執行個體，以提供更大的撰寫彈性
- v4 提供了 `ActivityHandler` 類別，可協助自動化處理不同類型的活動，例如訊息、交談更新和事件活動
- 若要移轉 NodeJS 聊天機器人，您必須以 TypeScript 或 JavaScript 建立新的 v4 NodeJS 聊天機器人。 請使用相關移轉文件所述的新 v4 建構重新建立聊天機器人邏輯

#### <a name="migrate-from-v3-to-v4"></a>從 v3 遷移至 v4

- 建立新專案及新增相依性
- 更新進入點及定義常數
- 建立對話，並使用 SDK v4 加以重新實作
- 更新聊天機器人程式碼以執行對話方塊
- 移植 `store.js` 公用程式檔案

如需詳細資訊，請參閱[將 SDK v3 Javascript 聊天機器人遷移至 v4](conversion-javascript.md)。

---

## <a name="additional-resources"></a>其他資源

下列額外資源會提供可在移轉期間有所幫助的詳細資訊。  

### <a name="ctabcsharp"></a>[C#](#tab/csharp)

<!-- _Mini-TOC with explainer for .NET topics_ -->
下列主題說明 .NET v3 和 v4 Bot Framework SDK 之間的差異、兩個版本之間的重大變更，以及要將聊天機器人從 v3 遷移至 v4 的步驟。

| 話題 | 說明 |
| :--- | :--- |
| [v3 和 v4.NET SDK 之間的差異](migration-about.md) |v3 和 v4 SDK 之間的常見差異 |
| [.NET 移轉快速參考](net-migration-quickreference.md) |v3 和 v4 SDK 之間的重大變更 |
| [將 .NET v3 bot 遷移至 Framework v4 bot](conversion-framework.md) |使用相同的專案類型從 v3 遷移至 v4 聊天機器人 |
| [將 .NET v3 聊天機器人遷移至 Core v4 聊天機器人](conversion-core.md) | 在新的 .NET Core 專案中將聊天機器人從 v3 遷移至 v4|

### <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

<!-- _Mini-TOC with explainer for JavaScript topics_ -->
下列主題說明 JavaScript v3 和 v4 Bot Framework SDK 之間的差異、兩個版本之間的重大變更，以及要將聊天機器人從 v3 遷移至 v4 的步驟。

| 話題 | 說明 |
| :--- | :--- |
| [v3 和 v4 JavaScript SDK 之間的差異](migration-about-javascript.md) | v3 和 v4 SDK 之間的常見差異 |
| [JavaScript 移轉快速參考](javascript-migration-quickreference.md)| v3 和 v4 SDK 之間的重大變更|
| [將 JavaScript v3 bot 遷移至 v4](conversion-javascript.md) |將聊天機器人從 v3 遷移至 v4 |

---

### <a name="code-samples"></a>程式碼範例

以下程式碼範例可讓您了解 Bot Framework SDK V4 或快速啟動您的專案。

| 範例 | 說明 |
| :--- | :--- |
| [Bot Framework 從 V3 到 V4 的移轉範例](https://github.com/microsoft/BotBuilder-Samples/tree/master/MigrationV3V4) <img width="200">| 從 Bot Framework V3 SDK 到 V4 SDK 的移轉範例 |
| [Bot Builder .NET 範例](https://github.com/microsoft/BotBuilder-Samples/tree/master/samples/csharp_dotnetcore) | Bot Builder C# .NET core 範例 |
| [Bot Builder JavaScript 範例](https://github.com/microsoft/BotBuilder-Samples/tree/master/samples/javascript_nodejs) | Bot Builder JavaScript (node.js) 範例 |
| [Bot Builder 所有範例](https://github.com/microsoft/BotBuilder-Samples/tree/master/samples) | Bot Builder 所有範例 |

### <a name="getting-help"></a>取得說明

下列資源提供了用於開發聊天機器人的其他資訊和支援。

[Bot Framework 其他資源](https://docs.microsoft.com/en-us/azure/bot-service/bot-service-resources-links-help?view=azure-bot-service-4.0)

### <a name="references"></a>參考

請參閱下列資源以取得詳細資料和背景資訊。

| 話題 | 說明 |
| :--- | :--- |
| [Bot Framework 中的新功能](https://docs.microsoft.com/en-us/azure/bot-service/what-is-new?view=azure-bot-service-4.0) | Bot Framework 和 Azure Bot 服務的重要功能和改進|
|[Bot 的運作方式](../bot-builder-basics.md)|聊天機器人的內部機制|
|[管理狀態](../bot-builder-concept-state.md)|可簡化狀態管理的抽象概念|
|[對話方塊程式庫](../bot-builder-concept-dialog.md)| 用來管理對話的中心概念|
|[傳送及接收文字訊息](../bot-builder-howto-send-messages.md)|聊天機器人與使用者通訊的主要方式|
|[傳送媒體](../bot-builder-howto-add-media-attachments.md)|媒體附件，例如影像、視訊、音訊和檔案| 
|[循序對話流程](../bot-builder-dialog-manage-conversation-flow.md)| 詢問是聊天機器人與使用者互動的主要方式|
|[儲存使用者和對話資料](../bot-builder-howto-v4-state.md)|無狀態下追蹤對話|
|[複雜流程](../bot-builder-dialog-manage-complex-conversation-flow.md)|管理複雜的對話流程 |
|[重複使用對話方塊](../bot-builder-compositcontrol.md)|建立獨立對話方塊來處理特定案例|
|[中斷](../bot-builder-howto-handle-user-interrupt.md)| 處理中斷以建立強固的聊天機器人|
|[活動結構描述](https://aka.ms/botSpecs-activitySchema)|人類和自動化軟體的結構描述|
