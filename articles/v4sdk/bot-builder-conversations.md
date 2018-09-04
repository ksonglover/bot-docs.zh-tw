---
title: 在 Bot Builder SDK 內的交談| Microsoft Docs
description: 說明 Bot Builder SDK 中的交談。
keywords: 交談流程, 辨識意圖, 單一回合, 多回合
author: jonathanfingold
ms.author: jonathanfingold
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 04/11/2018
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 32486cb024dfe852a7478ccba4a0eedc476431b0
ms.sourcegitcommit: ee63d9dc1944a6843368bdabf5878950229f61d0
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/23/2018
ms.locfileid: "42795027"
---
# <a name="conversation-flow"></a>交談流程
[!INCLUDE [pre-release-label](../includes/pre-release-label.md)]

由於 Bot 可視為交談式使用者介面，因此交談流程是指我們與使用者的互動方式及可採用的形式。 採用適當的交談流程有助於改善使用者互動，以及 Bot 的效能。

在設計 Bot 交談流程時，您要決定使用者向 Bot 發出訊息後，Bot 應該如何回應。 Bot 會先根據使用者的訊息辨識工作或交談主題。 Bot 會尋找使用者訊息中的字詞或模式，或利用 [Language Understanding (LUIS)](bot-builder-concept-luis.md) 和 QnA Maker 這類服務，以判斷與使用者訊息相關的工作或主題 (稱為「*意圖*」)。 

Bot 辨識出使用者意圖後，Bot 將視情況在單次回覆內滿足使用者的要求，或在單次回合結束交談，否則可能就需要多回合。 如為多回合的交談流程，Bot Builder SDK 可提供用於追蹤交談的[狀態管理](./bot-builder-howto-v4-state.md)作業、用於詢問更多資訊的[提示](bot-builder-prompts.md)，以及模擬交談流程的[交談方塊](bot-builder-dialog-manage-conversation-flow.md)。 

在複雜的 Bot 和多重子系統中，您可使用多項服務辨識意圖 (Bot 的每個子元件使用一項)。 將交談式子系統合併至一個 Bot 後，[分派工具](bot-builder-tutorial-dispatch.md)即可從集中位置取得多項服務的結果。 
<!-- 
A conversation identifies a series of activities sent between a bot and a user on a specific channel and represents an interaction between one or more bots and either a _direct_ conversation with a specific user or a _group_ conversation with multiple users.
A bot communicates with a user on a channel by receiving activities from, and sending activities to the user.

- Each user has an ID that is unique per channel.
- Each conversation has an ID that is unique per channel.
- The channel sets the conversation ID when it starts the conversation.
- The bot cannot start a conversation; however, once it has a conversation ID, it can resume that conversation.
- Not all channels support group conversations.
-->

## <a name="single-turn-conversation"></a>單一回合交談

最簡單的交談流程是單一回合。 在單一回合流程中，Bot 可在一個回合內完成工作，也就是在一則使用者訊息和一則 Bot 回覆的過程中完成。 



<!-- 
The EchoBot sample in the BotBuilder SDK is a single-turn bot. Here are other examples of single turn conversation flow:
* A bot for getting the weather report, that just tells the user what the weather is, when they say "What's the weather?".
* An IoT bot that responds to "turn on the lights" by calling an IoT service. -->

<!-- The following isn't always true, it's a generalization --> 最簡單的單一回合 Bot 不需要記錄交談狀態。 每次 Bot 收到訊息後，將僅根據目前內送訊息的內容回應，不需要參考過往交談回合的記錄。

![單一回合天氣 Bot](./media/concept-conversation/weather-single-turn.png)

天氣 Bot 採用單一回合流程，其僅提供使用者天氣預報，不用來回詢問城市或日期。 顯示天氣預報的所有邏輯皆根據 Bot 剛剛收到的訊息。 在對話的每個回合中，Bot 都會收到[回合內容](bot-builder-concept-activity-processing.md#turn-context)，其便可利用這些內容判斷後續動作及對話走向。 

## <a name="multiple-turns"></a>多回合

大部分類型的交談都無法在單一回合內完成，因此 Bot 也可採用多回合交談流程。 需要多次交談回合的情況包括：

 * Bot 需要提示使用者提供額外資訊以利其完成工作。 Bot 需要追蹤其是否已具有完成工作所需的一切參數。
 * Bot 需要引導使用者完成程序中的步驟，例如提交訂單。 Bot 需要追蹤使用者在步驟順序中的位置。

例如，假設天氣 Bot 採用多回合流程，且 Bot 以詢問城市來回應「天氣如何？」 這個問題。

![多回合天氣 Bot](./media/concept-conversation/weather-multi-turn.png)

使用者回覆 Bot 的城市問題提示後，Bot 收到「西雅圖」，因此必須儲存部分情境，以判斷目前的訊息是用來回覆先前提示，並且屬於取得天氣的要求。 多回合 Bot 會記錄狀態以確保能適當回覆新訊息。

<!--
```
// TBD: snippet showing receiving message and using ConversationProperties
```
-->

請參閱[管理狀態](bot-builder-storage-concept.md)，了解如何管理狀態的概觀；如需參考範例，請參閱[如何運用使用者和交談屬性](bot-builder-howto-v4-state.md)。

> [!NOTE]
> 採用 REST API 用戶端的多回合交談必須記錄自身狀態，例如在資料庫或資料表儲存體中。 

## <a name="conversation-topics"></a>交談主題

您可將 Bot 設計為能處理多種類型的工作。 例如，您的 Bot 可提供不同交談流程，包括歡迎使用者、提交訂單、取消要求和取得協助。 若要處理在不同工作或交談主題之間切換的情況，其中一個方式就是從目前訊息辨識意圖 (使用者想做什麼)。 

### <a name="recognize-intent"></a>辨識意圖

Bot Builder SDK 提供的_辨識器_可處理每個內送訊息，如此您的 Bot 即可起始適當的交談流程。 在_接收回呼_之前，辨識器會檢視使用者的訊息內容以判斷意圖，然後使用接收回呼中的回合內容物件，將意圖傳回至 Bot，並在[回合內容](bot-builder-concept-activity-processing.md#turn-context)物件中將其儲存為**最高意圖**。 

判斷出**最高意圖**的辨識器只要使用規則運算式、Language Understanding (LUIS) 或您開發的其他邏輯做為中介軟體即可。 以下是辨識器範例：
   
* 您使用規則運算式設定辨識器，以偵測每次使用者提到「協助」一詞。
* 您使用 Language Understanding (LUIS) 提供各種使用者要求協助的範例，再將其對應至「協助」意圖，藉此訓練服務。
* 您建立自己的辨識器中介軟體，其可檢查內送活動，並在每次偵測到其他語言的訊息時傳回「翻譯」意圖。

如需詳細資訊，請參閱 [Language Understanding with LUIS](bot-builder-concept-luis.md)。 <!-- TODO: ADD THIS TOPIC OR SNIPPET-->

### <a name="consider-how-to-interrupt-conversation-flow-or-change-topics"></a>請思考如何中斷交談流程或變更主題

若要追蹤自己目前位於交談中的位置，其中一個方式是使用[交談狀態](bot-builder-howto-v4-state.md)儲存與目前作用中主題相關的資訊，或目前已依序完成哪些步驟。

如果 Bot 機制變得更複雜，您也可想像成是一連串交談流程在堆疊中發生；例如，Bot 會叫用新順序的流程，然後叫用產品搜尋流程； 然後，使用者會選取產品並確認，完成產品搜尋流程後，接著完成訂單。

不過，交談很少依循邏輯路徑線性發生。 使用者並不會以堆疊式對話，而會頻繁改變心意。 請思考下列範例：

![使用者說出非預期的內容](./media/concept-conversation/interruption.png)

雖然您的 Bot 可能會以邏輯方式建構對話方塊的流程堆疊，但使用者可能會決定要進行完全不同的事項，或是詢問可能與目前主題不相關的問題。 在範例中，使用者會詢問問題，而不是提供流程所預期的是/否回應。 您的 Bot 應該如何回應？

* 堅持由使用者先回答問題。
* 忽略使用者先前已完成的所有項目、重設整個流程堆疊，並嘗試回答使用者的問題來從頭開始。
* 嘗試回答使用者的問題，然後返回是/否問題，接著嘗試從該處繼續執行。

這個問題沒有任何「正確」答案，因為最好的解決方案將取決於您案例的細節，以及使用者會如何合理預期 Bot 回應。 如需詳細資訊，請參閱[處理使用者中斷](bot-builder-howto-handle-user-interrupt.md)。

> [!TIP]
> 如果您使用 Bot Builder SDK for Node.Js，則可透過[交談方塊](bot-builder-dialog-manage-conversation-flow.md)管理交談流程。

## <a name="conversation-lifetime"></a>交談存留期

<!-- Note: these activities are dependent on whether the channel actually sends them. Also, we should add links -->每當 Bot 已加入交談、其他成員已加入或從交談中移除，或交談中繼資料已變更，Bot 會收到_交談更新_活動。
您可能會希望 Bot 透過向使用者顯示歡迎訊息或自我介紹的方式，回應交談更新活動。

Bot 會收到_結束交談_活動，表示使用者已結束交談。 Bot 會傳送_結束交談_ 活動，向使用者表示交談已結束。 如果您已儲存交談相關資訊，交談結束後，也許會想要清除這些資訊。

<!--  Types of conversations

Your bot can support multi-turn interactions where it prompts users for multiple peices of information. It can be focused on a very specific task or support multiple types of tasks. 
The Bot Builder SDK has some built-in support for Language Understatnding (LUIS) and QnA Maker for adding natural language "question and answer" features to your bot.

<!--TODO: Add with links when these topics are available:
[Conversation flow] and other design articles.
[Using recognizers] [Using state and storage] and other how tos.
-->
## <a name="conversations-channels-and-users"></a>交談, 通道, 使用者

交談可能是與特定使用者的_直接_交談，或與多位使用者的_群組_交談。
Bot 在通道上和使用者通訊，是透過與使用者彼此傳送及接收活動的方式來進行。

- 每位使用者在每個通道都有一個唯一識別碼。
- 每個交談在每個通道都有一個唯一識別碼。
- 通道開啟交談時，會設定交談識別碼。
- 不過若 Bot 已有交談識別碼就無法開啟交談，但可繼續已有的交談。
- 並非所有通道都支援群組對話。

## <a name="next-steps"></a>後續步驟

對於較為複雜的交談 (例如上文中提及的交談)，我們必須要將資料留存更久的時間。 接下來說明狀態和儲存體。

> [!div class="nextstepaction"]
> [狀態和儲存體](bot-builder-storage-concept.md)

<!-- In addition, your bot can send activities back to the user, either _proactively_, in response to internal logic, or _reactively_, in response to an activity from the user or channel.-->
<!--TODO: Link to messaging how tos.-->

<!--  TODO: Change to next steps, one for each of LUIS and State
## See also

- Activities
- Adapter
- Context
- Proactive messaging
- State
-->

[QnAMaker]:(bot-builder-luis-and-qna.md#using-qna-maker)

<!-- TODO: Update when the Dispatch concept is pushed -->
[Dispatch]:(bot-builder-concept-luis.md)
