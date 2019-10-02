---
title: 活動概觀 | Microsoft Docs
description: 了解適用於 .NET 的 Bot Framework SDK 內可以使用的不同活動類型。
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: cce7537124810504c9b92d86ad0ef7bd3b3ac5e9
ms.sourcegitcommit: d493caf74b87b790c99bcdaddb30682251e3fdd4
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/26/2019
ms.locfileid: "71278984"
---
# <a name="activities-overview"></a>活動概觀

[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]

[!INCLUDE [Activity concept overview](../includes/snippet-dotnet-concept-activity.md)]

## <a name="activity-types-in-the-bot-framework-sdk-for-net"></a>適用於 .NET 的 Bot Framework SDK 中的活動類型

適用於 .NET 的 Bot Framework SDK 支援下列活動類型。

| Activity.Type | 介面 | 說明 |
|------|------|------|
| [message](#message) | IMessageActivity | 代表 Bot 和使用者之間的通訊。 |
| [conversationUpdate](#conversationupdate) | IConversationUpdateActivity | 表示 Bot 已新增至對話、其他成員已新增至對話中或從對話中移除，或對話中繼資料已變更。 |
| [contactRelationUpdate](#contactrelationupdate) | IContactRelationUpdateActivity | 表示 Bot 已新增至使用者的連絡人清單，或從使用者的連絡人清單中移除。 |
| [typing](#typing) | ITypingActivity | 表示使用者或在對話另一端的 Bot 正在編譯回應。 | 
| [deleteUserData](#deleteuserdata) | n/a | 表示使用者已要求 Bot 刪除任何可能已儲存的使用者資料。 |
| [endOfConversation](#endofconversation) | IEndOfConversationActivity | 表示對話結束。 |
| [event](#event) | IEventActivity | 代表傳送給 Bot 使用者看不到的通訊。 |
| [invoke](#invoke) | IInvokeActivity | 代表傳送給 Bot 以要求其執行特定作業的通訊。 此活動類型由 Microsoft Bot Framework 保留供內部使用。 |
| [messageReaction](#messagereaction) | IMessageReactionActivity | 表示使用者已對現有活動做出回應。 例如，使用者按了訊息上的「讚」按鈕。 |

## <a name="message"></a>message

您的 Bot 會傳送 **message** 活動以傳達資訊，及接收來自使用者的 **message** 活動。 某些訊息可能只包含純文字，而其他訊息可能包含更豐富的內容，例如[要讀出的文字](bot-builder-dotnet-text-to-speech.md)、[建議的動作](bot-builder-dotnet-add-suggested-actions.md)、[媒體附件](bot-builder-dotnet-add-media-attachments.md)、[複合式資訊卡 (Rich Card)](bot-builder-dotnet-add-rich-card-attachments.md)，和[通道特定資料](bot-builder-dotnet-channeldata.md)。 如需常用訊息屬性的資訊，請參閱[建立訊息](bot-builder-dotnet-create-messages.md)。

## <a name="conversationupdate"></a>conversationUpdate

每當 Bot 已新增至對話、其他成員已新增至對話中或從對話中移除，或對話中繼資料已變更，Bot 就會收到 **conversationUpdate** 活動。 

如果成員已新增至對話，活動的 `MembersAdded` 屬性就會包含 `ChannelAccount` 物件陣列以識別新成員。 

若要判斷 Bot 是否已新增至對話 (亦即成為其中一個新成員)，請評估活動的 `Recipient.Id` 值 (亦即 Bot 的 ID) 是否符合 `MembersAdded` 陣列中任何帳戶的 `Id` 屬性。

如果成員已從對話中移除，`MembersRemoved` 屬性會包含 `ChannelAccount` 物件陣列以識別移除的成員。 

> [!TIP]
> 如果您的 Bot 收到 **conversationUpdate** 活動指出使用者已加入對話，您可以選擇傳送歡迎訊息給使用者作為回應。 

## <a name="contactrelationupdate"></a>contactRelationUpdate

每當 Bot 加入使用者的連絡人清單，或從中移除時，Bot 會收到 **contactRelationUpdate** 活動。 活動的 `Action` 屬性值 (新增 | 移除) 會指出 Bot 是否已加入使用者的連絡人清單，或者是否已從中移除。

## <a name="typing"></a>typing

Bot 會收到 **typing** 活動，表示使用者正在輸入回應。 Bot 會傳送 **typing** 活動，向使用者表示 Bot 正在運作以完成要求或編譯回應。 

## <a name="deleteuserdata"></a>deleteUserData

當使用者要求刪除 Bot 先前為其保存的任何資料，Bot 會收到 **deleteUserData** 活動。 如果您的 Bot 收到此類型的活動，就會為提出要求的使用者刪除先前已儲存的任何個人識別資訊 (PII)。

## <a name="endofconversation"></a>endOfConversation 

Bot 會收到 **endOfConversation** 活動，表示使用者已結束對話。 Bot 會傳送 **endOfConversation** 活動，向使用者表示對話已結束。 

## <a name="event"></a>事件

Bot 可能會從外部程序或服務收到 **event** 活動，代表想要傳達資訊給您的 Bot，而不向使用者顯示該資訊。 **event** 活動的傳送者通常不會預期 Bot 以任何方式確認收到活動。

## <a name="invoke"></a>invoke

Bot 可能會收到 **invoke** 活動，代表執行特定作業的要求。 **invoke** 活動的傳送者通常會預期 Bot 透過 HTTP 回應確認收到活動。 此活動類型由 Microsoft Bot Framework 保留供內部使用。

## <a name="messagereaction"></a>messageReaction

當使用者對現有活動做出回應，某些通道會傳送 **messageReaction** 活動給您的 Bot。 例如，使用者按了訊息上的「讚」按鈕。 **ReplyToId** 屬性會指出使用者回應了哪一個活動。

**messageReaction** 活動可能對應到通道所定義任何數目的 **messageReactionTypes**。 比方說，「讚」或「PlusOne」為通道可能傳送的反應類型。 

## <a name="additional-resources"></a>其他資源

- [傳送及接收活動](bot-builder-dotnet-connector.md)
- [建立訊息](bot-builder-dotnet-create-messages.md)
- [活動類別](https://aka.ms/ActivityClass-dotnet-API)
