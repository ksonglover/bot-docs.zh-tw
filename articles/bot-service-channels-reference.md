---
title: 通道參考
description: Bot Framework 通道參考
keywords: 通道參考, bot 建立器通道, bot framework 通道
author: ivorb
ms.author: v-ivorb
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: tools
ms.date: 03/01/2019
ms.openlocfilehash: 28f284e4d69cbef7a1741d298b3ae9e6e127e9dd
ms.sourcegitcommit: 721bb09f10524b0cb3961d7131966f57501734b8
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 04/12/2019
ms.locfileid: "59541084"
---
# <a name="categorized-activities-by-channel"></a>依通路分類的活動

下表顯示哪些事件 (線上的活動) 會來自於哪些通道。

這是資料表的索引鍵：

符號              | 意義
:------------------:|:------------------------------------------------
:white_check_mark:  |Bot 應會收到此活動
:x:                 |Bot 應**永遠不會**收到此活動
:white_large_square:|目前未決定 Bot 是否會收到此活動

可以將活動有意義地分割成不同的類別。 每個類別中會有一個可能的活動資料表。

<a name="conversational"></a>對話
--------------

 \                      | Cortana            | Direct Line        | Direct Line (網路聊天) | 電子郵件              | Facebook           | GroupMe            | Kik                | Teams              | Slack              | Skype   | 商務用 Skype | Telegram | Twilio  
:---------------------- | :-----:            | :----------------: | :--------------------: |:----:              | :------:           | :-----:            | :-----:            | :---:              | :---:              | :---:   | :------------: | :------: | :----:  
訊息                 | :white_check_mark: | :white_check_mark: | :white_check_mark:     | :white_check_mark: | :white_check_mark: | :white_check_mark: | :white_check_mark: | :white_check_mark: | :white_check_mark: | :white_check_mark: | :white_check_mark:        | :white_check_mark:  | :white_check_mark: 
MessageReaction         | :x:                | :x:                | :x:                    | :x:                | :x:                | :x:                | :x:                | :white_check_mark: | :x:                | :x:      | :x:             | :x:       | :x:      

- 所有的通道均傳送訊息活動。
- 使用對話方塊時，通常應一律將訊息活動傳遞到對話方塊上。
- 即使 MessageReaction 是對話很重要的一部分，上述規則可能不適用於 MessageReaction。
- 邏輯上有兩種類型的 MessageReaction：新增和移除


> [!TIP]
> 「訊息反應」就像是之前意見中的「讚」。 可能會隨機發生，因此可以視為與按鈕類似。 此活動目前是由 Teams 通道傳送。  


<a name="welcome"></a>歡迎使用
-------

 \                         | Cortana            | Direct Line        | Direct Line (網路聊天) | 電子郵件   | Facebook             | GroupMe | Kik     | Teams   | Slack   | Skype   | 商務用 Skype | Telegram | Twilio  
:----------------------    | :-----:            | :---------:        | :--------------------: |:----:   | :------:             | :-----: | :-----: | :---:   | :---:   | :---:   | :------------: | :------: | :----:  
ConversationUpdate         | :white_check_mark: | :white_check_mark: | :white_check_mark:     | :x:     | :white_large_square: | :white_check_mark: | :white_check_mark: | :white_check_mark: | :white_check_mark: | :x:      | :x:             | :white_check_mark:  | :x:     
ContactRelationUpdate      | :x:                | :x:                | :x:                    | :x:     | :x:                  | :x:      | :x:      | :x:      | :x:      | :white_check_mark: | :white_check_mark:        | :x:       | :x:     

- 通道通常會傳送 ConversationUpdate 活動。
- 邏輯上有兩種類型的 MessageReaction：新增和移除
- 如此很容易讓人假設透過連接 ConversationUpdate.Added 即可簡單地實作 Bot「歡迎使用」行為，而這麼做有時的確可行。
- 不過，這是簡化的方法，為了產生可靠的「歡迎使用」行為，Bot 實作也可能需要使用狀態。


<a name="application-extensibility"></a>應用程式擴充性
-------------------------

 \                      | Cortana            | Direct Line        | Direct Line (網路聊天) | 電子郵件   | Facebook | GroupMe | Kik     | Teams   | Slack   | Skype   | 商務用 Skype | Telegram | Twilio  
:---------------------- | :-----:            | :---------:        | :--------------------: |:----:   | :------: | :-----: | :-----: | :---:   | :---:   | :---:   | :------------: | :------: | :----:  
Event.*                    | :white_large_square: | :white_check_mark: | :white_check_mark:    | :white_large_square: | :white_large_square:  | :white_large_square: | :white_large_square: | :white_large_square:  | :white_large_square:  | :white_large_square:  | :white_large_square:           | :white_large_square:     | :white_large_square:  
Event.CreateConversation   | :white_large_square: | :white_large_square: | :white_large_square:   | :white_large_square: | :white_large_square:  | :white_large_square: | :white_large_square: | :white_large_square:  | :white_large_square:  | :white_large_square:  | :white_large_square:           | :white_large_square:     | :white_large_square:  
Event.ContinueConversation | :white_large_square: | :white_large_square: | :white_large_square:   | :white_large_square: | :white_large_square:  | :white_large_square: | :white_large_square: | :white_large_square:  | :white_large_square:  | :white_large_square:  | :white_large_square:           | :white_large_square:     | :white_large_square:  

- 事件活動是 Direct Line (_aka Web Chat_) 中的一個擴充性機制。
- 同時擁有用戶端和伺服器的應用程式，可以選擇使用此事件活動透過服務來打開自己事件的通道。


<a name="microsoft-teams"></a>Microsoft Teams
------------------

 \                      | Cortana            | Direct Line        | Direct Line (網路聊天) | 電子郵件   | Facebook | GroupMe | Kik     | Teams   | Slack   | Skype   | 商務用 Skype | Telegram | Twilio  
:---------------------- | :-----:            | :---------:        | :--------------------: |:----:   | :------: | :-----: | :-----: | :---:   | :---:   | :---:   | :------------: | :------: | :----:  
Invoke.TeamsVerification   | :x:      | :x:          | :x: | :x:   | :x:       | :x:      | :x:      | :white_check_mark: | :x:    | :x:    | :x:             | :x:       | :x:     
Invoke.ComposeResponse     | :x:      | :x:          | :x: | :x:   | :x:       | :x:      | :x:      | :white_check_mark: | :x:    | :x:    | :x:             | :x:       | :x:     

- 除了許多其他類型的活動外，Microsoft Teams 還定義了一些特定叫用活動的小組。
- 叫用活動是專屬於應用程式，而不是用戶端定義的內容。
- 活動的「叫用」特定子類型，沒有通用概念。
- 叫用是 Bot 上目前唯一會觸發要求-回覆行為的活動。

這非常重要：若使用 OAuth 提示的對話方塊，則 Invoke.TeamsVerification 活動必須轉寄給對話方塊。


<a name="message-update"></a>訊息更新
--------------

 \                      | Cortana            | Direct Line        | Direct Line (網路聊天) | 電子郵件   | Facebook | GroupMe | Kik     | Teams   | Slack   | Skype   | 商務用 Skype | Telegram | Twilio  
:---------------------- | :-----:            | :---------:        | :--------------------: |:----:   | :------: | :-----: | :-----: | :---:   | :---:   | :---:   | :------------: | :------: | :----:  
MessageUpdate | :x:      | :x:          | :x:    | :x: | :x:      | :x:      | :x:  | :white_check_mark: | :white_large_square:  | :x:    | :x:             | :x:       | :x:     
MessageDelete | :x:      | :x:          | :x:    | :x: | :x:      | :x:      | :x:  | :white_check_mark: | :white_large_square:  | :x:    | :x:             | :x:       | :x:     

- 訊息更新目前由 Teams 提供支援。


<a name="oauth"></a>OAuth
-------

 \                      | Cortana            | Direct Line        | Direct Line (網路聊天) | 電子郵件   | Facebook | GroupMe | Kik     | Teams   | Slack   | Skype   | 商務用 Skype | Telegram | Twilio  
:---------------------- | :-----:            | :---------:        | :--------------------: |:----:   | :------: | :-----: | :-----: | :---:   | :---:   | :---:   | :------------: | :------: | :----:  
Event.TokenResponse| :white_large_square:  | :white_check_mark:   | :white_check_mark:    | :x:    | :white_large_square: | :white_large_square: | :white_large_square: | :x:    | :white_large_square: | :white_large_square: | :white_large_square:       | :white_large_square: | :white_large_square: 

這非常重要：若使用 OAuth 提示的對話方塊，則 Event.TokenResponse Activity 活動必須轉寄給對話方塊。


<a name="uncategorized"></a>未分類 
-------------

 \                      | Cortana  | Direct Line        | Direct Line (網路聊天) | 電子郵件 | Facebook | GroupMe | Kik     | Teams | Slack | Skype | 商務用 Skype | Telegram | Twilio  
:---------------------- | :-----:  | :---------:        | :--------------------: |:----: | :------: | :-----: | :-----: | :---: | :---: | :---: | :------------: | :------: | :----:  
EndOfConversation       | :x:      | :white_check_mark: | :white_check_mark:     | :x:   | :x:      | :x:     | :x:     | :x:   | :x:   | :x:   | :x:            | :x:      | :x:     
InstallationUpdate      | :x:      | :white_check_mark: | :white_check_mark:     | :x:   | :x:      | :x:     | :x:     | :x:   | :x:   | :x:   | :x:            | :x:      | :x:     
輸入                  | :x:      | :white_check_mark: | :white_check_mark:     | :x:   | :x:      | :x:     | :x:     | :x:   | :x:   | :x:   | :x:            | :x:      | :x:     
Handoff                 | :x:      | :x:                | :x:                    | :x:   | :x:      | :x:     | :x:     | :x:   | :x:   | :x:   | :x:            | :x:      | :x:     


<a name="out-of-use-includes-payment-specific-invoke"></a>不使用 (包括付款特定叫用)
---------------------------------------------
- DeleteUserData 
- Invoke.PaymentRequest  
- Invoke.Address
- Ping

---

## <a name="summary-of-activities-supported-per-channel"></a>每個通道支援活動的摘要

<a name="cortana"></a>Cortana
-------
- 訊息
- ConversationUpdate
- _Event.TokenResponse_
- _EndOfConversation (在視窗關閉時？)_

<a name="direct-line"></a>Direct Line
--------
- 訊息
- ConversationUpdate
- Event.TokenResponse
- Event.*
- _Event.CreateConversation_
- _Event.ContinueConversation_

<a name="email"></a>電子郵件
-----
- 訊息

<a name="facebook"></a>Facebook
--------
- 訊息
- _Event.TokenResponse_

<a name="groupme"></a>GroupMe
-------
- 訊息
- ConversationUpdate
- _Event.TokenResponse_

<a name="kik"></a>Kik
---
- 訊息
- ConversationUpdate
- _Event.TokenResponse_

<a name="teams"></a>Teams
-----
- 訊息
- ConversationUpdate
- MessageReaction
- MessageUpdate
- MessageDelete
- Invoke.TeamsVerification
- Invoke.ComposeResponse


<a name="slack"></a>Slack
-----
- 訊息
- ConversationUpdate
- _Event.TokenResponse_


<a name="skype"></a>Skype
-----
- 訊息
- ContactRelationUpdate
- _Event.TokenResponse_


<a name="skype-business"></a>商務用 Skype
--------------
- 訊息
- ContactRelationUpdate 
- _Event.TokenResponse_


<a name="telegram"></a>Telegram
--------
- 訊息
- ConversationUpdate
- _Event.TokenResponse_


<a name="twilio"></a>Twilio
------
- 訊息

## <a name="summary-table-all-activities-to-all-channels"></a>所有通道的所有活動摘要表

 \                         | Cortana              | Direct Line          | Direct Line (網路聊天) | 電子郵件                | Facebook             | GroupMe | Kik     | Teams   | Slack   | Skype   | 商務用 Skype | Telegram | Twilio  
:----------------------    | :-----:              | :---------:          | :--------------------: |:----:                | :------:             | :-----: | :-----: | :---:   | :---:   | :---:   | :------------: | :------: | :----:  
訊息                    | :white_check_mark:   | :white_check_mark:   | :white_check_mark:     | :white_check_mark:   | :white_check_mark:   | :white_check_mark: | :white_check_mark: | :white_check_mark: | :white_check_mark: | :white_check_mark: | :white_check_mark:        | :white_check_mark:  | :white_check_mark: 
MessageReaction            | :x:                  | :x:                  | :x:                    | :x:                  | :x:                  | :x:      | :x:      | :white_check_mark: | :x:      | :x:      | :x:             | :x:       | :x:      
ConversationUpdate         | :white_check_mark:   | :white_check_mark:   | :white_check_mark:     | :x:                  | :white_large_square: | :white_check_mark: | :white_check_mark: | :white_check_mark: | :white_check_mark: | :x:      | :x:             | :white_check_mark:  | :x:     
ContactRelationUpdate      | :x:                  | :x:                  | :x:                    | :x:                  | :x:                  | :x:      | :x:      | :x:      | :x:      | :white_check_mark: | :white_check_mark:        | :x:       | :x:     
Event.*                    | :white_large_square: | :white_check_mark:   | :white_check_mark:     | :white_large_square: | :white_large_square: | :white_large_square: | :white_large_square: | :white_large_square:  | :white_large_square:  | :white_large_square:  | :white_large_square:           | :white_large_square:     | :white_large_square:  
Event.CreateConversation   | :white_large_square: | :white_large_square: | :white_large_square:   | :white_large_square: | :white_large_square: | :white_large_square: | :white_large_square: | :white_large_square:  | :white_large_square:  | :white_large_square:  | :white_large_square:           | :white_large_square:     | :white_large_square:  
Event.ContinueConversation | :white_large_square: | :white_large_square: | :white_large_square:   | :white_large_square: | :white_large_square: | :white_large_square: | :white_large_square: | :white_large_square:  | :white_large_square:  | :white_large_square:  | :white_large_square:           | :white_large_square:     | :white_large_square:  
Invoke.TeamsVerification   | :x:                  | :x:                  | :x:                    | :x:                  | :x:                  | :x:      | :x:      | :white_check_mark: | :x:    | :x:    | :x:             | :x:       | :x:     
Invoke.ComposeResponse     | :x:                  | :x:                  | :x:                    | :x:                  | :x:                  | :x:      | :x:      | :white_check_mark: | :x:    | :x:    | :x:             | :x:       | :x:     
MessageUpdate              | :x:                  | :x:                  | :x:                    | :x:                  | :x:                  | :x:      | :x:  | :white_check_mark: | :white_large_square:  | :x:    | :x:             | :x:       | :x:     
MessageDelete              | :x:                  | :x:                  | :x:                    | :x:                  | :x:                  | :x:      | :x:  | :white_check_mark: | :white_large_square:  | :x:    | :x:             | :x:       | :x:     
Event.TokenResponse        | :white_large_square: | :white_check_mark:   | :white_check_mark:     | :x:                  | :white_large_square: | :white_large_square: | :white_large_square: | :x:    | :white_large_square: | :white_large_square: | :white_large_square:       | :white_large_square: | :white_large_square: 
EndOfConversation          | :x:                  | :white_check_mark:   | :white_check_mark:     | :x:                  | :x:      | :x:     | :x:     | :x:   | :x:   | :x:   | :x:            | :x:      | :x:     
InstallationUpdate         | :x:                  | :white_check_mark:   | :white_check_mark:     | :x:                  | :x:      | :x:     | :x:     | :x:   | :x:   | :x:   | :x:            | :x:      | :x:     
輸入                     | :x:                  | :white_check_mark:   | :white_check_mark:     | :x:                  | :x:      | :x:     | :x:     | :x:   | :x:   | :x:   | :x:            | :x:      | :x:     
Handoff                    | :x:                  | :x:                  | :x:                    | :x:                  | :x:      | :x:     | :x:     | :x:   | :x:   | :x:   | :x:            | :x:      | :x:     

## <a name="web-chat"></a>網路聊天 
網路聊天將傳送：
- "message"：包含「文字」和/或「附件」
- "event"：包含「名稱」和「值」(如 JSON/字串)
- "typing"：如果使用者設定選項，亦即 "sendTypingIndicator" 網路聊天不會傳送 "contactRelationUpdate"。 而網路聊天不支援 "messageReaction"，未明確要求我們支援這項功能。

根據預設，網路聊天會轉譯：
- "message"：會根據活動中的選項轉譯為浮動切換或堆疊
- "typing"：將轉譯 5 秒並隱藏，或直到下一個活動開始
- "conversationUpdate"：會隱藏
- "event"：會隱藏
- 其他：會顯示警告方塊 (我們從未在生產環境中看到)。您可以將此轉譯管線修改為新增、移除或取代任何自訂轉譯。

您可以使用網路聊天傳送任何活動類型和承載，我們不會以文件記錄也不會建議這項功能。 您應該改為使用 "event" 活動。
