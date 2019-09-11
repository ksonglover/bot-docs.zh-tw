---
title: Bot Framework User-Agent 要求 | Microsoft Docs
description: 了解 Bot Framework 向 Web 伺服器發出的要求。
author: JohnD-ms
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 12/13/2017
ms.openlocfilehash: f86e130fff83e8ad1604ec4ea531d58793fb8409
ms.sourcegitcommit: a6d02ec4738e7fc90b7108934740e9077667f3c5
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/04/2019
ms.locfileid: "70297471"
---
# <a name="bot-framework-user-agent-requests"></a>Bot Framework User-Agent 要求

如果您正在閱讀這則訊息，您可能已從 Microsoft Bot Framework 服務收到要求。 本指南會協助您了解這些要求的本質，並提供一些停止要求的步驟 (如果您想要的話)。

如果您從我們的服務收到要求，其中可能會有類似下列字串的已格式化 User-Agent 標頭：

```User-Agent: BF-DirectLine/3.0 (Microsoft-BotFramework/3.0 +https://botframework.com/ua)```

這個字串最重要的部分是 **Microsoft-BotFramework** 識別碼，其可供 Microsoft Bot Framework 這個工具和服務集合用來讓獨立軟體開發人員建立及操作他們自己的 Bot。

如果您是 Bot 開發人員，您可能已經知道系統為什麼會將這些要求導向您的服務。 如果您不是，請繼續閱讀以深入了解。

## <a name="why-is-microsoft-contacting-my-service"></a>為什麼 Microsoft 會連絡我的服務？

Bot Framework 會將 Skype 及 Facebook Messenger 等聊天服務上的使用者連線到 Bot，此 Bot 是 Web 伺服器，且其中會有 REST API 在可由網際網路存取的端點上執行。 Bot 的 HTTP 呼叫 (也稱為 Webhook 呼叫) 只會傳送到 Bot 開發人員 (已向 Bot Framework 開發人員入口網站註冊) 所指定的 URL。

如果您收到從 Bot Framework 服務到 Web 服務的不請自來要求，可能是因為開發人員不小心或故意輸入您的 URL 作為其 Bot 的 Webhook 回呼。

## <a name="to-stop-these-requests"></a>停止這些要求

如需協助阻止不想要的要求，使其無法到達您的 Web 服務，請連絡 [bf-reports@microsoft.com](mailto://bf-reports@microsoft.com)。
