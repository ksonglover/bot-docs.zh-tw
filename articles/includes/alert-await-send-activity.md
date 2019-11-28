---
ms.openlocfilehash: 99bd2b58bafd1c4ece57aad5decc710346f56f86
ms.sourcegitcommit: 08f9dc91152e0d4565368f72f547cdea1885af89
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 11/26/2019
ms.locfileid: "74536366"
---
> [!IMPORTANT]
> 處理主要 Bot 回合的執行緒可在其完成後處置內容物件。 **請務必`await`任何活動呼叫**，這樣主要執行緒才能先等候已產生的活動，再結束其正在處理的工作並處置回合內容。 否則，如果回應 (包含其處理常式) 佔用大量時間，並嘗試在內容物件上動作，則可能會取得「內容已處置」  錯誤。