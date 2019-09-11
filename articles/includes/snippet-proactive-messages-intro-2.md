---
ms.openlocfilehash: 4108f02a350d7e02d66db06cd7e65fa9c7a98e7d
ms.sourcegitcommit: fa6e775dcf95a4253ad854796f5906f33af05a42
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/16/2019
ms.locfileid: "70297366"
---
**臨機操作主動式訊息**是最簡單的主動式訊息類型。
Bot 只會在每次觸發時將訊息插入對話，不會顧及使用者目前是否與 Bot 在其他對話主題中，也不會嘗試以任何方式變更對話。

若要更順利地處理通知，請考慮使用其他方式將通知整合到對話流程中，例如在對話狀態中設定旗標，或將通知新增至佇列。

<!--Snip
A **dialog-based proactive message** is more complex than an ad hoc proactive message. 
Before it can inject this type of proactive message into the conversation, 
the bot must identify the context of the existing conversation and decide how (or if)
it will resume that conversation after the message interrupts. 

For example, consider a bot that needs to initiate a survey at a given point in time. 
When that time arrives, the bot stops the existing conversation with the user and 
redirects the user to a `SurveyDialog`. 
The `SurveyDialog` is added to the top of the dialog stack and takes control of the conversation. 
When the user finishes all required tasks at the `SurveyDialog`, the `SurveyDialog` closes,
 returning control to the previous dialog, where the user can continue with the prior topic of conversation.

A dialog-based proactive message is more than just simple notification. 
In sending the notification, the bot changes the topic of the existing conversation. 
It then must decide whether to resume that conversation later, or to abandon that conversation altogether by resetting the dialog stack. 
/Snip-->