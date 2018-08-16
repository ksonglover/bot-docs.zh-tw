---
title: 管理狀態資料 |Microsoft Docs
description: 了解如何藉由 Bot Builder SDK for .NET 儲存及擷取資料。
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 12/13/17
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: b218ae4ffd2ffbfe9144b4143f2600be15d688dd
ms.sourcegitcommit: f576981342fb3361216675815714e24281e20ddf
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/18/2018
ms.locfileid: "39299146"
---
# <a name="manage-state-data"></a>管理狀態資料
> [!div class="op_single_selector"]
> - [.NET](../dotnet/bot-builder-dotnet-state.md)
> - [Node.js](../nodejs/bot-builder-nodejs-state.md)

[!INCLUDE [State concept overview](../includes/snippet-dotnet-concept-state.md)]

## <a name="in-memory-data-storage"></a>記憶體中的資料存放區

記憶體中的資料存放區僅供測試之用。 這是可變更的臨時儲存體。 每次 Bot 重新啟動時，就會清除資料。 若要將記憶體內部儲存體運用於測試用途，請務必： 

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

## <a name="manage-custom-data-storage"></a>管理自訂資料存放區

在生產環境中基於效能和安全性理由，您可能必須執行自己的資料存放區，或下列其中一個資料存放區選項：

1. [藉由 Cosmos DB 管理狀態資料](bot-builder-dotnet-state-azure-cosmosdb.md)

2. [藉由資料表儲存體管理狀態資料](bot-builder-dotnet-state-azure-table-storage.md)

透過其中任一 [Azure 擴充功能](https://www.nuget.org/packages/Microsoft.Bot.Builder.Azure/)選項，.Bot Framework SDK for .NET 的資料設定和留存機制會與記憶體內部資料存放區的機制相同。

## <a name="bot-state-methods"></a>Bot 狀態方法

下表列出可用於管理狀態資料的方法。

| 方法 | 範圍設定為 | 目標 |                                                
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

每個[活動][ Activity]物件都包含您要用來管理狀態資料的屬性。

| 屬性 | 說明 | 使用案例 |
|----|----|----|
| `From` | 可唯一識別通道上的使用者 | 儲存和擷取與使用者相關聯的狀態資料 |
| `Conversation` | 可唯一識別交談 | 儲存和擷取與交談相關聯的狀態資料 |
| `From`和`Conversation` | 可唯一識別使用者和交談 | 儲存和擷取與特定交談內容中與特定使用者相關聯的狀態資料 |

> [!NOTE]
> 即便您選擇將狀態資料儲存在自己的資料庫，而非選擇使用 Bot Framework 狀態資料存放區，您也可以使用這些屬性值做為索引鍵。

## <a id="state-client"></a> 建立狀態用戶端

`StateClient` 物件可讓您使用 Bot Builder SDK for .NET 來管理狀態資料。 如果在您想要管理狀態資料的相同內容中，有您能夠存取的訊息，您就能在 `Activity` 物件呼叫 `GetStateClient` 方法來建立狀態用戶端。

[!code-csharp[Get State client](../includes/code/dotnet-state.cs#getStateClient1)]

如果在您想要管理狀態資料的相同內容中，沒有您能夠存取的訊息，您只要建立 `StateClient` 類別新的執行個體，就能建立狀態用戶端。 在此範例中，`microsoftAppId` 和 `microsoftAppPassword` 是您在 [Bot 建立](../bot-service-quickstart.md)程序中取得的 Bot Framework 驗證認證。

> [!NOTE]
> 若要尋找 Bot 的 **AppID** 和 **AppPassword**，請參閱 [MicrosoftAppID 和 MicrosoftAppPassword](~/bot-service-manage-overview.md#microsoftappid-and-microsoftapppassword)。

[!code-csharp[Get State client](../includes/code/dotnet-state.cs#getStateClient2)]

> [!NOTE]
> 預設狀態用戶端會儲存在集中服務中。 在某些通道，您可能需要使用裝載在通道本身內的狀態 API，以便讓狀態資料儲存在通道提供的相容儲存體中。

## <a name="get-state-data"></a>取得狀態資料

每個 "**Get...Data**" 方法都會傳回 `BotData` 物件，當中含有指定使用者和/或交談的狀態資料。 若要從 `BotData` 物件取得特定的屬性值，請呼叫 `GetProperty` 方法。 

以下程式碼範例示範如何從使用者資料中取得經過歸類的屬性。 

[!code-csharp[Get state property](../includes/code/dotnet-state.cs#getProperty1)]

以下程式碼範例示範如何從使用者資料中的複雜類型取得屬性。

[!code-csharp[Get state property](../includes/code/dotnet-state.cs#getProperty2)]

如果針對 "**Get...Data**" 方法呼叫而指定的使用者和/或交談中不存在狀態資料，則傳回的 `BotData` 物件會包含以下屬性值： 
- `BotData.Data` = null
- `BotData.ETag` = "*"

## <a name="save-state-data"></a>儲存狀態資料

若要儲存狀態資料，請先呼叫適當的 "**Get...Data**" 方法以取得 `BotData` 物件，再為每個想要更新的屬性呼叫 `SetProperty` 方法加以更新，然後呼叫適當的 "**Set...Data**"方法以儲存資料。 

> [!NOTE]
> 針對通道的每個使用者、通道的每個交談，以及通道的交談內容中的每個使用者儲存，您最多可以 32 KB 的資料。 

以下程式碼範例顯示如何儲存使用者資料中經過歸類的屬性。

[!code-csharp[Set state property](../includes/code/dotnet-state.cs#setProperty1)]

以下程式碼範例顯示如何儲存使用者資料中複雜類型的屬性。 

[!code-csharp[Set state property](../includes/code/dotnet-state.cs#setProperty2)]

## <a name="handle-concurrency-issues"></a>處理並行問題

如果 Bot 的其他執行個體變更了資料，當 Bot 嘗試儲存狀態資料時，可能會收到以下的 HTTP 狀態碼錯誤回應：**412 前置條件失敗**。 您可以設計自己的 Bot 來說明這類情形，如下列程式碼範例所示。

[!code-csharp[Handle exception saving state](../includes/code/dotnet-state.cs#handleException)]

## <a name="additional-resources"></a>其他資源

- [Bot Framework 疑難排解指南](../bot-service-troubleshoot-general-problems.md)
- <a href="/dotnet/api/?view=botbuilder-3.11.0" target="_blank">Bot Builder SDK for .NET 參考資料</a>

[Activity]: https://docs.botframework.com/en-us/csharp/builder/sdkreference/dc/d2f/class_microsoft_1_1_bot_1_1_connector_1_1_activity.html
