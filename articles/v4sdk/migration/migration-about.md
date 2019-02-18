---
title: v3 和 v4 SDK 之間的差異 | Microsoft Docs
description: 說明 v3 和 v4 SDK 之間的差異。
keywords: bot 移轉, formflow, 對話, 狀態
author: JonathanFingold
ms.author: v-jofing
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 02/11/2019
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 7e5440e7d47d88b7ff6827359e7eb621bce53e3c
ms.sourcegitcommit: 7f418bed4d0d8d398f824e951ac464c7c82b8c3e
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 02/14/2019
ms.locfileid: "56240484"
---
# <a name="differences-between-the-v3-and-v4-net-sdk"></a>v3 和 v4 .NET SDK 之間的差異

Bot Framework SDK 第 4 版支援與第 3 版相同的基礎 Bot Framework 服務。 不過，v4 是舊版 SDK 的重構，讓開發人員對於其 Bot 有更多的彈性和控制權。 SDK 中的主要變更包括：

- Bot 配接器的簡介。 此配接器是活動處理堆疊的一部分。
  - 此配接器會處理 Bot Framework 驗證。
  - 此配接器會管理通道和 Bot 回合處理常式之間的傳入和傳出流量，並封裝對 Bot Framework 連接器的呼叫。
  - 此配接器會初始化每個回合的內容。
  - 如需詳細資料，請參閱 [Bot 運作方式](../bot-builder-basics.md)。
- 重構的狀態管理。
  - 狀態資料不再於 Bot 內自動提供使用。
  - 現在會透過狀態管理物件和屬性存取子管理狀態。
  - 如需詳細資料，請參閱[管理狀態](../bot-builder-concept-state.md)。
- 新的 Dialogs 程式庫。
  - v3 對話需要針對新的對話程式庫重新撰寫。
  - 可評分項目已不存在。 將控制權交給您的對話之前，可以檢查回合處理常式中的「全域」命令。
  - 如需詳細資料，請參閱 [Dialogs 程式庫](../bot-builder-concept-dialog.md)。
- 支援 ASP.NET Core。
  - 建立新 C# Bot 的範本是以 ASP.NET Core 架構為目標。
  - 您仍然可將 ASP.NET 用於 Bot，但 v4 的重點在於支援 ASP.NET Core 架構。
  - 如需此架構的詳細資訊，請參閱 [ASP.NET Core 簡介](https://docs.microsoft.com/aspnet/core/)。

## <a name="activity-processing"></a>活動處理

當您針對 Bot 建立配接器時，您也會提供回合處理常式委派，該委派會接收來自通道和使用者的連入活動。 此配接器會針對每項接收的活動建立回合內容物件。 其會將回合內容物件傳遞給回合處理常式，然後在回合完成時處置物件。

回合處理常式可接收許多類型的活動。 一般情況下，您只想將「訊息」活動轉送給 Bot 包含的任何對話。 如需活動類型的詳細資訊，請參閱[活動結構描述](https://aka.ms/botSpecs-activitySchema)。

### <a name="handling-turns"></a>處理回合

您的回合處理常式必須符合 `BotCallbackHandler` 的簽章：

```csharp
public delegate Task BotCallbackHandler(
    ITurnContext turnContext,
    CancellationToken cancellationToken);
```

在處理回合時，使用回合內容來取得連入活動的相關資訊，並將活動傳送給使用者：

| | |
|-|-|
| 若要取得連入活動 | 取得回合內容的 `Activity` 屬性。 |
| 若要建立活動並傳送給使用者 | 呼叫回合內容的 `SendActivityAsync` 方法。<br/>如需詳細資訊，請參閱[傳送和接收簡訊](../bot-builder-howto-send-messages.md)和[將媒體新增至訊息](../bot-builder-howto-add-media-attachments.md) |

`MessageFactory` 類別會提供一些協助程式方法，以供建立活動及設定其格式。

### <a name="scorables-is-gone"></a>可評分項目已不存在

在 Bot 的訊息迴圈中處理這些項目。 如需如何透過 v4 對話執行此作業的說明，請參閱如何[處理使用者中斷](../bot-builder-howto-handle-user-interrupt.md)。

可組合的可評分分派樹狀目錄與可組合的鏈結對話 (例如_預設例外狀況_) 也都會消失。 重新產生這項功能的方法之一，就是在 Bot 的回合處理常式內實作。

## <a name="state-management"></a>狀態管理

v4 不會使用 `UserData`、`ConversationData` 及 `PrivateConversationData` 屬性和資料包來管理狀態。
如[管理狀態](../bot-builder-concept-state.md)所述，現在會透過狀態管理物件和屬性存取子管理狀態。

v4 會定義`UserState`、`ConversationState` 和 `PrivateConversationState` 類別，以便管理 Bot 的狀態資料。 您需要針對想保存的每個屬性建立狀態屬性存取子，而不只是讀取並寫入至預先定義的資料包。

### <a name="setting-up-state"></a>設定狀態

可能的話，在適用於 .NET Core 的 **Startup.cs** 或在適用於 .NET Framework 的 **Global.asax.cs** 中，應該將狀態設定為單例。

1. 初始化一或多個 `IStorage` 物件。 這代表 Bot 資料的備份存放區。
    v4 SDK 會提供一些[儲存層](../bot-builder-concept-state.md#storage-layer)。
    您也可以實作自己的儲存層，以連線到不同類型的存放區。
1. 然後，視需要建立並註冊[狀態管理](../bot-builder-concept-state.md#state-management)物件。
    您具有如 v3 可用的相同範圍，而且可以視需要建立其他範圍。
1. 然後，針對 Bot 所需的屬性建立並註冊[狀態屬性存取子](../bot-builder-concept-state.md#state-property-accessors)。
    在狀態管理物件內，每個屬性存取子都需要唯一的名稱。

### <a name="using-state"></a>使用狀態

每次建立 Bot 時，您可以使用相依性插入來存取這些狀態
(在 ASP.NET 中，系統會為每個回合建立 Bot 或訊息控制器的新執行個體)。使用狀態屬性存取子來取得及更新屬性，並使用狀態管理物件將任何變更寫入儲存體。 了解您應該將並行問題列入考量後，以下說明如何完成一些常見工作。

| | |
|-|-|
| 若要建立狀態屬性存取子 | 呼叫 `BotState.CreateProperty<T>`。<br/>`BotState` 是交談、私人交談和使用者狀態的抽象基底類別。 |
| 若要取得目前的屬性值 | 呼叫 `IStatePropertyAccessor<T>.GetAsync`。<br/>如果之前尚未設定任何值，則會使用預設處理站參數來產生值。 |
| 若要更新目前快取的屬性值 | 呼叫 `IStatePropertyAccessor<T>.SetAsync`。<br/>這只會更新快取，而不會更新支援儲存層。 |
| 若要將狀態變更保存至儲存體 | 在結束回合處理常式之前，針對狀態已變更的狀態管理物件，呼叫 `BotState.SaveChangesAsync`。 |

如需詳細資訊，請參閱[儲存狀態](../bot-builder-concept-state.md#saving-state)。

### <a name="managing-concurrency"></a>管理並行存取

您的 Bot 可能需要管理狀態並行。 如需詳細資訊，請參閱[管理狀態](../bot-builder-concept-state.md#saving-state)的**儲存狀態**一節，以及**直接寫入儲存體**的[使用 eTag 管理並行](../bot-builder-howto-v4-storage.md#manage-concurrency-using-etags)一節。

## <a name="dialogs-library"></a>對話方塊程式庫

以下是對話的一些主要變更：

- Dialogs 程式庫現在是個別的 NuGet 套件：**Microsoft.Bot.Builder.Dialogs**。
- 對話類別不再必須是可序列化。 透過 `DialogState` 狀態屬性存取子管理對話狀態。
  - 現在會保存回合之間的對話狀態屬性，而不是對話物件本身。
- `IDialogContext` 介面會由 `DialogContext` 類別所取代。 在回合內，您會建立「對話集」的對話內容。
  - 此對話內容會封裝對話堆疊 (舊的堆疊框架)。 這項資訊會保存在對話狀態屬性內。
- `IDialog` 介面會由抽象 `Dialog` 類別所取代。

### <a name="defining-dialogs"></a>定義對話

您現在有一些選項可定義對話：

- 瀑布式對話，也就是 `WaterfallDialog` 類別的執行個體。

  其設計訴求是要搭配提示對話運作，而提示對話會提示使用者輸入並驗證各種類型的使用者輸入。 請參閱[提示輸入](../bot-builder-prompts.md)。

  這會為您將大部分的程序自動化，但是對您的對話程式碼強加特定形式，請參閱[循序交談流程](../bot-builder-dialog-manage-conversation-flow.md)。 不過，您可藉由將多個對話新增至對話集來建立其他控制流程，請參閱[進階交談流程](../bot-builder-dialog-manage-complex-conversation-flow.md)。

- 元件對話，其衍生自 `ComponentDialog` 類別。

  這可讓您封裝對話程式碼，而不會與外部內容發生命名衝突。 請參閱[重複使用對話](../bot-builder-compositcontrol.md)。

- 自訂對話，其衍生自抽象 `Dialog` 類別。

  這讓您的對話運作方式擁有最大彈性，但您也需要深入了解對話堆疊的實作方式。

若要存取對話，您需要在「對話集」中放入其執行個體，然後為該集合產生「對話內容」。

當您建立對話集時，您需要提供對話狀態屬性存取子。 這可讓架構保存回合之間的對話狀態。 [管理狀態](../bot-builder-concept-state.md)說明如何在 v4 中管理狀態。

### <a name="using-dialogs"></a>使用對話

以下是 v3 的常見作業清單，以及在瀑布式對話內完成這些作業的方式。 請注意，瀑布式對話的每個步驟預計傳回 `DialogTurnResult` 值。 若未如此，瀑布可能提早結束。

| 作業 | v3 | v4 |
|:---|:---|:---|
| 處理對話的開始 | 實作 `IDialog.StartAsync` | 讓此項成為瀑布式對話的第一個步驟。 |
| 傳送活動 | 呼叫 `IDialogContext.PostAsync`。 | 呼叫 `ITurnContext.SendActivityAsync`。<br/>使用步驟內容的 `Context` 屬性來取得回合內容。  |
| 等候使用者回應 | 使用 `IAwaitable<IMessageActivity>` 參數並呼叫 `IDialogContext.Wait`。 | 傳回等候 `ITurnContext.PromptAsync` 開始提示對話。 然後在瀑布的下一個步驟中擷取結果。 |
| 處理對話的接續 | 呼叫 `IDialogContext.Wait`。 | 在瀑布式對話中新增其他步驟，或實作 `Dialog.ContinueDialogAsync` |
| 表示在使用者的下一則訊息前處理結束 | 呼叫 `IDialogContext.Wait`。 | 傳回 `Dialog.EndOfTurn`。 |
| 開始子對話 | 呼叫 `IDialogContext.Call`。 | 傳回等候步驟內容的 `BeginDialogAsync` 方法。<br/>如果子對話傳回值，該值可透過步驟內容的 `Result` 屬性用於瀑布的下一個步驟。 |
| 以新對話取代目前的對話 | 呼叫 `IDialogContext.Forward`。 | 傳回等候 `ITurnContext.ReplaceDialogAsync`。 |
| 表示目前對話已完成的訊號 | 呼叫 `IDialogContext.Done`。 | 傳回等候步驟內容的 `EndDialogAsync` 方法。 |
| 對話失敗。 | 呼叫 `IDialogContext.Fail`。 | 擲回要在 Bot 的另一個層級攔截的例外狀況、結束狀態為 `Cancelled` 的步驟，或呼叫該步驟或對話內容的 `CancelAllDialogsAsync`。<br/>請注意，在 v4 中對話內的例外狀況會隨著 C# 堆疊傳播，而不是隨著對話堆疊傳播。 |

v4 程式碼的其他注意事項：

- v4 中的各種 `Prompt` 衍生類別會以包含兩個步驟的個別對話形式實作使用者提示。 請參閱如何[使用對話提示收集使用者輸入](../bot-builder-prompts.md)。
- 使用 `DialogSet.CreateContextAsync` 建立目前回合的對話內容。
- 在對話內，使用 `DialogContext.Context` 屬性來取得目前的回合內容。
- 瀑布步驟具有 `WaterfallStepContext` 參數，其衍生自 `DialogContext`。
- 所有具體的對話和提示類別均衍生自抽象的 `Dialog` 類別。
- 您會在建立元件對話時指派識別碼。 對話集中的每個對話都需要被指派該集合內唯一的識別碼。

### <a name="passing-state-between-and-within-dialogs"></a>在對話之間和之內傳遞狀態

**Dialogs 程式庫**一文的[對話狀態](../bot-builder-concept-dialog.md#dialog-state)、[瀑布步驟內容屬性](../bot-builder-concept-dialog.md#waterfall-step-context-properties)和[使用對話](../bot-builder-concept-dialog.md#using-dialogs)章節說明如何在 v4 中管理對話狀態。

### <a name="iawaitable-is-gone"></a>IAwaitable 已不存在

若要取得使用者在回合中的活動，請從回合內容中取得。

若要提示使用者並接收提示的結果：

- 將適當的提示執行個體新增到對話集。
- 從瀑布式對話中的步驟呼叫提示。
- 從下一個步驟中步驟內容的 `Result` 屬性擷取結果。

### <a name="formflow"></a>Formflow

在 V3，Formflow 是 C# SDK 的一部分，但不屬於 JavaScript SDK。 其不是 v4 SDK 的一部分，但有適用於 C# 的社群版本。

| NuGet 套件名稱 | 社群 GitHub 存放庫 |
|-|-|
| Bot.Builder.Community.Dialogs.Formflow | [BotBuilderCommunity/botbuilder-community-dotnet/libraries/Bot.Builder.Community.Dialogs.FormFlow](https://github.com/BotBuilderCommunity/botbuilder-community-dotnet/tree/master/libraries/Bot.Builder.Community.Dialogs.FormFlow) |

## <a name="additional-resources"></a>其他資源

- [將 .NET SDK v3 bot 遷移至 v4](conversion-framework.md)