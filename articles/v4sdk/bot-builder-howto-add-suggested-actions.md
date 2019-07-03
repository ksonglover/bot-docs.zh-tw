---
title: 使用按鈕進行輸入 | Microsoft Docs
description: 了解如何使用適用於 JavaScript 的 Bot Framework SDK 在訊息內傳送建議的動作。
keywords: 建訢動作, 按鈕, 額外輸入
author: Kaiqb
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 05/23/2019
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 43c9b0f062f4db909612f3fb9dc048be65c64c57
ms.sourcegitcommit: d29d3d7ccef401aa1e84e19e623db33b5ff13e63
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/17/2019
ms.locfileid: "67160635"
---
# <a name="use-button-for-input"></a>使用按鈕進行輸入

[!INCLUDE[applies-to](../includes/applies-to.md)]

您可以讓 Bot 顯示可供使用者點選，以提供輸入的按鈕。 按鈕讓使用者只要點選按鈕，即可回答問題或進行選取，而無需使用鍵盤輸入回應，藉以提升使用者體驗。 不同於複合式資訊卡 (Rich Card) 中出現的按鈕 (即使在點選之後，使用者仍可看見並可存取)，出現在建議動作窗格內的按鈕會在使用者進行選取後消失。 這可以避免使用者在對話中點選過時的按鈕，並簡化 Bot 的開發 (因為您不需要再說明該情境)。 

## <a name="suggest-action-using-button"></a>使用按鈕建議動作

「建議動作」  可讓 Bot 顯示按鈕。 您可以建立一份將在交談單一回合顯示給使用者的建議動作清單 (也稱為「快速回覆」)： 

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

這裡顯示的原始程式碼是以[建議動作範例](https://aka.ms/SuggestedActionsCSharp)為基礎。

[!code-csharp[suggested actions](~/../botbuilder-samples/samples/csharp_dotnetcore/08.suggested-actions/Bots/SuggestedActionsBot.cs?range=87-101)]

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

這裡顯示的原始程式碼是以[建議動作範例](https://aka.ms/SuggestActionsJS)為基礎。

[!code-javascript[suggested actions](~/../botbuilder-samples/samples/javascript_nodejs/08.suggested-actions/bots/suggestedActionsBot.js?range=61-64)]

---

## <a name="additional-resources"></a>其他資源

您可以在此存取完整的原始程式碼：[CSharp 範例](https://aka.ms/SuggestedActionsCSharp)或 [JavaScript 範例](https://aka.ms/SuggestActionsJS)。

## <a name="next-steps"></a>後續步驟

> [!div class="nextstepaction"]
> [儲存使用者和對話資料](./bot-builder-howto-v4-state.md)
