---
title: 主動式傳訊 | Microsoft Docs
description: 了解如何主動傳訊。
keywords: 歡迎使用者, 起始對話
author: ivorb
ms.author: v-ivorb
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 03/21/2018
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 47f3dd9405a36260a01b28bcc389355d6726ae11
ms.sourcegitcommit: f576981342fb3361216675815714e24281e20ddf
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/18/2018
ms.locfileid: "39299223"
---
# <a name="proactive-messaging"></a>主動式傳訊
[!INCLUDE [pre-release-label](../includes/pre-release-label.md)]
<!--
When you think about the exchange of messages between your bot and the user, you're probably thinking about the scenario where the user sends a message to your bot and your bot then replies to the user with a message of its own. We call this _reactive messaging_ and it's by far the most common flow that you should optimize your bot's code for.

It is possible, however, for your bot to initiate a conversation with the user by sending them a message first. We call this _proactive messaging_ and while the code you'll write to send a proactive message is very similar to what you'd write in the reactive case, there are a few differences that are worth exploring.

The first thing to note is that before you can send a proactive message to a user, the user will have to send at least one reactive style message to your bot. There are two reasons for this.

1. You need to get the user's `ConversationReference` and save it somewhere for future use. You can think of the conversation reference as the user's address, as it contains information about the channel they came in on, their user ID, the conversation ID, and even the server that should receive any future messages. This object is simple JSON and should be saved whole without tampering.
2. Most channels by policy won't let a bot initiate conversations with users they've never spoken to before. Depending on the channel the user might need to explicitly add the bot to a conversation or at a minimum send an initial message to the bot.

> ![NOTE]
> This bot currently runs properly only when deployed to Azure. However, you can test the bot without publishing it.

A common case of proactive messaging comes when our bot is performing a time-consuming task. In this case, we send a **typing** activity indicates to the user that the bot is in a *processing* mode, and then follow it up with a proactive message once our processing has completed.
-->

[!include[Introduction to proactive messages - part 1](../includes/snippet-proactive-messages-intro-1.md)] 

## <a name="types-of-proactive-messages"></a>主動式訊息的類型 

[!include[Introduction to proactive messages - part 2](../includes/snippet-proactive-messages-intro-2.md)] 

## <a name="next-steps"></a>後續步驟

現在，您已熟悉活動、傳訊和對話流程，接著來看看 Bot 的其他重要層面，讓我們從使用 LUIS 來了解語言開始。

> [!div class="nextstepaction"]
> [語言理解](bot-builder-concept-luis.md)