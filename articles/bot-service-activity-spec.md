---
title: Bot Framework 規格 | Microsoft Docs
description: Bot Framework 規格
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 03/07/2018
ms.openlocfilehash: 06e2289dd0176364467d34846ffa7716483f6578
ms.sourcegitcommit: b78fe3d8dd604c4f7233740658a229e85b8535dd
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/24/2018
ms.locfileid: "50000405"
---
# <a name="bot-framework----activity"></a>Bot Framework -- 活動

## <a name="abstract"></a>摘要

Bot Framework 活動結構描述是對話動作的應用程式層級表示，這些動作由人類和自動化軟體來進行。 結構描述包含通訊文字、多媒體和非內容動作 (例如社交互動和輸入指標) 的佈建。

此結構描述會在 Bot Framework 通訊協定內使用，而且由 Microsoft 交談系統和許多資源中的互通型 Bot 及用戶端進行實作。

## <a name="table-of-contents"></a>目錄

1. [簡介](#introduction)
2. [基本活動結構](#basic-activity-structure)
3. [訊息活動](#message-activity)
4. [連絡人關係更新活動](#contact-relation-update-activity)
5. [對話更新活動](#conversation-update-activity)
6. [結束對話活動](#end-of-conversation-activity)
7. [事件活動](#event-activity)
8. [叫用活動](#invoke-activity)
9. [安裝更新活動](#installation-update-activity)
10. [訊息刪除活動](#message-delete-activity)
11. [訊息更新活動](#message-update-activity)
12. [訊息回應活動](#message-reaction-activity)
13. [輸入活動](#typing-activity)
14. [複雜類型](#complex-types)
15. [參考](#references)
16. [附錄 I - 變更](#appendix-i---changes)
17. [附錄 II - 非 IRI 實體類型](#appendix-ii---non-iri-entity-types)
18. [附錄 III - 使用叫用活動的通訊協定](#appendix-iii---protocols-using-the-invoke-activity)

## <a name="introduction"></a>簡介

### <a name="overview"></a>概觀

Bot Framework 活動結構描述代表人類及自動化軟體在聊天應用程式、電子郵件和其他文字互動程式中的對話行為。 每個活動物件皆包含類型欄位及代表一個動作：最常見的是傳送文字內容，但也包括多媒體附件與非內容行為，例如「喜歡」按鈕或輸入指標。

本文件提供每種活動類型的意義，並說明可能包含的必要和選擇性欄位。 也會定義用戶端和伺服器角色，並提供每個參與者須主控哪些欄位，以及可以忽略哪些欄位的指引。

此規格會產生三個角色：用戶端，代表使用者傳送與接收活動；Bot，傳送和接收活動 (通常是自動化架構)；通道，儲存及轉送用戶端與 Bot 之間的活動。

雖然此規格需要在角色之間傳送的活動，但該傳送的確切性質不會在此說明。

為求簡便，此規格中不會定義視覺化的互動式卡片。 但是您可以參閱[調適型卡片](https://adaptivecards.io) [[11](#references)] 規格。 此卡片及其他未定義的卡片類型，可能會是 Bot Framework 活動內的附件。

### <a name="requirements"></a>需求

此文件中「必須 (MUST)」、「不得 (MUST NOT)」、「必要 (REQUIRED)」、「應 (SHALL)」、「不應 (SHALL NOT)」、「應該 (SHOULD)」、「不應該 (SHOULD NOT)」、「建議 (RECOMMENDED)」、「可以 (MAY)」與「選擇性 (OPTIONAL)」需依 [RFC 2119](https://tools.ietf.org/html/rfc2119) [[1](#references)] 所述來解釋。

如果實作不符合一個或多個所執行通訊協定的「必須」或「必要」層級需求，則此實作不符合規範。 如果實作符合其通訊協定的所有「必須」或「必要」層級及所有「應該」層級需求，則視為「無條件符合規範」；若是符合其通訊協定的所有「必須」層級需求，但不完全符合「應該」層級需求，則視為「有條件符合規範」。

### <a name="numbered-requirements"></a>已編號的需求

以 `RXXXX` 標記開頭的行都是特定需求，其設計訴求是為了能在此文件之外的討論中有編號可參照。 這些需求不會比在 `RXXXX` 行之外產生的標準陳述式有更多或更少權重。

`R1000`：此規格的編輯者「可以」新增 `RXXXX` 需求。 他們「應該」要尋找可保留文件流程的 `RXXXX` 數值。

`R1001`：編輯者「不得」對現有的 `RXXXX` 需求進行重新編號。

`R1002`：編輯者「可以」刪除或修改 `RXXXX` 需求。 如果修改了需求，但需求的主題多半維持不變，則編輯者「應該」保留現有的 `RXXXX` 值。

`R1003`：編輯者「不應該」重複使用已淘汰的 `RXXXX` 值。 本文結束時，「可以」繼續維護已刪除的值清單。

### <a name="terminology"></a>術語

activity
> 由 Bot、通道或用戶端表示，並且符合活動結構描述的動作。

channel
> 傳送和接收活動，並將活動來回轉換為聊天或應用程式行為的軟體。 通道是活動資料的授權存放區。

Bot
> 傳送和接收活動，並產生自動化、半自動化或完全手動回應的軟體。 Bot 具有向通道註冊的端點。

用戶端
> 傳送和接收活動的軟體，通常代表人類使用者。 用戶端沒有端點。

傳送者
> 傳送活動的軟體。

接收者
> 接受活動的軟體。

endpoint
> 能以程式設計方式來定址的位置，也就是 Bot 或通道可以接收活動的位置。

位址
> 可與使用者或 Bot 連線的識別碼或位址。

field
> 活動內或巢狀物件內的具名值。

### <a name="overall-organization"></a>整體組織

活動物件是名稱/值配對的一般清單，其中有些是基本物件，有些是複雜物件 (巢狀)。 活動物件通常會以 JSON 格式表示，但也可以投射到 .Net 或 JavaScript 中的記憶體內資料結構。

活動的 `type` 欄位會控制活動和其中欄位的意義。 根據參與者扮演的角色 (用戶端、Bot 或通道)，每個欄位可能是必要、選擇性或可略過欄位。 例如，`id` 欄位由通道控制，所以在某些情況下是必要欄位，但如果由 Bot 來傳送此欄位，則此欄位可忽略。

描述活動和任何參與者身分識別的欄位 (例如 `type` 和 `from` 欄位)，皆可在所有活動之間共用。 對許多程式設計語言來說，在核心基底類型上組織這些欄位相當方便，更具體地說，活動類型就是衍生自基底類型。

當儲存或傳送活動時，有些欄位可能會在傳輸機制中重複。 比方說，如果活動透過 HTTP POST 傳送給包含對話識別碼的 URL，則接收者可以推斷其值，而且此值不需要存在活動的主體內。 本文件只說明這些欄位的抽象需求，而控制通訊協定負責決定這些值是否必須明確宣告，或者是否允許隱含或推斷的值。

當 Bot 或用戶端將活動傳送給通道時，這通常是要記錄的活動要求。 當通道傳送活動給 Bot 或用戶端時，這通常是活動已記錄的通知。

## <a name="basic-activity-structure"></a>基本活動結構

此區段會定義活動物件的基本結構需求。

活動物件包含名稱/值配對的一般清單，我們將其稱為「欄位」。 欄位可能是基本類型。 通常會使用 JSON 作為交換格式，雖然並非所有活動都必須一律序列化為 JSON，但它們必須是可序列化為 JSON 的。 這可讓實作依賴一組簡單的慣例來處理已知和未知的活動欄位。

`R2001`：活動「必須」可以序列化為 JSON 格式，並遵守欄位唯一性等限制。

`R2002`：接收者「可以」允許大小寫不正確的欄位名稱，但這不是必要的。 接收者「可以」拒絕所含欄位未使用正確大小寫的活動。

`R2003`：如果活動包含的欄位值類型不符合此規格中所述的值類型，則接收者「可以」拒絕這些活動。

`R2004`：除非另有註明，否則傳送者「不應該」讓字串欄位包含空的字串值。

`R2005`：除非另有註明，否則傳送者「可以」在活動內或任何巢狀的複雜物件內包含其他欄位。 接收者「必須」接受不了解的欄位。

`R2006`：接收者「應該」接受不了解的事件類型。

### <a name="type"></a>類型

`type` 欄位會控制每個活動的意義，並且都是依照慣例的簡短字串 (例如「`message`」)。 傳送者可以定義自己的應用程式層類型，但選擇的值最好不會與未來定義完善的值相衝突。 如果傳送者會使用 URI 作為類型值，則「不應該」實作 URI 階梯型比較來建立相等值。

`R2010`：活動「必須」包含 `type` 欄位，並且使用字串值類型。

`R2011`：如果兩個 `type` 值在序數上是相同的，則這兩個值才算相等。

`R2012`：傳送者「可以」產生未在這份文件中定義的活動 `type` 值。

`R2013`：通道「應該」拒絕不了解的活動類型。

`R2014`：Bot 或用戶端「應該」略過不了解的活動類型。

### <a name="channel-id"></a>通道識別碼

`channelId` 欄位會建立活動的通道和授權存放區。 `channelId` 欄位的值是字串類型。

`R2020`：活動「必須」包含 `channelId` 欄位，並且使用字串值類型。

`R2021`：如果兩個 `channelId` 值在序數上是相同的，則這兩個值才算相等。

`R2022`：如果沒有預期的 `channelId` 值，則通道「可以」略過或拒絕收到的任何活動。

### <a name="id"></a>ID

一旦在通道中記錄活動之後，`id` 欄位就會建立此活動的身分識別。 由於傳輸中的活動尚未進行記錄，因此沒有身分識別。 並非所有活動都會獲派身分識別 (例如，[輸入活動](#typing-activity)可能永遠不會獲得 `id` 指派。)`id` 欄位的值是字串類型。

`R2030`：通道「應該」包括 `id` 欄位 (如果此欄位可供該活動使用)。

`R2031`：用戶端及 Bot「不應該」在他們產生的活動中包含 `id` 欄位。

為了方便實作，應假設其他參與者沒有複雜的活動識別碼知識，而且他們只會使用序數比較來建立相等性。

比方說，通道可能會使用十六進位編碼的 GUID 作為每個活動的識別碼。 即使以大寫編碼的 GUID 在邏輯上等於以小寫編碼 GUID，但傳送者應該盡可能不要使用這些替代的編碼方式。 每個識別碼的正規化版本皆由授權存放區 (也就是通道) 所建立。

`R2032`：產生 `id` 值時，傳送者「應該」選擇其相等性可由序數比較來建立的值。 不過，對於在序數上不相等的兩個值，如果傳送者和接收者對此特殊狀況有了解，則「可以」允許邏輯上的相等性。

`id` 欄位的設計可允許刪除重複資料，但這在大部分的應用程式中是禁止的。

`R2033`：接收者「可以」根據識別碼來刪除活動，不過傳送者「不應該」依賴接收者執行此刪除重複資料的動作。

### <a name="timestamp"></a>Timestamp

`timestamp` 欄位會記錄活動發生時的確切 UTC 時間。 由於運算系統在本質上是分散性的，因此通道 (授權存放區) 記錄活動時的時間相當重要。 用戶端或 Bot 起始活動時的時間可能會各自在 `localTimestamp` 欄位中傳送。 `timestamp` 欄位的值是在字串內以 [ISO 8601](https://www.iso.org/iso-8601-date-and-time-format.html) [[2](#references)] 編碼的日期時間。

`R2040`：通道「應該」包括 `timestamp` 欄位 (如果此欄位可供該活動使用)。

`R2041`：用戶端及 Bot「不應該」在他們產生的活動中包含 `timestamp` 欄位。

`R2042`：用戶端及 Bot「不應該」使用 `timestamp` 來拒絕活動，因為這些活動出現的順序可能不正確。 不過，他們「可以」在 UI 中或針對下游處理使用 `timestamp` 來排序活動。

`R2043`：傳送者「應該」一律使用「將 `timestamp` 欄位的值編碼為 UTC」，而且他們「應該」一律包含 Z 作為值之中的明確 UTC 標記。

### <a name="local-timestamp"></a>本地時間戳記

`localTimestamp` 欄位表示活動產生時的日期時間和時區位移。 這可能與記錄活動時的 UTC `timestamp` 不同。 `localTimestamp` 欄位的值是在字串內以 ISO 8601 [[3](#references)] 編碼的日期時間。

`R2050`：用戶端及 Bot「可以」在他們的活動中包含 `localTimestamp` 欄位。 他們「應該」在編碼的值之中，明確列出時區位移。

`R2051`：通道將來自傳送者的活動轉送給收件者時，「應該」保留 `localTimestamp`。

### <a name="from"></a>從

`from` 欄位會描述產生活動的用戶端、Bot 或通道。 `from` 欄位值是[通道帳戶](#channel-account)類型的複雜物件。

`from.id` 欄位會識別誰產生活動。 大多數情況下，這會是系統內的另一個使用者或 Bot。 在某些情況下，`from` 欄位會識別通道本身。

`R2060`：通道產生活動時「必須」包含 `from` 和 `from.id` 欄位。

`R2061`：Bot 和用戶端在產生活動時，「應該」包含 `from` 和 `from.id` 欄位。 通道「可以」因為遺漏 `from` 和 `from.id` 欄位而拒絕活動。

`from.name` 是選擇性欄位，代表通道內的帳戶顯示名稱。 通道「應該」包含此值，以便用戶端和 Bot 填入其 UI 和後端系統。 Bot 和用戶端「不應該」傳送此值給具有此存放區中央記錄的通道，但「可以」傳送此值給會在每個活動上填入此值的通道 (例如電子郵件通道)。

`R2062`：如果有 `from` 欄位，而且 `from.name` 有可供使用，則通道「應該」包含 `from.name` 欄位。

`R2063`：Bot 和用戶端「不應該」包含 `from.name` 欄位，除非該欄位對通道內的語意理解有所幫助。

### <a name="recipient"></a>收件者

`recipient` 欄位會描述要接收此活動的用戶端或 Bot。 此欄位只有在將活動確實傳送給一個收件者時才有意義；如果活動會發佈給多個收件者，則不具任何意義 (如同將活動傳送給通道一樣)。 此欄位的用途是讓收件者可識別自己本身。 當用戶端或 Bot 在通道內有多個身分識別時，這會很有幫助。 `recipient` 欄位值是[通道帳戶](#channel-account)類型的複雜物件。

`R2070`：通道在傳送活動給單一收件者時，「必須」包含 `recipient` 和 `recipient.id` 欄位。

`R2071`：Bot 和用戶端在產生活動時，「不應該」包含 `recipient` 欄位。

`recipient.name` 是選擇性欄位，代表通道內的帳戶顯示名稱。 通道「應該」包含此值，以便用戶端和 Bot 填入其 UI 和後端系統。

`R2072`：如果有 `recipient` 欄位，而且有 `recipient.name` 可供使用，則通道「應該」包含 `recipient.name` 欄位。

### <a name="conversation"></a>交談

`conversation` 欄位會描述活動所在的對話。 `conversation` 欄位值是[帳戶](#conversation-account)類型的複雜物件。

`R2080`：通道、Bot 和用戶端在產生活動時，「必須」包含 `conversation` 和 `conversation.id` 欄位。

`conversation.name` 是選擇性欄位，代表對話的顯示名稱 (如果對話存在而且可供使用)。

`R2081`：通道「應該」包含 `conversation.name` 和 `conversation.isGroup` 欄位 (如果有的話)。

`R2082`：Bot 和用戶端「不應該」包含 `conversation.name` 欄位，除非該欄位對通道內的語意理解有所幫助。

`R2083`：Bot 及用戶端「不應該」在他們產生的活動中包含 `conversation.isGroup` 欄位。

### <a name="reply-to-id"></a>回覆識別碼

`replyToId` 欄位會識別以目前活動作為回覆的前一個活動。 此欄位可讓一段對話和評論透過巢狀方式成為參與者之間的對話。 `replyToId` 只在目前的對話中有效。 (請參閱 [relatesTo](#relates-to) 來參照其他對話。)`replyToId` 欄位的值是字串。

`R2090`：當活動是另一個活動的回覆時，傳送者「應該」在該活動上包含 `replyToId`。

`R2091`：如果活動的 `replyToId` 未參考對話內的有效活動，則通道「可以」拒絕該活動。

`R2092`：如果知道通道不會用到 `replyToId` 欄位，即使所傳送活動是給另一個活動的回應，Bot 和用戶端還是「可以」略過該欄位。

### <a name="entities"></a>實體

`entities` 欄位包含此活動相關中繼資料物件的一般清單。 不同於附件 (請參閱[附件](#attachments)欄位)，實體不一定會顯示為可與使用者互動的內容項目，而且如果不是了解的實體則可忽略。 傳送者可以包含他們認為對接收者有用的實體，即使他們不確定接收者是否可以接受這些實體。 每個 `entities` 清單項目的值都是[實體](#entity)類型的複雜物件。

`R2100`：傳送者「應該」略過未包含項目的 `entities` 欄位。

`R2101`：如果實體具有不同意思，傳送者「可以」傳送多個相同類型的實體。

`R2102`：傳送者「不得」包含兩個或多個具有相同類型和內容的實體。

`R2103`：傳送者和接收者「不應該」依賴活動中所包含的特定實體順序。

### <a name="channel-data"></a>通道資料

活動結構描述中的擴充性資料主要會在 `channelData` 欄位中進行組織。 這可簡化 SDK (用於實作通訊協定) 中的管道配置。 傳送或接收活動的通道會定義 `channelData` 物件格式。

`R2200`：通道「不應該」定義屬於 JSON 基本類型 (例如字串、整數) 的 `channelData` 格式。 相反地，應該將 `channelData` 定義為複雜類型或維持未定義狀態。

`R2201`：如果目前通道的 `channelData` 格式為未定義狀態，則接收者「應該」略過 `channelData` 的內容。

### <a name="service-url"></a>服務 URL

活動通常不會同步傳送，並且會使用不同的傳輸連線來傳送和接收流量。 通道會使用 `serviceUrl` 欄位來表示可能會送出回覆給目前活動的 URL。 `serviceUrl` 欄位的值是字串類型。

`R2300`：通道「必須」在傳送給 Bot 的所有活動中包含 `serviceUrl` 欄位。

`R2301`：對於表露出已知道通道端點的用戶端，通道「不應該」在傳送給用戶端的活動中包含 `serviceUrl` 欄位。

`R2302`：Bot 及用戶端「不應該」在他們產生的活動中填入 `serviceUrl` 欄位。

`R2302`：針對 Bot 和用戶端所傳送的活動，通道「必須」略過其中的 `serviceUrl` 值。

`R2304`：通道「應該」使用穩定的值作為 `serviceUrl` 欄位，因為 Bot 可能會長期保留這些值。

## <a name="message-activity"></a>訊息活動

訊息活動代表要在對話介面中顯示的內容。 訊息活動可包含文字、語音、互動式卡片及二進位檔或未知附件；在這些項目中，通道一般最多需要一個來讓訊息活動語式正確。

訊息活動會透過 `message` 的 `type` 值來識別。

### <a name="text"></a>文字

`text` 欄位包含文字內容，可以是 Markdown 格式、XML 或純文字。 格式由 [`textFormat`](#text-format) 欄位控制，如果未指定或模稜兩可，則採用純文字格式。 `text` 欄位的值是字串類型。

`R3000`：`text` 欄位「可以」包含空字串，表示傳送沒有內容的文字。

`R3001`：通道在處理 `markdown` 格式的文字時，「應該」使用可針對該通道適當降級的方式。

### <a name="text-format"></a>文字格式

`textFormat` 欄位會表示 [`text`](#text) 欄位應解譯為 [Markdown](https://daringfireball.net/projects/markdown/) [[4](#references)]、純文字或 XML。 `textFormat` 欄位的值是字串類型，已定義的值為 `markdown`、`plain` 和 `xml`。 預設值為 `plain`。 此欄位並沒有設計為可使用任意值來進行擴充。

`textFormat` 欄位可控制附件等項目中的其他欄位。此關係會在這些欄位中說明 (本文的其他部分)。

`R3010`：如果傳送者包含 `textFormat` 欄位，則「應該」只傳送已定義的值。

`R3011`：如果 `textFormat` 的值為 `plain`，傳送者「應該」略過此欄位。

`R3012`：接收者「應該」將未定義的值解譯為 `plain`。

`R3013`：Bot 和用戶端「不應該」傳送 `xml` 值，除非他們事先知道通道可支援此值，以及具有所支援 XML 方言的特性。

`R3014`：通道「不應該」將 `markdown` 或 `xml` 內容傳送給 Bot。

`R3015`：通道至少「應該」接受使用 `plain` 和 `markdown` 的 `textformat` 值。

`R3016`：通道「可以」拒絕使用 `xml` 值的 `textformat`。

### <a name="locale"></a>地區設定

`locale` 欄位會傳達 [`text`](#text) 欄位的語言代碼。 `locale` 欄位值是字串內的 [ISO 639](https://www.iso.org/iso-639-language-codes.html) [[5](#references)] 代碼。

`R3020`：接收者「應該」將遺漏和未知的 `locale` 欄位值視為未知。

`R3021`：接收者「不應該」拒絕地區設定不明的活動。

### <a name="speak"></a>口語

`speak` 欄位會指示透過文字轉換語音系統說出活動的方式。 只會在預設值不適用的情況下，才會使用此欄位來自訂語音轉譯。 它會取代活動中所有內容的語音合成，包括文字、附件和摘要。 `speak` 欄位值是字串內以 [SSML](https://www.w3.org/TR/speech-synthesis/) [[6](#references)] 編碼的內容。 可使用未包裝的文字，而且此文字會自動升級為空白 SSML。

`R3030`：`speak` 欄位「可以」包含空字串，表示沒有語音可產生。

`R3031`：無法產生語音的接收者「應該」略過 `speak` 欄位。

`R3032`：如果找不到 SSML 根項目，則接收者「必須」考慮讓所含內容成為 SSML `<speak>` 標記的內部內容，並以有效的 [XML 初構](https://www.w3.org/TR/xml/#sec-prolog-dtd)[[7](#references)] 作為開始，否則須升級至有效的 SSML 文件。

`R3033`：接收者「不應該」使用 XML DTD 或結構描述解析來包含交談 XML 片段以外的遠端資源。

`R3034`：通道「不應該」將 `speak` 欄位傳送給 Bot。

### <a name="input-hint"></a>輸入提示

`inputHint` 欄位會指出活動產生者是否預期得到回應。 此欄位主要用於具有模態使用者介面的通道，而且通常不會用於有連續聊天動態的通道。 `inputHint` 欄位的值是字串類型，已定義的值為 `accepting`、`expecting` 和 `ignoring`。 預設值為 `accepting`。

`R3040`：如果傳送者包含 `inputHint` 欄位，則「應該」只傳送已定義的值。

`R3041`：如果 Bot 將活動傳送至使用 `inputHint` 的通道，則 Bot「應該」包含該欄位，即使值是 `accepting` 也一樣。

`R3042`：接收者「應該」將未定義的值解譯為 `accepting`。

### <a name="attachments"></a>附件

`attachments` 欄位包含要顯示為此活動一部份的物件一般清單。 每個 `attachments` 清單項目的值都是[附件](#attachment)類型的複雜物件。

`R3050`：傳送者「應該」略過未包含項目的 `attachments` 欄位。

`R3051`：傳送者「可以」傳送多個相同類型的實體。

`R3052`：接收者「可以」將類型未知的附件視為可下載的文件。

`R3053`：接收者「應該」在處理內容時保留附件順序，除非轉譯限制強制執行變更，例如，先群組影像再群組文件。

### <a name="attachment-layout"></a>附件配置

`attachmentLayout` 欄位會指示使用者介面轉譯器如何呈現 [ `attachments`](#attachments) 欄位中包含的內容。 `attachmentLayout` 欄位的值是字串類型，已定義的值為 `list` 和 `carousel`。 預設值為 `list`。

`R3060`：如果傳送者包含 `attachmentLayout` 欄位，則「應該」只傳送已定義的值。

`R3061`：接收者「應該」將未定義的值解譯為 `list`。

### <a name="summary"></a>總結

若通道上有此通道不支援的 [`attachments`](#attachments)，則 `summary` 欄位會包含用來取代這些附件的文字。 `summary` 欄位的值是字串類型。

`R3070`：接收者「應該」讓 `summary` 欄位在邏輯上遵循 `text` 欄位。

`R3071`：通道「不應該」將 `summary` 欄位傳送給 Bot。

`R3072`：可處理活動內所有附件的通道「應該」略過 `summary` 欄位。

### <a name="suggested-actions"></a>建議動作

`suggestedActions` 欄位包含互動動作的承載，而這些動作可能會向使用者顯示。 是否支援 `suggestedActions` 和其表徵主要取決於通道。 `suggestedActions` 欄位值是[建議動作](#suggested-actions-2)類型的複雜物件。

### <a name="value"></a>值

`value` 欄位包含所傳送活動專屬的程式設計承載。 其意義和用途會在本文件其他區段中定義，並說明其用法。

`R3080`：傳送者「不應該」包含基本類型 (例如字串、整數) 的 `value` 欄位。 `value` 欄位「應該」是複雜類型或「應該」略過。

### <a name="expiration"></a>到期

`expiration` 欄位包含應將活動視為「過期」的時間，而且不會向收件者顯示。 `expiration` 欄位的值是在字串內以 [ISO 8601](https://www.iso.org/iso-8601-date-and-time-format.html) [[2](#references)] 編碼的日期時間。

`R3090`：傳送者「應該」一律使用「將 `expiration` 欄位的值編碼為 UTC」，而且他們「應該」一律包含 Z 作為值之中的明確 UTC 標記。

### <a name="importance"></a>重要性

`importance` 欄位包含一組列舉的值，用來將活動的相對重要性告知收件者。  由接收者決定要如何將這些重要提示對應至使用者體驗。 `importance` 欄位的值是字串類型，已定義的值為 `low`、`normal` 和 `high`。 預設值為 `normal`。

`R3100`：如果傳送者包含 `importance` 欄位，則「應該」只傳送已定義的值。

`R3101`：接收者「應該」將未定義的值解譯為 `normal`。

### <a name="deliverymode"></a>DeliveryMode

`deliveryMode` 欄位包含其中一組列舉的值，用來將活動的替代傳遞路徑告知收件者。  `DeliveryMode` 欄位的值是字串類型，已定義的值為 `normal` 和 `notification`。 預設值為 `normal`。

`R3110`：如果傳送者包含 `deliveryMode` 欄位，則「應該」只傳送已定義的值。

`R3111`：接收者「應該」將未定義的值解譯為 `normal`。

## <a name="contact-relation-update-activity"></a>連絡人關係更新活動

連絡人關係更新活動表示，通道內的收件者與使用者關係發生變更。 連絡人關係更新活動通常不會包含使用者產生的內容。 `from` 欄位中的使用者 (通常是起始更新的使用者，但並不一定) 和 `recipient` 欄位中的使用者或 Bot 之間會有關係更新的情況，而這會由連絡人關係更新活動來描述。

連絡人關係更新活動會透過 `contactRelationUpdate` 的 `type` 值來識別。

### <a name="action"></a>動作

`action` 欄位會描述連絡人關係更新活動的意義。 `action` 欄位的值是字串。 已定義的值只有 `add` 和 `remove`，表示 `from` 和 `recipient` 欄位中使用者/Bot 之間的關係。

## <a name="conversation-update-activity"></a>對話更新活動

對話更新活動會描述對話成員、描述、存在或其他項目的變更。 對話更新活動通常不會包含使用者產生的內容。 要更新的對話會在 `conversation` 欄位中描述。

對話更新活動會透過 `conversationUpdate` 的 `type` 值來識別。

`R4100`：傳送者「可以」在對話更新活動中包含零個或多個 `membersAdded`、`membersRemoved`、`topicName` 和 `historyDisclosed` 欄位。

`R4101`：在 `membersAdded` 和 `membersRemoved` 欄位中，每個 `channelAccount` (透過 `id` 欄位識別)「應該」最多出現一次。 識別碼「不應該」同時出現在這兩個欄位中。 識別碼「不應該」在任一個欄位中重複出現。

`R4102`：如果未從對話中新增或移除通道帳戶，通道「不應該」使用對話更新活動來包含通道帳戶欄位 (例如 `name`) 的變更。

`R4103`：如果活動未發出要變更 `topicName` 或 `historyDisclosed` 欄位值的訊號，則通道「不應該」傳送這兩個欄位。

### <a name="members-added"></a>新增的成員

`membersAdded` 欄位包含新增至對話的通道參與者 (Bot 或使用者) 清單。 `membersAdded` 欄位值是 [`channelAccount`](#channel-account) 類型的陣列。

### <a name="members-removed"></a>移除的成員

`membersRemoved` 欄位包含從對話移除的通道參與者 (Bot 或使用者) 清單。 `membersRemoved` 欄位值是 [`channelAccount`](#channel-account) 類型的陣列。

### <a name="topic-name"></a>主題名稱

`topicName` 欄位包含對話的文字主題或描述。 `topicName` 欄位的值是字串類型。

### <a name="history-disclosed"></a>公開的記錄

`historyDisclosed` 欄位已淘汰。

`R4110`：傳送者「不應該」包含 `historyDisclosed` 欄位。

## <a name="end-of-conversation-activity"></a>結束對話活動

結束對話活動是以收件者觀點來發出對話結束的訊號。 這可能是因為對話已經完全結束，或是因為收件者從對話中移除時，難以判斷其是否已結束對話。 要結束的對話會在 `conversation` 欄位中描述。

結束對話活動會透過 `endOfConversation` 的 `type` 值來識別。

`code` 和 `text` 都是選擇性欄位。

### <a name="code"></a>代碼

`code` 欄位包含程式設計值，用來描述對話結束的原因及方式。 `code` 欄位的值是字串類型，而意義會由傳送活動的通道定義。

### <a name="text"></a>文字

`text` 欄位包含要告知使用者的選用文字內容。 `text` 欄位的值是字串類型，而格式是純文字。

## <a name="event-activity"></a>事件活動

事件活動會將程式設計資訊從用戶端或通道傳遞給 Bot。 事件活動的意義會由 `name` 欄位定義，這在通道範圍內具有意義。 事件活動的設計訴求是傳達互動式資訊 (例如按鈕點選) 和 非互動式資訊 (例如用戶端自動更新內嵌語音模型的的通知)。

事件活動是[叫用活動](#invoke-activity)的非同步對應項目。 不同於叫用，事件可透過用戶端應用程式擴充功能來進行擴充。

事件活動會透過 `event`的 `type` 值和 `name` 欄位的特定值來識別。

`R5000`：通道「可以」允許在用戶端與 Bot 之間傳送應用程式定義的事件訊息 (如果用戶端允許自訂的話)。

### <a name="name"></a>名稱

`name` 欄位會控制事件的意義和 `value` 欄位的結構描述。 `name` 欄位的值是字串類型。

`R5001`：事件活動「必須」包含 `name` 欄位。

`R5002`：接收者若不了解事件活動的 `name` 欄位，則「必須」略過該事件活動。

### <a name="value"></a>值

`value` 欄位包含此事件專用的參數，如同事件名稱所定義。 `value` 欄位的值是複雜類型。

`R5100`：`value` 欄位「可以」遺漏或空白 (如果是由事件名稱定義的話)。

`R5101`：事件活動的擴充行為「不應該」要求接收者使用活動 `type` 和 `name` 欄位以外的任何資訊，來了解 `value` 欄位的結構描述。

### <a name="relates-to"></a>相關項目

`relatesTo` 欄位會參考另一個對話，並選擇性地參考該活動內的特定活動。 `relatesTo` 欄位值是對話參考類型的複雜物件。

`R5200`：`relatesTo` 參考的活動「不應該」位在由 `conversation` 欄位識別的對話中。

## <a name="invoke-activity"></a>叫用活動

叫用活動會將程式設計資訊從用戶端或通道傳遞給 Bot，並且讓對應項目在通道中傳回要用的承載。 叫用活動的意義會由 `name` 欄位定義，這在通道範圍內具有意義。 

叫用活動是[事件活動](#event-activity)的非同步對應項目。 事件活動是為了可擴充而設計的。 叫用活動的差別只在於將回應承載傳回通道的能力；由於通道必須決定處理這些回應承載的位置和方式，因此叫用只會在每個叫用名稱的明確支援已新增至通道時發揮用處。 所以，叫用沒有設計為泛型應用程式的擴充性機制。

叫用活動會透過 `invoke` 的 `type` 值和 `name` 欄位的特定值來識別。

定義的叫用活動清單包含在[附錄 III](#appendix-iii---protocols-using-the-invoke-activity) 之中。

`R5301`：通道「不應該」允許在用戶端和 Bot 之間傳送應用程式定義的叫用訊息。

### <a name="name"></a>名稱

`name` 欄位會控制叫用的意義和 `value` 欄位的結構描述。 `name` 欄位的值是字串類型。

`R5401`：叫用活動「必須」包含 `name` 欄位。

`R5402`：接收者若不了解事件活動的 `name` 欄位，則「必須」略過該事件活動。

### <a name="value"></a>值

`value` 欄位包含此事件專用的參數，如同事件名稱所定義。 `value` 欄位的值是複雜類型。

`R5500`：`value` 欄位「可以」遺漏或空白 (如果是由事件名稱定義的話)。

`R5501`：事件活動的擴充行為「不應該」要求接收者使用活動 `type` 和 `name` 欄位以外的任何資訊，來了解 `value` 欄位的結構描述。

### <a name="relates-to"></a>相關項目

`relatesTo` 欄位會參考另一個對話，並選擇性地參考該活動內的特定活動。 `relatesTo` 欄位值是[參考](#conversation-reference)類型的複雜物件。

`R5600`：`relatesTo` 參考的活動「不應該」位在由 `conversation` 欄位識別的對話中。

## <a name="installation-update-activity"></a>安裝更新活動

安裝更新活動代表在通道的組織單位內 (例如客戶租用戶或「小組」) 安裝或解除安裝 Bot。 安裝更新活動通常不代表新增或移除通道。

安裝更新活動會透過 `installationUpdate` 的 `type` 值來識別。

`R5700`：在通道內的租用戶、小組或其他組織單位中新增或移除 Bot 時，通道「可以」傳送安裝活動。

`R5701`：在通道內安裝或移除 Bot 時，通道「不應該」傳送安裝活動。

### <a name="action"></a>動作

`action` 欄位會描述安裝更新活動的意義。 `action` 欄位的值是字串。 已定義的值只有 `add` 和 `remove`。

## <a name="message-delete-activity"></a>訊息刪除活動

訊息刪除活動代表要刪除對話內的現有訊息活動。 已刪除活動可透過活動內的 `id` 和 `conversation` 欄位進行參照。

訊息刪除活動會透過 `messageDelete` 的 `type` 值來識別。

`R5800`：通道「可以」選擇傳送訊息刪除活動，來刪除對話內所有活動、刪除對話內的子集 (例如只刪除特定使用者的活動)，或不刪除對話內的任何活動。

`R5801`：通道「不應該」針對 Bot 未查覺到的對話或活動傳送訊息刪除活動。

`R5802`：如果 Bot 觸發刪除動作，則通道「不應該」將訊息刪除活動傳回給該 Bot。

`R5803`：如果訊息刪除活動與不是 `message` 類型的活動對應，則通道「不應該」傳送這些訊息刪除活動。

## <a name="message-update-activity"></a>訊息更新活動

訊息更新活動代表要更新對話內的現有訊息活動。 更新的活動可由活動中的 `id` 和 `conversation` 欄位進行參照，而訊息更新活動會包含已修訂訊息活動中的所有欄位。

訊息更新活動會透過 `messageUpdate` 的 `type` 值來識別。

`R5900`：通道「可以」選擇傳送訊息更新活動，來更新對話內所有活動、更新對話內的子集 (例如只更新特定使用者的活動)，或不更新對話內的任何活動。

`R5901`：如果 Bot 觸發更新動作，則通道「不應該」將訊息更新活動傳回給該 Bot。

`R5902`：如果訊息更新活動與不是 `message` 類型的活動對應，則通道「不應該」傳送這些訊息刪除活動。

## <a name="message-reaction-activity"></a>訊息回應活動

訊息回應活動代表對話內現有訊息活動上的社交互動。 原始活動可透過活動內的 `id` 和 `conversation` 欄位進行參照。 `from` 欄位代表回應的來源 (也就是回應訊息的使用者)。

訊息回應活動會透過 `messageReaction` 的 `type` 值來識別。

### <a name="reactions-added"></a>新增的回應

`reactionsAdded` 欄位包含新增至此活動的回應清單。 `reactionsAdded` 欄位值是 [`messageReaction`](#message-reaction) 類型的陣列。

### <a name="reactions-removed"></a>移除的回應

`reactionsRemoved` 欄位包含從此活動中移除的回應清單。 `reactionsRemoved` 欄位值是 [`messageReaction`](#message-reaction) 類型的陣列。

## <a name="typing-activity"></a>輸入活動

輸入活動代表使用者或 Bot 正在進行的輸入。 此活動通常會在使用者以按鍵進行輸入時傳送，但 Bot 也會使用此活動來表示它們在「思考」，並且也可用來表示向使用者收集來的音訊等等。

輸入活動會在 UI 中保存三秒。

輸入活動會透過 `typing` 的 `type` 值來識別。

`R6000`：如果可以，用戶端「應該」在收到輸入活動時，讓輸入指標顯示 3 秒。

`R6001`：除非對通道有所了解，否則傳送者傳送輸入活動的頻率「不應該」比每三秒一次還要頻繁。 (若要防止出現間隙，傳送者「可以」每兩秒傳送輸入活動。)

`R6002`：如果通道指派 [`id`](#id) 給輸入活動，則「可以」讓 Bot 和用戶端在輸入活動到期之前刪除該活動。

`R6003`：如果可以，通道「應該」傳送輸入活動給 Bot。

## <a name="complex-types"></a>複雜類型

此區段會定義活動結構描述內使用的複雜類型，如上面所述。

### <a name="attachment"></a>附件

附件是[訊息活動](#message-activity)中包含的內容：卡片、二進位文件及其他互動式內容。 這些附件會用來與文字內容搭配顯示。 內容可以透過 `contentUrl` 欄位內的 URL 資料 URI，或 `content` 欄位中的內嵌項目來傳送。

`R7100`：傳送者「不應該」在單一附件中同時包含 `content` 和 `contentUrl` 欄位。

#### <a name="content-type"></a>內容類型

`contentType` 欄位會描述 [MIME 媒體類型](https://www.iana.org/assignments/media-types/media-types.xhtml) [[8](#references)] 的附件內容。 `contentType` 欄位的值是字串類型。

#### <a name="content"></a>內容

`content` 欄位 (如果有的話) 會包含結構化的 JSON 物件附件。 `content` 欄位的值是複雜類型，由 `contentType` 欄位定義。

`R7110`：傳送者「不應該」在附件的 `content` 欄位中包含 JSON 基本類型。

#### <a name="content-url"></a>內容 URL

`contentUrl` 欄位 (如果有的話) 會包含附件內容的 URL。 通道通常會支援資料 URI (如 [RFC 2397](https://tools.ietf.org/html/rfc2397) [[9](#references)] 中所定義)。 `contentUrl` 欄位的值是字串類型。

`R7120`：接收者「應該」接受 HTTPS URL。

`R7121`：接收者「可以」接受 HTTPS URL。

`R7122`：通道「應該」接受資料 URI。

`R7123`：通道「不應該」將資料 URI 傳送給用戶端或 Bot。

#### <a name="name"></a>名稱

`name` 欄位會包含附件的選擇性名稱和檔名。 `name` 欄位的值是字串類型。

#### <a name="thumbnail-url"></a>縮圖 URL

某些用戶端能夠顯示非互動式附件的自訂縮圖，或是將互動式附件顯示為預留位置。 `thumbnailUrl` 欄位會識別此縮圖的來源。 通常也會允許使用資料 URI (如 [RFC 2397](https://tools.ietf.org/html/rfc2397) [[9](#references)] 中所定義)。

`R7140`：接收者「應該」接受 HTTPS URL。

`R7141`：接收者「可以」接受 HTTPS URL。

`R7142`：通道「應該」接受資料 URI。

`R7143`：通道「不應該」將 `thumbnailUrl` 欄位傳送給 Bot。

### <a name="card-action"></a>卡片動作

卡片動作代表可在卡片內使用，或作為[建議動作](#suggested-actions)使用的可點按或互動式按鈕。 這些動作會用來收集使用者的輸入。 儘管名稱是卡片動作，但並未限制為只能在卡片上使用。

卡片動作只有在傳送到通道時才有意義。

通道會決定每個動作在其使用者體驗中的顯示方式。 在大部分情況下，卡片都是可點按的。 但在其他情況下，卡片可能會透過語音輸入來選取。 如果通道未提供互動式啟用體驗 (例如，透過簡訊互動)，則通道可能完全不會支援啟用。 有關如何轉譯動作的決定，會由此文件其他部分的標準需求來控制 (可能在卡片格式中，或是在[建議動作](#suggested-actions)的定義中)。

#### <a name="type"></a>類型

`type` 欄位會描述按鈕的意義，以及按鈕啟動時的行為。 `type` 欄位值是依照慣例的簡短字串 (例如「`openUrl`」)。 請參閱後續章節，了解每個動作類型的特定需求。

* `messageBack` 動作類型代表透過聊天系統傳送的文字回應。
* `imBack` 動作類型代表新增至聊天動態的文字回應。
* `postBack` 動作類型代表不會新增至聊天動態的文字回應。
* `openUrl` 動作類型代表由用戶端處理的超連結。
* `downloadFile` 動作類型代表用於下載的超連結。
* `showImage` 動作類型代表可能要顯示的影像。
* `signin` 動作類型代表由用戶端登入系統處理的超連結。
* `playAudio` 動作類型代表可能要播放的音訊媒體。
* `playVideo` 動作類型代表可能要播放的視訊媒體。
* `call` 動作類型代表可能要撥打的電話號碼。
* `payment` 動作類型代表要提供付款的要求。

#### <a name="title"></a>標題

`title` 欄位包含要在按鈕表面顯示的文字。 `title` 欄位的值是字串類型，而且不包含標記。

此欄位適用於所有類型的動作。

`R7210`：通道「不應該」處理 `title` 欄位內的標記 (例如 Markdown)。

#### <a name="image"></a>映像

`image` 欄位包含一個 URL，其指向按鈕表面顯示的圖示。 通道通常會支援資料 URI (如 [RFC 2397](https://tools.ietf.org/html/rfc2397) [[9](#references)] 中所定義)。 `image` 欄位的值是字串類型。

此欄位適用於所有類型的動作。

`R7220`：通道「應該」接受 HTTPS URL。

`R7221`：通道「可以」接受 HTTPS URL。

`R7222`：通道「應該」接受資料 URI。

#### <a name="text"></a>文字

`text` 欄位包含按下按鈕時要傳送給 Bot 的文字內容，以及聊天動態中包含的文字內容。 `text` 欄位的內容可能會或可能不會顯示，這取決於按鈕類型。 `text` 欄位可能會包含標記，這會由活動根之中的 [`textFormat`](#text-format) 欄位來控制。 `text` 欄位的值是字串類型。

此欄位只會用在選取類型的動作上。 我們會在此文件的後續章節中詳細說明每個動作類型。

`R7230`：`text` 欄位「可以」包含空字串，表示傳送沒有內容的文字。

`R7231`：通道「應該」根據活動根之中的 [`textFormat`](#text-format) 欄位，來處理 `text` 欄位的內容。

#### <a name="display-text"></a>顯示文字

`displayText` 欄位包含按下按鈕時要納入聊天動態中的文字內容。 如果技術上允許，通道內「應該」一律顯示 `displayText` 欄位的內容。 `displayText` 欄位可能會包含標記，這會由活動根中的 [`textFormat`](#text-format) 欄位來控制。 `displayText` 欄位的值是字串類型。

此欄位只會用在選取類型的動作上。 我們會在此文件的後續章節中詳細說明每個動作類型。

`R7240`：`displayText` 欄位「可以」包含空字串，表示傳送沒有內容的文字。

`R7241`：通道「應該」根據活動根之中的 [`textFormat`](#text-format) 欄位，來處理 `displayText` 欄位的內容。

#### <a name="value"></a>值

`value` 欄位包含按下按鈕時，要傳送給 Bot 的程式設計內容。 `value` 欄位可以是基本或複雜類型，但某些活動類型會限制此欄位。

此欄位只會用在選取類型的動作上。 我們會在此文件的後續章節中詳細說明每個動作類型。

#### <a name="message-back"></a>訊息回復

`messageBack` 動作代表透過聊天系統傳送的文字回應。 訊息回復會使用下列欄位：
* `type` ("`messageBack`")
* `title`
* `image`
* `text`
* `displayText`
* `value` (任何類型)

`R7350`：傳送者「不應該」包含基本類型 (例如字串、整數) 的 `value` 欄位。 `value` 欄位「應該」是複雜類型或「應該」略過。

`R7351`：通道「可以」拒絕或捨棄不屬於複雜類型的 `value` 欄位。

`R7352`：啟動時，通道「必須」傳送 `message` 類型的活動給所有相關收件者。

`R7353`：如果通道支援儲存和傳送文字，則動作的 `text` 欄位內容「必須」在所產生訊息活動的 `text` 欄位中保存與傳送。

`R7352`：如果通道支援儲存和傳送其他程式設計值，則 `value` 欄位內容「必須」在所產生訊息活動的 `value` 欄位中保存與傳送。

`R7353`：如果通道支援保留聊天動態中的不同值 (之後會傳送給 Bot)，則通道「必須」在聊天記錄中包含 `displayText` 欄位。

`R7354`：如果通道不支援 `R7353`，但可支援記錄聊天動態中的文字，則通道「必須」在聊天記錄中包含 `text` 欄位。

#### <a name="im-back"></a>即時訊息回復

`imBack` 動作代表新增至聊天動態的文字回應。 訊息回復會使用下列欄位：
* `type` ("`imBack`")
* `title`
* `image`
* `value` (字串類型)

`R7360`：啟動時，通道「必須」傳送 `message` 類型的活動給所有相關收件者。

`R7361`：如果通道支援儲存和傳送文字，則 `title` 欄位內容「必須」在所產生訊息活動的 `text` 欄位中保存與傳送。

`R7362`：如果動作上遺漏 `title` 欄位，而且 `value` 欄位是字串類型，則通道「可以」在所產生訊息活動的 `text` 欄位中傳送 `value` 欄位內容。

`R7363`：如果通道支援記錄聊天動態中的文字，則通道「必須」在聊天記錄中包含 `title` 欄位的內容。

#### <a name="post-back"></a>貼文回復

`postBack` 動作代表不會新增至聊天動態的文字回應。 貼文回復會使用下列欄位：
* `type` ("`postBack`")
* `title`
* `image`
* `value` (字串類型)

`R7370`：啟動時，通道「必須」傳送 `message` 類型的活動給所有相關收件者。

`R7371`：啟動貼文回應動作時，通道「不應該」在聊天記錄中包含文字。

`R7372`：通道「必須」拒絕或捨棄不屬於字串類型的 `value` 欄位。

`R7373`：如果通道支援儲存和傳送文字，則 `value` 欄位內容「必須」在所產生訊息活動的 `text` 欄位中保存與傳送。

`R7374`：如果聊天動態中沒有記錄，通道就無法傳送訊息給 Bot，則通道「應該」使用 `title` 欄位作為顯示文字。

#### <a name="open-url-actions"></a>開啟 URL 動作

`openUrl` 動作代表由用戶端處理的超連結。 開啟 URL 會使用下列欄位：
* `type` ("`openUrl`")
* `title`
* `image`
* `value` (字串類型)

`R7380`：傳送者「必須」在`openUrl` 動作的 `value` 欄位中包含 URL。

`R7381`：接收者「可以」拒絕 `value` 欄位遺漏或不是字串的 `openUrl` 動作。

`R7382`：接收者「應該」拒絕或捨棄 `value` 欄位包含資料 URI (如 [RFC 2397](https://tools.ietf.org/html/rfc2397) [[9](#references)] 中所述) 的 `openUrl` 動作。

`R7383`：如果 `openUrl` 動作的 `value` URI 不是預期外的 URI 配置或值，則接收者「不應該」拒絕這些動作。

`R7384`：了解特定 URI 配置 (例如 HTTP) 的用戶端，「可以」處理內嵌轉譯器 (例如瀏覽器控制項) 中的 `openUrl` 動作。

`R7385`：若情況允許，用戶端應該將不是由 `R7354` 處理的 `openUrl` 動作委派給作業系統層級或殼層層級的 URI 處理常式來處理。

#### <a name="download-file-actions"></a>下載檔案動作

`downloadFile` 動作代表用於下載的超連結。 下載檔案會使用下列欄位：
* `type` ("`downloadFile`")
* `title`
* `image`
* `value` (字串類型)

`R7390`：傳送者「必須」在`downloadFile` 動作的 `value` 欄位中包含 URL。

`R7391`：接收者「可以」拒絕 `value` 欄位遺漏或不是字串的 `downloadFile` 動作。

`R7392`：接收者「應該」拒絕或捨棄 `value` 欄位包含資料 URI (如 [RFC 2397](https://tools.ietf.org/html/rfc2397) [[9](#references)] 中所述) 的 `downloadFile` 動作。

#### <a name="show-image-file-actions"></a>顯示影像檔的動作

`showImage` 動作代表可能要顯示的影像。 顯示影像會使用下列欄位：
* `type` ("`showImage`")
* `title`
* `image`
* `value` (字串類型)

`R7400`：傳送者「必須」在`showImage` 動作的 `value` 欄位中包含 URL。

`R7401`：接收者「可以」拒絕 `value` 欄位遺漏或不是字串的 `showImage` 動作。

`R7402`：接收者「可以」拒絕整個 `value` 欄位是資料 URI (如 [RFC 2397](https://tools.ietf.org/html/rfc2397) [[9](#references)] 中所定義) 的 `showImage` 動作。

#### <a name="signin"></a>登入

`signin` 動作代表由用戶端登入系統處理的超連結。 登入會使用下列欄位：
* `type` ("`signin`")
* `title`
* `image`
* `value` (字串類型)

`R7410`：傳送者「必須」在`signin` 動作的 `value` 欄位中包含 URL。

`R7411`：接收者「可以」拒絕 `value` 欄位遺漏或不是字串的 `signin` 動作。

`R7412`：接收者「必須」拒絕或捨棄 `value` 欄位包含資料 URI (如 [RFC 2397](https://tools.ietf.org/html/rfc2397) [[9](#references)] 中所述) 的 `signin` 動作。

#### <a name="play-audio"></a>播放音訊

`playAudio` 動作代表可能要播放的音訊媒體。 播放音訊會使用下列欄位：
* `type` ("`playAudio`")
* `title`
* `image`
* `value` (字串類型)

`R7420`：啟動時，通道「可以」傳送媒體事件。

`R7421`：通道「必須」拒絕或捨棄不屬於字串類型的 `value` 欄位。

`R7422`：如果傳送者事先不知道通道是否可支援資料 URI (如 [RFC 2397](https://tools.ietf.org/html/rfc2397) [[9](#references)] 中所述)，則「不應該」傳送資料 URI。

#### <a name="play-video"></a>播放視訊

`playAudio` 動作代表可能要播放的視訊媒體。 播放視訊會使用下列欄位：
* `type` ("`playVideo`")
* `title`
* `image`
* `value` (字串類型)

`R7430`：啟動時，通道「可以」傳送媒體事件。

`R7431`：通道「必須」拒絕或捨棄不屬於字串類型的 `value` 欄位。

`R7432`：如果傳送者事先不知道通道是否可支援資料 URI (如 [RFC 2397](https://tools.ietf.org/html/rfc2397) [[9](#references)] 中所述)，則「不應該」傳送資料 URI。

#### <a name="call"></a>呼叫

`call` 動作代表可能要撥打的電話號碼。 呼叫會使用下列欄位：
* `type` ("`call`")
* `title`
* `image`
* `value` (字串類型)

`R7440`：傳送者「必須」在 `signin` 動作的 `value` 欄位中包含 `tel` 配置的 URL。

`R7441`：如果 `signin` 動作的 `value` 欄位遺漏或不是 `tel` 配置的字串 URI，則接收者「必須」拒絕該動作。

#### <a name="payment"></a>付款

`payment` 動作代表要提供付款的要求。 付款會使用下列欄位：
* `type` ("`payment`")
* `title`
* `image`
* `value` ([付款要求](#payment-request)的複雜類型)

`R7450`：傳送者「必須」在 `payment` 動作的 `value` 欄位中，包含[付款要求](#payment-request)類型的複雜物件。

`R7451`：如果 `payment` 動作的 `value` 欄位遺漏或不是[付款要求](#payment-request)類型的複雜物件，則通道「必須」拒絕該動作。

### <a name="channel-account"></a>通道帳戶

通道帳戶代表通道內的身分識別。 通道帳戶包含識別碼，可用來識別並連絡該通道內的帳戶。 有時候這些識別碼會存在單一命名空間中 (例如 Skype 識別碼)；有時候，它們會在多部伺服器間結成同盟 (例如電子郵件地址)。 除了識別碼，通道帳戶還包含了顯示名稱和 Azure Active Directory (AAD) 物件識別碼。

#### <a name="channel-account-id"></a>通道帳戶識別碼

`id` 欄位是通道內的識別碼和位址。 `id` 欄位的值是字串。 例如，在使用電子郵件地址的通道中，`id` 可能是 "name@example.com"

`R7510`：不論帳戶識別碼在結構描述中的位置為何 (`from.id`、`recipient.id`、`membersAdded` 等)，通道都「應該」對帳戶識別碼使用相同值及慣例。 這可讓 Bot 和用戶端使用序數字串進行比較，以了解帳號識別碼何時在 `conversationUpdate` 活動的 `membersAdded` 欄位中或其他位置中有所描述。

#### <a name="channel-account-name"></a>通道帳戶名稱

`name` 欄位是通道內的選擇性易記名稱。 `name` 欄位的值是字串。 通道中的 `name` 範例是「John Doe」

#### <a name="channel-account-aad-object-id"></a>通道帳戶的 AAD 物件識別碼

`aadObjectId` 欄位是選擇性識別碼，對應至 Azure Active Directory (AAD) 內的帳戶物件識別碼。 `aadObjectId` 欄位的值是字串。

### <a name="conversation-account"></a>對話帳戶

對話帳戶代表通道內的對話身分識別。 在只支援兩個帳戶間單一對話 (例如 SMS) 的通道中，對話帳戶是持續性的，而且沒有預先決定的開始或結束。 在支援多個平行對話 (例如電子郵件) 的通道中，每個對話都可能有唯一識別碼。

#### <a name="conversation-account-id"></a>對話帳戶識別碼

`id` 欄位是通道內的識別碼。 此識別碼的格式由通道所定義，並且會以不透明字串的形式在整個通訊協定中使用。

通道「應該」針對對話中的所有參與者，選擇穩定的 `id` 值。 (例如，對於 1:1 對話的 `id` 欄位，就不適合使用另一個參與者的識別碼作為 `id` 值。 因為對其中一個參與者來說，這是不同的 `id`。 較好的選擇是整理兩個參與者的識別碼，並將它們串連在一起，這樣一來，此識別碼對兩者來說都是相同的。)

#### <a name="conversation-account-name"></a>對話帳戶名稱

`name` 欄位是通道內的選擇性易記對話名稱。 `name` 欄位的值是字串。

#### <a name="conversation-account-aad-object-id"></a>對話帳戶的 AAD 物件識別碼

`aadObjectId` 欄位是選擇性識別碼，對應至 Azure Active Directory (AAD) 內的對話物件識別碼。 `aadObjectId` 欄位的值是字串。

#### <a name="conversation-account-is-group"></a>對話帳戶是群組

`isGroup` 欄位用於指出活動產生時，對話是否要包含超過兩位參與者。 `isGroup` 欄位的值是布林值；如果省略，預設值是 `false`。 此欄位通常會控制參與者在通道中的提及行為，而且「應該」在兩個以上的參與者都能夠傳送和接收對話內的活動時，才能設定為 `true`。

### <a name="entity"></a>實體

實體包含有關活動或對話的中繼資料。 每個實體的意義和形狀皆由 `type` 欄位定義，而其他特定類型的欄位會作為 `type` 欄位的對等項目。

`R7600`：如果接收者不了解實體的類型，則「必須」略過這些實體。

`R7601`：如果接收者了解實體的類型，但因為語法錯誤等原因而無法處理，則「應該」略過這些實體。

將現有實體類型投射至活動實體格式的各方，應根據 IRI 與實體結構描述之間的繫結，解決 `type` 欄位名稱的衝突，以及與序列化需求 `R2001` 不相容的問題。

#### <a name="type"></a>類型

`type` 欄位為必要項目，用來定義實體的與意義與形狀。 `type` 主要包含 [IRI](https://tools.ietf.org/html/rfc3987) [[3](#references)]，但還是有少數非 IRI 實體類型，如[附錄 I](#appendix-ii---non-iri-entity-types) 中所定義。

`R7610`：傳送者「應該」只對[附錄 II](#appendix-ii---non-iri-entity-types) 中描述的類型使用非 IRI 類型名稱。

`R7611`：傳送者「可以」傳送[附錄 II](#appendix-ii---non-iri-entity-types) 中所述的 IRI 類型，前提是傳送者知道接收者了解這些類型。

`R7612`：傳送者「應該」針對[附錄 II](#appendix-ii---non-iri-entity-types) 中未定義的實體類型使用或建立 IRI。

### <a name="suggested-actions"></a>建議動作

建議動作可能會在訊息內容中傳送，以建立用戶端 UI 中的互動式動作項目。

`R7700`：用戶端若不支援能夠轉譯建議動作的 UI，則「應該」略過 `suggestedActions` 欄位。

`R7701`：如果 `actions` 欄位是空的，則傳送者「應該」略過 `suggestedActions` 欄位。

#### <a name="to"></a>至

`to` 欄位包含應該看到建議動作的通道帳戶識別碼。 此欄位可用來針對對話參與者的子集篩選動作。

`R7710`：如果 `to` 欄位遺漏或者為空白，用戶端「應該」對所有對話參與者顯示建議動作。

`R7711`：如果 `to` 欄位包含無效識別碼，則「應該」忽略這些值。

#### <a name="actions"></a>動作

`actions` 欄位包含要顯示的動作一般清單。 每個 `actions` 清單項目的值都是 `cardAction` 類型的複雜物件。

### <a name="message-reaction"></a>訊息回應

訊息回應代表社交互動 (「喜歡」、「+ 1」等等)。 訊息回應目前只包含單一欄位：`type` 欄位。

#### <a name="type"></a>類型

`type` 欄位會描述社交互動的類型。 `type` 欄位的值是字串，而意義會由發生互動的通道來定義。 雖然 `like` 和 `+1` 等這些值相當常見，但這些是根據慣例統整而來，而不是根據規則。

## <a name="references"></a>參考

1. [RFC 2119](https://tools.ietf.org/html/rfc2119)
2. [ISO 8601](https://www.iso.org/iso-8601-date-and-time-format.html)
3. [RFC 3987](https://tools.ietf.org/html/rfc3987)
4. [Markdown](https://daringfireball.net/projects/markdown/)
5. [ISO 639](https://www.iso.org/iso-639-language-codes.html)
6. [SSML](https://www.w3.org/TR/speech-synthesis/)
7. [XML](https://www.w3.org/TR/xml/)
8. [MIME 媒體類型](https://www.iana.org/assignments/media-types/media-types.xhtml)
9. [RFC 2397](https://tools.ietf.org/html/rfc2397)
10. [ISO 3166-1](https://www.iso.org/iso-3166-country-codes.html)
11. [調適型卡片](https://adaptivecards.io)

# <a name="appendix-i---changes"></a>附錄 I - 變更

## <a name="2018-02-07"></a>2018-02-07

* 初始草稿

# <a name="appendix-ii---non-iri-entity-types"></a>附錄 II - 非 IRI 實體類型

活動[實體](#entity)會傳達活動的額外中繼資料，例如使用者位置或正在使用的傳訊應用程式版本。 活動類型主要是 IRI，但有一小部分非 IRI 名稱也很常用。 本附錄完整列出可支援的非 IRI 實體類型。

| 類型           | URI 對等項目                          | 說明               |
| -------------- | --------------------------------------- | ------------------------- |
| GeoCoordinates | https://schema.org/GeoCoordinates       | Schema.org GeoCoordinates |
| 提及        | https://botframework.com/schema/mention | @-mention                 |
| 位置          | https://schema.org/Place                | Schema.org 位置          |
| 項目          | https://schema.org/Thing                | Schema.org 項目          |
| clientInfo     | N/A                                     | Skype 用戶端資訊         |

### <a name="clientinfo"></a>clientInfo

clientInfo 實體包含用戶端軟體 (用於傳送使用者訊息) 的擴充資訊。 其中有三個屬性，而所有屬性都是選擇性。

`R9201`：Bot「不應該」傳送 `clientInfo` 實體。

`R9202`：傳送者「應該」只在一或多個欄位已填入資料時，包含 `clientInfo`實體。

#### <a name="locale-deprecated"></a>地區設定 (已淘汰)

`locale` 欄位包含使用者的地區設定。 此欄位與活動根之中的 [`locale`](#locale) 欄位重複。 `locale` 欄位值是字串內的 [ISO 639](https://www.iso.org/iso-639-language-codes.html) [[5](#references)] 代碼。

`clientInfo` 中的 `locale` 欄位已淘汰。

`R9211`：接收者「不應該」使用 `clientInfo` 物件內的 `locale` 欄位。

`R9212`：傳送者「可以」基於相容性考量，填寫 `clientInfo` 中的 `locale` 欄位。 如果不需要與之前的接收者相容，傳送者「不應該」傳送 `locale` 屬性。

#### <a name="country"></a>國家 (地區)

`country` 欄位包含使用者所在地區的偵測結果。 此值可能與 [`locale`](#locale) 資料不同，因為 `country` 是偵測到的，而 `locale` 通常是使用者或應用程式設定。 `country` 欄位的值是 [ISO 3166-1](https://www.iso.org/iso-3166-country-codes.html) [[10](#references)] 的國家/地區代碼 (2 個或 3 個字母)。

`R9220`：通道「不應該」允許用戶端對 `country` 欄位指定任意值。 通道「應該」使用 GPS、位置 API 或 IP 位址偵測等機制，來設立產生要求的國家/地區。

#### <a name="platform"></a>平台

`platform` 欄位會描述用來產生活動的傳訊用戶端平台。 `platform` 欄位的值是字串，而可能值的清單及意義會由傳送這些值的通道來定義。

請注意，在有持續聊天動態的通道上，`platform` 通常有助於決定要包含的內容，但對內容格式並無幫助。 比方說，如果行動裝置上的使用者要求提供產品支援說明，Bot 可能會產生行動裝置專屬的說明。 但是，使用者可能接著要在電腦上重新開啟聊天動態，以便在該畫面上讀取說明，並且同時對行動裝置進行變更。 在此情況下，`platform` 欄位的用意在於通知內容，但應該在其他裝置上檢視內容。

`R9230`：Bot「不應該」使用 `platform` 欄位來控制回應資料的格式，除非 Bot 知道它們傳送的內容只會在有問題的裝置上顯示。

# <a name="appendix-iii---protocols-using-the-invoke-activity"></a>附錄 III - 使用叫用活動的通訊協定

[叫用活動](#invoke-activity)只能在 Bot Framework 通道支援的通訊協定中使用 (也就是指非泛型的擴充性機制)。 本附錄包含所有使用此活動的 Bot Framework 通訊協定清單。

## <a name="payments"></a>付款

Bot Framework 付款通訊協定會使用叫用來計算出貨和稅率等，並傳達已完成付款的強式確認。

付款通訊協定會定義三項作業 (在叫用活動的 `name` 欄位中定義)：
* `payment/shippingAddressChange`
* `payment/shppingOptionsChange`
* `payment/paymentResponse`

您可以在[要求付款](https://docs.microsoft.com/en-us/bot-framework/dotnet/bot-builder-dotnet-request-payment)頁面中找到更詳細的資訊。

## <a name="teams-compose-extension"></a>Teams 的撰寫擴充

Microsoft Teams 通道會使用適用於[撰寫擴充](https://docs.microsoft.com/en-us/microsoftteams/platform/concepts/messaging-extensions)的叫用。 此叫用的用法只適用於 Microsoft Teams。



