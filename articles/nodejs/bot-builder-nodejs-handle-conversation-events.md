---
title: 處理使用者和對話事件 | Microsoft Docs
description: 了解如何使用適用於 Node.js 的 Bot Framework SDK 來處理使用者加入交談之類的事件。
author: DucVo
ms.author: v-ducvo
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: a6149b750a4432f00268571df6d12b611114181f
ms.sourcegitcommit: 6a83b2c8ab2902121e8ee9531a7aa2d85b827396
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/12/2019
ms.locfileid: "67404915"
---
# <a name="handle-user-and-conversation-events"></a>處理使用者和對話事件

[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]

本文會示範 Bot 處理事件的方式，這些事件包括使用者加入對話、將 Bot 新增至連絡人清單，或是當 Bot 從對話中移除時會說**再見**等等。


## <a name="greet-a-user-on-conversation-join"></a>向加入對話的使用者打招呼
每當有成員加入或離開對話時，Bot Framework 會提供 [conversationUpdate][conversationUpdate] 事件來通知您的 Bot。 對話成員可以是使用者或 Bot。

下列程式碼片段可讓 Bot 向對話的新成員打招呼，或在 Bot 從對話中移除時說出**再見**。

[!INCLUDE [conversationUpdate sample Node.js](../includes/snippet-code-node-conversationupdate-1.md)]

## <a name="acknowledge-add-to-contacts-list"></a>認同新增至連絡人清單

[contactRelationUpdate][contactRelationUpdate] 事件會告訴 Bot 有使用者將您新增至他們的連絡人清單。

[!INCLUDE [contactRelationUpdate sample Node.js](../includes/snippet-code-node-contactrelationupdate-1.md)]

## <a name="add-a-first-run-dialog"></a>新增首次執行對話方塊

由於不是所有通道都支援 **conversationUpdate** 和 **contactRelationUpdate** 事件，若要向加入對話的使用者打招呼，通用的方式是新增首次執行對話方塊。

在下列範例中，我們已新增函式，以在每次有沒看過的使用者出現時觸發對話方塊。 您可以藉由提供動作的 [onFindAction][onFindAction] 處理常式，來自訂動作觸發的方式。 

[!INCLUDE [first-run sample Node.js](../includes/snippet-code-node-first-run-dialog-1.md)]

您也可以藉由提供 [onSelectAction][onSelectAction] 處理常式來自訂動作在觸發後的行為。 針對觸發動作，您可以提供 [onInterrupted][onInterrupted] 處理常式，以在中斷發生之前加以攔截。 如需詳細資訊，請參閱[處理使用者動作](bot-builder-nodejs-dialog-actions.md)

## <a name="additional-resources"></a>其他資源

* [conversationUpdate][conversationUpdate]
* [contactRelationUpdate][contactRelationUpdate]

[conversationUpdate]: https://docs.botframework.com/node/builder/chat-reference/interfaces/_botbuilder_d_.iconversationupdate.html
[contactRelationUpdate]: https://docs.botframework.com/node/builder/chat-reference/interfaces/_botbuilder_d_.icontactrelationupdate.html

[onFindAction]: https://docs.botframework.com/node/builder/chat-reference/interfaces/_botbuilder_d_.itriggeractionoptions#onfindaction
[onSelectAction]: https://docs.botframework.com/node/builder/chat-reference/interfaces/_botbuilder_d_.itriggeractionoptions#onselectaction
[onInterrupted]: https://docs.botframework.com/node/builder/chat-reference/interfaces/_botbuilder_d_.itriggeractionoptions#oninterrupted

[SendTyping]: https://docs.botframework.com/node/builder/chat-reference/classes/_botbuilder_d_.session#sendtyping
[IMessage]: http://docs.botframework.com/node/builder/chat-reference/interfaces/_botbuilder_d_.imessage
[ChatConnector]: https://docs.botframework.com/node/builder/chat-reference/classes/_botbuilder_d_.chatconnector.html
[session_userData]: https://docs.botframework.com/node/builder/chat-reference/classes/_botbuilder_d_.session.html#userdata
