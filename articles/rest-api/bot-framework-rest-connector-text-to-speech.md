---
title: 將語音新增至訊息 | Microsoft Docs
description: 了解如何使用 Bot Connector 服務將語音新增至訊息。
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 12/13/2017
ms.openlocfilehash: 2aac000b7e8dd52b00659ffecde5184df6c29991
ms.sourcegitcommit: b78fe3d8dd604c4f7233740658a229e85b8535dd
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/24/2018
ms.locfileid: "49998685"
---
# <a name="add-speech-to-messages"></a>將語音新增至訊息
> [!div class="op_single_selector"]
> - [.NET](../dotnet/bot-builder-dotnet-text-to-speech.md)
> - [Node.js](../nodejs/bot-builder-nodejs-text-to-speech.md)
> - [REST](../rest-api/bot-framework-rest-connector-text-to-speech.md)

如果您要為具備語音功能的通道 (例如 Cortana) 建置 Bot，您可以建構訊息，其中指定要由 Bot 讀出的文字。 您也可以指定[輸入提示](bot-framework-rest-connector-add-input-hints.md)，藉由指出您的 Bot 要接受、需要或忽略使用者輸入，來嘗試影響用戶端的麥克風狀態。

## <a name="specify-text-to-be-spoken-by-your-bot"></a>指定要由您的 Bot 讀出的文字

若要指定將由 Bot 在啟用語音通道上讀出的文字，請在代表您訊息的 [Activity][Activity] 物件中設定 `speak` 屬性。 您可以將 `speak` 屬性設定為純文字字串或格式化為<a href="https://msdn.microsoft.com/en-us/library/hh378377(v=office.14).aspx" target="_blank">語音合成標記語言 (SSML)</a> 的字串，SSML 是一種以 XML 為基礎的標記語言，可讓您控制 Bot 語音的各種特性，例如聲音、速率、音量、發音、音調等等。 

下列要求會傳送一則訊息，其中指定要顯示的文字和要讀出的文字，並指出 Bot [需要使用者輸入](bot-framework-rest-connector-add-input-hints.md)。 它會使用 <a href="https://msdn.microsoft.com/en-us/library/hh378377(v=office.14).aspx" target="_blank">SSML</a> 格式來指定 `speak` 屬性，表示應以適度強調的方式讀出 "sure" 這個字。 在此範例要求中，`https://smba.trafficmanager.net/apis` 表示基底 URI，和您提出要求之 Bot 的基底 URI 可能不同。 如需設定基底 URI 的詳細資訊，請參閱 [API 參考](bot-framework-rest-connector-api-reference.md#base-uri)。

```http
POST https://smba.trafficmanager.net/apis/v3/conversations/abcd1234/activities/5d5cdc723
Authorization: Bearer ACCESS_TOKEN
Content-Type: application/json
```

```json
{
    "type": "message",
    "from": {
        "id": "12345678",
        "name": "sender's name"
    },
    "conversation": {
        "id": "abcd1234",
        "name": "conversation's name"
   },
   "recipient": {
        "id": "1234abcd",
        "name": "recipient's name"
    },
    "text": "Are you sure that you want to cancel this transaction?",
    "speak": "Are you <emphasis level='moderate'>sure</emphasis> that you want to cancel this transaction?",
    "inputHint": "expectingInput",
    "replyToId": "5d5cdc723"
}
```

## <a name="input-hints"></a>輸入提示

當您在啟用語音功能的通道上傳送訊息時，您也可以藉由包含輸入提示以指出 Bot 要接受、需要或忽略使用者輸入，來嘗試影響用戶端的麥克風狀態。 如需詳細資訊，請參閱[將輸入提示新增至訊息](bot-framework-rest-connector-add-input-hints.md)。

## <a name="additional-resources"></a>其他資源

- [建立訊息](bot-framework-rest-connector-create-messages.md)
- [傳送及接收訊息](bot-framework-rest-connector-send-and-receive-messages.md)
- [將輸入提示新增至訊息](bot-framework-rest-connector-add-input-hints.md)
- <a href="https://msdn.microsoft.com/en-us/library/hh378377(v=office.14).aspx" target="_blank">語音合成標記語言 (SSML)</a>

[Activity]: bot-framework-rest-connector-api-reference.md#activity-object
