---
title: 設計知識型 Bot | Microsoft Docs
description: 了解如何透過不同方式設計能尋找並傳回資訊，以回應使用者輸入或查詢的知識型 Bot。
author: matvelloso
ms.author: mateusv
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: cognitive-services
ms.date: 12/13/2017
ms.openlocfilehash: 4e73a56eb94207de49d8684c4db26155554820f3
ms.sourcegitcommit: a295a90eac461f8b96770dd902ba44919acf33fc
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/26/2019
ms.locfileid: "67405856"
---
# <a name="design-knowledge-bots"></a>設計知識型 Bot

您可設計知識型 Bot 來提供有關任何主題的資訊。 例如，您可讓一個知識型 Bot 回答「這場研討會有哪些 Bot 活動？」、「下場雷鬼表演什麼時候？」或「誰是 Tame Impala？」等事件的問題。 再設計一個知識型 Bot 負責回答「我如何更新作業系統？」 或「我要去哪裡重設密碼？」這類 IT 相關問題。 而另一個 Bot 則可回答「誰是 John Doe？」或 「John Doe 的電子郵件地址是什麼？」等有關連絡資訊的問題。 

無論知識型 Bot 是專為哪些使用案例而設計，其基本目標皆相同：利用大量資料 (例如：SQL 資料庫中的關聯式資料、非關聯式存放區中的 JSON 資料或文件存放區中的 PDF)，尋找並傳回使用者要求的資訊。 

## <a name="search"></a>Search

搜尋功能在 Bot 中是相當重要的工具。 

首先，「模糊搜尋」不需要使用者提供精確的輸入內容，也能讓 Bot 傳回可能與使用者問題相關的資訊。 例如，如果使用者詢問音樂知識型 Bot 有關「impala」的資訊 (而非精準輸入「Tame Impala」)，則 Bot 還是可在回應中提供最有可能與該輸入相關的資訊。

![對話方塊結構](~/media/bot-service-design-pattern-knowledge-base/fuzzySearch2.png)

搜尋分數表示特定搜尋結果的可信度，Bot 便可依此排序結果，甚至根據可信度客製化其通訊內容。 例如，如果可信度很高，則 Bot 即可用「以下是最符合您搜尋的活動：」回應使用者。

![對話方塊結構](~/media/bot-service-design-pattern-knowledge-base/searchScore2.png)

如果可信度很低，則 Bot 可回答「嗯...您要找的是這些活動嗎？」

![對話方塊結構](~/media/bot-service-design-pattern-knowledge-base/searchScore1.png)

### <a name="using-search-to-guide-a-conversation"></a>使用搜尋功能引導交談

如果您建置 Bot 只是為了使用基本的搜尋引擎功能，則可能根本不需要 Bot。 交談式介面能提供使用者哪些無法從網頁瀏覽器上一般搜尋引擎取得的內容？ 

當知識型 Bot 是設計用於引導交談時，通常最能發揮效益。 交談就是指使用者 和 Bot 來回交換訊息，因此可讓 Bot 有機會進一步詢問以釐清問題、提供選項並驗證結果；而這些都是基本搜尋功能無法辦到的。 例如，以下 Bot 可透過交談引導使用者利用 Facet 和篩選條件過濾資料集，直到其找出使用者所需的資訊。

![對話方塊結構](~/media/bot-service-design-pattern-knowledge-base/guidedConvo1.png)

![對話方塊結構](~/media/bot-service-design-pattern-knowledge-base/guidedConvo2.png)

![對話方塊結構](~/media/bot-service-design-pattern-knowledge-base/guidedConvo3.png)

![對話方塊結構](~/media/bot-service-design-pattern-knowledge-base/guidedConvo4.png)

藉由處理使用者每個步驟的輸入內容並呈現相關選項，Bot 可引導使用者找到他們所需的資訊。 Bot 提供該資訊後，未來甚至可透過更有效率的方式提供引導，進而找出類似的資訊。 

![對話方塊結構](~/media/bot-service-design-pattern-knowledge-base/Training.png)

### <a name="azure-search"></a>Azure 搜尋服務

透過 <a href="https://azure.microsoft.com/services/search/" target="_blank">Azure 搜尋服務</a>，您可建立有效率的搜尋索引，讓 Bot 輕鬆搜尋、利用 Facet 過濾資料和篩選。 考慮使用 Azure 入口網站建立的搜尋索引。

![對話方塊結構](~/media/bot-service-design-pattern-knowledge-base/search3.PNG)

您想要存取資料存放區的所有屬性，因此將每個屬性設為「可擷取」。 您想要按姓名尋找音樂家，因此將 **Name** 屬性設為「可搜尋」。 最後，您希望可利用 Facet 進一步篩選音樂家的年代，因此將 **Eras** 屬性設為「可 Facet」和「可篩選」。 

Facet 功能可針對指定屬性判斷存在於資料存放區中的值，以及每個值的規模。 例如，此螢幕擷取畫面顯示資料存放區中有 5 個不同的年代：

![對話方塊結構](~/media/bot-service-design-pattern-knowledge-base/facet.png)

篩選可進一步僅選取特定屬性的指定執行個體。 例如，您可篩選上述結果集，以僅包含 **Era**等於「浪漫時期」的項目。 

> [!NOTE]
> 如需完整範例，以了解如何使用 Azure Document DB、Azure 搜尋服務和 Microsoft Bot Framework 建立知識型 Bot，請參閱<a href="https://github.com/ryanvolum/AzureSearchBot" target="_blank">範例 Bot</a>。
> 
> 簡單起見，以上範例顯示的搜尋索引是使用 Azure 入口網站建立。 
> 亦可透過程式設計方式建立索引。

## <a name="qna-maker"></a>QnA Maker

有些知識型 Bot 的功能僅單純回答常見問題集 (FAQ)。 
<a href="https://www.microsoft.com/cognitive-services/qnamaker" target="_blank">QnA Maker</a> 是功能強大的工具，專用於此使用案例。 QnA Maker 的內建功能可從現有的常見問題集網站抓取問答，此外還可讓您手動配置專屬的自訂問答清單。 QnA Maker 支援自然語言處理功能，甚至可針對用字與預期有些微不同的問題提供回答。 不過，其並無語意語言理解能力。 舉例來說，QnA Maker 就無法判斷「幼犬 (puppy)」是「狗 (dog)」的一種。 

使用 QnA Maker Web 介面，您即可設定具有三組問答的知識庫： 

![對話方塊結構](~/media/bot-service-design-pattern-knowledge-base/KnowledgeBaseConfig.png)

接著，您可提出一連串的問題進行測試： 

![對話方塊結構](~/media/bot-service-design-pattern-knowledge-base/exampleQnAConvo.png)

Bot 可正確回答直接對應至知識庫中已經配置的問題。 不過，卻無法正確回答「我能不能自備茶飲？」這樣的問題，因為此問題最類似於「我能不能自備伏特加？」的問題結構。 QnA Maker 之所以無法提供正確回答問題，是因為其本身並無法理解文字的意義。 QnA Maker 並不知道「茶」是一種無酒精飲料。 因此便會回答「禁帶酒精飲料」。

> [!TIP]
> 建立 QnA 組合，然後利用交談下方的 [檢查] 按鈕，針對每個錯誤回答選取其他答案，藉此進行測試並反覆訓練 Bot。 

## <a name="luis"></a>LUIS

有些知識型 Bot 需要自然語言處理 (NLP) 能力，以便分析使用者訊息並判斷其意圖。 
[Language Understanding (LUIS)](https://www.luis.ai) 提供快速且有效率的方式讓您為 Bot 新增 NLP 功能。 LUIS 可讓您依據需求從 Bing 和 Cortana 選用預先建置的現有模型，亦可打造專屬的特製化模型。 

使用龐大資料集時，以每種變異實體訓練 NLP 模型不一定可行。 例如，在音樂播放 Bot 中，使用者可傳送「播放雷鬼」、「播放 Bob Marley」或「播放 One Love」等訊息。 雖然 Bot 可將這每一個訊息對應到「playMusic」意圖，但若未以每個表演者、類別和歌曲名稱逐一訓練，NLP 模型可能無法識別該實體是類別、表演者或歌曲。 利用 NLP 模型識別類型為「音樂」的一般實體後，Bot 則可在其資料存放區中搜尋該實體，並接續處理。 

## <a name="combining-search-qna-maker-andor-luis"></a>結合搜尋、QnA Maker 和/或 LUIS

搜尋、QnA Maker 和 LUIS 各自都是強大的工具，而相互搭配運用則可打造出能擁有上述多種能力的知識型 Bot。

### <a name="luis-and-search"></a>LUIS 和搜尋

在[前文提及](#search)的音樂祭 Bot 範例中，Bot 可藉由顯示呈現節目表按鈕引導交談進行。 不過，此 Bot 也可使用 LUIS 整合自然語言理解能力，以判斷問題的意圖和實體，例如「Romit Girdhar 演奏什麼類型的音樂？」 接著，Bot 可使用音樂家姓名在 Azure 搜尋索引中搜尋。 
 
由於有很多可能的值，因此不太可能以每個音樂家的姓名來訓練模型，但您可提供足夠的代表性範例，讓 LUIS 可正確識別出眼前的實體。  例如，您可考慮提供以下音樂家姓名來訓練模型： 

![對話方塊結構](~/media/bot-service-design-pattern-knowledge-base/answerGenre.png)
![對話方塊結構](~/media/bot-service-design-pattern-knowledge-base/answerGenreOneWord.png)

使用新的表達方式如「披頭四演奏什麼類型的音樂？」測試此模型時，LUIS 可成功判斷意圖「answerGenre」並識別實體「披頭四」。 不過，若您送出較長的問題如「The Devil Makes Three 演奏什麼類型的音樂？」，LUIS 則只會將「The Devil」判斷為實體。

![對話方塊結構](~/media/bot-service-design-pattern-knowledge-base/devilMakesThreeScore.png)

利用可代表基本資料集的範例實體訓練模型，有助於提高 Bot 語言理解的準確性。 

> [!TIP]
> 一般而言，建議讓模型遇到在實體辨識中識別贅字詞時出錯，例如，從「請打電話給 John Smith」表達方式中識別「請 John Smith」；而不是以識別的字詞太少來做為出錯依據，例如，從「請打電話給 John Smith」表達方式識別「John」。 搜尋索引會忽略「請 John Smith」片語中無關的字詞如：「請」。 

### <a name="luis-and-qna-maker"></a>LUIS 和 QnA Maker

部分知識型 Bot 可能搭配 LUIS 使用 QnA Maker 回答基本問題，以判斷意圖、擷取實體並叫用其他更詳盡的對話方塊。 例如，請考慮使用簡易型 IT 技術支援中心 Bot。 此 Bot 可使用 QnA Maker 回答有關 Windows 或 Outlook 的基本問題，此外也必須協助需要意圖辨識及使用者和 Bot 來回通訊的狀況，例如：重設密碼。 有幾種方式可讓 Bot 混合實作 LUIS 和 QnA Maker：

1. 同時呼叫 QnA Maker 和 LUIS，然後使用第一個傳回特定閾值分數的資訊回應使用者。 
2. 先呼叫 LUIS，如果沒有任何意圖符合特定閾值分數 (即觸發「無」意圖)，則呼叫 QnA Maker。 或者，針對 QnA Maker 建立 LUIS 意圖，並使用對應至「QnAIntent」的範例 QnA 問題饋送至 LUIS 模型。 
3. 先呼叫 QnA Maker，如果沒有任何答案符合特定閾值分數，則呼叫 LUIS。 

Bot Framework SDK 具備 LUIS 和 QnA Maker 內建支援。 此可讓您觸發對話方塊，或使用 LUIS 和/或 QnA Maker 自動回答問題，而無須實作這兩項工具的自訂呼叫。 如需詳細資訊，請參閱[分派工具教學課程](https://docs.microsoft.com/azure/bot-service/bot-builder-tutorial-dispatch?view=azure-bot-service-4.0)。

> [!TIP]
> 實作 LUIS、QnA Maker 和/或 Azure 搜尋服務組合時，使用每項工具測試輸入，以判斷每個模型的閾值分數。 LUIS、QnA Maker 和 Azure 搜尋服務會各自使用不同的評分條件來產生分數，因此這些工具產生的分數不一定相容。 此外，LUIS 和 QnA Maker 會標準化分數。 在 LUIS 模型中視為「良好」的分數，在其他模型不一定分類到「良好」。 

## <a name="sample-code"></a>範例程式碼

- 如需相關範例以了解如何使用適用於 .NET 的 Bot Framework SDK 建立基本知識型 Bot，請參閱 GitHub 中的<a href="https://aka.ms/qna-with-appinsights" target="_blank">知識型 Bot 範例</a>。 
<!-- TODO: Do not have a current bot sample to work with this
- For a sample that shows how to create more complex knowledge bots using the Bot Framework SDK for .NET, see the <a href="https://github.com/Microsoft/BotBuilder-Samples/tree/master/CSharp/demo-Search" target="_blank">Search-powered Bots sample</a> in GitHub.
-->

[qnamakerTemplate]: https://docs.botframework.com/azure-bot-service/templates/qnamaker/#navtitle
