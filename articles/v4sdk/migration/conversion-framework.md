---
title: 在相同 .NET Framework 專案中遷移現有的 Bot | Microsoft Docs
description: 我們採用現有的 .NET v3 聊天機器人並將其遷移至 .NET v4 SDK，而且使用相同的專案。
keywords: bot 移轉, formflow, 對話, v3 bot
author: JonathanFingold
ms.author: v-jofing
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 05/23/2019
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 45830f099833c41c308b0f5a5e7b104986604e03
ms.sourcegitcommit: 93508adfb79523f610a919b361fc34f5c8dd3eff
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/02/2019
ms.locfileid: "67533391"
---
# <a name="migrate-a-net-v3-bot-to-a-net-framework-v4-bot"></a>將 .NET v3 聊天機器人遷移至 .NET Framework v4 聊天機器人

我們在本文中將 [v3 ContosoHelpdeskChatBot](https://github.com/microsoft/BotBuilder-Samples/tree/master/MigrationV3V4/CSharp/ContosoHelpdeskChatBot-V3) 轉換為 v4 Bot，而_不需轉換專案類型_。 其仍然是 .NET Framework 專案。
此轉換可細分成下列步驟：

1. 更新和安裝 NuGet 套件
1. 更新 Global.asax.cs 檔案
1. 更新 MessagesController 類別
1. 轉換您的對話

此轉換的結果是 [.NET Framework v4 ContosoHelpdeskChatBot](https://github.com/microsoft/BotBuilder-Samples/tree/master/MigrationV3V4/CSharp/ContosoHelpdeskChatBot-V4NetFramework)。
若要遷移至新專案中的 .NET Core v4 聊天機器人，請參閱[將 .NET v3 聊天機器人遷移至 .NET Core v4 聊天機器人](conversion-core.md)。

Bot Framework SDK v4 是以與 SDK v3 相同的基礎 REST API 作為基礎。 不過，SDK v4 是舊版 SDK 的重構，讓開發人員對於其 Bot 有更多的彈性和控制權。 SDK 中的主要變更包括：

- 透過狀態管理物件和屬性存取子管理狀態。
- 設定回合處理常式並將活動傳遞給它已變更。
- 可評分項目已不存在。 將控制權交給您的對話之前，可以檢查回合處理常式中的「全域」命令。
- 與前一版非常不同的新 Dialogs 程式庫。 您必須使用元件和瀑布式對話，以及適用於 v4 的 Formflow 對話社群實作，將舊的對話轉換至新的對話系統。

如需特定變更的詳細資訊，請參閱 [v3 和 v4 .NET SDK 之間差異](migration-about.md)。

## <a name="update-and-install-nuget-packages"></a>更新和安裝 NuGet 套件

1. 將 **Microsoft.Bot.Builder.Azure** 和 **Microsoft.Bot.Builder.Integration.AspNet.WebApi** 更新為最新穩定版本。

    這將也會更新 **Microsoft.Bot.Builder** 和 **Microsoft.Bot.Connector** 套件，因為它們是相依項目。

1. 刪除 **Microsoft.Bot.Builder.History** 套件。 這不是 v4 SDK 的一部分。
1. 新增 **Autofac.WebApi2**

    我們將使用它來協助在 ASP.NET 中插入相依性。

1. 新增 **Bot.Builder.Community.Dialogs.Formflow**

    這是一個社群程式庫，用於從 v3 Formflow 定義檔建置 v4 對話。 它以 **Microsoft.Bot.Builder.Dialogs** 作為其相依性之一，因此系統也會為我們安裝。

如果您在此時建置，則會收到編譯器錯誤。 您可以忽略這些錯誤。 完成轉換之後，我們就會有工作程式碼。

## <a name="update-your-globalasaxcs-file"></a>更新 Global.asax.cs 檔案

某些 Scaffolding 已變更，而且我們必須自行在 v4 中設定部份的[狀態管理](../bot-builder-concept-state.md)基礎結構。 例如，v4 會使用 Bot 配接器來處理驗證並將活動轉送到 Bot 程式碼，而我們會事先宣告我們的狀態屬性。

我們將建立 `DialogState` 的狀態屬性，我們現在需要該屬性才能在 v4 中支援對話。 我們將使用相依性插入來取得控制器和 Bot 程式碼的必要資訊。

在 **Global.asax.cs** 中：

1. 更新 `using` 陳述式：  
    [!code-csharp[Using statements](~/../botbuilder-samples/MigrationV3V4/CSharp/ContosoHelpdeskChatBot-V4NetFramework/ContosoHelpdeskChatBot/Global.asax.cs?range=4-13)]

1. 從 `Application_Start` 方法中移除這幾行：  
    [!code-csharp[Removed lines](~/../botbuilder-samples/MigrationV3V4/CSharp/ContosoHelpdeskChatBot-V3/ContosoHelpdeskChatBot/Global.asax.cs?range=23-24)]

    並且插入這一行：  
    [!code-csharp[Reference BotConfig.Register](~/../botbuilder-samples/MigrationV3V4/CSharp/ContosoHelpdeskChatBot-V4NetFramework/ContosoHelpdeskChatBot/Global.asax.cs?range=22)]

1. 移除不再參考的 `RegisterBotModules` 方法。

1. 以此 `BotConfig.Register` 方法取代 `BotConfig.UpdateConversationContainer` 方法，我們將在其中註冊支援相依性插入所需的物件。 此 Bot 不採用_使用者_和_私人交談_狀態，因此我們只建立交談狀態管理物件。  
    [!code-csharp[Define BotConfig.Register](~/../botbuilder-samples/MigrationV3V4/CSharp/ContosoHelpdeskChatBot-V4NetFramework/ContosoHelpdeskChatBot/Global.asax.cs?range=31-61)]

## <a name="update-your-messagescontroller-class"></a>更新 MessagesController 類別

這是 v4 中 Bot 起始回合的地方，因此需求大幅變更。 除了 Bot 的回合處理常式本身，可將大部分的內容視為重複使用文字。 在 **Controllers\MessagesController.cs** 檔案中：

1. 更新 `using` 陳述式：  
    [!code-csharp[Using statements](~/../botbuilder-samples/MigrationV3V4/CSharp/ContosoHelpdeskChatBot-V4NetFramework/ContosoHelpdeskChatBot/Controllers/MessagesController.cs?range=4-8)]

1. 從類別中移除 `[BotAuthentication]` 屬性。 在 v4 中，Bot 的配接器會處理驗證。

1. 新增這些欄位和一個建構函式，並初始化這些項目。 ASP.NET 和 Autofac 會使用相依性插入來取得參數值。 (為支援此功能，我們已在 **Global.asax.cs** 中註冊配接器和 Bot 物件。)  
    [!code-csharp[Fields and constructor](~/../botbuilder-samples/MigrationV3V4/CSharp/ContosoHelpdeskChatBot-V4NetFramework/ContosoHelpdeskChatBot/Controllers/MessagesController.cs?range=14-21)]

1. 取代 `Post` 方法的主體。 我們可以使用配接器來呼叫我們的 Bot 訊息迴圈 (回合處理常式)。  
    [!code-csharp[Post method](~/../botbuilder-samples/MigrationV3V4/CSharp/ContosoHelpdeskChatBot-V4NetFramework/ContosoHelpdeskChatBot/Controllers/MessagesController.cs?range=23-31)]

### <a name="delete-the-cancelscorable-and-globalmessagehandlersbotmodule-classes"></a>刪除 CancelScorable 和 GlobalMessageHandlersBotModule 類別

因為 v4 中不存在可評分項目，而且我們已更新回合處理常式以回應 `cancel` 訊息，所以可刪除 `CancelScorable` (在**Dialogs\CancelScorable.cs**) 和 `GlobalMessageHandlersBotModule` 類別。

## <a name="create-your-bot-class"></a>建立您的 Bot 類別

在 v4 中，回合處理常式或訊息迴圈邏輯是 Bot 檔案中的主要項目。 我們將從 `ActivityHandler` 衍生，此項目會定義常見活動類型的處理常式。

1. 建立 **Bots\DialogBots.cs** 檔案。

1. 更新 `using` 陳述式：  
    [!code-csharp[Using statements](~/../botbuilder-samples/MigrationV3V4/CSharp/ContosoHelpdeskChatBot-V4NetFramework/ContosoHelpdeskChatBot/Bots/DialogBot.cs?range=4-8)]

1. 從 `ActivityHandler` 衍生 `DialogBot`，並新增對話方塊的泛型參數。  
    [!code-csharp[Class definition](~/../botbuilder-samples/MigrationV3V4/CSharp/ContosoHelpdeskChatBot-V4NetFramework/ContosoHelpdeskChatBot/Bots/DialogBot.cs?range=19)]

1. 新增這些欄位和一個建構函式，並初始化這些項目。 同樣地，ASP.NET 和 Autofac 會使用相依性插入來取得參數值。  
    [!code-csharp[Fields and constructor](~/../botbuilder-samples/MigrationV3V4/CSharp/ContosoHelpdeskChatBot-V4NetFramework/ContosoHelpdeskChatBot/Bots/DialogBot.cs?range=21-28)]

1. 覆寫 `OnMessageActivityAsync` 以叫用我們的主要對話方塊。 (我們會簡短地定義 `Run` 擴充方法。)  
    [!code-csharp[OnMessageActivityAsync](~/../botbuilder-samples/MigrationV3V4/CSharp/ContosoHelpdeskChatBot-V4NetFramework/ContosoHelpdeskChatBot/Bots/DialogBot.cs?range=38-47)]

1. 覆寫 `OnTurnAsync`，以在回合結束時儲存我們的交談狀態。 在 v4 中，我們必須明確地執行此作業，以將狀態寫出到持續性層。 `ActivityHandler.OnTurnAsync` 方法會呼叫特定活動處理常式方法 (根據接收的活動類型)，因此我們會在呼叫基底方法之後儲存狀態。  
    [!code-csharp[OnTurnAsync](~/../botbuilder-samples/MigrationV3V4/CSharp/ContosoHelpdeskChatBot-V4NetFramework/ContosoHelpdeskChatBot/Bots/DialogBot.cs?range=30-36)]

### <a name="create-the-run-extension-method"></a>建立執行擴充方法

我們將建立擴充方法來合併從 Bot 執行裸機元件對話所需的程式碼。

建立 **DialogExtensions.cs** 檔案並實作 `Run` 擴充方法。  
[!code-csharp[The extension](~/../botbuilder-samples/MigrationV3V4/CSharp/ContosoHelpdeskChatBot-V4NetFramework/ContosoHelpdeskChatBot/DialogExtensions.cs?range=4-41)]

## <a name="convert-your-dialogs"></a>轉換您的對話

我們將對原始對話進行許多變更，以將其遷移至 v4 SDK。 現在不用擔心編譯器錯誤。 待我們完成轉換後，就會解決這些錯誤。
為了不會對原始程式碼進行不必要的修改，在我們完成移轉後，仍會有一些編譯器警告。

我們所有的對話都將衍生自 `ComponentDialog`，而不會實作 v3 的 `IDialog<object>` 介面。

此 Bot 有四個需要我們轉換的對話：

| | |
|---|---|
| [RootDialog](#update-the-root-dialog) | 顯示選項並開始其他對話。 |
| [InstallAppDialog](#update-the-install-app-dialog) | 處理在電腦上安裝應用程式的要求。 |
| [LocalAdminDialog](#update-the-local-admin-dialog) | 處理本機電腦系統管理權限的要求。 |
| [ResetPasswordDialog](#update-the-reset-password-dialog) | 處理重設密碼的要求。 |

這些對話會收集輸入，但不會在您的電腦上執行上述任何作業。

### <a name="make-solution-wide-dialog-changes"></a>進行全方案的對話變更

1. 針對整個方案，以 `ComponentDialog` 取代所有出現的 `IDialog<object>`。
1. 針對整個方案，以 `DialogContext` 取代所有出現的 `IDialogContext`。
1. 針對每個對話類別，移除 `[Serializable]` 屬性。

對話內的控制流程和傳訊不再以相同的方式處理，因此我們必須在轉換每個對話時修改這個屬性。

| 作業 | v3 程式碼 | v4 程式碼 |
| :--- | :--- | :--- |
| 處理對話的開始 | 實作 `IDialog.StartAsync` | 讓此項成為瀑布式對話的第一個步驟，或實作 `Dialog.BeginDialogAsync` |
| 處理對話的接續 | 呼叫 `IDialogContext.Wait` | 在瀑布式對話中新增其他步驟，或實作 `Dialog.ContinueDialogAsync` |
| 將訊息傳送給使用者 | 呼叫 `IDialogContext.PostAsync` | 呼叫 `ITurnContext.SendActivityAsync` |
| 開始子對話 | 呼叫 `IDialogContext.Call` | 呼叫 `DialogContext.BeginDialogAsync` |
| 表示目前對話已完成的訊號 | 呼叫 `IDialogContext.Done` | 呼叫 `DialogContext.EndDialogAsync` |
| 取得使用者的輸入 | 使用 `IAwaitable<IMessageActivity>` 參數 | 使用瀑布中的提示，或使用 `ITurnContext.Activity` |

v4 程式碼的注意事項：

- 在對話程式碼內，使用 `DialogContext.Context` 屬性來取得目前的回合內容。
- 瀑布步驟具有 `WaterfallStepContext` 參數，其衍生自 `DialogContext`。
- 所有具體的對話和提示類別均衍生自抽象的 `Dialog` 類別。
- 您會在建立元件對話時指派識別碼。 對話集中的每個對話都需要被指派該集合內唯一的識別碼。

### <a name="update-the-root-dialog"></a>更新根對話

在此 Bot 中，根對話會提示使用者從一組選項中進行選擇，然後根據該選擇開始進行子對話。 接著在交談存留期間執行迴圈。

- 我們可以將主要流程設定為瀑布式對話，這是 v4 SDK 中的新概念。 它會依序執行一組固定的步驟。 如需詳細資訊，請參閱[實作循序交談流程](~/v4sdk/bot-builder-dialog-manage-conversation-flow.md)。
- 提示作業現在會透過提示類別來處理，而這些類別是簡短的子對話，可提示輸入資料、執行最少的處理和驗證，並傳回一個值。 如需詳細資訊，請參閱[使用對話提示收集使用者輸入](~/v4sdk/bot-builder-prompts.md)。

在 **Dialogs/RootDialog.cs** 檔案中：

1. 更新 `using` 陳述式：  
    [!code-csharp[Using statements](~/../botbuilder-samples/MigrationV3V4/CSharp/ContosoHelpdeskChatBot-V4NetFramework/ContosoHelpdeskChatBot/Dialogs/RootDialog.cs?range=4-10)]

1. 我們需要將 `HelpdeskOptions` 選項從字串清單轉換為選擇清單。 這將搭配選擇提示使用，其將接受選擇號碼 (清單中)、選擇值，或任何選擇的同義字作為有效的輸入。  
    [!code-csharp[HelpDeskOptions](~/../botbuilder-samples/MigrationV3V4/CSharp/ContosoHelpdeskChatBot-V4NetFramework/ContosoHelpdeskChatBot/Dialogs/RootDialog.cs?range=28-33)]

1. 新增建構函式。 此程式碼會執行以下動作：
   - 每個對話執行個體會在建立時被指派識別碼。 對話識別碼是對話要新增到其中的對話集的一部分。 還記得 Bot 已透過 `MessageController` 類別中的對話方塊物件初始化。 每個 `ComponentDialog` 都有自己的內部對話集，以及自己的對話識別碼集。
   - 它會新增其他對話 (包括選擇提示) 作為子對話。 在此，我們只會使用每個對話識別碼的類別名稱。
   - 它會定義包含三個步驟的瀑布式對話。 我們會立刻進行實作。
     - 對話會先提示使用者選擇要執行的工作。
     - 然後，開始與該選擇相關聯的子對話。
     - 最後，自行重新開始。
   - 瀑布的每個步驟都是一項委派，而我們接下來會實作這些步驟，並從原始對話中取得現有的程式碼。
   - 當您啟動元件對話時，就會啟動其「初始對話」  。 根據預設，這是新增至元件對話的第一個子對話。 我們會明確地設定 `InitialDialogId` 屬性，這表示主要瀑布式對話方塊不必是您新增至對話集的第一個對話方塊。 比方說，如果您想要先新增提示，這可讓您執行這項操作，而不會造成執行階段問題。  
    [!code-csharp[Constructor](~/../botbuilder-samples/MigrationV3V4/CSharp/ContosoHelpdeskChatBot-V4NetFramework/ContosoHelpdeskChatBot/Dialogs/RootDialog.cs?range=35-49)]

1. 我們可以刪除 `StartAsync` 方法。 當元件對話開始時，它會自動開始它「初始」  對話。 在此情況下，這就是我們在建構函式中定義的瀑布式對話。 該對話也會自動在其第一個步驟開始。

1. 我們將會刪除 `MessageReceivedAsync` 和 `ShowOptions` 方法，並以瀑布的第一個步驟取代。 這兩種方法會先映入使用者的眼簾，並要求他們選擇其中一個可用的選項。
   - 您可以在此看到選擇清單，而系統會提供問候和錯誤訊息作為我們的選擇提示呼叫中的選項。
   - 我們不需要指定要在對話中呼叫的下一個方法，因為瀑布會在選擇提示完成時繼續下一個步驟。
   - 選擇提示將會執行迴圈，直到它收到有效的輸入，或取消整個對話堆疊為止。  
    [!code-csharp[PromptForOptionsAsync](~/../botbuilder-samples/MigrationV3V4/CSharp/ContosoHelpdeskChatBot-V4NetFramework/ContosoHelpdeskChatBot/Dialogs/RootDialog.cs?range=51-65)]

1. 我們可以使用瀑布的第二個步驟取代 `OnOptionSelected`。 我們仍會根據使用者的輸入開始子對話。
   - 選擇提示會傳回 `FoundChoice` 值。 這會顯示在步驟內容的 `Result` 屬性中。 對話堆疊會將所有傳回值視為物件。 如果傳回值來自您的其中一個對話，您便知道物件值是何種類型。 如需每個提示類型傳回的內容清單，請參閱[提示類型](../bot-builder-concept-dialog.md#prompt-types)。
   - 因為選擇提示不會擲回例外狀況，所以可移除 try-catch 區塊。
   - 我們必須新增一項通過，此方法才能一律傳回適當的值。 此程式碼應該永遠不會被叫用，但如果叫用，就會讓對話「正常失敗」。  
    [!code-csharp[ShowChildDialogAsync](~/../botbuilder-samples/MigrationV3V4/CSharp/ContosoHelpdeskChatBot-V4NetFramework/ContosoHelpdeskChatBot/Dialogs/RootDialog.cs?range=67-102)]

1. 最後，使用瀑布的最後一個步驟取代舊的 `ResumeAfterOptionDialog` 方法。
    - 我們會將堆疊上的原始執行個體取代為本身的新執行個體，進而重新開始瀑布式對話，而不是如同我們在原始對話中一樣結束對話並傳回票證號碼。 我們可以這麼做，因為原始應用程式一律忽略傳回值 (票證號碼)，並重新開始進行根對話。  
    [!code-csharp[ResumeAfterAsync](~/../botbuilder-samples/MigrationV3V4/CSharp/ContosoHelpdeskChatBot-V4NetFramework/ContosoHelpdeskChatBot/Dialogs/RootDialog.cs?range=104-138)]

### <a name="update-the-install-app-dialog"></a>更新安裝應用程式對話

安裝應用程式對話會執行一些邏輯工作，我們會將其設定為包含 4 個步驟的瀑布式對話。 如何將現有程式碼分解成瀑布步驟是每個對話的邏輯活動。 每個步驟都會註明程式碼來自的原始方法。

1. 要求使用者提供搜尋字串。
1. 查詢資料庫中可能的相符項目。
   - 如果有一項命中，請選取此項目並繼續執行。
   - 如果有多項命中，則會要求使用者選擇一項。
   - 如果沒有命中項目，就會結束對話。
1. 要求使用者提供要安裝應用程式的機器。
1. 將資訊寫入資料庫並傳送確認訊息。

在 **Dialogs/InstallAppDialog.cs** 檔案中：

1. 更新 `using` 陳述式：  
    [!code-csharp[Using statements](~/../botbuilder-samples/MigrationV3V4/CSharp/ContosoHelpdeskChatBot-V4NetFramework/ContosoHelpdeskChatBot/Dialogs/InstallAppDialog.cs?range=4-11)]

1. 為我們用來追蹤所收集資訊的金鑰定義常數。  
    [!code-csharp[Key ID](~/../botbuilder-samples/MigrationV3V4/CSharp/ContosoHelpdeskChatBot-V4NetFramework/ContosoHelpdeskChatBot/Dialogs/InstallAppDialog.cs?range=17-18)]

1. 新增建構函式並初始化元件的對話集。  
    [!code-csharp[Constructor](~/../botbuilder-samples/MigrationV3V4/CSharp/ContosoHelpdeskChatBot-V4NetFramework/ContosoHelpdeskChatBot/Dialogs/InstallAppDialog.cs?range=20-33)]

1. 我們可以使用瀑布的第一個步驟取代 `StartAsync`。
    - 我們不必自行管理狀態，所以會追蹤對話狀態中的安裝應用程式物件。
    - 要求使用者輸入資料的訊息會變成提示呼叫中的選項。  
    [!code-csharp[GetSearchTermAsync](~/../botbuilder-samples/MigrationV3V4/CSharp/ContosoHelpdeskChatBot-V4NetFramework/ContosoHelpdeskChatBot/Dialogs/InstallAppDialog.cs?range=35-50)]

1. 我們可以使用瀑布的第二個步驟取代 `appNameAsync` 和 `multipleAppsAsync`。
    - 我們現在會取得提示結果，而不只是查看使用者的最後一則訊息。
    - 資料庫查詢和 if 陳述式的組織方式與在 `appNameAsync` 中相同。 已更新 if 陳述式的每個區塊中的程式碼，可搭配 v4 對話運作。
        - 如果我們有一次命中，我們將會更新對話狀態並繼續進行下一個步驟。
        - 如果您有多項命中，我們將使用選擇提示來要求使用者從選項清單中進行選擇。 這表示我們可以刪除 `multipleAppsAsync`。
        - 如果我們沒有命中，我們會結束此對話並傳回 null 給根對話。  
    [!code-csharp[ResolveAppNameAsync](~/../botbuilder-samples/MigrationV3V4/CSharp/ContosoHelpdeskChatBot-V4NetFramework/ContosoHelpdeskChatBot/Dialogs/InstallAppDialog.cs?range=52-91)]

1. `appNameAsync` 也在解析查詢之後，要求使用者提供其機器名稱。 我們將在瀑布的下一個步驟中擷取邏輯的該部分。
    - 同樣地，在 v4 中，我們必須自行管理狀態。 唯一比較麻煩的事，就是我們可以透過上一個步驟中的兩個不同邏輯分支來抵達此步驟。
    - 我們會使用與之前相同的文字提示來要求使用者提供機器名稱，只是這次提供不同的選項。  
    [!code-csharp[GetMachineNameAsync](~/../botbuilder-samples/MigrationV3V4/CSharp/ContosoHelpdeskChatBot-V4NetFramework/ContosoHelpdeskChatBot/Dialogs/InstallAppDialog.cs?range=93-114)]

1. `machineNameAsync` 中的邏輯會包裝在瀑布的最後一個步驟中。
    - 我們可從文字提示結果中擷取機器名稱並更新對話狀態。
    - 我們正在移除呼叫以更新資料庫，因為支援的程式碼位於不同的專案中。
    - 然後我們會將成功訊息傳送給使用者並結束對話。  
    [!code-csharp[SubmitRequestAsync](~/../botbuilder-samples/MigrationV3V4/CSharp/ContosoHelpdeskChatBot-V4NetFramework/ContosoHelpdeskChatBot/Dialogs/InstallAppDialog.cs?range=116-135)]

1. 為了模擬資料庫呼叫，我們模擬 `getAppsAsync` 來查詢靜態清單，而不是查詢資料庫。  
    [!code-csharp[GetAppsAsync](~/../botbuilder-samples/MigrationV3V4/CSharp/ContosoHelpdeskChatBot-V4NetFramework/ContosoHelpdeskChatBot/Dialogs/InstallAppDialog.cs?range=137-200)]

### <a name="update-the-local-admin-dialog"></a>更新本機系統管理對話

在 v3 中，此對話會先映入使用者的眼簾、開始 Formflow 對話，然後將結果儲存至資料庫。 這可輕鬆地轉換成包含兩個步驟的瀑布。

1. 更新 `using` 陳述式。 請注意，此對話包含 v3 Formflow 對話。 在 v4 中，我們可以使用社群 Formflow 程式庫。  
    [!code-csharp[Using statements](~/../botbuilder-samples/MigrationV3V4/CSharp/ContosoHelpdeskChatBot-V4NetFramework/ContosoHelpdeskChatBot/Dialogs/LocalAdminDialog.cs?range=4-8)]

1. 我們可以移除 `LocalAdmin` 的執行個體屬性，因為可在對話狀態中取得結果。

1. 新增建構函式並初始化元件的對話集。 Formflow 對話會以相同的方式建立。 我們只是將它新增至建構函式中元件的對話集。  
    [!code-csharp[Constructor](~/../botbuilder-samples/MigrationV3V4/CSharp/ContosoHelpdeskChatBot-V4NetFramework/ContosoHelpdeskChatBot/Dialogs/LocalAdminDialog.cs?range=14-23)]

1. 我們可以使用瀑布的第一個步驟取代 `StartAsync`。 我們已經在建構函式中建立 Formflow，而其他兩個陳述式會轉譯為此陳述式。 請注意，`FormBuilder` 會將模型類型名稱指派為所產生對話方塊的識別碼，而此模型的對話是 `LocalAdminPrompt`。  
    [!code-csharp[BeginFormflowAsync](~/../botbuilder-samples/MigrationV3V4/CSharp/ContosoHelpdeskChatBot-V4NetFramework/ContosoHelpdeskChatBot/Dialogs/LocalAdminDialog.cs?range=25-35)]

1. 我們可以使用瀑布的第二個步驟取代 `ResumeAfterLocalAdminFormDialog`。 我們必須從步驟內容中取得傳回值，而不是從執行個體屬性中取得。  
    [!code-csharp[SaveResultAsync](~/../botbuilder-samples/MigrationV3V4/CSharp/ContosoHelpdeskChatBot-V4NetFramework/ContosoHelpdeskChatBot/Dialogs/LocalAdminDialog.cs?range=37-50)]

1. `BuildLocalAdminForm` 大致維持相同，但我們沒有讓 Formflow 更新執行個體屬性。  
    [!code-csharp[BuildLocalAdminForm](~/../botbuilder-samples/MigrationV3V4/CSharp/ContosoHelpdeskChatBot-V4NetFramework/ContosoHelpdeskChatBot/Dialogs/LocalAdminDialog.cs?range=52-76)]

### <a name="update-the-reset-password-dialog"></a>更新重設密碼對話

在 v3 中，此對話會先映入使用者的眼簾、透過密碼授權給使用者、結束或開始 Formflow 對話，然後重設密碼。 這仍會轉譯成瀑布。

1. 更新 `using` 陳述式。 請注意，此對話包含 v3 Formflow 對話。 在 v4 中，我們可以使用社群 Formflow 程式庫。  
    [!code-csharp[Using statements](~/../botbuilder-samples/MigrationV3V4/CSharp/ContosoHelpdeskChatBot-V4NetFramework/ContosoHelpdeskChatBot/Dialogs/ResetPasswordDialog.cs?range=4-9)]

1. 新增建構函式並初始化元件的對話集。 Formflow 對話會以相同的方式建立。 我們只是將它新增至建構函式中元件的對話集。  
    [!code-csharp[Constructor](~/../botbuilder-samples/MigrationV3V4/CSharp/ContosoHelpdeskChatBot-V4NetFramework/ContosoHelpdeskChatBot/Dialogs/ResetPasswordDialog.cs?range=15-25)]

1. 我們可以使用瀑布的第一個步驟取代 `StartAsync`。 我們已經在建構函式中建立 Formflow。 否則，我們會保留相同的邏輯，只是將 v3 呼叫轉譯成 v4 對等項目。  
    [!code-csharp[BeginFormflowAsync](~/../botbuilder-samples/MigrationV3V4/CSharp/ContosoHelpdeskChatBot-V4NetFramework/ContosoHelpdeskChatBot/Dialogs/ResetPasswordDialog.cs?range=27-45)]

1. `sendPassCode` 主要留作練習。 已將原始程式碼註解排除，而此方法只會傳回 true。 此外，原始 Bot 中並未使用電子郵件地址，所以可再次予以移除。  
    [!code-csharp[SendPassCode](~/../botbuilder-samples/MigrationV3V4/CSharp/ContosoHelpdeskChatBot-V4NetFramework/ContosoHelpdeskChatBot/Dialogs/ResetPasswordDialog.cs?range=47-81)]

1. `BuildResetPasswordForm` 沒有任何變更。

1. 我們可以使用瀑布的第二個步驟取代 `ResumeAfterResetPasswordFormDialog`，並將從步驟內容中取得傳回值。 我們已移除原始對話未做任何處理的電子郵件地址，並提供了虛擬結果，而非查詢資料庫。 我們會保留相同的邏輯，只是將 v3 呼叫轉譯成 v4 對等項目。  
    [!code-csharp[ProcessRequestAsync](~/../botbuilder-samples/MigrationV3V4/CSharp/ContosoHelpdeskChatBot-V4NetFramework/ContosoHelpdeskChatBot/Dialogs/ResetPasswordDialog.cs?range=90-113)]

### <a name="update-models-as-necessary"></a>視需要更新模型

在某些參考 Formflow 程式庫的模型中，我們需要更新`using` 陳述式。

1. 在 `LocalAdminPrompt` 中，將它們變更如下：  
    [!code-csharp[Using statements](~/../botbuilder-samples/MigrationV3V4/CSharp/ContosoHelpdeskChatBot-V4NetFramework/ContosoHelpdeskChatBot/Models/LocalAdminPrompt.cs?range=4)]

1. 在 `ResetPasswordPrompt` 中，將它們變更如下：  
    [!code-csharp[Using statements](~/../botbuilder-samples/MigrationV3V4/CSharp/ContosoHelpdeskChatBot-V4NetFramework/ContosoHelpdeskChatBot/Models/ResetPasswordPrompt.cs?range=4-5)]

## <a name="update-webconfig"></a>更新 Web.config

註解排除 **MicrosoftAppId** 和 **MicrosoftAppPassword** 的組態金鑰。 這可讓您在本機進行 Bot 偵錯，而不需將這些值提供給模擬器。

## <a name="run-and-test-your-bot-in-the-emulator"></a>在模擬器中執行並測試您的 Bot

此時，我們應該能夠在 IIS 中本機執行 Bot，然後將它與模擬器連結。

1. 在 IIS 中執行 Bot。
1. 啟動模擬器，然後連線到 Bot 的端點 (例如： **http://localhost:3978/api/messages** )。
    - 如果這是您第一次執行 Bot，請按一下 [檔案] > [新的 Bot]  ，並遵循畫面上的指示。 否則，請按一下 [檔案] > [開啟 Bot]  以開啟現有的 Bot。
    - 在組態中再次檢查連接埠設定。 例如，如果 Bot 在瀏覽器中開啟至 `http://localhost:3979/`，則在模擬器中，將 Bot 的端點設定為 `http://localhost:3979/api/messages`。
1. 四個對話應該都可以運作，而且您可以在瀑布步驟中設定中斷點，以檢查哪個對話內容和對話狀態是在這些點上。

## <a name="additional-resources"></a>其他資源

v4 概念性主題：

- [Bot 的運作方式](../bot-builder-basics.md)
- [管理狀態](../bot-builder-concept-state.md)
- [對話方塊程式庫](../bot-builder-concept-dialog.md)

v4 作法主題：

- [傳送及接收文字訊息](../bot-builder-howto-send-messages.md)
- [儲存使用者和對話資料](../bot-builder-howto-v4-state.md)
- [實作循序對話流程](../bot-builder-dialog-manage-conversation-flow.md)
- [使用模擬器進行偵錯](../../bot-service-debug-emulator.md)
- [將遙測資料新增至 Bot](../bot-builder-telemetry.md)
