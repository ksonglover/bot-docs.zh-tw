---
title: 管理狀態 | Microsoft Docs
description: 說明 Bot Builder SDK 內的狀態運作方式。
keywords: 狀態, bot 狀態, 對話狀態, 使用者狀態
author: ivorb
ms.author: v-ivorb
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 11/15/2018
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 366a985e839c8a79fcd8794c139e2e8130a05335
ms.sourcegitcommit: 6cb37f43947273a58b2b7624579852b72b0e13ea
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 11/22/2018
ms.locfileid: "52288868"
---
# <a name="managing-state"></a>管理狀態

Bot 內的狀態會遵循與新式 Web 應用程式相同的架構，而 Bot Framework SDK 會提供一些抽象概念，讓狀態管理變得更容易。

如同 Web 應用程式，Bot 原本是無狀態的；不同的 Bot 執行個體可處理任何指定的交談回合。 有些 Bot 偏好此種隱含性 - Bot 不需要額外資訊即可運作，或所需的資訊保證在內送訊息中。 而其他 Bot 則需要狀態 (例如，我們在交談中的位置或先前收到的使用者相關資料) 才能進行有用的交談。

**為什麼需要狀態？**

維護狀態可讓 Bot 藉由記住有關使用者或交談的特定事項，以便進行更有意義的交談。 比方說，如果您之前已跟使用者談話，即可儲存其先前的相關資訊，您就不需要再次詢問。 狀態也可讓資料保存超過目前的回合，以便 Bot 保存多回合交談過程中的資訊。

由於屬於 Bot，所以使用狀態有幾個層次，我們將在此討論：儲存層、狀態管理和狀態屬性存取子。

## <a name="storage-layer"></a>儲存層

從後端說起，這是狀態資訊的實際儲存位置，也就是我們的「儲存層」。 這可視為實體儲存體，例如記憶體內部、Azure 或第三方伺服器。

Bot Framework SDK 提供儲存層的實作，例如記憶體內部儲存體 (可供在本機測試) 和 Azure 儲存體或 CosmosDB (可供測試和部署至雲端)。

## <a name="state-management"></a>狀態管理

「狀態管理」可自動讀取 Bot 的狀態以及將其寫入至基礎儲存層。 狀態會儲存為「狀態屬性」，這是 Bot 無須擔心特定基礎實作，即可透過狀態管理物件讀取和寫入的有效索引鍵 / 值組。 這些狀態屬性會定義該資訊的儲存方式。 例如，當您擷取定義為特定類別或物件的屬性時，您知道該資料將如何結構化。

這些狀態屬性會歸併為限域「貯體」，而這只是協助組織這些屬性的集合。 SDK 包含三個「貯體」：

- 使用者狀態
- 交談狀態
- 私人交談狀態

上述所有貯體都是「Bot 狀態」類別的子類別，可加以衍生來定義其他類型的貯體。

視貯體而定，這些預先定義的貯體受限於特定可見性：

- 不論交談內容為何，使用者狀態適用於 Bot 在該頻道上與該使用者正在交談的任何回合
- 不論使用者為何 (亦即群組交談)，交談狀態適用於特定交談中的任何回合
- 私人交談狀態受限於特定交談和該特定使用者

每個預先定義的貯體所用的索引鍵為使用者和交談所特有。 設定狀態屬性的值時，系統會使用回合內容內含的資訊在內部為您定義索引鍵，以確保每個使用者或交談取均位於正確的貯體和屬性。 具體而言，索引鍵的定義如下所示：

- 使用者狀態會使用「頻道識別碼」和和「傳送者識別碼」來建立索引鍵。 例如，_{Activity.ChannelId}/users/{Activity.From.Id}#YourPropertyName_
- 交談狀態會使用「頻道識別碼」和「交談識別碼」來建立索引鍵。 例如，_{Activity.ChannelId}/conversations/{Activity.Conversation.Id}#YourPropertyName_
- 私人交談狀態會使用「頻道識別碼」、「傳送者識別碼」和「交談識別碼」來建立索引鍵。 例如，_{Activity.ChannelId}/conversations/{Activity.Conversation.Id}/users/{Activity.From.Id}#YourPropertyName_

如需使用這些預先定義貯體的詳細資訊，請參閱[狀態操作說明文章](bot-builder-howto-v4-state.md)。

## <a name="state-property-accessors"></a>狀態屬性存取子

「狀態屬性存取子」用來實際讀取或寫入其中一個狀態屬性，並提供 *get*、*set* 和 *delete* 方法，以便存取一個回合內的狀態屬性。 若要建立存取子，您必須提供屬性名稱，這通常發生於您初始化 Bot 時。 然後，您可以使用該存取子來取得及操作 Bot 狀態的該屬性。

存取子允許 SDK 從基礎儲存體中取得狀態，並且為您更新 Bot 的「狀態快取」。 狀態快取是由 Bot 維護的本機快取，可為您儲存狀態物件，並允許不需存取基礎儲存體的讀取和寫入作業。 如果狀態尚未位於快取中，則呼叫存取子的 *get* 方法可擷取狀態並其放入快取中。 擷取之後，即可如同本機變數一樣操作狀態屬性。

存取子的 *delete* 方法可從快取中移除屬性，也可從基礎儲存體中刪除屬性。

> [!IMPORTANT]
> 第一次呼叫存取子的 *get* 方法時，您必須提供 Factory 方法來建立物件 (如果其尚未存在於您的狀態中)。 如果未提供任何 Factory 方法，則會發生例外狀況。 在[狀態操作說明文章](bot-builder-howto-v4-state.md)中可找到有關如何使用 Factory 方法的詳細資訊。

對於經由存取子所取得的狀態屬性，若要保存您對其所做的任何變更，就必須更新狀態快取中的屬性。 您可以透過呼叫存取子的 *set* 方法，該方法會在快取中設定您的屬性值，而且適用於稍後在該回合中讀取或更新屬性。 若要將該資料實際保存到基礎儲存體 (因而在目前回合之後提供使用)，您必須接著[儲存狀態](#saving-state)。

### <a name="how-the-state-property-accessor-methods-work"></a>狀態屬性存取子方法的運作方式

存取子方法是 Bot 與狀態互動的主要方式。 每個存取子方法的運作方式，以及基礎層的互動方式，如下所示：

- 存取子的 *get*
    - 存取子會要求狀態快取中的屬性
    - 如果屬性位於快取中，請將其傳回。 否則，從狀態管理物件中取得。
        - 如果屬相尚未存在於狀態中，請使用存取子的 *get* 呼叫中所提供的 Factory 方法。
- 存取子的 *set*
    - 使用新的屬性值更新狀態快取。
- 狀態管理物件的 *save changes*
    - 檢查狀態快取中的屬性變更。
    - 將該屬性寫入至儲存體。

## <a name="saving-state"></a>儲存狀態

當您呼叫存取子的 set 方法來記錄已更新的狀態時，該狀態屬性尚未儲存到保存的儲存體，而改為只儲存到 Bot 的狀態快取。 若要將狀態快取中的所有變更儲存到保存狀態，您必須呼叫狀態管理物件的「儲存變更」方法，該方法適用於上述 Bot 狀態類別 (例如使用者狀態或交談狀態) 的實作。

針對狀態管理物件 (也就是上述的貯體) 呼叫「儲存變更」方法，就會在狀態快取中儲存您為該貯體設定的所有屬性，但不適合存在於 Bot 狀態中的任何其他貯體。

> [!TIP]
> Bot 狀態會實作「以最後寫入者為準」行為，因此最後寫入的狀態將蓋過先前寫入的狀態。 這對許多應用程式來說可能還可以運作，但會有些顧慮，特別是在向外延展的案例中，其中可能正在進行某種程度的並行或延遲作業。

如果您有一些自訂中介軟體，可能在回合處理常式完成後更新狀態，請考慮[在中介軟體中處理狀態](bot-builder-concept-middleware.md#handling-state-in-middleware)。

## <a name="additional-resources"></a>其他資源

- [對話狀態](bot-builder-concept-dialog.md#dialog-state)
- [直接寫入儲存體](bot-builder-howto-v4-storage.md)
- [儲存交談和使用者資料](bot-builder-howto-v4-state.md)