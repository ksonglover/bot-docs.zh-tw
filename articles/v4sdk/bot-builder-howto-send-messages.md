---
title: 傳送訊息 | Microsoft Docs
description: 了解如何在 Bot 建立器 SDK 內傳送訊息。
keywords: 傳送訊息, 訊息活動, 簡單的文字訊息, 語音, 語音訊息
author: ivorb
ms.author: v-ivorb
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 08/23/2018
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: b56ad56e19691cfbd7b39606832ed10fce951aa3
ms.sourcegitcommit: f89ed979eb6321232fb21100ef376d9b0d5113c9
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/24/2018
ms.locfileid: "42914588"
---
# <a name="send-messages"></a>傳送訊息

[!INCLUDE [pre-release-label](../includes/pre-release-label.md)]

您的 Bot 與使用者進行往來通訊的主要方式，都是透過**訊息**活動。 有些訊息可能只包含純文字，有些則可能包含更豐富的內容，例如卡片或附件。 Bot 回合處理常式會接收來自使用者的訊息，而您可以從該處將回應傳送給使用者。 [回合內容](bot-builder-concept-activity-processing.md#turn-context)物件會提供將訊息傳回給使用者的方法。 如需一般活動處理方式的詳細資訊，請參閱[活動處理](bot-builder-concept-activity-processing.md)。

本文說明如何傳送簡單的文字和語音訊息。 如需傳送豐富內容的相關資訊，請參閱如何[新增豐富的媒體附件](bot-builder-howto-add-media-attachments.md)。 如需如何使用提示物件的相關資訊，請參閱如何[提示使用者輸入](bot-builder-prompts.md)。

## <a name="send-a-simple-text-message"></a>傳送簡單的文字訊息

若要傳送簡單的文字訊息，請指定要以活動的形式傳送的字串：

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

在 Bot 的 **OnTurn** 方法中，使用回合內容物件的 **SendActivity** 方法傳送單一訊息回應。 您也可以使用該物件的 **SendActivities** 方法同時傳送多個回應。

```cs
await context.SendActivity("Greetings from sample message.");
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

在 Bot 的回合處理常式中，使用回合內容物件的 **sendActivity** 方法傳送單一訊息回應。 您也可以使用該物件的 **sendActivities** 方法同時傳送多個回應。

```javascript
await context.sendActivity("Greetings from sample message.");
```

---

## <a name="send-a-spoken-message"></a>傳送語音訊息

某些通道支援具備語音功能的 Bot，而可讓 Bot 向使用者說話。 訊息中可同時包含撰寫和讀出的內容。

> [!NOTE]
> 不支援語音的通道則會忽略語音內容。

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

使用選用的 **speak** 參數，提供要在回應中讀出的文字。

```cs
await context.SendActivity(
    "This is the text to be displayed.",
    "This is the text to be spoken.");
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

若要新增語音，您必須以 `Microsoft.Bot.Builder.MessageFactory` 建置訊息。 `MessageFactory` 較常與[豐富的媒體](bot-builder-howto-add-media-attachments.md)搭配使用，因此在那方面會有較多說明，但我們目前只需要將其運用於此領域。

```javascript
// Require MessageFactory from botbuilder
const {MessageFactory} = require('botbuilder');

const basicMessage = MessageFactory.text('This is the text that will be displayed.', 'This is the text that will be spoken.');
await context.sendActivity(basicMessage);
```

---
