---
title: 儲存使用者和對話資料 | Microsoft Docs
description: 了解如何透過 Bot Framework SDK 儲存及擷取資料。
keywords: 交談狀態, 使用者狀態, 交談, 儲存狀態, 管理 Bot 狀態
author: ivorb
ms.author: v-ivorb
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 4/16/19
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 38a8034687ef1a0b8b3bcf3e01d3b33b91bdfd18
ms.sourcegitcommit: f84b56beecd41debe6baf056e98332f20b646bda
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 05/03/2019
ms.locfileid: "65033221"
---
# <a name="save-user-and-conversation-data"></a>儲存使用者和對話資料

[!INCLUDE[applies-to](../includes/applies-to.md)]

Bot 原本就是無狀態。 部署 Bot 後，在不同的回合中，Bot 便無法在相同程序中或同部電腦上執行。 不過，Bot 可能需要追蹤對話的內容，以便管理其行為，以及記住先前問題的答案。 Bot Framework SDK 的狀態和儲存體功能可讓您將狀態新增至 Bot。 Bot 會使用狀態管理和儲存體物件來管理和保存狀態。 狀態管理員會提供抽象層，讓您使用屬性存取子存取狀態屬性，而不受基礎儲存體類型所影響。

## <a name="prerequisites"></a>必要條件
- 需具備 [Bot 基本概念](bot-builder-basics.md)以及 Bot 如何[管理狀態](bot-builder-concept-state.md)的知識。
- 本文中的程式碼是以**狀態管理 Bot 範例**為基礎。 您需要 [CSharp](https://aka.ms/statebot-sample-cs) 或 [JavaScript](https://aka.ms/statebot-sample-js) 中的一份範例。

## <a name="about-this-sample"></a>關於此範例
收到使用者輸入後，此範例會檢查儲存的對話狀態，以查看之前是否已提示此使用者提供名稱。 如果沒有，則會要求使用者的名稱，並將該輸入儲存在使用者狀態中。 如果有，則使用者狀態中儲存的名稱就會用來與使用者對話，而其輸入的資料 (包括接收時間與輸入通道識別碼) 會傳回給使用者。 時間和通道識別碼值會從使用者對話資料中擷取，然後再儲存到對話狀態。 下圖顯示 Bot、使用者設定檔及對話資料類別之間的關聯性。

## <a name="ctabcsharp"></a>[C#](#tab/csharp)
![狀態 Bot 範例](media/StateBotSample-Overview.png)

## <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)
![狀態 Bot 範例](media/StateBotSample-JS-Overview.png)

---

## <a name="define-classes"></a>定義類別

## <a name="ctabcsharp"></a>[C#](#tab/csharp)

設定狀態管理的第一步是定義類別，以包含我們要在使用者和對話狀態中管理的所有資訊。 在此範例中，我們已定義下列項目：

- 在 **UserProfile.cs** 中，我們針對 Bot 要收集的使用者資訊定義了 `UserProfile` 類別。 
- 在 **ConversationData.cs** 中，我們定義了 `ConversationData` 類別來控制收集使用者資訊時的對話狀態。

下列程式碼範例示範如何建立 UserProfile 類別的定義。

**UserProfile.cs** [!code-csharp[UserProfile](~/../BotBuilder-Samples/samples/csharp_dotnetcore/45.state-management/UserProfile.cs?range=7-11)]

## <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

第一個步驟需要包含 `UserState` 和 `ConversationState` 定義的 BotBuilder 服務。

**index.js** [!code-javascript[BotService](~/../BotBuilder-Samples/samples/javascript_nodejs/45.state-management/index.js?range=7-9)]

---

## <a name="create-conversation-and-user-state-objects"></a>建立對話和使用者狀態物件

## <a name="ctabcsharp"></a>[C#](#tab/csharp)

接下來，我們會註冊用來建立 `UserState` 和 `ConversationState` 物件的 `MemoryStorage`。 使用者和對話狀態物件會在 `Startup` 上建立，而相依性會插入 Bot 建構函式。 為 Bot 註冊的其他服務為：認證提供者、配接器及 Bot 實作。

**Startup.cs** [!code-csharp[ConfigureServices](~/../BotBuilder-Samples/samples/csharp_dotnetcore/45.state-management/Startup.cs?range=16-38)]

**StateManagementBot.cs** [!code-csharp[StateManagement](~/../BotBuilder-Samples/samples/csharp_dotnetcore/45.state-management/bots/StateManagementBot.cs?range=15-22)]

## <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

接下來，我們會註冊用來建立 `UserState` 和 `ConversationState` 物件的 `MemoryStorage`。

**index.js** [!code-javascript[DefineMemoryStore](~/../BotBuilder-Samples/samples/javascript_nodejs/45.state-management/index.js?range=32-38)]

---

## <a name="add-state-property-accessors"></a>新增狀態屬性存取子

## <a name="ctabcsharp"></a>[C#](#tab/csharp)

現在我們要使用可提供控制代碼給 `BotState` 物件的 `CreateProperty` 方法建立屬性存取子。 每個狀態屬性存取子，都可讓您取得或設定相關聯的狀態屬性值。 在我們使用狀態屬性之前，我們會使用每個存取子從儲存體載入屬性，並從狀態快取中取得該屬性。 為取得狀態屬性相關金鑰的正確範圍，我們會呼叫 `GetAsync` 方法。

**StateManagementBot.cs** [!code-csharp[StateAccessors](~/../BotBuilder-Samples/samples/csharp_dotnetcore/45.state-management/bots/StateManagementBot.cs?range=38-46)]

## <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

現在我們要建立 `UserState` 和 `ConversationState` 的屬性存取子。 每個狀態屬性存取子，都可讓您取得或設定相關聯的狀態屬性值。 我們使用每個存取子從儲存體載入相關聯的屬性，並從快取中擷取其目前的狀態。

**StateManagementBot.js**。
[!code-javascript[BotService](~/../BotBuilder-Samples/samples/javascript_nodejs/45.state-management/bots/stateManagementBot.js?range=6-19)]

---

## <a name="access-state-from-your-bot"></a>從 Bot 存取狀態
前一節討論了初始階段步驟，以將狀態屬性存取子新增至 Bot。 現在，我們可以在執行階段使用這些存取子，來讀取和寫入狀態資訊。 以下範例程式碼會使用下列邏輯流程：
- 如果 userProfile.Name 為空白，且 conversationData.PromptedUserForName 為 _true_，我們就會擷取所提供的使用者名稱，並將其儲存在使用者狀態中。
- 如果 userProfile.Name 為空白，且 conversationData.PromptedUserForName 為 _false_，我們就會要求使用者的名稱。
- 如果先前已儲存 userProfile.Name，則我們會從使用者輸入中擷取訊息時間和通道識別碼，將所有資料回應給使用者，並將擷取的資料儲存在對話狀態中。

## <a name="ctabcsharp"></a>[C#](#tab/csharp)

**StateManagementBot.cs** [!code-csharp[OnMessageActivityAsync](~/../BotBuilder-Samples/samples/csharp_dotnetcore/45.state-management/bots/StateManagementBot.cs?range=38-85)]

結束回合處理常式之前，我們會使用狀態管理物件的 SaveChangesAsync() 方法，將所有狀態變更寫回儲存體。

**StateManagementBot.cs** [!code-csharp[OnTurnAsync](~/../BotBuilder-Samples/samples/csharp_dotnetcore/45.state-management/bots/StateManagementBot.cs?range=24-31)]

## <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

**StateManagementBot.js** [!code-javascript[OnMessage](~/../BotBuilder-Samples/samples/javascript_nodejs/45.state-management/bots/stateManagementBot.js?range=21-54)]

結束每個對話方塊回合之前，我們會使用狀態管理物件的 _saveChanges()_ 方法，藉由將狀態寫回儲存體來保留所有變更。

**StateManagementBot.js** [!code-javascript[OnDialog](~/../BotBuilder-Samples/samples/javascript_nodejs/45.state-management/bots/stateManagementBot.js?range=60-67)]

---

## <a name="test-the-bot"></a>測試 Bot

下載並安裝最新版 [Bot Framework Emulator](https://aka.ms/bot-framework-emulator-readme)

1. 在您的電腦本機執行範例。 如需相關指示，請參閱 [C# 範例](https://aka.ms/statebot-sample-cs)或 [JS 範例](https://aka.ms/statebot-sample-js)的讀我檔案。
1. 使用模擬器來測試 Bot，如下所示。

![測試狀態 Bot 範例](media/state-bot-testing-emulator.png)

## <a name="additional-resources"></a>其他資源

**隱私權：** 如果您想要儲存使用者的個人資料，請務必符合[一般資料保護規範](https://blog.botframework.com/2018/04/23/general-data-protection-regulation-gdpr)。

**狀態管理：** 所有狀態管理呼叫都是非同步的，預設會使用「最後寫入為準」。 實務上，您應該在 Bot 中儘可能密集取得、設定及儲存狀態。

**重要商務資料：** 使用 Bot 狀態來儲存喜好設定、使用者名稱或其所排序的最後一個項目，但不要來儲存重要的商務資料。 對於重要資料，請[建立自己的儲存體元件](bot-builder-custom-storage.md)，或直接寫入[儲存體](bot-builder-howto-v4-storage.md)。

**Recognizer-Text：** 此範例會使用 Microsoft/Recognizers-Text 程式庫來剖析及驗證使用者輸入。 如需詳細資訊，請參閱[概觀](https://github.com/Microsoft/Recognizers-Text#microsoft-recognizers-text-overview)頁面。

## <a name="next-steps"></a>後續步驟

您現在已知道如何設定狀態來協助您讀取 Bot 資料以及將其寫入儲存體，讓我們說明如何詢問使用者一連串的問題、驗證其答案，以及儲存其輸入。

> [!div class="nextstepaction"]
> [建立您自己的提示，以收集使用者輸入](bot-builder-primitive-prompts.md)。
