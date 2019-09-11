---
title: 將輸入提示新增至訊息 | Microsoft Docs
description: 了解如何使用適用於 .NET 的 Bot Framework SDK，將輸入提示新增至訊息。
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: d9d98c217afcf3eb1759284bcb0d46d9001bf383
ms.sourcegitcommit: a6d02ec4738e7fc90b7108934740e9077667f3c5
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/04/2019
ms.locfileid: "70298464"
---
# <a name="add-input-hints-to-messages"></a>將輸入提示新增至訊息

[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]

> [!div class="op_single_selector"]
> - [.NET](../dotnet/bot-builder-dotnet-add-input-hints.md)
> - [Node.js](../nodejs/bot-builder-nodejs-send-input-hints.md)
> - [REST](../rest-api/bot-framework-rest-connector-add-input-hints.md)

您可藉由指定訊息的輸入提示，指出您的 Bot 在訊息傳遞給用戶端之後，會接受、需要或忽略使用者輸入。 這可讓用戶端為許多通道設定相應的使用者輸入控制項狀態。 例如，如果訊息的輸入提示指出 Bot 會忽略使用者輸入，則用戶端可關閉麥克風並停用輸入方塊，以防止使用者提供輸入。

## <a name="accepting-input"></a>接受輸入

若要指出 Bot 要被動接受輸入，但不等候使用者回應，請將訊息的輸入提示設定為 `InputHints.AcceptingInput`。 在許多通道上，這會啟用用戶端的輸入方塊並關閉麥克風，但使用者仍可存取。 例如，如果使用者按住麥克風按鈕，Cortana 會開啟麥克風並接受使用者輸入。 下列程式碼範例會建立訊息，指出 Bot 會接受使用者輸入。

```cs
Activity reply = activity.CreateReply("This is the text that will be displayed.");
reply.Speak = "This is the text that will be spoken.";
reply.InputHint = InputHints.AcceptingInput;
await connector.Conversations.ReplyToActivityAsync(reply);
```

## <a name="expecting-input"></a>需要輸入

若要指出 Bot 要等候使用者回應，請將訊息的輸入提示設定為 `InputHints.ExpectingInput`。 在許多通道上，這會啟用用戶端的輸入方塊並開啓麥克風。 下列程式碼範例會建立訊息來指出 Bot 需要使用者輸入。

```cs
Activity reply = activity.CreateReply("This is the text that will be displayed.");
reply.Speak = "This is the text that will be spoken.";
reply.InputHint = InputHints.ExpectingInput;
await connector.Conversations.ReplyToActivityAsync(reply);
```

## <a name="ignoring-input"></a>忽略輸入

若要指出 Bot 尚未準備好接收使用者輸入，請將訊息的輸入提示設定為 `InputHints.IgnorningInput`。 在許多通道上，這會停用用戶端的輸入方塊並關閉麥克風。 下列程式碼範例會建立訊息來指出 Bot 會忽略使用者輸入。

```cs
Activity reply = activity.CreateReply("This is the text that will be displayed.");
reply.Speak = "This is the text that will be spoken.";
reply.InputHint = InputHints.IgnoringInput;
await connector.Conversations.ReplyToActivityAsync(reply);
```

## <a name="default-values-for-input-hint"></a>輸入提示的預設值

如果您未設定訊息的輸入提示，Bot Framework SDK 會使用下列邏輯自動為您設定：

- 如果 Bot 傳送提示，訊息的輸入提示將指定 Bot **需要輸入**。</li>
- 如果 Bot 傳送單一訊息，訊息的輸入提示將指定 Bot 會**接受輸入**。</li>
- 如果 Bot 傳送一系列的連續訊息，系列中的所有訊息 (但最後一則訊息除外) 的輸入提示都將指定 Bot 會**忽略輸入**，而系列中最後一則訊息的輸入提示將指定 Bot 會**接受輸入**。

## <a name="additional-resources"></a>其他資源

- [建立訊息](bot-builder-dotnet-create-messages.md)
- [將語音新增至訊息](bot-builder-dotnet-text-to-speech.md)
- <a href="https://docs.botframework.com/csharp/builder/sdkreference/dc/d2f/class_microsoft_1_1_bot_1_1_connector_1_1_activity.html" target="_blank">Activity 類別</a> \(英文\)
- <a href="/dotnet/api/microsoft.bot.connector.inputhints" target="_blank">InputHints 類別</a>
