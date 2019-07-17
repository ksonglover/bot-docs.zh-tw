---
ms.openlocfilehash: 91b2729aa10c8ac1985e62845126296e8141ef77
ms.sourcegitcommit: fa6e775dcf95a4253ad854796f5906f33af05a42
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/16/2019
ms.locfileid: "68230630"
---
> [!IMPORTANT]
> 處理主要 Bot 回合的執行緒可在其完成後處置內容物件。 **請務必`await`任何活動呼叫**，這樣主要執行緒才能先等候已產生的活動，再結束其正在處理的工作並處置回合內容。 否則，如果回應 (包含其處理常式) 佔用大量時間，並嘗試在內容物件上動作，則可能會取得「內容已處置」  錯誤。