---
ms.openlocfilehash: 86faca976bc95a5e91e17749096cd148483edc61
ms.sourcegitcommit: 980612a922b8290b2faadaca193496c4117e415a
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 04/26/2019
ms.locfileid: "64563692"
---
## <a name="payment-process-overview"></a>付款程序概觀

付款程序包含三個不同的部分：

1. Bot 傳送付款要求。

2. 使用者登入 Microsoft 帳戶來提供付款、運送和連絡資訊。 回呼會傳送至 Bot 以指出 Bot 何時需要執行某些作業 (更新運送地址、更新運送選項、完成付款)。

3. Bot 會處理收到的回呼，包括更新運送地址、更新運送選項和完成付款。 

Bot 只需要實作此程序的步驟一和步驟三，步驟二會在 Bot 的內容之外進行。 
