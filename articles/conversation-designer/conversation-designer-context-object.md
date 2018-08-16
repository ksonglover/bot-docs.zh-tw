---
title: 適用於內容物件的 API 參考 | Microsoft Docs
description: 了解如何在對話設計工具 Bot 中參考內容物件。
author: vkannan
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 12/13/2017
ROBOTS: NoIndex, NoFollow
ms.openlocfilehash: 5ba70189c16815539524d3c9046da03ed0344b9b
ms.sourcegitcommit: f576981342fb3361216675815714e24281e20ddf
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/18/2018
ms.locfileid: "39298793"
---
# <a name="api-reference"></a>API 參考
> [!IMPORTANT]
> 對話設計工具尚未提供給所有客戶。 有關對話設計工具可用性的詳細資訊會於本年度稍後提供。

對話設計工具讓您能夠將自訂商務邏輯新增到您的 Bot。 這些指令碼函式都會在**指令碼**編輯器中加以實作。 函式會在不同位置中連結，例如，條件式回應範本、此工作的「觸發程序」或「動作」，或是對話狀態，而且它們全部都會在其函式參數中接收到類型為 **[IConversationContext]** 的 `context` 物件。

下列程式碼範例所示的簽章適用於含有所傳入之 **context** 物件的條件式回應函式。

```javascript
/**
* @param {IConversationContext} context
*/
module.exports.NewConditionalResponse_onRun = function(context) {
    // Business logic here
    return true; // Returns a boolean
}
```

透過 `context` 物件，您可以存取使用者與 Bot 之間對話的相關資訊。

## <a name="script-callback-functions"></a>指令碼回呼函式

您所建立的自訂指令碼回呼函式可能會採用多種形式。 雖然您可以為它們提供不同的名稱，但在功能上，它們會採用下列其中一種形式。

| 形式 | 參數 | 傳回類型 | 說明 |
| ---- | ---- | ---- | ---- |
| 回應之前的函式 | context | **void** 或 **promise** | 在提供回應之前要執行的函式。 |
| 處理函式 | context | **void** 或 **promise** | 執行商務邏輯的函式。 |
| 決策函式 | context | **string** 或 **promise** | 可根據商務邏輯進行決策的函式。 傳回字串應該符合來自[決策](conversation-designer-dialogues.md#decision-state)區塊的條件。 |
| 程式碼辨識器函式 | context | **boolean** 或 **promise** | 在發生**指令碼觸發程序**時執行的自訂商務邏輯。 傳回 `true` 以指出相符項目。 否則，會傳回 `false` 以取消相符項目。 |
| onRecognize 函式 | context | **boolean** 或 **promise** | 只有在有來自 LUIS 辨識器的相符項目時才會執行的函式。 使用這個回呼函式來處理 LUIS 實體，並傳回適當的 **boolean** 值。 傳回 `true` 以指出相符項目。 否則，會傳回 `false` 以取消相符項目。 |

## <a name="iconversationcontext-interface"></a>IConversationContext 介面

`IConversationContext` 介面會追蹤使用者與 Bot 之間的對話資訊。 所有透過對話設計工具連結的自訂函式都將接收到 `context` 物件作為參數引數。

## <a name="context-properties"></a>內容屬性
`context` 物件會公開下列屬性。

| Name |  代碼 | 說明 |
| ---- | ---- | ---- |
| `request` | `context.request` | 取得包含 Bot 活動的要求物件。  |
| | `context.request.attachment` | 可能包含調適型卡片的附加活動。 |
| | `context.request.text` | 包含來自用戶端之傳入文字訊息的文字活動。 |
| | `context.request.speak` | 包含來自用戶端之口說文字 (如果可用) 的語音活動。 |
| | `context.request.type` | 指定活動類型 (預設值：「訊息」)。 |
| `responses` | `context.responses` | 維護一個活動陣列，這些活動將在目前狀態或程式碼後置執行結束時傳送回用戶端。 |
| | `context.responses.push` | 將活動新增至回應。 |
| `global` | `context.global` | 包含您所定義之對話式資料的 JavaScript 物件。 此物件會在整個對話期間持續存在。 |
| `local` | `context.local` | 包含您所定義之工作資料的 JavaScript 物件。 此物件會在特定工作期間持續存在。 LUIS 意圖一律會傳回給本機內容。 如果您想要保存 LUIS 結果，請考慮將它複製到 `context.global` 內容。 |
| | `context.local['@description']` | 傳回從 LUIS 接收到的原始實體。 |
| `sticky` | `context.sticky` | 指出目前的工作名稱 |
| `currentTemplate` | `context.currentTemplate` | 一個[條件式回應範本](conversation-designer-response-templates.md#conditional-response-templates)，可呼叫來進行顯示與語音評估。 此物件包含下列三個屬性： <br/>1. **name**：目前範本的名稱。 <br/>2. **modalityDisplay**：指出形式是否與顯示評估相關聯的布林值。 <br/>3. **modalitySpeak**：指出形式是否與語音評估相關聯的布林值。 |

## <a name="context-methods"></a>內容方法
`context` 物件會公開下列方法。

| Name | 傳回類型 | 代碼 | 說明 |
| ---- | ---- | ---- | ---- |
| `getCurrentTurn` | **number** | `context.getCurrentTurn();` | 如果您正在執行重新提示，請從堆疊頂端的框架中取得該回合。 |

## <a name="next-step"></a>後續步驟
> [!div class="nextstepaction"]
> [建立 Bot](conversation-designer-create-bot.md)
