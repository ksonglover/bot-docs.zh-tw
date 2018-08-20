---
title: 適用於 Bot 的 Language Understanding | Microsoft Docs
description: 了解如何運用 Microsoft 認知服務為 Bot 加入人工智慧。
keywords: LUIS, 意圖, 辨識器, 分派工具, qna, qna maker
author: DeniseMak
ms.author: v-demak
manager: rstand
ms.topic: article
ms.prod: bot-framework
ms.date: 03/22/2018
ms.reviewer: ''
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: ec7385db42264b44303e6abd6a7edde06bfe84e9
ms.sourcegitcommit: dcbc8ad992a3e242a11ebcdf0ee99714d919a877
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/30/2018
ms.locfileid: "39352887"
---
# <a name="language-understanding-for-bots"></a>適用於 Bot 的 Language Understanding
[!INCLUDE [pre-release-label](../includes/pre-release-label.md)]

Bot 可以使用各種對話風格，從結構化和引導式到自由形式和開放式。 Bot 需要根據使用者的談話內容來決定在對話流程中下一步該做什麼，而在開放式對話中，使用者的回覆還有更多可能。

| 引導式 | 開啟 |
|------|------|
| 我是旅遊 Bot。 請選取下列選項之一：尋找班機、尋找旅館、尋找租車。 | 我可以協助您預訂旅遊行程。 您要執行什麼動作？ |
| 您還需要其他項目嗎？ 請按一下 [是] 或 [否]。 | 您還需要其他項目嗎？ |

使用者與 Bot 通常會以自由形式進行互動，因此 Bot 需要了解自然語言及與內容相關的語言。 這篇文章說明 Language Understanding (LUIS) 如何幫助您判斷使用者想要什麼、識別句子中的概念和實體，最終讓您的 Bot 以適當的動作來回應。

## <a name="recognize-intent"></a>辨識意圖

[LUIS](https://www.luis.ai) (英文) 可協助您判斷使用者的**意圖**，也就是從使用者說話內容判斷他們想要做什麼，以便您的 Bot 可以適當回應。 當使用者對 Bot 說話的內容未遵循可預測的結構或特定模式時，LUIS 會特別有幫助。 如果 Bot 擁有對話式使用者介面，使用者可在其中說話或輸入回應，「語句」可能會有無數的變化 (語句是使用者的口語或文字輸入)。

例如，設想旅遊 Bot 的使用者要求預訂班機可使用的許多表達方法。 



![以各種不同形式的語句來預訂班機](media/cognitive-services-add-bot-language/cognitive-services-luis-utterances.png)

這些語句可能有不同的結構，並包含各種您想不到的「班機」同義詞。 對 Bot 而言，撰寫符合所有語句的邏輯，並與其他包含相同字組的意圖區別開來十分困難。 此外，Bot 必須擷取「實體」，也就是其他重要的字組，例如位置和時間。 LUIS 可在內容中為您識別意圖和實體，以簡化此程序。

當您設計用於自然語言輸入的 Bot 時，您要判斷 Bot 必須辨識哪些意圖和實體，並思考其如何連接到 Bot 所採取的動作。 在 <a href="https://www.luis.ai" target="_blank">LUIS</a> (英文) 中，藉由為每個意圖提供範例，並標記其中的實體，您可以定義自訂意圖和實體並指定其行為。

Bot 會使用由 LUIS 辨識的意圖來判斷對話主題，或開始對話流程。 例如，當使用者說：「我想要預訂班機」，Bot 會偵測到 BookFlight 意圖，並叫用開始搜尋航班的對話流程。 LUIS 會在觸發意圖的原始語句以及在對話流程中，偵測目的地城市和起飛日期等實體。 當 Bot 擁有所需的所有資訊後，就可以滿足使用者意圖。


![由 BookFlight 意圖觸發與 Bot 的對話](media/cognitive-services-add-bot-language/cognitive-services-luis-conversation-high-level.png)


### <a name="recognize-intent-in-common-scenarios"></a>在常見案例中辨識意圖

為了節省開發時間，LUIS 提供預先訓練的語言模型，可辨識 Bot 常見類別的常見語句。 <!-- Consider if you'll use prebuilt or custom intents and entities: -->

* **預先建置的領域**是預先訓練且已就緒可供使用的意圖和實體集合，可搭配用於常見案例，如約會、提醒、管理、保健、娛樂、通訊、預約等。 **公用程式**預先建置領域可協助 Bot 處理一般工作，如取消、確認、說明、重複、及停止。 請參閱 [C#]( https://github.com/Microsoft/botbuilder-dotnet/tree/master/samples-final/8.AspNetCore-LUIS-Bot) (英文) 或 [JavaScript](https://github.com/Microsoft/botbuilder-js/tree/master/samples/luis-bot-es6) (英文) 的提醒範例，以取得在 Bot 中使用預先建置領域的範例，並查看 LUIS 提供的[預先建置的領域](https://docs.microsoft.com/en-us/azure/cognitive-services/LUIS/luis-how-to-use-prebuilt-domains)。
* **預先建置的實體**可協助 Bot 辨識常見類型的資訊，例如日期、時間、數字、溫度、貨幣、地理和年齡。
請參閱[擷取具類型的 LUIS 結果][luis-v4-typed-entities]，以取得使用 LUIS 擷取日期的範例。 請參閱[使用預先建置的實體](https://docs.microsoft.com/en-us/azure/cognitive-services/LUIS/pre-builtentities)，了解 LUIS 可辨識的類型背景資訊。 



<!-- TODO: Link to Bot Framework design guidance about LUIS apps, when this is ready --> 


## <a name="how-your-bot-gets-messages-from-luis"></a>您的 Bot 如何從 LUIS 取得訊息
每當整合 LUIS 的 Bot 收到語句，Bot 會將它傳送至 LUIS 應用程式，其會傳回包含意圖和實體的 JSON 回應。 Bot 建立器 SDK 會提供功能 (實作為[中介軟體](bot-builder-concept-middleware.md)) 以自動處理 LUIS 的回應，並將其傳遞至您的 Bot。 您可以使用 Bot _接收處理常式_中的_回合內容_，根據 LUIS 回應中的意圖來路由傳送對話流程。 

![意圖和實體如何傳遞至您的 Bot](./media/cognitive-services-add-bot-language/cognitive-services-luis-message-flow-bot-code.png)

請參閱下列項目以開始整合 LUIS 應用程式與 Bot：

* [將 LUIS 用於對話流程][luis-v4-how-to]

## <a name="best-practices-for-language-understanding"></a>Language Understanding 的最佳做法
為您的 Bot 設計語言模型時，請考慮下列做法。

### <a name="consider-the-number-of-intents"></a>考慮意圖數目
LUIS 應用程式會將語句分類成多個類別以辨識意圖。 結果自然是，要從一大堆意圖中判斷正確類別，可能會降低 LUIS 應用程式區分意圖的能力。 

減少意圖數目的其中一種方法是使用階層式設計。 想像個人助理 Bot 的情況，它有三個與天氣相關的意圖、三個與居家自動化相關的意圖，以及三個其他公用程式的意圖，分別是說明、取消和問候。 如果您將所有意圖放在同一 LUIS 應用程式中，就有 9 個意圖，而當您將功能新增到 Bot 時，就會有數十個意圖。 因此，您可以使用發送器 LUIS 應用程式，判斷使用者的要求是針對天氣、居家自動化或公用程式，然後針對發送器決定的類別呼叫 LUIS 應用程式。 在此情況下，每個 LUIS 應用程式只會從 3 個意圖開始。 

### <a name="use-a-none-intent"></a>使用 None 意圖 
通常情況是 Bot 的使用者會說出非預期或與目前對話流程不相關的內容。 None 意圖即是用來處理這些訊息。 如果您未訓練意圖來處理後援、預設或「非上述之一」的情況，LUIS 應用程式只能將訊息分類到已定義的意圖中。 比方說，假設您的 LUIS 應用程式具有兩個意圖：`HomeAutomation.TurnOn` 和 `HomeAutomation.TurnOff`。 如果這些是僅有的意圖，而輸入是像「安排星期五的約會」等無關項目，您的 LUIS 應用程式仍必須將該訊息分類為 HomeAutomation.TurnOn 或 HomeAutomation.TurnOff。 如果您的 LUIS 應用程式有一些 `None` 意圖的範例，您可以在 Bot 中提供一些後援邏輯來處理非預期的語句。  

### <a name="review-the-utterances-that-luis-app-receives"></a>檢閱 LUIS 應用程式收到的語句
LUIS 應用程式提供功能，可藉由檢閱使用者傳送的訊息，改善應用程式效能。 請參閱[檢閱建議語句](https://docs.microsoft.com/en-us/azure/cognitive-services/LUIS/label-suggested-utterances)的逐步解說。

## <a name="integrate-multiple-luis-apps-and-qna-services-with-the-dispatch-tool"></a>使用分派工具整合多個 LUIS 應用程式和 QnA 服務

<!-- 1. Modular. 2. Better performance for classification --> 建置可了解多個對話主題的多用途 Bot 時，您可以開始分別為每個功能開發服務，然後將其整合在一起。 這些服務可包括 Language Understanding (LUIS) 應用程式和 QnAMaker 服務。 以下是一些範例案例，其中 Bot 可能會結合多個 LUIS 應用程式、多個 QnAMaker 服務或兩者的組合：

* 個人助理 Bot 可讓使用者叫用各種命令。 每個類別的命令會形成可個別開發的「技能」，且每項技能都具有 LUIS 應用程式。
* Bot 會搜尋許多知識庫來尋找常見問題集 (FAQ) 的解答。 
* 用於企業的 Bot 擁有可建立客戶帳戶和下訂單的 LUIS 應用程式，也有針對常見問題集的 QnAMaker 服務。  

### <a name="the-dispatch-tool"></a>分派工具
分派工具可透過建立「分派應用程式」 (這是新的 LUIS 應用程式，可將訊息路由傳送至適當的 LUIS 和 QnAMaker 服務)，協助您將多個 LUIS 應用程式和 QnA Maker 服務與您的 Bot 整合。 請參閱[分派教學課程](./bot-builder-tutorial-dispatch.md)，取得在一個 Bot 中結合多個 LUIS 應用程式和 QnA Maker 的逐步教學。

## <a name="use-luis-to-improve-speech-recognition"></a>使用 LUIS 改善語音辨識

對於使用者會與其對話的 Bot，與 LUIS 整合可協助 Bot 識別從語音轉換文字時可能會遭誤解的字組。  例如，在西洋棋的情境中，使用者可能會說：「騎士移動到 A 7 (Move knight to A 7)」。 若沒有使用者意圖的背景，該語句可能會識別為：「移動晚上 287 (Move night 287)」。 藉由建立代表西洋棋和座標的實體，並將其標記在語句中，您可以提供語音辨識的背景來識別它們。 您可以使用與 Bing 語音整合的 Bot Framework 通道 (例如 Web Chat、Bot Framework 模擬器和 Cortana) 來[啟用語音辨識預備][speechrecognitionpriming]。  


## <a name="additional-resources"></a>其他資源

- [語言理解](~/bot-service-concept-intelligence.md#language-understanding)
- <a href="https://www.luis.ai" target="_blank">LUIS 網站</a>

<!-- Links -->
[luis_home]: https://docs.microsoft.com/en-us/azure/cognitive-services/luis/home
[middleware]: bot-builder-concept-middleware.md
<!-- TODO: this link is a placeholder, need to find existing speech priming article -->
[speechrecognitionpriming]: ../bot-service-channel-connect-webchat-speech.md



[luis-v4-typed-entities]: bot-builder-howto-v4-luisgen.md
[luis-v4-how-to]: bot-builder-howto-v4-luis.md
[luis-v4-cs-quickstart]: https://github.com/Microsoft/botbuilder-dotnet/wiki/Using-LUIS-and-QnA-Maker
[luis-v4-js-quickstart]: https://github.com/Microsoft/botbuilder-js/wiki/Using-LUIS-and-QnA-Maker

## <a name="next-steps"></a>後續步驟

認知服務可提供方法讓您的 Bot 更有智慧。

> [!div class="nextstepaction"]
> [適用於 Bot 的認知服務](../bot-service-concept-intelligence.md)
