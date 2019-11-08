---
title: 適用於 Microsoft Teams 的 Bot 如何運作
description: 本文接續 Microsoft Teams 專屬 Bot 如何運作的文章
author: WashingtonKayaker
ms.author: kamrani
manager: kamrani
ms.topic: overview
ms.service: bot-service
ms.date: 10/31/2019
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 3a7e60e1fa72402b5f783676f5281579a2be5371
ms.sourcegitcommit: 490810d278d1c8207330b132f28a5eaf2b37bd07
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 11/05/2019
ms.locfileid: "73592201"
---
# <a name="how-microsoft-teams-bots-work"></a>Microsoft Teams Bot 的運作方式

此簡介會以 [Bot 運作方式](https://docs.microsoft.com/azure/bot-service/bot-builder-basics)文章中所討論的內容為基礎，您應該先熟悉該文章後，再閱讀此簡介。

在專為 Microsoft Teams 開發的 Bot 之間，主要差異在於活動的處理方式。 Microsoft Teams 活動處理常式是從 Bot Framework 的活動處理常式衍生而來，該處理常式會路由所有小組活動，然後才允許處理任何非小組專屬的活動。

## <a name="teams-activity-handlers"></a>小組活動處理常式

就像任何其他 Bot 一樣，當專為 Microsoft Teams 所設計的 Bot 收到活動時，該 Bot 就會將活動傳遞至其*活動處理常式*。 實際上，所有活動都會透過稱為*回合處理常式*的基底處理常式來路由。 *回合處理常式*會呼叫所需的活動處理常式，以處理所收到的任何活動類型。 專為 Microsoft Teams 所設計的 Bot 因為衍生自 _Teams_ `Activity Handler` 類別而有所不同，而此類別則是衍生自 Bot Framework 的 `Activity Handler` 類別。  _Teams_ `Activity Handler` 類別包含將在本文中討論的各種 Microsoft Teams 專屬活動處理常式。

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

如同使用 Microsoft Bot Framework 建立的任何 Bot，如果 Bot 收到訊息活動，則回合處理常式會看見該傳入活動並將其傳送給 `OnMessageActivityAsync` 活動處理常式。 此功能維持不變，不過，如果 Bot 收到對話更新活動，則回合處理常式會看到傳入活動，並將其傳送至 `OnConversationUpdateActivityAsync` _Teams_ 活動處理常式，以先檢查是否有任何 Teams 專屬的事件，然後將其傳遞至 Bot Framework 的活動處理常式 (如果找不到的話)。

在 Teams 活動處理常式類別中，有兩個主要的 Teams 活動處理常式：`OnConversationUpdateActivityAsync` 會路由所有對話更新活動，而 `OnInvokeActivityAsync` 會路由所有 Teams 叫用活動。  `How bots work` 文章中 [Bot 邏輯](https://aka.ms/how-bots-work#bot-logic)章節所述的所有活動處理常式，都會繼續以非 Teams Bot 的運作方式來運作，但處理成員新增和移除活動的方式除外，這些處理方式在小組情況中會有所不同，因為新成員會加入至小組，而不是訊息執行緒。  如需詳細資訊，請參閱 [Bot 邏輯](#bot-logic)一節中的＜Teams 對話更新活動＞  資料表。

若要針對這些 Teams 專屬活動處理常式實作您的邏輯，您將會覆寫 Bot 中的這些方法，如下面 [Bot 邏輯](#bot-logic)一節所示。 上述每個處理常式都沒有基底實作，所以只要新增您想要的覆寫邏輯即可。

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

如同使用 Microsoft Bot Framework 建立的任何 Bot，如果 Bot 收到訊息活動，則回合處理常式會看見該傳入活動並將其傳送給 `onMessage` 活動處理常式。 此功能維持不變，不過，如果 Bot 收到對話更新活動，則回合處理常式會看到傳入活動，並將其傳送至 `dispatchConversationUpdateActivity` Teams  活動處理常式，以先檢查是否有任何 Teams 專屬的事件，然後將其傳遞至 Bot Framework 的活動處理常式 (如果找不到的話)。

在 Teams 活動處理常式類別中，有兩個主要的 Teams 活動處理常式：`dispatchConversationUpdateActivity` 會路由所有對話更新活動，而 `onInvokeActivity` 會路由所有 Teams 叫用活動。  `How bots work` 文章中 [Bot 邏輯](https://aka.ms/how-bots-work#bot-logic)章節所述的所有活動處理常式，都會繼續以非 Teams Bot 的運作方式來運作，但處理成員新增和移除活動的方式除外，這些處理方式在小組情況中會有所不同，因為新成員會加入至小組，而不是訊息執行緒。  如需詳細資訊，請參閱 [Bot 邏輯](#bot-logic)一節中的＜Teams 對話更新活動＞  資料表。

如同任何 Bot 處理常式，若要針對 Teams 處理常式實作您的邏輯，您將會覆寫 Bot 中的這些方法，如下面 [Bot 邏輯](#bot-logic)一節所示。 針對每個處理常式，定義您的 Bot 邏輯，然後**務必在結束時呼叫 `next()`** 。 呼叫 `next()`，確定下一個處理常式已執行。

---

### <a name="bot-logic"></a>Bot 邏輯

Bot 邏輯會處理來自從您一個或多個 Bot 通道的傳入活動，並產生傳出活動來回應。  這對於衍生自 Teams 活動處理常式類別的 Bot 仍然成立，該 Bot 會先檢查 Teams 活動，然後將所有其他活動傳遞至 Bot Framework 的活動處理常式。

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

#### <a name="teams-conversation-update-activities"></a>Teams 對話更新活動

以下是從 `OnConversationUpdateActivityAsync` Teams  活動處理常式呼叫的所有 Teams 活動處理常式清單。 [對話更新事件](https://aka.ms/azure-bot-subscribe-to-conversation-events)文章會說明如何在 Bot 中使用其中每一個事件。

| 事件 | 處理常式 | 說明 |
| :-- | :-- | :-- |
| channelCreated | `OnTeamsChannelCreatedAsync` | 覆寫此項目以處理即將建立的 Teams 通道。 如需詳細資訊，請參閱[通道已建立](https://aka.ms/azure-bot-subscribe-to-conversation-events#channel-created)。 |
| channelDeleted | `OnTeamsChannelDeletedAsync` | 覆寫此項目以處理即將刪除的 Teams 通道。 如需詳細資訊，請參閱[通道已刪除](https://aka.ms/azure-bot-subscribe-to-conversation-events#channel-deleted)。 |
| channelRenamed | `OnTeamsChannelRenamedAsync` | 覆寫此項目以處理即將重新命名的 Teams 通道。 如需詳細資訊，請參閱[通道已重新命名](https://aka.ms/azure-bot-subscribe-to-conversation-events#channel-renamed)。 |
| teamRenamed | `OnTeamsTeamRenamedAsync` | `return Task.CompletedTask;` 覆寫此項目以處理即將重新命名的 Teams 小組。 如需詳細資訊，請參閱[小組已重新命名](https://aka.ms/azure-bot-subscribe-to-conversation-events#team-renamed)。 |
| MembersAdded | `OnTeamsMembersAddedAsync` | 呼叫 `ActivityHandler` 中的 `OnMembersAddedAsync` 方法。 覆寫此項目以處理要加入小組的成員。 如需詳細資訊，請參閱[小組成員已新增](https://aka.ms/azure-bot-subscribe-to-conversation-events#Team-Member-Added)。|
| MembersRemoved | `OnTeamsMembersRemovedAsync` | 呼叫 `ActivityHandler` 中的 `OnMembersRemovedAsync` 方法。 覆寫此項目以處理要離開小組的成員。 如需詳細資訊，請參閱[小組成員已移除](https://aka.ms/azure-bot-subscribe-to-conversation-events#Team-Member-Removed)。|


<!--
| Event | Handler | Description |
| :-- | :-- | :-- |
| channelCreated | `OnTeamsChannelCreatedAsync` | Override this to handle a Teams channel being created. |
| channelDeleted | `OnTeamsChannelDeletedAsync` | Override this to handle a Teams channel being deleted. |
| channelRenamed | `OnTeamsChannelRenamedAsync` | Override this to handle a Teams channel being renamed. |
| teamRenamed | `OnTeamsTeamRenamedAsync` | `return Task.CompletedTask;` Override this to handle a Teams Team being Renamed. |
| MembersAdded | `OnTeamsMembersAddedAsync` | Calls the `OnMembersAddedAsync` method in `ActivityHandler`. Override this to handle members joining a team. |
| MembersRemoved | `OnTeamsMembersRemovedAsync` | Calls the `OnMembersRemovedAsync` method in `ActivityHandler`. Override this to handle members leaving a team. |
-->

#### <a name="teams-invoke--activities"></a>Teams 叫用活動

以下是從 `OnInvokeActivityAsync` Teams  活動處理常式呼叫的所有 Teams 活動處理常式清單：

| 叫用類型                    | 處理常式                              | 說明                                                  |
| :-----------------------------  | :----------------------------------- | :----------------------------------------------------------- |
| CardAction.Invoke               | `OnTeamsCardActionInvokeAsync`       | Teams 卡片動作叫用。 |
| fileConsent/invoke              | `OnTeamsFileConsentAcceptAsync`      | Teams 檔案同意接受。 |
| fileConsent/invoke              | `OnTeamsFileConsentAsync`            | Teams 檔案同意。 |
| fileConsent/invoke              | `OnTeamsFileConsentDeclineAsync`     | Teams 檔案同意。 |
| actionableMessage/executeAction | `OnTeamsO365ConnectorCardActionAsync` | Teams O365 連接器卡動作。 |
| signin/verifyState              | `OnTeamsSigninVerifyStateAsync`      | Teams 登入驗證狀態。 |
| task/fetch                      | `OnTeamsTaskModuleFetchAsync`        | Teams 工作模組擷取。 |
| task/submit                     | `OnTeamsTaskModuleSubmitAsync`       | Teams 工作模組提交。 |

以上所列的叫用活動皆適用於 Teams 中的對話式 Bot。 Bot Framework SDK 也支援訊息延伸模組專屬的叫用。 如需詳細資訊，請參閱[什麼是訊息延伸模組](https://aka.ms/azure-bot-what-are-messaging-extensions)。

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

#### <a name="teams-conversation-update-activities"></a>Teams 對話更新活動

以下是從 `dispatchConversationUpdateActivity` Teams  活動處理常式呼叫的所有 Teams 活動處理常式清單。 [對話更新事件](https://aka.ms/azure-bot-subscribe-to-conversation-events)文章會說明如何在 Bot 中使用其中每一個事件。

| 事件 | 處理常式 | 說明 |
| :-- | :-- | :-- |
| channelCreated | `OnTeamsChannelCreatedEvent` | 覆寫此項目以處理即將建立的 Teams 通道。 如需詳細資訊，請參閱[通道已建立](https://aka.ms/azure-bot-subscribe-to-conversation-events#channel-created)。 |
| channelDeleted | `OnTeamsChannelDeletedEvent` | 覆寫此項目以處理即將刪除的 Teams 通道。 如需詳細資訊，請參閱[通道已刪除](https://aka.ms/azure-bot-subscribe-to-conversation-events#channel-deleted)。|
| channelRenamed | `OnTeamsChannelRenamedEvent` | 覆寫此項目以處理即將重新命名的 Teams 通道。 如需詳細資訊，請參閱[通道已重新命名](https://aka.ms/azure-bot-subscribe-to-conversation-events#channel-renamed)。 |
| teamRenamed | `OnTeamsTeamRenamedEvent` | `return Task.CompletedTask;` 覆寫此項目以處理即將重新命名的 Teams 小組。 如需詳細資訊，請參閱[小組已重新命名](https://aka.ms/azure-bot-subscribe-to-conversation-events#team-renamed)。 |
| MembersAdded | `OnTeamsMembersAddedEvent` | 呼叫 `ActivityHandler` 中的 `OnMembersAddedEvent` 方法。 覆寫此項目以處理要加入小組的成員。 如需詳細資訊，請參閱[小組成員已新增](https://aka.ms/azure-bot-subscribe-to-conversation-events#Team-Member-Added)。 |
| MembersRemoved | `OnTeamsMembersRemovedEvent` | 呼叫 `ActivityHandler` 中的 `OnMembersRemovedEvent` 方法。 覆寫此項目以處理要離開小組的成員。 如需詳細資訊，請參閱[小組成員已移除](https://aka.ms/azure-bot-subscribe-to-conversation-events#Team-Member-Removed)。 |

<!--
| Event | Handler | Description |
| :-- | :-- | :-- |
| channelCreated | `OnTeamsChannelCreatedEvent` | Override this to handle a Teams channel being created. |
| channelDeleted | `OnTeamsChannelDeletedEvent` | Override this to handle a Teams channel being deleted. |
| channelRenamed | `OnTeamsChannelRenamedEvent` | Override this to handle a Teams channel being renamed. |
| teamRenamed | `OnTeamsTeamRenamedEvent` | `return Task.CompletedTask;` Override this to handle a Teams Team being Renamed. |
| MembersAdded | `OnTeamsMembersAddedEvent` | Calls the `OnMembersAddedEvent` method in `ActivityHandler`. Override this to handle members joining a team. |
| MembersRemoved | `OnTeamsMembersRemovedEvent` | Calls the `OnMembersRemovedEvent` method in `ActivityHandler`. Override this to handle members leaving a team. |
-->

#### <a name="teams-invoke--activities"></a>Teams 叫用活動

以下是從 `onInvokeActivity` Teams  活動處理常式呼叫的所有 Teams 活動處理常式清單：

| 叫用類型                    | 處理常式                              | 說明                                                  |
| :-----------------------------  | :----------------------------------- | :----------------------------------------------------------- |
| CardAction.Invoke               | `handleTeamsCardActionInvoke`       | Teams 卡片動作叫用。 |
| fileConsent/invoke              | `handleTeamsFileConsentAccept`      | Teams 檔案同意接受。 |
| fileConsent/invoke              | `handleTeamsFileConsent`            | Teams 檔案同意。 |
| fileConsent/invoke              | `handleTeamsFileConsentDecline`     | Teams 檔案同意。 |
| actionableMessage/executeAction | `handleTeamsO365ConnectorCardAction` | Teams O365 連接器卡動作。 |
| signin/verifyState              | `handleTeamsSigninVerifyState`      | Teams 登入驗證狀態。 |
| task/fetch                      | `handleTeamsTaskModuleFetch`        | Teams 工作模組擷取。 |
| task/submit                     | `handleTeamsTaskModuleSubmit`       | Teams 工作模組提交。 |

以上所列的叫用活動皆適用於 Teams 中的對話式 Bot。 Bot Framework SDK 也支援訊息延伸模組專屬的叫用。 如需詳細資訊，請參閱[什麼是訊息延伸模組](https://aka.ms/azure-bot-what-are-messaging-extensions)。

---

## <a name="next-steps"></a>後續步驟
若要建立 Teams Bot，請參閱 Microsoft Teams 開發人員[文件](https://aka.ms/teams-docs)。