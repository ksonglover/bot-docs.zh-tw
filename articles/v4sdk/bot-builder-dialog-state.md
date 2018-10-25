---
title: 對話方塊狀態 | Microsoft Docs
description: 說明 Bot Builder SDK 內的狀態運作方式。
keywords: 狀態, bot 狀態, 對話狀態, 使用者狀態
author: johnataylor
ms.author: johtaylo
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 9/21/2018
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 4da715650eb1c3e7b284aaa47aa5ce4ac7280f4c
ms.sourcegitcommit: b78fe3d8dd604c4f7233740658a229e85b8535dd
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/24/2018
ms.locfileid: "49998755"
---
# <a name="dialog-state"></a>對話方塊狀態

對話方塊是一種實作多回合對話邏輯的方法，因此，這是 SDK 中依賴各種保存狀態的功能範例。 

對話方塊式 Bot 通常會將 DialogSet 集合保留為其 Bot 實作中的成員變數。 DialogSet 會透過「存取子」物件的控點來建立。 

存取子是在 SDK 狀態模型的中心概念。 存取子會實作 IStatePropertyAccessor 介面，這表示其必須提供 Get、Set 和 Delete 方法的實作。 若要建立存取子，您須為其提供屬性名稱。 

使用存取子的程式碼可呼叫其 Get、Set 和 Delete 方法，並知道這些方法會指向具有該名稱的屬性。 當較高層的元件需要某些狀態持續存在時，此元件就該使用存取子。 此方式可讓不同案例帶入自己的狀態儲存實作，但仍使用相同的較高層程式碼。 例如，對話方塊會使用存取子，而這是其存取保存狀態的唯一方式。

![對話方塊狀態](media/bot-builder-dialog-state.png)

當 Bot 的 OnTurn 受到呼叫時，Bot 會藉由呼叫 DialogSet 上的 CreateContext 來初始化對話方塊子系統。 DialogContext 的建立需要狀態，因此 DialogSet 會使用其存取子來「取得」適當的對話方塊狀態 JSON。 SDK 會以 BotState 類別的形式提供存取子實作。 需要利用狀態實作的應用程式可以建立 BotState 子類別，以繼承存取子的實作。 SDK 中包含兩個 BotState 子類別：

- UserState
- ConversationState

UserState 和 ConversationState 會在基礎儲存體上使用金鑰。 金鑰會向下傳遞至實體儲存體。 從邏輯上來說，對於根據存取子命名的屬性而言，金鑰是個命名空間。 在內部，BotState 實作會使用 TurnContext 上的輸入活動，以動態方式產生所使用的儲存體金鑰。

- UserState 會使用頻道識別碼 (Channel Id) 和傳送者識別碼 (From Id) 來建立金鑰。例如：_{Activity.ChannelId}/conversations/{Activity.From.Id}#DialogState_
- ConversationState 會使用頻道識別碼和對話識別碼 (Conversation Id) 來建立金鑰。例如：_{Activity.ChannelId}/users/{Activity.Conversation.Id}#YourPropertyName_

應用程式必須提供存取子，而適當儲存體金鑰 (以動態方式建立) 的繫結和屬性名稱都會在幕後發生。 存取子的 BotState 實作包含一些最佳化： 

- 第一個最佳化是快取和消極式載入。 實際儲存體實作上的載入，會延遲到存取子的第一個 Get 受到呼叫後再進行，而載入結果會保留在快取中。 此快取的參考會使用相對應 BotState 所提供的金鑰新增至 TurnContext。 因此對應到 UserState 的快取將會保留在名為 "UserState" 的欄位，而對應到 ConversationState 的快取會保留在名為 “ConversationState” 的欄位。 各種對「存取子」物件發出的呼叫都會收到 TurnContext，因此可以繼續並擷取適當的快取。

- 第二個最佳化是 BotState 類別包含邏輯，可判斷狀態上是否已進行任何變更。 只有在變更已執行時，BotState 類別才會在基礎儲存體上執行實際的儲存作業。

- BotState 實作中的第三個最佳化是位在稱為 BotStateSet 的類別中。 BotStateSet 是 BotState 物件的集合，此集合會將其 SaveChanges 方法上的呼叫平行委派給每個集合成員。

很明顯地，BotState 上的 SaveChanges 呼叫不是 IStatePropertyAccessor 介面的一部分。 這是因為 SaveChanges 是 BotState 實作的特定最佳化，而不是模型的核心層面。 具體來說，對話方塊之類的程式碼對 BotState 一無所知，更別說是 SaveChanges 了。 事實上，對話方塊的程式碼只會與存取子結合。 這麼做的目的是，SaveChanges 方法應在對話方塊系統執行完成後，於對話方塊系統的外部進行呼叫。 例如，如下圖所示，該方法可以從 Bot 的 OnTurn 方法內進行呼叫。

請務必注意，BotState 實作會帶來一些具體的語義。 值得注意的是，其支援「以最後寫入者為準」的行為，因此最後寫入的狀態將蓋過先前寫入的狀態。 這對許多應用程式來說可能還可以正常運作，但會有些顧慮，特別是在向外延展的案例中，其中可能正在進行某程度的並行作業。 如果這是個問題，那麼解答就是實作您自己的存取子，並將其傳遞至對話方塊。 還有許多其他替代方法，例如，解決方案可能會選擇利用 eTag 條件，這在雲端儲存體服務 (例如 Azure 儲存體) 上相當熱門。 在此情況下，解決方案可能會實作其他重要部分。 比方說，解決方案可能會緩衝處理輸出活動，並只遵循成功的儲存作業來傳送這些活動。 要注意的重點是，此行為並不是 BotState 實作的行為，而是應用程式會在存取子層級提供或插入的項目。

BotState 實作本身具有插入式儲存體模型。 這會遵循簡單的載入/儲存模式，而且 SDK 會提供兩個用於生產環境的替代實作。 一個適用於 Azure 儲存體，另一個則適用於 CosmosDB，而記憶體中的實作僅用於測試目的。 不過，這裡要注意的重點是，「以最後寫入者為準」的語意會由 BotState 實作決定。

## <a name="handling-state-in-middleware"></a>處理中介軟體中的狀態
如上圖所示，SaveAllChanges 的呼叫會在 Bot 的 OnTurn 結束時進行。 以下是著重於該呼叫的相同圖表。

![狀態中介軟體問題](media/bot-builder-dialog-state-problem.png)

此方法的問題是，若是在 Bot 的 OnTurn 方法傳回之後，從某些自訂中介軟體更新任何狀態，則狀態不會儲存到永久性儲存體。 解決方案是藉由將 AutoSaveChangesMiddleware 新增至中介軟體的堆疊結尾，以將 SaveAllChanges 的呼叫移到完成自訂中介軟體之後。 執行如下所示。

![狀態中介軟體解決方案](media/bot-builder-dialog-state-solution.png)

## <a name="additional-resources"></a>其他資源
如需詳細資訊，請參閱 GitHub 上的 Bot Builder SDK [[C#](https://github.com/Microsoft/BotBuilder-dotnet) | [JavaScript](https://github.com/Microsoft/BotBuilder-js)]。
