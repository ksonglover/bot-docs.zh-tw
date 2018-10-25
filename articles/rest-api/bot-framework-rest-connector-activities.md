---
title: 活動概觀 | Microsoft Docs
description: 了解 Bot 連接器服務中的不同活動類型。
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 12/13/2017
ms.openlocfilehash: b246e9e07243e4064f92e72ee3909541f642714e
ms.sourcegitcommit: b78fe3d8dd604c4f7233740658a229e85b8535dd
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/24/2018
ms.locfileid: "49999925"
---
# <a name="activities-overview"></a>活動概觀

Bot 連接器服務在 Bot 和通道 (使用者) 之間藉由傳遞[活動][Activity]物件來交換資訊。 活動的最常見的類型是**訊息**，但是還有其他活動類型可用來將各種類型的資訊傳達給 Bot 或通道。 

## <a name="activity-types-in-the-bot-connector-service"></a>Bot 連接器服務中的活動類型

以下為 Bot 連接器服務支援的活動類型。

| 活動類型 | 說明 |
|------|------|------|
| Message | 代表 Bot 和使用者之間的通訊。 |
| conversationUpdate | 表示 Bot 已新增至對話、其他成員已新增至對話或從對話中移除，或對話中繼資料已變更。 |
| contactRelationUpdate | 表示 Bot 已新增至連絡人清單，或從使用者的連絡人清單中移除。 |
| typing | 表示使用者或在對話另一端的 Bot 正在編譯回應。 | 
| deleteUserData | 表示使用者已要求 Bot 刪除任何可能已儲存的使用者資料。 |
| endOfConversation | 表示對話結束。 |

## <a name="message"></a>Message

您的 Bot 會傳送 **message** 活動以傳達資訊，及接收來自使用者的 **message** 活動。 有些郵件可能是純文字，而其他郵件可能包含更豐富的內容，例如[媒體附件](bot-framework-rest-connector-add-media-attachments.md)、[按鈕，以及卡片](bot-framework-rest-connector-add-rich-cards.md)或是[特定通道的資料](bot-framework-rest-connector-channeldata.md)。 如需常用訊息屬性的資訊，請參閱[建立訊息](bot-framework-rest-connector-create-messages.md)。 如需有關如何傳送和接收訊息的資訊，請參閱[傳送和接收訊息](bot-framework-rest-connector-send-and-receive-messages.md)。 

## <a name="conversationupdate"></a>conversationUpdate

每當 Bot 已加入對話、其他成員已加入或從對話中移除，或對話中繼資料已變更，Bot 會收到 **conversationUpdate** 活動。 

如果成員已加入對話，則活動的 `addedMembers` 屬性會識別新成員。 如果成員已從對話中移除，則 `removedMembers` 屬性會識別移除的成員。 

> [!TIP]
> 如果您的 Bot 收到 **conversationUpdate** 活動指出使用者已加入對話，您可以選擇傳送歡迎訊息給使用者做為回應。 

## <a name="contactrelationupdate"></a>contactRelationUpdate

每當 Bot 加入使用者的連絡人清單，或從中移除時，Bot 會收到 **contactRelationUpdate** 活動。 活動的 `action` 屬性值 (新增 | 移除) 會指出 Bot 是否已加入使用者的連絡人清單，或者是否已從中移除。

## <a name="typing"></a>typing

Bot 會收到 **typing** 活動，表示使用者正在輸入回應。 Bot 會傳送 **typing** 活動，向使用者表示 Bot 正在運作以完成要求或編譯回應。 

## <a name="deleteuserdata"></a>deleteUserData

當使用者要求刪除 Bot 先前為其保存的任何資料，Bot 會收到 **deleteUserData** 活動。 如果您的 Bot 收到此類型的活動，就會為提出要求的使用者刪除先前已儲存的任何個人識別資訊 (PII)。

## <a name="endofconversation"></a>endOfConversation 

Bot 會收到 **endOfConversation** 活動，表示使用者已結束對話。 Bot 會傳送 **endOfConversation** 活動，向使用者表示對話已結束。 

## <a name="additional-resources"></a>其他資源

- [建立訊息](bot-framework-rest-connector-create-messages.md)
- [傳送及接收訊息](bot-framework-rest-connector-send-and-receive-messages.md)

[Activity]: bot-framework-rest-connector-api-reference.md#activity-object