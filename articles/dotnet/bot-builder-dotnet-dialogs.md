---
title: 對話概觀 | Microsoft Docs
description: 了解如何在適用於 .NET 的 Bot Framework SDK 中使用對話，建立交談模型及管理交談流程。
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: 3089e7a073f6a6d9af3a3720954af3a915106888
ms.sourcegitcommit: b15cf37afc4f57d13ca6636d4227433809562f8b
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 01/11/2019
ms.locfileid: "54224993"
---
# <a name="dialogs-in-the-bot-framework-sdk-for-net"></a>適用於 .NET 的 Bot Framework SDK 中的對話

[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]

> [!div class="op_single_selector"]
> - [.NET](../dotnet/bot-builder-dotnet-dialogs.md)
> - [Node.js](../nodejs/bot-builder-nodejs-dialog-overview.md)

當您使用適用於 .NET 的 Bot Framework SDK 建立 Bot 時，您可以使用對話來建立交談模型及管理[交談流程](../bot-service-design-conversation-flow.md)。 每個對話都是抽象概念，可在實作 `IDialog` 的 C# 類別中封裝本身的狀態。 對話可使用其他對話組成以便充分地重複使用，而且對話內容會維護[對話的堆疊](../bot-service-design-conversation-flow.md#dialog-stack)，這些對話在交談的任何時間點均處於作用中狀態。 

由對話 \(dialog\) 組成的對話 \(conversation\) 可移植到不同電腦，這使得您的 Bot 實作能夠加以調整。 當您在適用於 .NET 的 Bot Framework SDK 中使用對話時，交談狀態 (對話堆疊和堆疊中每個對話的狀態) 會自動儲存到您選擇的[狀態資料](bot-builder-dotnet-state.md)儲存體。 這可讓 Bot 的服務程式碼成為無狀態，如同 Web 應用程式不需要在 Web 伺服器記憶體中儲存工作階段狀態一樣。 

## <a name="echo-bot-example"></a>回應 Bot 範例

請考慮使用這個回應 Bot 範例，此範例描述如何變更在[快速入門](bot-builder-dotnet-quickstart.md)教學課程中建立的 Bot，使其使用與使用者交換訊息的對話。

> [!TIP]
> 若要遵循此範例進行操作，請使用[快速入門](bot-builder-dotnet-quickstart.md)教學課程中的指示來建立 Bot，然後更新其 **MessagesController.cs** 檔案，如下所述。

### <a name="messagescontrollercs"></a>MessagesController.cs 

在適用於 .NET 的 Bot Framework SDK 中，[Builder][builderLibrary] 程式庫可讓您實作對話。 若要存取相關類別，請匯入 `Dialogs` 命名空間。

[!code-csharp[Using statement](../includes/code/dotnet-dialogs.cs#usingStatement)]

接下來，將這個 `EchoDialog` 類別新增至 **MessagesController.cs** 來代表交談。 

[!code-csharp[EchoDialog class](../includes/code/dotnet-dialogs.cs#echobot1)]

然後，透過呼叫 `Conversation.SendAsync` 方法，將 `EchoDialog` 類別連接到 `Post`。

[!code-csharp[Post method](../includes/code/dotnet-dialogs.cs#echobot2)]

### <a name="implementation-details"></a>實作詳細資料 

`Post` 方法已標記為 `async`，因為 Bot 建立器使用 C# 功能來處理非同步通訊。 它會傳回 `Task` 物件，代表負責傳送傳入訊息回覆的工作。 如果發生例外狀況，方法所傳回的 `Task` 將會包含例外狀況資訊。 

`Conversation.SendAsync` 方法是使用適用於 .NET 的 Bot Framework SDK 實作對話的關鍵。 它會遵循<a href="https://en.wikipedia.org/wiki/Dependency_inversion_principle" target="_blank">相依性反轉準則</a>，並執行下列步驟：

1. 具現化必要元件  
2. 從 `IBotDataStore` 將交談狀態 (對話堆疊和堆疊中每個對話的狀態) 還原序列化
3. 繼續 Bot 暫停並等候訊息的交談流程
4. 傳送回覆
5. 將已更新的交談狀態序列化，並將其儲存回 `IBotDataStore`

第一次啟動交談時，對話不會包含狀態，因此 `Conversation.SendAsync` 會建構 `EchoDialog`，並呼叫其 `StartAsync` 方法。 `StartAsync` 方法會呼叫 `IDialogContext.Wait`　，繼續委派以指定在收到新訊息時應該呼叫的方法 (`MessageReceivedAsync`)。 

`MessageReceivedAsync` 方法會等候訊息，張貼回應，並等候下一則訊息。 每次呼叫 `IDialogContext.Wait` 時，Bot 會進入暫停狀態，因此可以在接收訊息的任何電腦上重新啟動。 

使用上述程式碼範例建立的 Bot 會回覆使用者傳送的每則訊息，只需回應前置文字為 'You said: ' 的使用者訊息即可。 由於 Bot 是使用對話建立的，因此它能進化到支援更複雜的交談，而不需要明確地管理狀態。

## <a name="echo-bot-with-state-example"></a>以狀態回應 Bot 的範例

下一個範例會新增功能來追蹤對話狀態，以上一個範例為基礎進行建置。 當 `EchoDialog` 類別如下列程式碼範例所示更新時，Bot 會回覆使用者傳送的每則訊息，只需回應前置數字 (`count`) 且後接文字 'You said: ' 的使用者訊息即可。 Bot 將在每次回覆時繼續遞增 `count`，直到使用者選擇重設計數為止。

### <a name="messagescontrollercs"></a>MessagesController.cs 

[!code-csharp[EchoDialog class](../includes/code/dotnet-dialogs.cs#echobot3)]

### <a name="implementation-details"></a>實作詳細資料

如同第一個範例，在收到新訊息時會呼叫 `MessageReceivedAsync` 方法。 不過，這次 `MessageReceivedAsync` 方法會在回應之前評估使用者的訊息。 如果使用者的訊息是 "reset"，則內建的 `PromptDialog.Confirm` 提示會繁衍子對話，要求使用者確認計數重設。 子對話有自己的專用狀態，不會干擾父對話的狀態。 當使用者回應提示時，子對話的結果會傳遞至 `AfterResetAsync` 方法，這可將訊息傳送給使用者，以指出計數是否已重設，然後呼叫 `IDialogContext.Wait` 繼續回到下一則訊息的 `MessageReceivedAsync`。

## <a name="dialog-context"></a>對話內容

傳入每個對話方法的 `IDialogContext` 介面可存取對話所需的服務，用來儲存狀態並與通道進行通訊。 `IDialogContext` 介面包含三個介面：[Internals.IBotData][iBotData]、[Internals.IBotToUser][iBotToUser] 和 [Internals.IDialogStack][iDialogStack]。 

### <a name="internalsibotdata"></a>Internals.IBotData

`Internals.IBotData` 介面可以存取每位使用者、每個交談和連接器所維護之私用交談的狀態資料。 每位使用者的狀態資料適用於儲存與特定交談無關的使用者資料，每個交談的資料適用於儲存有關交談的一般資料，而私用交談資料則適用於儲存與特定交談相關的使用者資料。 

### <a name="internalsibottouser"></a>Internals.IBotToUser

`Internals.IBotToUser` 提供方法將訊息從 Bot 傳送給使用者。 訊息可能會內嵌在 Web API 方法呼叫的回應中傳送，或者使用[連接器用戶端](bot-builder-dotnet-connector.md#create-a-connector-client)直接傳送。 透過對話內容傳送和接收訊息可確保 `Internals.IBotData` 狀態會通過連接器。

### <a name="internalsidialogstack"></a>Internals.IDialogStack

`Internals.IDialogStack` 提供方法來管理[對話堆疊](../bot-service-design-conversation-flow.md#dialog-stack)。 大部分的情況下，將會為您自動管理對話堆疊。 不過，可能會有您想要明確地管理堆疊的情況。 例如，您可能想要呼叫子對話並將它新增至對話堆疊的頂端、將目前的對話標示為完成 (藉此將它從對話堆疊中移除，並將結果傳回堆疊中較早的對話)、暫停目前的對話直到來自使用者的訊息抵達，或甚至完全重設對話堆疊。

## <a name="serialization"></a>序列化

對話堆疊和所有使用中對話的狀態都會序列化為每位使用者、每個交談的 [IBotDataBag][iBotDataBag]。 序列化的 Blob 會保存在 Bot 從[連接器](bot-builder-dotnet-concepts.md#connector)中傳送和接收的訊息中。 若要序列化，`Dialog` 類別必須包含 `[Serializable]` 屬性。 [Builder][builderLibrary] 程式庫中的所有 `IDialog` 實作都會標示為可序列化。 

[Chain 方法](#dialog-chains)為可用於 LINQ 查詢語法的對話提供 Fluent 介面。 LINQ 查詢語法的已編譯形式通常會使用匿名方法。 如果這些匿名方法不會參考本機變數的環境，則這些匿名方法沒有狀態，並可透過極簡方式序列化。 不過，如果匿名方法會擷取環境中的任何區域變數，產生的結束物件 (由編譯器產生) 不會標示為可序列化。 在此情況下，Bot 建立器會擲回 `ClosureCaptureException` 以找出問題。

若要使用反映來序列化未標示為可序列化的類別，建立器程式庫包含可供您用來註冊 [Autofac][autofac] 的反映型序列化代理。

[!code-csharp[Serialization](../includes/code/dotnet-dialogs.cs#serialization)]

## <a id="dialog-chains"></a> 對話鏈結

您可以使用 `IDialogStack.Call<R>` 和 `IDialogStack.Done<R>` 以明確管理使用中對話的堆疊，同時也可以使用下列流暢的 [Chain][chain] 方法隱含地管理使用中對話的堆疊。


|           方法            |  類型   |                                 注意                                  |
|-----------------------------|---------|------------------------------------------------------------------------|
|     Chain.Select<T, R>      |  LINQ   |           支援 LINQ 查詢語法中的 "select" 和 "let"。            |
|  Chain.SelectMany<T, C, R>  |  LINQ   |            支援 LINQ 查詢語法中的後續 "from"。            |
|       Chain.Where<T>        |  LINQ   |                 支援 LINQ 查詢語法中的 "where"。                 |
|        Chain.From<T>        | 鏈結  |                具現化對話的新執行個體。                |
|       Chain.Return<T>       | 鏈結  |                將常數值傳回到鏈結。                |
|         Chain.Do<T>         | 鏈結  |               允許鏈結內的副作用。                |
|  Chain.ContinueWith<T, R>   | 鏈結  |                      對話的簡單鏈結。                       |
|       Chain.Unwrap<T>       | 鏈結  |                  解除包裝對話中巢狀處理的對話。                   |
| Chain.DefaultIfException<T> | 鏈結  | 接受來自先前結果的例外狀況，並傳回 default(T)。 |
|        Chain.Loop<T>        | 分支  |                   循環整個對話鏈結。                   |
|        Chain.Fold<T>        | 分支  |   將來自對話列舉的結果摺疊成單一結果。   |
|     Chain.Switch<T, R>      | 分支  |            支援分支到不同的對話鏈結。            |
|     Chain.PostToUser<T>     | 訊息 |                      將訊息張貼給使用者。                      |
|     Chain.WaitToBot<T>      | 訊息 |                    等候將訊息傳送至 Bot。                     |
|    Chain.PostToChain<T>     | 訊息 |              使用來自使用者的訊息開始鏈結。              |

### <a name="examples"></a>範例 

LINQ 查詢語法會使用 `Chain.Select<T, R>` 方法。

[!code-csharp[Chain.Select](../includes/code/dotnet-dialogs.cs#chain1)]

或者，`Chain.SelectMany<T, C, R>` 方法。

[!code-csharp[Chain.SelectMany](../includes/code/dotnet-dialogs.cs#chain2)]

`Chain.PostToUser<T>` 和 `Chain.WaitToBot<T>` 方法會將來自 Bot 的訊息張貼給使用者，反之亦然。

[!code-csharp[Chain.PostToUser](../includes/code/dotnet-dialogs.cs#chain3)]

`Chain.Switch<T, R>` 方法會分支交談對話流程。

[!code-csharp[Chain.Switch](../includes/code/dotnet-dialogs.cs#chain4)]

如果 `Chain.Switch<T, R>` 傳回巢狀 `IDialog<IDialog<T>>`，則內部 `IDialog<T>` 可以使用 `Chain.Unwrap<T>` 來解除包裝。 這可讓交談分支到已鏈結對話的不同路徑，可能的長度不等。 此範例示範分支對話的更完整範例，該對話是使用隱含的堆疊管理以更流暢的鏈結樣式所撰寫而成。

[!code-csharp[Chain.Switch](../includes/code/dotnet-dialogs.cs#chain5)]

## <a name="next-steps"></a>後續步驟

對話可管理 Bot 與使用者之間的交談流程。 對話會定義與使用者互動的方式。 Bot 可以使用堆疊中組織的許多對話，來引導與使用者的交談。 在下一節中，請查看當您建立和關閉堆疊中的對話時，對話堆疊如何成長和壓縮。

> [!div class="nextstepaction"]
> [使用對話管理交交談流程](bot-builder-dotnet-manage-conversation-flow.md)


[builderLibrary]: /dotnet/api/microsoft.bot.builder.dialogs

[iBotData]: /dotnet/api/microsoft.bot.builder.dialogs.internals.ibotdata

[iBotToUser]: /dotnet/api/microsoft.bot.builder.dialogs.internals.ibottouser

[iDialogStack]: /dotnet/api/microsoft.bot.builder.dialogs.internals.idialogstack

[iBotDataBag]: /dotnet/api/microsoft.bot.builder.dialogs.ibotdatabag

[autofac]: /dotnet/api/microsoft.bot.builder.autofac.base

[chain]: /dotnet/api/microsoft.bot.builder.dialogs.chain
