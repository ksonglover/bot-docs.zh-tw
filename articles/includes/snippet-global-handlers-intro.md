---
ms.openlocfilehash: b5809b6d46cdc09035efb36c3ea58c2ca9dc6c3e
ms.sourcegitcommit: fa6e775dcf95a4253ad854796f5906f33af05a42
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/16/2019
ms.locfileid: "68230726"
---
使用者通常會嘗試使用「說明」、「取消」或「重新開始」等關鍵字，來存取 Bot 內的某些功能。 這通常會發生在對話中，而 Bot 這時會預期不同的回應。 藉由實作**全域訊息處理常式**，您可以將 Bot 設計為會以妥善的方式處理這類要求。
處理常式會檢查使用者輸入中是否有您指定的關鍵字 (例如，「說明」、「取消」或「重新開始」)，並做出適當回應。 

![使用者的說話方式](~/media/designing-bots/capabilities/trigger-actions.png)

> [!NOTE]
> 在全域訊息處理常式中定義邏輯，可讓所有對話方塊都能存取該邏輯。 您可以設定個別的對話方塊和提示，安全地忽略關鍵字。
