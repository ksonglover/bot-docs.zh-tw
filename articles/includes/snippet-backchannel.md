---
ms.openlocfilehash: b320aadc876074a76fe209ad55a81cb70b1ddcac
ms.sourcegitcommit: fa6e775dcf95a4253ad854796f5906f33af05a42
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/16/2019
ms.locfileid: "68230554"
---
<a href="https://github.com/Microsoft/BotFramework-WebChat" target="_blank">開放原始碼網站聊天控制項</a>使用 [直接線路 API](https://docs.botframework.com/restapi/directline3/#navtitle) 與 Bot 進行通訊，如此會允許 `activities` 在用戶端與 Bot 之間來回傳送。 最常見的活動類型是 `message`，但也有其他類型。 比方說，活動類型 `typing` 指明使用者正在輸入或是 bot 正在編譯回覆的工作。 

您可以使用 backchannel 機制在用戶端與 Bot 之間交換資訊，而不需要藉由將活動類型設定為 `event` 而將這些資料呈現給使用者。 網路聊天控制項會自動忽略 `type="event"` 的任何活動。