---
title: 將 Bot 升級為 Bot Framework API 第 3 版 |Microsoft Docs
description: 了解如何將 Bot 升級為 Bot Framework API 第 3 版。
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 12/13/2017
ms.openlocfilehash: ca54abf89f8967b2b109895326d13a601030fdec
ms.sourcegitcommit: f576981342fb3361216675815714e24281e20ddf
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/18/2018
ms.locfileid: "39299402"
---
# <a name="upgrade-your-bot-to-bot-framework-api-v3"></a>將 Bot 升級為 Bot Framework API 第 3 版

在 Build 2016，Microsoft 宣告了 Microsoft Bot Framework 與其 Bot 連接器 API 的初始反覆項目，以及 Bot 建立器和 Bot 連接器 SDK。 從那時開始，我們一直在收集您的意見反應，並積極致力於改善 REST API 和 SDK。

在 2016 年 7 月，已發行 Bot Framework API 第 3 版，並淘汰了 Bot Framework API 第 1 版。 使用 API 第 1 版的 Bot 已於 2016 年 12 月停止在 Skype 上運作，並於 2017 年 2 月 23 日停止在所有其餘的通道上運作。 如果您已建立使用 API 第 1 版的 Bot，並且想要使其再次運作，您必須遵循本文中的指示將它升級為 API 第 3 版。 若要確保您從頭到尾了解升級程序，請在開始之前完整閱讀本文。 

## <a name="step-1-get-your-app-id-and-password-from-the-bot-framework-portal"></a>步驟 1：從 Bot Framework 入口網站取得您的應用程式識別碼和密碼

登入 [Bot Framework 入口網站](https://dev.botframework.com/)，按一下 [My bots] \(我的 Bot\)，然後選取您的 Bot 以開啟其儀表板。 接下來，按一下位於頁面右上角附近的 [SETTINGS] \(設定\) 連結。 

在設定頁面的 [Configuration] \(組態\) 區段內，檢查 [App ID] \(應用程式識別碼\) 欄位的內容，然後根據 [App ID] \(應用程式識別碼\) 欄位是否已填入資料，繼續進行接下來的步驟。

### <a name="case-1-app-id-field-is-already-populated"></a>案例 1：已填入 [App ID] \(應用程式識別碼\) 欄位

如果已填入 [App ID] \(應用程式識別碼\) 欄位，請完成下列步驟：

1. 按一下 [Manage Microsoft App ID and password] \(管理 Microsoft 應用程式識別碼和密碼\)。  
![組態](~/media/upgrade/manage-app-id.png)

2. 按一下 [Generate New Password] \(產生新密碼\)。  
![產生新密碼](~/media/upgrade/generate-new-password.png)

3. 複製並儲存新密碼以及 MSA 應用程式識別碼；您在未來將需要這些值。  
![新密碼](~/media/upgrade/new-password-generated.png)

### <a name="case-2-app-id-field-is-empty"></a>案例 2：[App ID] \(應用程式識別碼\) 欄位是空的

如果 [App ID] \(應用程式識別碼\) 欄位是空的，請完成下列步驟：

1. 按一下 [Create Microsoft App ID and password] \(建立 Microsoft 應用程式識別碼和密碼\)。  
   ![建立應用程式識別碼和密碼](~/media/upgrade/generate-appid-and-password.png)
   > [!IMPORTANT]
   > 尚未選取 [Version 3.0] \(3.0 版\) 選項按鈕。 在[更新 Bot 程式碼](#update-code)之後，您將於稍後執行此動作。</div>

2. 按一下 [Generate a password to continue] \(產生密碼以繼續\)。  
   ![產生應用程式密碼](~/media/upgrade/generate-a-password-to-continue.png)

3. 複製並儲存新密碼以及 MSA 應用程式識別碼；您在未來將需要這些值。  
   ![新密碼](~/media/upgrade/new-password-generated.png)

4. 按一下 [Finish and go back to Bot Framework] \(完成並返回 Bot Framework\)。  
   ![完成並返回 Bot Framework](~/media/upgrade/finish-and-go-back-to-bot-framework.png)

5. 返回 Bot Framework 入口網站中的 Bot 設定頁面，捲動到頁面底部，然後按一下 [Save changes] \(儲存變更\)。  
   ![儲存變更](~/media/upgrade/save-changes.png)

## <a id="update-code"></a> 步驟 2：將 Bot 程式碼更新為 3.0 版

若要將 Bot 程式碼更新為 3.0 版，請完成下列步驟：

1. 針對 Bot 的語言更新為最新版的 [Bot 建立器 SDK](https://github.com/Microsoft/BotBuilder)。
2. 根據下列指導方針，更新您的程式碼以套用必要的變更。
3. 使用 [Bot Framework 模擬器](~/bot-service-debug-emulator.md)在本機上測試 Bot，然後在雲端中測試 Bot。

下列各節描述 API 第 1 版與 API 第 3 版之間的主要差異。 在程式碼更新為 API 第 3 版之後，您可以在 Bot Framework 入口網站中[更新 Bot 設定](#step-3)來完成升級程序。

### <a name="botbuilder-and-connector-are-now-one-sdk"></a>Bot 建立器和連接器現在是一個 SDK

您現在可以在單一 Bot 建立器 SDK 中取得這兩個程式庫，而不需要使用多個 NuGet 套件 (或 NPM 模組) 為建立器和連接器安裝個別的 SDK：

- 適用於 .NET 的 Bot 建立器 SDK：`Microsoft.Bot.Builder` NuGet 套件
- 適用於 Node.js 的 Bot 建立器 SDK：`botbuilder` NPM 模組

獨立的 `Microsoft.Bot.Connector` SDK 現在已淘汰，不再進行維護。

### <a name="message-is-now-activity"></a>Message 現在是 Activity

在 API 第 3 版中，`Message` 物件已取代為 `Activity` 物件。 活動的最常見的類型是**訊息**，但是還有其他活動類型可用來將各種類型的資訊傳達給 Bot 或通道。 如需訊息的詳細資訊，請參閱[建立訊息](~/dotnet/bot-builder-dotnet-create-messages.md)和[傳送及接收活動](~/dotnet/bot-builder-dotnet-connector.md)。

### <a name="activity-types--events"></a>活動類型和事件

某些事件已在 API 第 3 版中重新命名及/或重構。 此外，已在連接器中新增 `ActivityTypes` 列舉，因此不需要記住特定的活動類型。

- `conversationUpdate` 活動類型會以單一方法取代在交談中新增/移除的 Bot 使用者。
- 新的 `typing` 活動類型可讓 Bot 指出它正在編譯回應，以及知道使用者在何時鍵入回應。
- 新的 `contactRelationUpdate` 活動類型可讓 Bot 知道已在使用者的連絡人清單中新增還是移除該 Bot。

當 Bot 接收到 `conversationUpdate` 活動時，`MembersRemoved` 屬性和 `MembersAdded` 屬性會指出在交談中新增或移除的人員。 當 Bot 接收到 `contactRelationUpdate` 活動時，`Action` 屬性會指出使用者已在其連絡人清單中新增 Bot 還是移除 Bot。 如需活動類型的詳細資訊，請參閱[活動概觀](~/dotnet/bot-builder-dotnet-activities.md)。

### <a name="addressing-messages"></a>為訊息定址

在訊息內指定傳送者、接收者和通道資訊的位置在 API 第 3 版中略有變更：

|API 第 1 版欄位 | API 第 3 版欄位|
|--------|--------|
| `From` 物件 | `From` 物件 |
| `To` 物件 | `Recipient` 物件 |
| `ChannelConversationID` 屬性 | `Conversation` 物件|
| `ChannelId` 屬性 | `ChannelId` 屬性 |

如需為訊息定址的詳細資訊，請參閱[傳送及接收活動](~/dotnet/bot-builder-dotnet-connector.md)。

### <a name="sending-replies"></a>傳送回覆

在 Bot Framework API 第 3 版中，使用者的所有回覆都會透過個別初始化的 HTTP 要求以非同步方式傳送，而不是針對 Bot 的傳入訊息使用 HTTP POST 以內嵌方式傳送。 由於沒有任何訊息會透過連接器以內嵌方式傳回給使用者，所以 Bot 之 Post 方法的傳回型別會是 `HttpResponseMessage`。 這表示 Bot 不會同步「傳回」您想要傳送給使用者的字串，但卻改為在程式碼的任何位置傳送回覆訊息，而不需要作為傳入 POST 的回應傳回。 這兩種方法都會傳送一則訊息給交談：

- `SendToConversation`
- `ReplyToConversation`

`SendToConversation`　方法會將指定的訊息附加到交談的結尾，而 `ReplyToConversation`方法 (適用於支援它的交談) 則會將指定的訊息作為交談中先前訊息的直接回覆。 如需這些方法的詳細資訊，請參閱[傳送及接收活動](~/dotnet/bot-builder-dotnet-connector.md)。

### <a name="starting-conversations"></a>開始交談

在 Bot Framework API 第 3 版中，您可以透過下列方式開始交談：使用新方法 `CreateDirectConversation` 開始與單一使用者進行私人交談，或使用新方法 `CreateConversation` 開始與多個使用者進行群組交談。 如需開始交談的詳細資訊，請參閱[傳送及接收活動](~/dotnet/bot-builder-dotnet-connector.md#start-a-conversation)。

### <a name="attachments-and-options"></a>附件和選項

Bot Framework API 第 3 版引入了更健全的附件和卡片實作。 API 第 3 版不再支援 `Options` 類型，而是改由卡片取代。 如需如何使用 .NET 將附件新增至訊息的詳細資訊，請參閱[將媒體附件新增至訊息](~/dotnet/bot-builder-dotnet-add-media-attachments.md)和[將豐富的卡片附件新增至訊息](~/dotnet/bot-builder-dotnet-add-rich-card-attachments.md)。

### <a name="bot-data-storage-bot-state"></a>Bot 資料儲存體 (Bot 狀態)

在 Bot Framework API 第 1 版中，用於管理 Bot 狀態資料的 API 包含在傳訊 API 中。 而在 Bot Framework API 第 3 版中，這些 API 是分開的。 現在，您必須使用 Bot 狀態服務來取得狀態資料 (而不是假設它將包含在 `Message` 物件內)，以及儲存狀態資料 (而不是作為 `Message` 物件的一部分傳遞)。 如需使用 Bot 狀態服務管理 Bot 狀態資料的資訊，請參閱[管理狀態資料](~/dotnet/bot-builder-dotnet-state.md)。

> [!IMPORTANT]
> 建議您不要將 Bot Framework State Service API 用於生產環境，未來版本可能會將其淘汰。 建議更新 Bot 程式碼以使用記憶體內部儲存體進行測試，或將任何一個 **Azure 延伸模組**用於生產 Bot。 如需詳細資訊，請參閱**管理狀態資料**主題，以了解 [.NET](~/dotnet/bot-builder-dotnet-state.md) 或[節點](~/nodejs/bot-builder-nodejs-state.md)實作。

### <a name="webconfig-changes"></a>Web.config 變更

Bot Framework API 第 1 版已使用在 **Web.Config** 中的下列索引鍵來儲存驗證屬性：

- `AppID`
- `AppSecret`

Bot Framework API 第 3 版則會使用在 **Web.Config** 中的下列索引鍵來儲存驗證屬性：

- `MicrosoftAppID`
- `MicrosoftAppPassword`

## <a id="step-3"></a> 步驟 3：在 Bot Framework 入口網站中更新 Bot 設定

在您將 Bot 程式碼升級為 API 第 3 版並將它部署到雲端之後，請完成下列步驟來完成升級程序： 

1. 登入 [Bot Framework 入口網站](https://dev.botframework.com/)。

2. 按一下 [My bots] \(我的機器人\)，然後選取 Bot 以開啟其儀表板。 

3. 按一下位於頁面右上角附近的 [SETTINGS] \(設定\) 連結。 

4. 在 [Configuration] \(組態\) 區段內的 [Version 3.0] \(3.0 版\) 底下，將 Bot 端點貼入 [Messaging endpoint] \(傳訊端點\) 欄位。  
![第 3 版組態](~/media/upgrade/paste-new-v3-enpoint-url.png)

5. 選取 [Version 3.0] \(3.0 版\) 選項按鈕。  
![選取 3.0 版](~/media/upgrade/switch-to-v3-endpoint.png)

6. 捲動到頁面底部，然後按一下 [Save changes] \(儲存變更\)。  
![儲存變更](~/media/upgrade/save-changes.png)