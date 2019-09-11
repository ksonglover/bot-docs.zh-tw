---
title: Bot Framework Direct Line API 3.0 中的重要概念 | Microsoft Docs
description: 了解 Bot Framework Direct Line API 3.0 中的重要概念。
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 01/06/2019
ms.openlocfilehash: 6727530ec6267a63e28e103bdfc12183ebed8016
ms.sourcegitcommit: a6d02ec4738e7fc90b7108934740e9077667f3c5
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/04/2019
ms.locfileid: "70299588"
---
# <a name="key-concepts-in-direct-line-api-30"></a>Direct Line API 3.0 中的重要概念

您可以使用 Direct Line API 啟用您的 Bot 和您自有用戶端應用程式的對話。 本文介紹 Direct Line API 3.0 中的重要概念，並提供相關開發人員資源的資訊。

## <a name="authentication"></a>Authentication

Direct Line API 3.0 要求可以使用您從 <a href="https://dev.botframework.com/" target="_blank">Bot Framework 入口網站</a> 中的 Direct Line 頻道設定頁面取得的**祕密**來驗證，或使用您在執行階段取得的**權杖**來驗證。 如需詳細資訊，請參閱[驗證](bot-framework-rest-direct-line-3-0-authentication.md)。

## <a name="starting-a-conversation"></a>開始對話

直接線路對話是由用戶端明確地開啟，且只要 Bot 和用戶端在參與且具有有效認證，對話就會繼續執行。 如需詳細資訊，請參閱[開始對話](bot-framework-rest-direct-line-3-0-start-conversation.md)。

## <a name="sending-messages"></a>傳送訊息

用戶端可藉由使用 Direct Line API 3.0，來透過發出 `HTTP POST` 要求將訊息傳送給您的 Bot。 用戶端可能會每個要求都傳送單一訊息。 如需詳細資訊，請參閱[將活動傳送到 Bot](bot-framework-rest-direct-line-3-0-send-activity.md)。

## <a name="receiving-messages"></a>接收訊息

用戶端可以使用 Direct Line API 3.0 透過`WebSocket`串流或發出`HTTP GET`要求從 Bot 接收訊息。 用戶端可以使用這些技術從 `ActivitySet` 的 Bot 一次接收多則訊息。 如需詳細資訊，請參閱[接收來自 Bot 的活動](bot-framework-rest-direct-line-3-0-receive-activities.md)。

## <a name="developer-resources"></a>開發人員資源

### <a name="client-libraries"></a>用戶端程式庫

Bot Framework 提供用戶端程式庫，可輔助透過 C# 和 Node.js 存取 Direct Line API 3.0。 

- 若要在 Visual Studio 專案中使用 .NET 用戶端程式庫，請安裝 `Microsoft.Bot.Connector.DirectLine` <a href="https://www.nuget.org/packages/Microsoft.Bot.Connector.DirectLine" target="_blank">NuGet 套件</a>。 

- 若要使用 Node.js 用戶端程式庫，請使用 <a href="https://www.npmjs.com/package/botframework-directlinejs" target="_blank">NPM</a> (或<a href="https://github.com/Microsoft/BotFramework-DirectLineJS" target="_blank">下載</a>來源) 安裝 `botframework-directlinejs` 程式庫。

::: moniker range="azure-bot-service-3.0"

### <a name="sample-code"></a>範例程式碼

<a href="https://github.com/Microsoft/BotBuilder-Samples/tree/v3-sdk-samples" target="_blank">BotBuilder-範例</a> GitHub 存放庫包含多個範例，說明如何搭配 C# 和 Node.js 使用 Direct Line API 3.0。

| 範例 | 語言 | 說明 |
|----|----|----|
| <a href="https://github.com/Microsoft/BotBuilder-Samples/tree/v3-sdk-samples/CSharp/core-DirectLine" target="_blank">Direct Line Bot 範例</a> | C# | Bot 範例和自訂用戶端，會使用 Direct Line API 彼此通訊。 |
| <a href="https://github.com/Microsoft/BotBuilder-Samples/tree/v3-sdk-samples/CSharp/core-DirectLineWebSockets" target="_blank">Direct Line Bot 範例 (使用用戶端 WebSocket)</a> | C# | Bot 範例和自訂用戶端，會使用 Direct Line API 和 WebSocket 來彼此通訊。 |
| <a href="https://github.com/Microsoft/BotBuilder-Samples/tree/v3-sdk-samples/Node/core-DirectLine" target="_blank">Direct Line Bot 範例</a> | JavaScript | Bot 範例和自訂用戶端，會使用 Direct Line API 彼此通訊。 |
| <a href="https://github.com/Microsoft/BotBuilder-Samples/tree/v3-sdk-samples/Node/core-DirectLineWebSockets" target="_blank">Direct Line Bot 範例 (使用用戶端 WebSocket)</a> | JavaScript | Bot 範例和自訂用戶端，會使用 Direct Line API 和 WebSocket 來彼此通訊。 |

::: moniker-end

### <a name="web-chat-control"></a>網路聊天控制項 

Bot Framework 提供可讓您將使用直接線路的Bot 內嵌到您用戶端應用程式的控制項。 如需詳細資訊，請參閱 <a href="https://github.com/Microsoft/BotFramework-WebChat" target="_blank">Microsoft Bot Framework WebChat 控制項</a>。
