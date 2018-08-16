---
title: 儲存狀態和存取資料 | Microsoft Docs
description: 描述 Bot Builder SDK 中有哪些狀態管理員、對話狀態和使用者狀態。
keywords: LUIS, 對話狀態, 使用者狀態, 儲存體, 管理狀態
author: DeniseMak
ms.author: v-demak
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 02/15/2018
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 56814ab12a85d18e52b0d5ec83fd81682f3b9f60
ms.sourcegitcommit: f95702d27abbd242c902eeb218d55a72df56ce56
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/19/2018
ms.locfileid: "39300578"
---
# <a name="save-state-and-access-data"></a>儲存狀態和存取資料
[!INCLUDE [pre-release-label](../includes/pre-release-label.md)]

Bot 的設計是否精良，關鍵之一是必須能夠追蹤交談的上下文，讓 Bot 記住先前問題的答案這類事項。
根據 Bot 的用途，您甚至需要持續追蹤狀態，或將資料儲存超過對話存留期的時間。
Bot 必須記住*狀態*的資訊才能適當回應內送的訊息。 Bot Builder SDK 會提供類別，以便用將狀態資料儲存和擷取為與使用者或對話相關聯的物件。

* **對話屬性**可協助您的 Bot 追蹤目前 Bot 與使用者進行中的對話。 如果您的 Bot 需要完成一系列的步驟或在對話主題之間進行切換，您可以使用對話屬性來按照順序管理步驟或追蹤目前的主題。 由於對話屬性會反映目前對話的狀態，您通常會在工作階段的結尾在 Bot 收到_結束對話_活動時，清除對話屬性。
* **使用者屬性**有許多用途，例如判斷使用者先前離開對話的位置，或單純地以名字來問候回來的使用者。 如果您儲存使用者的喜好設定，下次聊天時就可以使用該資訊來自訂對話。 比方說，您可能會在有使用者感興趣的新聞文章，或有可用約會時通知使用者。 如果 Bot 收到_刪除使用者資料_活動，請進行清除。

您可以使用[儲存體](bot-builder-howto-v4-storage.md)來讀取和寫入永續性儲存體。 這可讓您的 Bot 執行動作，例如更新共用資源、記錄出席回函或投票，或讀取歷史氣象資料。 您可以使用與應用程式相同的方式，讓 Bot 在與使用者的對話中利用儲存體來達成目標。

<!-- 
*Conversation state* pertains to the current conversation that the user is having with your bot. When the conversation ends, your bot deletes this data.

You can also store *user state* that persists after a conversation ends. For example, if you store a user's preferences, you can use that information to customize the conversation the next time you chat. For example, you might alert the user to a news article about a topic that interests her, or alert a user when an appointment becomes available. 
-->

<!-- You should generally avoid saving state using a global variable or function closures.
Doing so will create issues when you want to scale out your bot. Instead, use the conversation state and user state middleware that the BotBuilder SDK provides --> 


## <a name="types-of-underlying-storage"></a>基礎儲存體類型

SDK 會提供 Bot 狀態管理員中介軟體來保存對話和使用者狀態。 您可以使用 Bot 的內容來存取狀態。 此狀態管理員可以使用 Azure 資料表儲存體、檔案儲存體或記憶體儲存體作為基礎資料儲存體。 您也可以為 Bot 建立自己的儲存體元件。

您可以將使用 Azure 資料表儲存體建置的 Bot 設計為無狀態，且可在多個計算節點進行調整。

> [!NOTE] 
> 檔案和記憶體的儲存體不會跨節點調整。

## <a name="writing-directly-to-storage"></a>直接寫入儲存體

您也可以使用 Bot Builder SDK 在儲存體中直接讀取和寫入資料，而不使用中介軟體或 Bot 內容。 這種方式適用於來自 Bot 對話流程以外來源的 Bot 使用者。

例如，假設您的 Bot 允許使用者要求取得氣象報告，且 Bot 會透過外部資料庫讀取氣象報告，以擷取特定日期的報告內容。 氣象資料庫的內容不需依賴使用者的資訊或對話內容，因此您可以直接在儲存體中讀取，不需使用狀態管理員。  請參閱[如何直接寫入到儲存體](bot-builder-howto-v4-storage.md)的範例。

## <a name="next-steps"></a>後續步驟

接下來，請深入了解活動的處理方式，以及我們回應的方式。

> [!div class="nextstepaction"]
> [活動處理](bot-builder-concept-activity-processing.md)

## <a name="additional-resources"></a>其他資源

- [如何儲存狀態](bot-builder-howto-v4-state.md)
- [如何直接寫入到儲存體](bot-builder-howto-v4-storage.md)
