---
title: v3 和 v4 NodeJS SDK 之間的差異 | Microsoft Docs
description: 說明 v3 和 v4 NodeJS SDK 之間的差異。
keywords: 聊天機器人移轉, 對話方塊, 狀態
author: mmiele
ms.author: v-mimiel
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 07/08/2019
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: d1fce8dfc6a8097ac14f72a9fe596560ddaac1a2
ms.sourcegitcommit: a6d02ec4738e7fc90b7108934740e9077667f3c5
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/04/2019
ms.locfileid: "70299016"
---
# <a name="differences-between-the-v3-and-v4-javascript-sdk"></a>v3 和 v4 JavaScript SDK 之間的差異

Bot Framework SDK 第 4 版支援與第 3 版相同的基礎 Bot Framework 服務。 不過，v4 是舊版 SDK 的重構，讓開發人員對於其 Bot 有更多的彈性和控制權。 SDK 中的主要變更包括：

- 聊天機器人配接器的簡介
  - 聊天機器人配接器是活動處理堆疊的一部分
  - 會處理 Bot Framework 驗證
  - 會管理通道和 Bot 回合處理常式之間的傳入和傳出流量，並封裝對 Bot Framework 連接器的呼叫
  - 會初始化每個回合的內容
  - 如需詳細資訊，請參閱[聊天機器人的運作方式](../bot-builder-basics.md)。
- 狀態管理已重構
  - 狀態資料不再於 Bot 內自動提供使用
  - 現在會透過狀態管理物件和屬性存取子來管理
  - 如需詳細資訊，請參閱[管理狀態](../bot-builder-concept-state.md)
- 新增對話程式庫
  - v3 對話必須針對新的對話程式庫重新撰寫
  - 可評分項目已不存在。 將控制權交給您的對話之前，可以檢查「全域」命令。 根據您設計 v4 Bot 的方式，這可能位在訊息處理常式或父代對話方塊中。 如需範例，請參閱如何[處理使用者中斷](../bot-builder-howto-handle-user-interrupt.md)。
  - 如需詳細資訊，請參閱[對話方塊程式庫](../bot-builder-concept-dialog.md)。

## <a name="activity-processing"></a>活動處理

當您為 Bot 建立配接器時，您也會提供訊息處理常式委派，該委派會接收來自通道和使用者的連入活動。 此配接器會針對每項接收的活動建立回合內容物件。 其會將回合內容物件傳遞給 Bot 的回合處理常式，然後在回合完成時處置物件。

回合處理常式可接收許多類型的活動。 一般情況下，您只想將「訊息」  活動轉送給 Bot 包含的任何對話。 如果您從 `ActivityHandler` 衍生 Bot，Bot 的回合處理常式會將所有訊息活動轉送至 `OnMessage`。 覆寫此方法來加入訊息處理邏輯。 如需活動類型的詳細資訊，請參閱[活動結構描述](https://github.com/Microsoft/botframework-sdk/blob/master/specs/botframework-activity/botframework-activity.md)。

### <a name="handling-turns"></a>處理回合

在處理訊息時，使用回合內容來取得連入活動的相關資訊，並將活動傳送給使用者：

|活動 |說明 |
|:---|:---|
| 若要取得連入活動 | 取得回合內容的 `Activity` 屬性。 |
| 若要建立活動並傳送給使用者 | 呼叫回合內容的 `SendActivity` 方法。<br/> 如需詳細資訊，請參閱[傳送和接收簡訊](../../rest-api/bot-framework-rest-direct-line-1-1-receive-messages.md)和[將媒體新增至訊息](../bot-builder-howto-add-media-attachments.md)。 |

`MessageFactory` 類別會提供一些協助程式方法，以供建立活動及設定其格式。

### <a name="interruptions"></a>中斷

在 Bot 的訊息迴圈中處理這些項目。 如需如何透過 v4 對話執行此作業的說明，請參閱如何[處理使用者中斷](../bot-builder-howto-handle-user-interrupt.md)。

## <a name="state-management"></a>狀態管理

在 v3 中，您可以將對話資料儲存在 Bot 狀態服務中，此服務屬於 Bot Framework 所提供的較大型服務套件。 不過，此服務已在 2018 年 3 月 31 日淘汰。 從 v4 開始，管理狀態的設計考量就像任何 Web 應用程式一樣，並且有許多選項可用。 通常，在記憶體和相同程序中進行快取是最簡單的方式；不過，針對生產應用程式，您應該更永久地儲存狀態，例如儲存在 SQL 或 NoSQL 資料庫，或是儲存為 blob。

v4 不會使用 `UserData`、`ConversationData` 及 `PrivateConversationData` 屬性和資料包來管理狀態。
如[管理狀態](../bot-builder-concept-state.md)所述，現在會透過狀態管理物件和屬性存取子管理狀態。

v4 會定義`UserState`、`ConversationState` 和 `PrivateConversationState` 類別，以便管理 Bot 的狀態資料。 您需要針對想保存的每個屬性建立狀態屬性存取子，而不只是讀取並寫入至預先定義的資料包。

### <a name="setting-up-state"></a>設定狀態

狀態應該設定於應用程式的進入點檔案，通常是 NodeJS 應用程式中的 'index.js' 或 'app.js'。 

1. 初始化一或多個物件，以實作 botbuilder-core 所提供的 `Storage` 介面。 這代表 Bot 資料的備份存放區。
    v4 SDK 會提供一些[儲存層](../bot-builder-concept-state.md#storage-layer)。
    您也可以實作自己的儲存層，以連線到不同類型的存放區。
1. 然後，視需要建立並註冊[狀態管理](../bot-builder-concept-state.md#state-management)物件。
    您具有如 v3 可用的相同範圍，而且可以視需要建立其他範圍。
1. 最後，針對聊天機器人所需的屬性建立並註冊[狀態屬性存取子](../bot-builder-concept-state.md#state-property-accessors)。
    在狀態管理物件內，每個屬性存取子都需要唯一的名稱。

### <a name="using-state"></a>使用狀態

使用狀態屬性存取子來取得及更新屬性，並使用狀態管理物件將任何變更寫入儲存體。 了解您應該將並行問題列入考量後，以下說明如何完成一些常見工作。

| Task|說明 |
|:---|:---|
| 若要建立狀態屬性存取子 | 呼叫 `BotState` 物件的 `createProperty` 方法。 <br/>`BotState` 是交談、私人交談和使用者狀態的抽象基底類別。 |
| 若要取得目前的屬性值 | 呼叫 `StatePropertyAccessor.get(TurnContext)`。<br/>如果之前尚未設定任何值，則會使用預設處理站參數來產生值。 |
| 若要將狀態變更保存至儲存體 | 在結束回合處理常式之前，針對狀態已變更的狀態管理物件，呼叫 `BotState.saveChanges(TurnContext, boolean)`。 |

### <a name="managing-concurrency"></a>管理並行存取

您的 Bot 可能需要管理狀態並行。 如需詳細資訊，請參閱[管理狀態](../bot-builder-concept-state.md#saving-state)的**儲存狀態**一節，以及**直接寫入儲存體**的[使用 eTag 管理並行](../bot-builder-howto-v4-storage.md#manage-concurrency-using-etags)一節。

## <a name="dialogs-library"></a>對話方塊程式庫

以下是對話的一些主要變更：

- 對話程式庫現在變成個別的 npm 套件：**botbuilder-dialogs**。
- 透過 `DialogState` 狀態類別和屬性存取子管理對話方塊狀態。
  - 現在會保存回合之間的對話狀態屬性，而不是對話物件本身。
- 在回合內，您會建立「對話集」  的對話內容。
  - 此對話方塊內容會封裝對話方塊堆疊。 這項資訊會保存在對話狀態屬性內。
- 這兩個版本都包含抽象的 `Dialog` 類別，不過在 v3 中，其會擴充 'ActionSet' 類別，而在 v4 則會擴充 'Object'。

### <a name="defining-dialogs"></a>定義對話

雖然所提供的 v3 能夠使用 `Dialog` 類別彈性地實作對話方塊，但這表示您必須針對功能 (例如驗證) 實作自己的程式碼。 在 v4 中，現在有提示類別可自動驗證使用者輸入、將其限制為特定類型 (例如整數)，並在使用者提供有效輸入之前，反覆提示使用者。 一般來說，這表示開發人員可撰寫較少的程式碼。

您現在有一些選項可定義對話：

|對話方塊類型| 說明 |
|:---|:---|
| 元件對話，其衍生自 `ComponentDialog` 類別 | 可讓您封裝對話程式碼，而不會與外部內容發生命名衝突。 請參閱[重複使用對話](../bot-builder-concept-dialog.md)。|
| 瀑布式對話，也就是 `WaterfallDialog` 類別的執行個體 | 其設計訴求是要搭配提示對話運作，而提示對話會提示使用者輸入並驗證各種類型的使用者輸入。 瀑布式對話會為您將大部分的程序自動化，但是對您的對話方塊程式碼強加特定形式，請參閱[循序對話流程](../bot-builder-dialog-manage-conversation-flow.md)。 |
| 自訂對話，其衍生自抽象 `Dialog` 類別 | 這讓您的對話運作方式擁有最大彈性，但您也需要深入了解對話堆疊的實作方式。 |

當您建立瀑布式對話時，您會在建構函式中定義對話方塊的步驟。 執行的步驟順序會完全遵循您宣告的方式，並且會自動地逐一前進。

您也可以使用多個對話方塊來建立複雜的控制流程；請參閱[進階對話流程](../bot-builder-dialog-manage-complex-conversation-flow.md)。

若要存取對話，您需要在「對話集」  中放入其執行個體，然後為該集合產生「對話內容」  。 當您建立對話集時，您需要提供對話狀態屬性存取子。 這可讓架構保存回合之間的對話狀態。 [管理狀態](../bot-builder-concept-state.md)說明如何在 v4 中管理狀態。

### <a name="using-dialogs"></a>使用對話

以下是 v3 的常見作業清單，以及在瀑布式對話內完成這些作業的方式。 請注意，瀑布式對話的每個步驟預計傳回 `DialogTurnResult` 值。 若未如此，瀑布可能提早結束。

| 作業 | v3 | v4 |
|:---|:---|:---|
| 處理對話的開始 | 呼叫 `session.beginDialog`，並傳入對話方塊的識別碼 | 撥電話給 `DialogContext.beginDialog` |
| 傳送活動 | 呼叫 `session.send`。 | 呼叫 `TurnContext.sendActivity`。<br/>使用步驟內容的 `Context` 屬性來取得回合內容 (`step.context.sendActivity`)。  |
| 等候使用者回應 | 從瀑布式步驟內呼叫提示字元，例如：`builder.Prompts.text(session, 'Please enter your destination')`。 在下一個步驟擷取回應。 | 傳回等候 `TurnContext.prompt` 開始提示對話。 然後在瀑布的下一個步驟中擷取結果。 |
| 處理對話的接續 | 自動 | 在瀑布式對話中新增其他步驟，或實作 `Dialog.continueDialog` |
| 表示在使用者的下一則訊息前處理結束 | 呼叫 `session.endDialog`。 | 傳回 `Dialog.EndOfTurn`。 |
| 開始子對話 | 呼叫 `session.beginDialog`。 | 傳回等候步驟內容的 `beginDialog` 方法。<br/>如果子對話傳回值，該值可透過步驟內容的 `Result` 屬性用於瀑布的下一個步驟。 |
| 以新對話取代目前的對話 | 呼叫 `session.replaceDialog`。 | 傳回等候 `ITurnContext.replaceDialog`。 |
| 表示目前對話已完成的訊號 | 呼叫 `session.endDialog`。 | 傳回等候步驟內容的 `endDialog` 方法。 |
| 對話失敗。 | 呼叫 `session.pruneDialogStack`。 | 擲回要在 Bot 的另一個層級攔截的例外狀況、結束狀態為 `Cancelled` 的步驟，或呼叫該步驟或對話內容的 `cancelAllDialogs`。 |

v4 程式碼的其他注意事項：

- v4 中的各種 `Prompt` 衍生類別會以包含兩個步驟的個別對話形式實作使用者提示。 請參閱如何 [實作循序對話流程][sequential-flow]。
- 使用 `DialogSet.createContext` 建立目前回合的對話內容。
- 在對話內，使用 `DialogContext.context` 屬性來取得目前的回合內容。
- 瀑布步驟具有 `WaterfallStepContext` 參數，其衍生自 `DialogContext`。
- 所有具體的對話和提示類別均衍生自抽象的 `Dialog` 類別。
- 您會在建立元件對話時指派識別碼。 對話集中的每個對話都需要被指派該集合內唯一的識別碼。

### <a name="passing-state-between-and-within-dialogs"></a>在對話之間和之內傳遞狀態

**Dialogs 程式庫**一文的[對話狀態](../bot-builder-concept-dialog.md#dialog-state)、[瀑布步驟內容屬性](../bot-builder-concept-dialog.md#waterfall-step-context-properties)和[使用對話](../bot-builder-concept-dialog.md#using-dialogs)章節說明如何在 v4 中管理對話狀態。

### <a name="get-user-response"></a>取得使用者回應

若要取得使用者在回合中的活動，請從回合內容中取得。

若要提示使用者並接收提示的結果：

- 將適當的提示執行個體新增到對話集。
- 從瀑布式對話中的步驟呼叫提示。
- 從下一個步驟中步驟內容的 `Result` 屬性擷取結果。

## <a name="additional-resources"></a>其他資源

請參閱下列資源以取得詳細資料和背景資訊。

| 話題 | 說明 |
| :--- | :--- |
|[將 JavaScript SDK v3 bot 遷移至 v4](https://docs.microsoft.com/en-us/azure/bot-service/migration/conversion-javascript?view=azure-bot-service-4.0)| 將 v3 JavaScript 聊天機器人植入到 v4|
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