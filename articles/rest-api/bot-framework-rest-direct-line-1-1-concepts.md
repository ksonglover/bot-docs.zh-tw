---
title: Bot Framework 直接線路 API 1.1 中的重要概念 | Microsoft Docs
description: 了解 Bot Framework 直接線路 API 1.1 中的重要概念。
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 12/13/2017
ms.openlocfilehash: a049d77d506fa3fa678a079de52aa424264847c9
ms.sourcegitcommit: b78fe3d8dd604c4f7233740658a229e85b8535dd
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/24/2018
ms.locfileid: "49997045"
---
# <a name="key-concepts-in-direct-line-api-11"></a>直接線路 API 1.1 中的重要概念

您可以使用直接線路 API 啟用您的 Bot 和您自有用戶端應用程式的對話。 

> [!IMPORTANT]
> 本文介紹直接線路 API 1.1 中的重要概念，並提供相關開發人員資源的資訊。 如果要在用戶端應用程式與 Bot 之間建立新連線，請改為使用[直接線路 API 3.0](bot-framework-rest-direct-line-3-0-concepts.md)。

## <a name="authentication"></a>驗證

直接線路 API 1.1 要求可以使用您從 <a href="https://dev.botframework.com/" target="_blank">Bot Framework 入口網站</a> \(英文\) 中的 直接線路頻道設定頁面取得的**祕密**來驗證，或使用您在執行階段取得的**權杖**來驗證。  如需詳細資訊，請參閱[驗證](bot-framework-rest-direct-line-1-1-authentication.md)。

## <a name="starting-a-conversation"></a>開始對話

直接線路對話是由用戶端明確地開啟，且只要 Bot 和用戶端在參與且具有有效認證，對話就會繼續執行。 如需詳細資訊，請參閱[開始對話](bot-framework-rest-direct-line-1-1-start-conversation.md)。

## <a name="sending-messages"></a>傳送訊息

使用直接線路 API 1.1，用戶端就能透過發出 `HTTP POST` 要求將訊息傳送給您的 Bot。 用戶端可能會為每個要求都傳送單一訊息。 如需詳細資訊，請參閱[將訊息傳送至 Bot](bot-framework-rest-direct-line-1-1-send-message.md)。

## <a name="receiving-messages"></a>接收訊息

使用直接線路 API 1.1，用戶端就能透過使用 `HTTP GET` 要求來輪詢以接收訊息。 用戶端對於每個要求的回應，可能會從 Bot 收到屬於 `MessageSet` 一部分的多個訊息。 如需詳細資訊，請參閱[ 接收來自 Bot 的訊息](bot-framework-rest-direct-line-1-1-receive-messages.md)。

## <a name="developer-resources"></a>開發人員資源

### <a name="client-library"></a>用戶端程式庫

Bot Framework 提供用戶端程式庫，有助於透過 C# 存取直接線路 API 1.1。 若要在 Visual Studio 專案中使用用戶端程式庫，請安裝 `Microsoft.Bot.Connector.DirectLine` <a href="https://www.nuget.org/packages/Microsoft.Bot.Connector.DirectLine/1.1.1" target="_blank">v1.x NuGet 套件</a> \(英文\)。 

除了使用 C# 用戶端程式庫之外，您也可以使用<a href="https://docs.botframework.com/en-us/restapi/directline/swagger.json" target="_blank">直接線路 API 1.1 Swagger 檔案</a>以所選的語言產生自己的用戶端程式庫。

### <a name="web-chat-control"></a>Web 聊天控制項 

Bot Framework 提供可讓您將使用直接線路的Bot 內嵌到您用戶端應用程式的控制項。 如需詳細資訊，請參閱 <a href="https://github.com/Microsoft/BotFramework-WebChat" target="_blank">Microsoft Bot Framework WebChat 控制項</a>。
