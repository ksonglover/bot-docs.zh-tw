---
title: 將交談從 Bot 切換為人類 | Microsoft Docs
description: 了解如何針對使用者與 Bot 開始交談，然後必須遞交給人類的情況進行設計。
author: matvelloso
ms.author: mateusv
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 12/13/2017
ms.openlocfilehash: 6b50df60c3a8165198e8f9a55964f2f596d62406
ms.sourcegitcommit: b78fe3d8dd604c4f7233740658a229e85b8535dd
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/24/2018
ms.locfileid: "49998725"
---
# <a name="transition-conversations-from-bot-to-human"></a>將交談從 Bot 切換為人類

無論 Bot 擁有多高的人工智慧，有時仍可能需要將交談遞交給人類。 Bot 應該要辨識何時需要進行遞交，並為使用者提供明確且順暢的切換。

## <a name="scenarios-that-require-human-involvement"></a>需要人為介入的情況

有各種不同的情況可能需要 Bot 將交談的控制權切換給人類。 這些情況其中一些是「分級」、「呈報」及「監督」。 

### <a name="triage"></a>分級

一般技術支援中心的來電是從一些 Bot 可以輕鬆回答的基本問題開始。 作為輸入要求之第一個回應者的 Bot，可以收集使用者名稱、地址、問題的描述或任何其他相關資訊，然後將交談的控制權切換給代理人員。 使用 Bot 來分類連入要求，可讓代理人員將時間花在解決問題，而不是收集資訊。

### <a name="escalation"></a>升級

在技術支援中心案例中，Bot 除了收集資訊　(例如重設使用者的密碼) 之外，還可以回答基本問題並解決簡單的問題。 不過，如果交談指出使用者的問題太過複雜而需要人為介入，Bot 必須呈報此問題給人類的代理人員。 若要實作這種類型的案例，Bot 必須能夠區別它可以獨立解決的問題，以及必須呈報給人的問題。 Bot 可透過許多方法來判定它需要將交談的控制權移轉給人類。 

#### <a name="user-driven-menus"></a>使用者導向的功能表

也許讓 Bot 處理這個難題的最簡單方式，是向使用者呈現包含多個選項的功能表。 Bot 可獨立處理的工作會出現在標示為 [與代理人員聊天] 之連結上方的功能表。 這種類型的實作不需要任何進階的機器學習服務或自然語言理解。 當使用者選取 [與代理人員聊天] 選項時，Bot 只會將交談的控制權移轉給人類的代理人員。 

#### <a name="scenario-driven"></a>案例導向

Bot 可能會根據它是否判斷為能夠處理手邊的案例，來決定是否要轉移控制權。 Bot 會收集有關使用者要求的一些資訊，，然後查詢其功能的內部清單，以判定它是否能夠處理該要求。 如果 Bot 判定它能夠處理要求，就會這樣做，但如果 Bot 判定要求超出它可以解決的問題範圍，則會將交談的控制權轉移給人類的代理人員。

#### <a name="natural-language"></a>自然語言

自然語言理解和情感分析可協助 Bot 決定何時要將交談的控制權轉移給人類的代理人員。 當試圖判斷使用者何時感到挫折，或想要與人類的代理人員交談時，這極具價值。 
 
Bot 透過使用可推斷情感的<a href="https://www.microsoft.com/cognitive-services/en-us/text-analytics-api" target="blank">文字分析 API</a>或使用 <a href="https://www.luis.ai" target="_blank">LUIS API</a> 來分析使用者訊息的內容。 


> [!TIP]
> 自然語言理解不一定是判定 Bot 何時應該將交談控制權移轉給人類的最佳方法。 與人類一樣，Bot　不見得一律能夠猜對，但無效的回應會讓使用者感到挫折。 不過，如果使用者從包含有效選項的功能表中進行選取，Bot 一律會適當回應該輸入。 

### <a name="supervision"></a>監督

在某些情況下，人類的代理人員想要監視交談，而不是控制。

例如，請考慮使用技術支援中心案例，其中 Bot 正在與使用者通訊以診斷電腦問題。 機器學習服務模型可協助 Bot 判定問題的最可能原因；不過，在建議使用者採取特定動作之前，Bot 可以私下與人類代理人員確認診斷和解決方法，並要求授權以繼續進行。 代理人員接著會按一下按鈕，Bot 即會向使用者呈現解決方案，這個問題就解決了。 Bot 仍然在執行大部分的工作，但代理人員保留最終決定的控制權。 

## <a name="transitioning-control-of-the-conversation"></a>切換交談的控制權 

當 Bot 決定要將交談的控制權移轉給人類時，它可以通知使用者即將進行移轉並將交談置入「等待」狀態，直到它確認代理人員有空為止。 

Bot 正在等待人類時，它可能會以預設回應 (例如「在佇例中等待」) 自動回答所有的連入使用者訊息。 此外，如果使用者傳送特定訊息 (例如「沒關係」或「取消」)，您可以讓 Bot 從「等待」狀態中移除交談。

您可以指定在設計 Bot 時，如何將代理人員指派給等待的使用者。 例如，Bot 可以實作一個簡單的佇列系統：先進先出。更複雜的邏輯會根據地理位置、語言或某些其他因素，將使用者指派給代理人員。 Bot 還可以向代理人員呈現某種類型的 UI，供他們用來選取使用者。 當代理人員有空時，其會連線到 Bot 並加入交談。

> [!IMPORTANT]
> 即使在代理人員參與之後，Bot 仍然是交談的幕後推動者。 使用者和代理人員永遠不會直接彼此通訊；它們只是透過 Bot 路由傳送訊息。 

## <a name="routing-messages-between-user-and-agent"></a>在使用者與代理人員之間路由傳送訊息

代理人員連線到 Bot 之後，Bot 就會開始在使用者與代理人員之間路由傳送訊息。 雖然看起來使用者和代理人員是直接與彼此聊天，但實際上他們是透過 Bot 交換訊息。 Bot 會接收來自使用者的訊息並將這些訊息傳送給代理人員，接著接收來自代理人員的訊息，並將這些訊息傳送給使用者。 

> [!NOTE]
> 在更進階的案例中，Bot 可以承擔除了只是在使用者與代理人員之間路由傳送訊息之外的責任。 例如，Bot 可以決定哪個回應合適，並且只要求代理人員確認以繼續進行。

## <a name="sample-code"></a>範例程式碼

如需示範如何使用適用於 Node.js 的 Bot 建立器 SDK 將交談從 Bot 遞交給人的完整範例，請參閱 GitHub 中的 <a href="https://github.com/palindromed/Bot-HandOff" target="_blank">Bot 遞交範例</a>。

## <a name="additional-resources"></a>其他資源

::: moniker range="azure-bot-service-4.0"

- [對話方塊](v4sdk/bot-builder-dialog-manage-conversation-flow.md)
- <a href="https://www.microsoft.com/cognitive-services/en-us/text-analytics-api" target="blank">文字分析 API</a>

::: moniker-end

::: moniker range="azure-bot-service-3.0"

- [使用對話方塊來管理對話流程 (.NET)](~/dotnet/bot-builder-dotnet-manage-conversation-flow.md)
- [使用對話方塊來管理對話流程 (Node.js)](~/nodejs/bot-builder-nodejs-manage-conversation-flow.md)
- <a href="https://www.microsoft.com/cognitive-services/en-us/text-analytics-api" target="blank">文字分析 API</a>


::: moniker-end

