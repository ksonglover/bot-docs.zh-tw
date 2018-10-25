---
title: 將語音新增至訊息 | Microsoft Docs
description: 了解如何使用適用於 Node.js 的 Bot 建立器 SDK 將語音新增至訊息。
author: v-ducvo
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: 3c17097197ba4b6ed0523d84a81974d9cc9fe3b5
ms.sourcegitcommit: b78fe3d8dd604c4f7233740658a229e85b8535dd
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/24/2018
ms.locfileid: "49999127"
---
# <a name="add-speech-to-messages"></a>將語音新增至訊息

[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]

> [!div class="op_single_selector"]
> - [.NET](../dotnet/bot-builder-dotnet-text-to-speech.md)
> - [Node.js](../nodejs/bot-builder-nodejs-text-to-speech.md)
> - [REST](../rest-api/bot-framework-rest-connector-text-to-speech.md)

如果您要為具備語音功能的通道 (例如 Cortana) 建置 Bot，您可以建構訊息，其中指定要由 Bot 讀出的文字。 您也可以指定[輸入提示](bot-builder-nodejs-send-input-hints.md)，藉由指出您的 Bot 要接受、需要或忽略使用者輸入，來嘗試影響用戶端的麥克風狀態。

## <a name="specify-text-to-be-spoken-by-your-bot"></a>指定要由 Bot 讀出的文字

使用適用於 Node.js 的 Bot 建立器 SDK 時，有多種方式可用來指定要由 Bot 在具備語音功能的通道上讀出的文字。 您可以設定 `IMessage.speak` 屬性，並使用 `session.send()` 方法傳送訊息、使用 `session.say()` 方法 (傳入指定顯示文字、語音文字和選項的參數) 傳送訊息，或使用內建的提示 (指定選項 `speak` 和 `retrySpeak`) 傳送訊息。

### <a id="message-speak"></a> IMessage.speak 

如果您要建立將會使用 `session.send()` 方法來傳送的訊息，請設定 `speak`屬性以指定要由 Bot 讀出的文字。 下列程式碼範例會建立一則訊息，其中指定要讀出的文字，並指出 Bot 會[接受使用者輸入](bot-builder-nodejs-send-input-hints.md)。

[!code-javascript[IMessage.speak](../includes/code/node-text-to-speech.js#IMessageSpeak)]

### <a id="session-say"></a> session.say()

除了使用 `session.send()` 以外，您也可以呼叫 `session.say()` 方法以建立和傳送訊息，並且讓訊息指定要讀出的文字、要顯示的文字和其他選項。 此方法定義如下：

`session.say(displayText: string, speechText: string, options?: object)`

| 參數 | 說明 |
|----|----|
| `displayText` | 要顯示的文字。 |
| `speechText` | 要讀出的文字 (採用純文字或 <a href="https://msdn.microsoft.com/en-us/library/hh378377(v=office.14).aspx" target="_blank">SSML</a> 格式)。 |
| `options` | 可包含附件或[輸入提示](bot-builder-nodejs-send-input-hints.md)的 [IMessage][IMessage] 物件。 |

下列程式碼範例會傳送一則訊息，其中指定要顯示的文字和要讀出的文字，並指出 Bot 會[忽略使用者輸入](bot-builder-nodejs-send-input-hints.md)。

[!code-javascript[Session.say()](../includes/code/node-text-to-speech.js#SessionSay)]

### <a id="prompt-options"></a> 提示選項

使用任何內建提示時，您可以設定選項 `speak` 和 `retrySpeak`，以指定要由您的 Bot 讀出的文字。 下列程式碼範例會建立一則提示，其中指定要顯示的文字、一開始要讀出的文字，和等待使用者輸入一段時間後要讀出的文字。 它會指出 Bot [需要使用者輸入](bot-builder-nodejs-send-input-hints.md)，並使用 [SSML](#ssml) 格式指定 "sure" 這個字應以適度強調的方式讀出。

[!code-javascript[Prompt](../includes/code/node-text-to-speech.js#Prompt)]

## <a id="ssml"></a>語音合成標記語言 (SSML)

若要指定要由 Bot 讀出的文字，您可以使用純文字字串或格式化為語音合成標記語言 (SSML) 的字串，SSML 是一種以 XML 為基礎的標記語言，可讓您控制 Bot 語音的各種特性，例如聲音、速率、音量、發音、音調等等。 如需 SSML 的詳細資訊，請參閱<a href="https://msdn.microsoft.com/en-us/library/hh378377(v=office.14).aspx" target="_blank">合成標記語言參考</a>。

> [!TIP]
> 使用 <a href="https://www.npmjs.com/search?q=ssml" target="_blank">SSML 程式庫</a>建立格式正確的 SSML。

## <a name="input-hints"></a>輸入提示

當您在啟用語音功能的通道上傳送訊息時，您也可以藉由包含輸入提示以指出 Bot 要接受、需要或忽略使用者輸入，來嘗試影響用戶端的麥克風狀態。 如需詳細資訊，請參閱[將輸入提示新增至訊息](bot-builder-nodejs-send-input-hints.md)。

## <a name="sample-code"></a>範例程式碼 

如需完整範例以了解如何使用適用於 .NET 的 Bot 建立器 SDK ，來建立具備語音功能的 Bot，請參閱 GitHub 中的<a href="https://github.com/Microsoft/BotBuilder-Samples/tree/master/Node/demo-RollerSkill" target="_blank">骰子機範例</a>。

## <a name="additional-resources"></a>其他資源

- <a href="https://msdn.microsoft.com/en-us/library/hh378377(v=office.14).aspx" target="_blank">語音合成標記語言 (SSML)</a> \(英文\)
- <a href="https://github.com/Microsoft/BotBuilder-Samples/tree/master/Node/demo-RollerSkill" target="_blank">骰子機範例 (GitHub)</a>
- [適用於 Node.js 的 Bot 建立器 SDK 參考資料][SDKReference]

[SDKReference]: https://docs.botframework.com/en-us/node/builder/chat-reference/modules/_botbuilder_d_.html

[Message]: https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.message

[IMessage]: http://docs.botframework.com/en-us/node/builder/chat-reference/interfaces/_botbuilder_d_.imessage
