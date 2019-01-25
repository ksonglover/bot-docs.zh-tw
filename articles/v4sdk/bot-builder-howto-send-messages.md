---
title: 傳送及接收文字訊息 | Microsoft Docs
description: 了解如何在 Bot Framework SDK 內傳送及接收文字訊息。
keywords: 傳送訊息, 訊息活動, 簡單的文字訊息, 文字訊息, 接收訊息
author: ivorb
ms.author: v-ivorb
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 01/16/2019
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: ff52a62353df8983d94bbd09276de4ae94e6535e
ms.sourcegitcommit: c6ce4c42fc56ce1e12b45358d2c747fb77eb74e2
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 01/22/2019
ms.locfileid: "54453872"
---
# <a name="send-and-receive-text-message"></a>傳送及接收文字訊息

[!INCLUDE [pre-release-label](../includes/pre-release-label.md)]

您的 Bot 與使用者進行往來通訊的主要方式，都是透過**訊息**活動。 有些訊息可能只包含純文字，有些則可能包含更豐富的內容，例如卡片或附件。 Bot 回合處理常式會接收來自使用者的訊息，而您可以從該處將回應傳送給使用者。 回合內容物件會提供將訊息傳回給使用者的方法。 本文說明如何傳送簡單的文字訊息。

大部分文字欄位都支援 Markdown，但是支援可能因通道而有所不同。

## <a name="send-a-text-message"></a>傳送文字訊息

若要傳送簡單的文字訊息，請指定要以活動的形式傳送的字串：

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

在 Bot 的 `OnTurnAsync` 方法中，使用回合內容物件的 `SendActivityAsync` 方法傳送單一訊息回應。 您也可以使用該物件的 `SendActivitiesAsync` 方法同時傳送多個回應。

```cs
await turnContext.SendActivityAsync($"Welcome!");
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

在 Bot 的 `onTurn` 處理常式中，使用回合內容物件的 `sendActivity` 方法傳送單一訊息回應。 您也可以使用該物件的 `sendActivities` 方法同時傳送多個回應。

```javascript
await context.sendActivity("Welcome!");
```
---
## <a name="receive-a-text-message"></a>接收文字訊息

若要接收簡單的文字訊息，請使用 *activity* 物件的 *text* 屬性。 

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

在 Bot 的 `OnTurnAsync` 方法中，使用下列程式碼來接收訊息。 

```cs
var responseMessage = turnContext.Activity.Text;
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

在 Bot 的 `OnTurnAsync` 方法中，使用下列程式碼來接收訊息。

```javascript
let text = turnContext.activity.text;
```

---

## <a name="additional-resources"></a>其他資源

- 如需一般活動處理方式的詳細資訊，請參閱[活動處理](~/v4sdk/bot-builder-basics.md#the-activity-processing-stack)。
- 如需傳送豐富內容的相關資訊，請參閱如何新增[豐富的媒體](bot-builder-howto-add-media-attachments.md)附件。
- 如需格式化的詳細資訊，請參閱 Bot Framework 活動結構描述的[訊息活動區段](https://aka.ms/botSpecs-activitySchema#message-activity)。
