---
redirect_url: /bot-framework/bot-builder-howto-v4-state
ms.openlocfilehash: a0d2b1295be1271e827d617ad09878ee8cfcd356
ms.sourcegitcommit: b15cf37afc4f57d13ca6636d4227433809562f8b
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 01/11/2019
ms.locfileid: "54223913"
---
<a name="--"></a><!--
---
標題：狀態和儲存體 | Microsoft Docs 描述：描述 Bot Framework SDK 中有哪些狀態管理員、交談狀態和使用者狀態。
關鍵字：LUIS, 交談狀態, 使用者狀態, 儲存體, 管理狀態 作者：DeniseMak ms.author: v-demak manager: kamrani ms.topic: article ms.service: bot-service ms.subservice: sdk ms.date:02/15/2018 monikerRange: 'azure-bot-service-4.0'
---

# <a name="state-and-storage"></a>狀態和儲存體
[!INCLUDE [pre-release-label](../includes/pre-release-label.md)]

Bot 的設計是否精良，關鍵之一是必須能夠追蹤交談的上下文，讓 Bot 記住先前問題的答案這類事項。
根據 Bot 的用途，您甚至需要持續追蹤狀態，或將資料儲存超過對話存留期的時間。
Bot 必須記住*狀態*的資訊才能適當回應內送的訊息。 Bot Framework SDK 會提供類別，以便用將狀態資料儲存和擷取為與使用者或交談相關聯的物件。

* **對話屬性**可協助您的 Bot 追蹤目前 Bot 與使用者進行中的對話。 如果您的 Bot 需要完成一系列的步驟或在對話主題之間進行切換，您可以使用對話屬性來按照順序管理步驟或追蹤目前的主題。 由於對話屬性會反映目前對話的狀態，您通常會在對話結尾當 Bot 收到_結束對話_活動時，清除這些屬性。
* **使用者屬性**有許多用途，例如判斷使用者先前離開對話的位置，或單純地以名字來問候回來的使用者。 如果您儲存使用者的喜好設定，下次聊天時就可以使用該資訊來自訂對話。 比方說，您可能會在有使用者感興趣的新聞文章，或有可用約會時通知使用者。 如果 Bot 收到_刪除使用者資料_活動，請進行清除。

您可以使用[儲存體](bot-builder-howto-v4-storage.md)來讀取和寫入永續性儲存體。 這可讓您的 Bot 執行動作，例如更新共用資源、記錄出席回函或投票，或讀取歷史氣象資料。 您可以使用與應用程式相同的方式，讓 Bot 在與使用者的對話中利用儲存體來達成目標。

<!-- 
*Conversation state* pertains to the current conversation that the user is having with your bot. When the conversation ends, your bot deletes this data.

You can also store *user state* that persists after a conversation ends. For example, if you store a user's preferences, you can use that information to customize the conversation the next time you chat. For example, you might alert the user to a news article about a topic that interests her, or alert a user when an appointment becomes available. 
-->

<!-- You should generally avoid saving state using a global variable or function closures.
Doing so will create issues when you want to scale out your bot. Instead, use the conversation state and user state middleware that the BotBuilder SDK provides --> 

<!--
## Types of underlying storage

The SDK provides bot state manager middleware to persist conversation and user state. State can be accessed using the bot's context. This state manager can use Azure Table Storage, file storage, or memory storage as the underlying data storage. You can also create your own storage components for your bot.

Bots built using Azure Table Storage can be designed to be stateless and scalable across multiple compute nodes.

> [!NOTE] 
> File and memory storage won't scale across nodes.

## Writing directly to storage

You can also use the Bot Framework SDK to read and write data directly to storage, without using middleware or without using the bot context. This can be appropriate to data that your bot uses, that comes from a source outside your bot's conversation flow.

For example, let's say your bot allows the user to ask for the weather report, and your bot retrieves the weather report for a specified date, by reading it from an external database. The content of the weather database isn't dependent on user information or the conversation context, so you could just read it directly from storage instead of using the state manager.  See [How to write directly to storage](bot-builder-howto-v4-storage.md) for an example.

## Next steps

Next, lets get into how activities are processed, in depth, and how we respond to them.

> [!div class="nextstepaction"]
> [Activity Processing](bot-builder-concept-activity-processing.md)

## Additional resources

- [How to save state](bot-builder-howto-v4-state.md)
- [How to write directly to storage](bot-builder-howto-v4-storage.md)

-->
