---
title: 定義程式碼辨識器作為工作觸發程序 | Microsoft Docs
description: 了解如何使用自訂程式碼辨識器作為工作觸發程序。
author: vkannan
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 12/13/2017
ROBOTS: NoIndex, NoFollow
ms.openlocfilehash: 6d44cd299e46971b2218bb78d16e454d5df3c1d9
ms.sourcegitcommit: f576981342fb3361216675815714e24281e20ddf
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/18/2018
ms.locfileid: "39298796"
---
# <a name="define-code-recognizer-as-task-trigger"></a>定義程式碼辨識器作為工作觸發程序
> [!IMPORTANT]
> 對話設計工具尚未提供給所有客戶。 有關對話設計工具可用性的詳細資訊會於本年度稍後提供。

程式碼辨識器可讓您編寫自訂指令碼，以協助執行工作。 Regex 型的運算式評估或呼叫其他服務，可用來協助判斷使用者的意圖。 自訂指令碼將指示對話執行階段來觸發工作。 

## <a name="create-a-script-function"></a>建立指令碼函式
若要使用指令碼作為辨識器，在工作編輯器中，選擇「指令碼函式」作為辨識器、指定函式的名稱，然後按一下 [建立/檢視函式] 以開始編輯指令碼。 您也可以按一下 [建立/檢視函式]，但不指定指令碼名稱，即會使用預設名稱為您建立空白函式。 

## <a name="script-trigger-function-parameter"></a>指令碼觸發程序函式參數

指定的指令碼回呼函式將一律使用 [`context`](conversation-designer-context-object.md) 物件來呼叫。

`context` 物件同時包含 `taskEntities` 和 `contextEntities`。 工作實體是已針對此工作定義或產生的實體，而內容實體是屬性包，其中的實體可在與使用者對話期間加以保留。

## <a name="return-value"></a>傳回值

**指令碼觸發程序**函式預期會傳回布林值。

## <a name="sample-regex-based-recognizer"></a>範例 Regex 型辨識器
下列程式碼範例會使用 Regex 來處理要求並傳回布林值，讓對話執行階段可以判斷要執行哪一個指令碼觸發程序。

```javascript
module.exports.fnFindPhoneTrigger = function(context) {
    if (context.request.text.includes("call") || context.request.text.includes("ring")) {
        return true;
    } else {
        return false;
    }
} 
```

## <a name="next-step"></a>後續步驟
> [!div class="nextstepaction"]
> [回覆動作](conversation-designer-reply.md)
