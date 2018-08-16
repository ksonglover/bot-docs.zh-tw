---
title: 定義回覆作為執行動作 | Microsoft Docs
description: 了解如何定義回覆作為執行動作。
author: vkannan
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 12/13/2017
ROBOTS: NoIndex, NoFollow
ms.openlocfilehash: 344a771a55cf729587863a70398ef944863ac738
ms.sourcegitcommit: f576981342fb3361216675815714e24281e20ddf
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/18/2018
ms.locfileid: "39299875"
---
# <a name="define-a-reply-as-a-do-action"></a>定義回覆作為執行動作

回覆可以是顯示文字、語音文字或豐富卡片的任意組合。 透過此動作，Bot 會將回覆傳送給使用者，並完成工作。 回覆是單一回合的對話，這表示工作不會預期來自使用者的後續追蹤訊息。 若要將回覆設定為「回覆」動作，請從 [執行] 動作下拉式清單中選取 [提供回覆]。 然後，您可以將簡單的回應輸入到 [Bot 給使用者的回應] 欄位中。

您可以選擇性地包含[調適型卡片](conversation-designer-adaptive-cards.md)與簡單的回應。 您也可以在傳送回應給使用者之前，先選擇性地提供自訂指令碼函式來執行。 當您建立或編輯工作時，這些選項可供您使用。 

## <a name="next-step"></a>後續步驟
> [!div class="nextstepaction"]
> [指令碼動作](conversation-designer-script-function.md)
