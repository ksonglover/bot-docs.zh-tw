---
title: 測試 Bot | Microsoft Docs
description: 測試和偵錯您的對話設計工具 Bot。
author: vkannan
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 12/13/2017
ROBOTS: NoIndex, NoFollow
ms.openlocfilehash: b08bd96bc7c413ff7cb6db4899c8c0d3ad613ab8
ms.sourcegitcommit: f576981342fb3361216675815714e24281e20ddf
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/18/2018
ms.locfileid: "39300374"
---
# <a name="test-your-conversation-designer-bot"></a>測試對話設計工具 Bot
> [!IMPORTANT]
> 對話設計工具尚未提供給所有客戶使用。 有關對話設計工具可用性的詳細資訊會於本年度稍後提供。

對話設計工具會透過數個不同的輸出視窗 (位於畫面底)，提供實用的偵錯工具。 按一下視窗右上角的 [測試] 按鈕，即可開始進行測試和偵錯。 

## <a name="validation-errors"></a>驗證錯誤
發生驗證錯誤有幾個條件。 例如︰ 
- 您缺少觸發程序的回應 
- 在流程上部分完成資訊
- 缺少條件式回應範本、決策和程序狀態的觸發程序和後置程式碼
- 在範本參考中偵測到的遞迴或無限迴圈 
- 您的對話方塊有未連線至流程的孤立狀態。
- 您新增了一項工作，但缺少觸發程序或動作 


## <a name="javascript-output"></a>JavaScript 輸出
您可以將指令碼附加至回應的執行，以提供其他功能。 如果指令碼中有錯誤，它將會在此顯示。 您可以將 `console.log()`、`error.log()` 或 `debug.log()` 新增至您的 Bot 商務邏輯，其會在輸出視窗中列出訊息。 例如︰

``` javascript
module.exports.Respond_beforeResponse = function(context) {
    console.log(JSON.stringify(context));
}
```

## <a name="runtime-output"></a>執行階段輸出
執行階段所產生的錯誤或例外狀況會在此顯示。 例如，如果您的 Bot 無法回應，請查看輸出以調查造成例外狀況或錯誤的原因。 如果輸出視窗中有太多訊息，您可以按一下 [全部清除]，然後藉由傳送一則訊息給 Bot，再次測試您的 Bot，以查看錯誤詳細資料。 

## <a name="next-step"></a>後續步驟
> [!div class="nextstepaction"]
> [匯入和匯出 Bot](conversation-designer-export-import-bot.md)
