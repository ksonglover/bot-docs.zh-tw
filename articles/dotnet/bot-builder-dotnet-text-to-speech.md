---
title: 將語音新增至訊息 | Microsoft Docs
description: 了解如何使用適用於 .NET 的 Bot Framework SDK 將語音新增至訊息。
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: 6d47fe41821b0ea8a61555b8139f2969f204d8dd
ms.sourcegitcommit: a6d02ec4738e7fc90b7108934740e9077667f3c5
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/04/2019
ms.locfileid: "70297309"
---
# <a name="add-speech-to-messages"></a>將語音新增至訊息

[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]

> [!div class="op_single_selector"]
> - [.NET](../dotnet/bot-builder-dotnet-text-to-speech.md)
> - [Node.js](../nodejs/bot-builder-nodejs-text-to-speech.md)
> - [REST](../rest-api/bot-framework-rest-connector-text-to-speech.md)

如果您要為具備語音功能的通道 (例如 Cortana) 建置 Bot，您可以建構訊息，其中指定要由 Bot 讀出的文字。 您也可以指定[輸入提示](bot-builder-dotnet-add-input-hints.md)，藉由指出您的 Bot 要接受、需要或忽略使用者輸入，來嘗試影響用戶端的麥克風狀態。

## <a name="specify-text-to-be-spoken-by-your-bot"></a>指定要由 Bot 讀出的文字

使用適用於 .NET 的 Bot Framework SDK 時，有多種方式可用來指定要由 Bot 在具備語音功能的通道上讀出的文字。 您可以設定[訊息][IMessageActivity]的 `Speak` 屬性、呼叫 `IDialogContext.SayAsync()` 方法，或在使用內建提示傳送訊息時指定提示選項 `speak` 和 `retrySpeak`。

### <a id="message-speak"></a> IMessageActivity.Speak

如果您要建立[訊息][IMessageActivity]並設定其個別屬性，您可以設定訊息的 `Speak` 屬性來指定要由您的 Bot 讀出的文字。 下列程式碼範例會建立一則訊息，其中指定要顯示的文字和要讀出的文字，並指出 Bot 會[接受使用者輸入](bot-builder-dotnet-add-input-hints.md)。

[!code-csharp[Set speak property](../includes/code/dotnet-text-to-speech.cs#Speak1)]

### <a id="say-async"></a> IDialogContext.SayAsync()

如果您正在使用[對話](bot-builder-dotnet-dialogs.md)，您也可以呼叫 `SayAsync()` 方法來建立和傳送訊息，並在訊息內指定要讀出的文字、要顯示的文字和其他選項。 下列程式碼範例會建立一則訊息，其中指定要顯示的文字和要讀出的文字。

[!code-csharp[Call SayAsync()](../includes/code/dotnet-text-to-speech.cs#Speak2)]

### <a id="prompt-options"></a> 提示選項

使用任何內建提示時，您可以設定選項 `speak` 和 `retrySpeak`，以指定要由您的 Bot 讀出的文字。 下列程式碼範例會建立一則提示，其中指定要顯示的文字、一開始要讀出的文字，以及等待使用者輸入一段時間後要讀出的文字。 它會使用 [SSML](#ssml) 格式，來表示應以適度強調的方式讀出 "sure" 這個字。

[!code-csharp[Set Prompt options](../includes/code/dotnet-text-to-speech.cs#Speak3)]

## <a id="ssml"></a> 語音合成標記語言 (SSML)

若要指定要由 Bot 讀出的文字，您可以提供 Bot 格式為語音合成標記語言 (SSML) 的字串。 SSML 是以 XML 為基礎的標記語言 (而且必須是有效的 XML)，可讓您控制 Bot 語音，例如語音、速率、磁碟區、發音及音調等各種特性。 如需 SSML 的詳細資訊，請參閱<a href="https://msdn.microsoft.com/library/hh378377(v=office.14).aspx" target="_blank">語音合成標記語言參考</a> \(英文\)。

當提供 SSML 格式化字串時，可能會省略外部的 SSML 包裝函式元素。

## <a name="input-hints"></a>輸入提示

當您在啟用語音功能的通道上傳送訊息時，您也可以藉由包含輸入提示以指出 Bot 要接受、需要或忽略使用者輸入，來嘗試影響用戶端的麥克風狀態。 如需詳細資訊，請參閱[將輸入提示新增至訊息](bot-builder-dotnet-add-input-hints.md)。

## <a name="sample-code"></a>範例程式碼 

如需完整範例以了解如何使用適用於 .NET 的 Bot Framework SDK 建立具備語音功能的 Bot，請參閱 GitHub 中的<a href="https://github.com/Microsoft/BotBuilder-Samples/tree/v3-sdk-samples/CSharp" target="_blank">骰子機技能範例</a>。

## <a name="additional-resources"></a>其他資源

- [建立訊息](bot-builder-dotnet-create-messages.md)
- [將輸入提示新增至訊息](bot-builder-dotnet-add-input-hints.md)
- <a href="https://msdn.microsoft.com/library/hh378377(v=office.14).aspx" target="_blank">語音合成標記語言 (SSML)</a> \(英文\)
- <a href="https://github.com/Microsoft/BotBuilder-Samples/tree/v3-sdk-samples/CSharp/demo-RollerSkill" target="_blank">骰子機技能範例 (GitHub)</a> \(英文\)
- <a href="https://docs.botframework.com/csharp/builder/sdkreference/dc/d2f/class_microsoft_1_1_bot_1_1_connector_1_1_activity.html" target="_blank">Activity 類別</a> \(英文\)
- <a href="/dotnet/api/microsoft.bot.connector.imessageactivity" target="_blank">IMessageActivity 介面</a>
- <a href="/dotnet/api/microsoft.bot.builder.dialogs.internals.dialogcontext" target="_blank">DialogContext 類別</a>
- <a href="/dotnet/api/microsoft.bot.builder.dialogs.internals.prompt-2" target="_blank">Prompt 類別</a>

[IMessageActivity]: /dotnet/api/microsoft.bot.connector.imessageactivity

