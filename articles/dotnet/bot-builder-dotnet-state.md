---
title: 管理狀態資料 | Microsoft Docs
description: 了解如何透過適用於 .NET 的 Bot Framework SDK 儲存及擷取資料。
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: 9dc2519e74147d1147e2d5f2cb7fba883bd1269c
ms.sourcegitcommit: a6d02ec4738e7fc90b7108934740e9077667f3c5
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/04/2019
ms.locfileid: "70298794"
---
# <a name="manage-state-data"></a>管理狀態資料

[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]

> [!div class="op_single_selector"]
> - [.NET](../dotnet/bot-builder-dotnet-state.md)
> - [Node.js](../nodejs/bot-builder-nodejs-state.md)

[!INCLUDE [State concept overview](../includes/snippet-dotnet-concept-state.md)]

## <a name="in-memory-data-storage"></a>記憶體內部的資料儲存體

記憶體內部的資料儲存體僅供測試之用。 這是可變更的臨時儲存體。 每次 Bot 重新啟動時，就會清除資料。 若要將記憶體內部儲存體運用於測試用途，請務必： 

安裝下列 NuGet 封裝： 
- Microsoft.Bot.Builder.Azure
- Autofac.WebApi2

在 **Application_Start** 方法中，建立新記憶體內部儲存體的執行個體，並註冊新的資料存放區：

```cs
// Global.asax file

var store = new InMemoryDataStore();

Conversation.UpdateContainer(
           builder =>
           {
               builder.Register(c => store)
                         .Keyed<IBotDataStore<BotData>>(AzureModule.Key_DataStore)
                         .AsSelf()
                         .SingleInstance();

               builder.Register(c => new CachingBotDataStore(store,
                          CachingBotDataStoreConsistencyPolicy
                          .ETagBasedConsistency))
                          .As<IBotDataStore<BotData>>()
                          .AsSelf()
                          .InstancePerLifetimeScope();


           });
GlobalConfiguration.Configure(WebApiConfig.Register);

```

您可以使用這個方法來設定自己的自訂資料存放區，或使用任一項 *Azure 擴充功能*。

## <a name="manage-custom-data-storage"></a>管理自訂資料儲存體

在生產環境中基於效能和安全性理由，您可能要實作自己的資料儲存體，或考量實作下列其中一個資料儲存體選項：

1. [使用 Cosmos DB 管理狀態資料](bot-builder-dotnet-state-azure-cosmosdb.md)

2. [藉由資料表儲存體管理狀態資料](bot-builder-dotnet-state-azure-table-storage.md)

透過其中任一 [Azure 擴充功能](https://www.nuget.org/packages/Microsoft.Bot.Builder.Azure/)選項，.Bot Framework SDK for .NET 的資料設定和留存機制會與記憶體內部資料存放區的機制相同。

## <a name="bot-state-methods"></a>Bot 狀態方法

下表列出可用於管理狀態資料的方法。

| 方法 | 目標範圍 | 目標 |                                                
|----|----|----|
| `GetUserData` | 使用者 | 取得之前儲存供指定通道使用者使用的狀態資料 |
| `GetConversationData` | 交談 | 取得之前針對指定通道交談儲存的狀態資料 |
| `GetPrivateConversationData` | 使用者和交談 | 取得之前儲存供指定通道交談內使用者使用的狀態資料 |
| `SetUserData` | 使用者 | 針對指定通道的使用者儲存的狀態資料 |
| `SetConversationData` | 交談 | 針對指定通道的交談儲存的狀態資料。 <br/><br/>**注意**：由於 `DeleteStateForUser` 方法並不會刪除已使用`SetConversationData` 方法所儲存的資料，因此您「切勿」使用這個方法來儲存使用者個人識別資訊 (PII)。 |
| `SetPrivateConversationData` | 使用者和交談 | 針對指定通道的交談中之使用者儲存的狀態資料 |
| `DeleteStateForUser` | 使用者 | 刪除之前已使用 `SetUserData` 方法或 `SetPrivateConversationData` 方法所儲存的使用者狀態資料。 <br/><br/>**注意**：Bot 接收 [deleteUserData](bot-builder-dotnet-activities.md#deleteuserdata) 或 [contactRelationUpdate](bot-builder-dotnet-activities.md#contactrelationupdate) 類型的活動，指出 Bot 已從使用者的連絡人清單中遭到移除時，Bot 就應呼叫這個方法。 |

如果您的 Bot 使用 "**Set...Data**"　方法來儲存狀態資料，日後 Bot 在相同環境中收到的訊息就會包含該資料，這些資料可以使用相應的 "**Get...Data**" 方法來存取。

## <a name="useful-properties-for-managing-state-data"></a>適用於管理狀態資料的實用屬性

每個[活動][Activity]物件都包含您要用來管理狀態資料的屬性。

| 屬性 | 說明 | 使用案例 |
|----|----|----|
| `From` | 可唯一識別通道上的使用者 | 儲存和擷取與使用者相關聯的狀態資料 |
| `Conversation` | 可唯一識別交談 | 儲存和擷取與交談相關聯的狀態資料 |
| `From`和`Conversation` | 可唯一識別使用者和交談 | 儲存和擷取與特定交談內容中與特定使用者相關聯的狀態資料 |

> [!NOTE]
> 即便您選擇將狀態資料儲存在自己的資料庫，而非選擇使用 Bot Framework 狀態資料存放區，您也可以使用這些屬性值做為索引鍵。

## <a name="handle-concurrency-issues"></a>處理並行問題

如果 Bot 的其他執行個體變更了資料，當 Bot 嘗試儲存狀態資料時，可能會收到以下的 HTTP 狀態碼錯誤回應：**412 前置條件失敗**。 您可以設計自己的 Bot 來說明這類情形，如下列程式碼範例所示。

[!code-csharp[Handle exception saving state](../includes/code/dotnet-state.cs#handleException)]

## <a name="additional-resources"></a>其他資源

- [Bot Framework 疑難排解指南](../bot-service-troubleshoot-general-problems.md)
- <a href="/dotnet/api/?view=botbuilder-3.11.0" target="_blank">適用於 .NET 的 Bot Framework SDK 參考</a>

[Activity]: https://docs.botframework.com/csharp/builder/sdkreference/dc/d2f/class_microsoft_1_1_bot_1_1_connector_1_1_activity.html
