---
title: 將輸入提示新增至訊息 | Microsoft Docs
description: 了解如何使用 Bot Framework SDK 將輸入提示新增至訊息。
keywords: Inputhints, 接受輸入, 必須有輸入, 忽略輸入, 語音
author: ivorb
ms.author: v-ivorb
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 04/18/2019
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: e788501e7bd4cc109677f0e6870eac95c0696e36
ms.sourcegitcommit: aea57820b8a137047d59491b45320cf268043861
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 04/22/2019
ms.locfileid: "59904951"
---
# <a name="add-input-hints-to-messages"></a>將輸入提示新增至訊息

[!INCLUDE[applies-to](../includes/applies-to.md)]

您可藉由指定訊息的輸入提示，指出您的 Bot 在訊息傳遞給用戶端之後，會接受、需要或忽略使用者輸入。 這可讓用戶端為許多通道設定相應的使用者輸入控制項狀態。 例如，如果訊息輸入提示指出 Bot 忽略使用者輸入，則用戶端可關閉麥克風並停用輸入方塊，以防止使用者提供輸入。

請確定輸入提示中有納入所需的程式庫。

# <a name="ctabcs"></a>[C#](#tab/cs)

```cs
using Microsoft.Bot.Schema;
```

<!--TODO: Remove the following remark after the next release of the NuGet packages.-->

這些範例所使用的 **MessageFactory** 類別定義於下列命名空間中。

```cs
using Microsoft.Bot.Builder.Core.Extensions;
```

# <a name="javascripttabjs"></a>[JavaScript](#tab/js)

```javascript
const {InputHints, MessageFactory} = require('botbuilder');
```

---

## <a name="accepting-input"></a>接受輸入

若要指出 Bot 是被動接受輸入，但不等候使用者回應，請將訊息的輸入提示設定為「接受輸入」。 在許多通道上，這會啟用用戶端的輸入方塊並關閉麥克風，但使用者仍可存取。 例如，如果使用者按住 [麥克風] 按鈕，Cortana 會開啟麥克風接受使用者輸入。 下列程式碼會建立訊息來指出 Bot 接受使用者輸入。

# <a name="ctabcs"></a>[C#](#tab/cs)

```csharp
var reply = MessageFactory.Text(
    "This is the text that will be displayed.",
    "This is the text that will be spoken.",
    InputHints.AcceptingInput);
await context.SendActivityAsync(reply).ConfigureAwait(false);
```

# <a name="javascripttabjs"></a>[JavaScript](#tab/js)

```javascript
const basicMessage = MessageFactory.text('This is the text that will be displayed.', 'This is the text that will be spoken.', InputHints.AcceptingInput);
await context.sendActivity(basicMessage);
```

---

## <a name="expecting-input"></a>需要輸入

若要指出 Bot 要等候使用者回應，請將訊息的輸入提示設定為「必須有輸入」。 在許多通道上，這會啟用用戶端的輸入方塊並開啓麥克風。 下列程式碼範例會建立訊息來指出 Bot 需要使用者輸入。

# <a name="ctabcs"></a>[C#](#tab/cs)

```csharp
var reply = MessageFactory.Text(
    "This is the text that will be displayed.",
    "This is the text that will be spoken.",
    InputHints.ExpectingInput);
await context.SendActivityAsync(reply).ConfigureAwait(false);
```

# <a name="javascripttabjs"></a>[JavaScript](#tab/js)

```javascript
const basicMessage = MessageFactory.text('This is the text that will be displayed.', 'This is the text that will be spoken.', InputHints.ExpectingInput);
await context.sendActivity(basicMessage);
```

---

## <a name="ignoring-input"></a>忽略輸入

若要指出 Bot 尚未準備接收使用者輸入，請將訊息的輸入提示設定為「忽略輸入」。 在許多通道上，這會停用用戶端的輸入方塊並關閉麥克風。 下列程式碼範例會建立訊息來指出 Bot 會忽略使用者輸入。

# <a name="ctabcs"></a>[C#](#tab/cs)

```csharp
var reply = MessageFactory.Text(
    "This is the text that will be displayed.",
    "This is the text that will be spoken.",
    InputHints.IgnoringInput);
await context.SendActivityAsync(reply).ConfigureAwait(false);
```

# <a name="javascripttabjs"></a>[JavaScript](#tab/js)

```javascript
const basicMessage = MessageFactory.text('This is the text that will be displayed.', 'This is the text that will be spoken.', InputHints.IgnoringInput);
await context.sendActivity(basicMessage);
```

---

## <a name="default-values-for-input-hint"></a>輸入提示的預設值

如果您未設定訊息的輸入提示，Bot Framework SDK 會使用下列邏輯自動為您設定：

- 如果 Bot 傳送提示，訊息的輸入提示將指定 Bot **需要輸入**。</li>
- 如果 Bot 傳送單一訊息，訊息的輸入提示會指定 Bot **接受輸入**。</li>
- 如果 Bot 傳送一系列的連續訊息，所有訊息 (系列中的最後一則訊息除外) 的輸入提示會指定 Bot 會**忽略輸入**，而且系列中最後一則訊息的輸入提示會指定 Bot 會**接受輸入**。

