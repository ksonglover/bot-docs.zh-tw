---
title: 認知服務 | Microsoft Docs
description: 了解如何運用 Microsoft 認知服務為 Bot 加入人工智慧。
keywords: 語言力解, 知識擷取, 語音辨識, web 搜尋
author: RobStand
ms.author: rstand
manager: rstand
ms.topic: article
ms.prod: bot-framework
ms.date: 12/17/2017
ms.openlocfilehash: 85b3c86582ce1d653159e0d36516b4a90f5a3220
ms.sourcegitcommit: 1abc32353c20acd103e0383121db21b705e5eec3
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/21/2018
ms.locfileid: "42756551"
---
# <a name="cognitive-services"></a>認知服務

您可以藉由 Microsoft 認知服務，應用各領域專家不斷開發之一系列的強大 AI 演算法，包括電腦視覺、語音、自然語言處理、知識擷取和 Web 搜尋。 此服務簡化了各種 AI 工作，只要幾行程式碼，您就能快速在 Bot 中新增最新的智慧技術。 API 整合了大部分的新語言和平台。 API 亦可持續改善與學習，變得更聰明，讓服務體驗隨時保持最新狀態。 

智慧 Bot 回應的方式就如同真人思考。 他們會從不同來源發掘資訊並擷取知識，以提供實用的解答，最重要的是，還可從每次累積的經驗中學習，持續精進本身能力。 

## <a name="language-understanding"></a>語言理解

使用者與 Bot 的互動大多格式自由，因此 Bot 需要了解自然語言及與內容相關的語言。 認知服務語言 API 提供強大的語言模型，可協助您判斷使用者想要什麼、識別句子中的概念和實體，最終讓您的 Bot 以適當的動作來回應。 這五個 API 支援多種文字分析功能，例如拼字檢查、情感偵測、語言模型，且可從文字中擷取正確且豐富的深入解析。 

認知服務能為這些 API 提供語言理解能力：

- <a href="https://www.microsoft.com/cognitive-services/en-us/language-understanding-intelligent-service-luis" target="_blank">Language Understanding Intelligent Service (LUIS)</a> 可使用預先建置或自訂訓練的語言模型處理自然語言。 如需更多詳細資訊，請參閱 [Bot 的 Language Understanding](v4sdk/bot-builder-concept-luis.md)

- <a href="https://www.microsoft.com/cognitive-services/en-us/text-analytics-api" target="_blank">文字分析 API</a> 可從文字中偵測情緒、主要片語、主題和語言。

- <a href="https://www.microsoft.com/cognitive-services/en-us/bing-spell-check-api" target="_blank">Bing 拼字檢查 API</a> 提供強大的拼字檢查功能，且可辨識名稱、品牌名稱和俚語之間的差異。

- <a href="https://docs.microsoft.com/en-us/azure/machine-learning/studio/text-analytics-module-tutorial" target ="_blank">Azure Machine Learning Studio 中的文字分析模型</a>可讓您建置及操作文字分析模型，例如：詞形還原及文字預先處理。 這些模型可協助您解決文件分類或情緒分析等問題。

深入了解採用 Microsoft 認知服務的 [Language Understanding][language]。

## <a name="knowledge-extraction"></a>知識擷取

認知服務提供四種知識 API，可讓您識別非結構化文字中的具名實體或片語、新增個人化建議、根據自然解譯使用者查詢提供自動完成建議，以及搜尋學術文章及其他研究，如同個人化常見問答集服務。

- <a href="https://www.microsoft.com/cognitive-services/en-us/entity-linking-intelligence-service" target="_blank">實體連結智慧服務</a>可使用文字中提及的相關實體標註非結構化文字。 視內容而定，相同的字詞或片語可能指涉不同事物。 此服務了解所提供之文字中的情境，且能識別您文字中的每個實體。    

- <a href="https://www.microsoft.com/cognitive-services/en-us/knowledge-exploration-service" target="_blank">知識探索服務</a>可解譯自然語言使用者查詢字串，再傳回加上註釋的解譯，進而支援可預期使用者輸入內容的豐富搜尋及自動完成功能體驗。 立即查詢完成建議和預期查詢調整皆以您自己的資料和應用程式特定文法為依據，可讓使用者執行更快的查詢。    

- <a href="https://www.microsoft.com/cognitive-services/en-us/academic-knowledge-api" target="_blank">學術知識 API</a> 可從 <a href="https://www.microsoft.com/en-us/research/project/microsoft-academic-graph/" target="_blank">Microsoft Academic Graph</a> 傳回學術研究文章、作者、期刊、研討會、主題和大學資訊。 學術知識 API 以網域特定形式的範例內建於知識探索服務，可使用圖表式的對話方塊提供知識庫，且可從數千萬筆研究相關實體中搜尋內容。 搜尋主題、教授、大學或研討會，然後 API 為提供相關的出版刊物及相關實體。 其文法亦支援自然查詢如：「Michael Jordan 在 2010 年後有關機器學習的研究文章」。

- <a href="https://qnamaker.ai" target="_blank">QnA Maker</a> 是一項簡單易用的免費 REST API 及 Web 服務，可訓練 AI 以自然的交談方式回應使用者問題。 QnA Maker 具備最佳化機器學習邏輯，以及整合領先業界語言處理技術的能力，可抽取半結構化資料，例如：由問與答組成清楚且實用的解答。

深入了解採用 Microsoft 認知服務的[知識擷取][knowledge]。

## <a name="speech-recognition-and-conversion"></a>語音辨識和轉換

使用語音 API 為 Bot 新增進階語音技術，即可運用領先業界的語音轉文字演算法及語音辨識功能。 語音 API 採用內建的語言和原音模式，其可高準確涵蓋各種情境。 

對於需要進一步自訂的應用程式，您可使用自訂辨識智慧型服務 (CRIS)。 此可讓您根據應用程式的詞彙，甚至使用者的說話方式進行調整，以自訂語音辨識器的原音模型。

認知服務中有三個語音 API 可處理或合成語音：

- <a href="https://www.microsoft.com/cognitive-services/en-us/speech-api" target="_blank">Bing 語音 API</a> 支援語音轉文字和文字轉語音功能。
- <a href="https://www.microsoft.com/cognitive-services/en-us/custom-recognition-intelligent-service-cris" target="_blank">自訂辨識智慧型服務 (CRIS)</a> 可讓您建立自訂語音辨識模型，並根據應用程式的詞彙或使用者的說話方式調整語音轉文字設定。
- <a href="https://www.microsoft.com/cognitive-services/en-us/speaker-recognition-api" target="_blank">說話者辨識 API</a> 可透過聲音辨識及驗證說話者。

下列資源提供有關新增語音辨識功能至 Bot 的額外資訊。

* [應用程式的 Bot 交談影片概觀](https://channel9.msdn.com/events/Build/2017/P4114)
* [適用於 UWP 或 Xamarin 應用程式的 Microsoft.Bot.Client 程式庫](https://aka.ms/BotClient)
* [Bot 用戶端程式庫範例](https://aka.ms/BotClientSample)
* [支援語音的 WebChat 用戶端](https://aka.ms/BFWebChat)

深入了解採用 Microsoft 認知服務的[語音辨識和轉換][speech]。

## <a name="web-search"></a>Web 搜尋

Bing 搜尋 API 可讓您新增智慧 Web 搜尋功能至 Bot。 只要使用幾行程式碼，即可存取數十億筆網頁、影像、影片、新聞及其他結果類型。 您可設定 API 按地理位置、市場或語言傳回結果，以提升相關性。 您可使用支援的搜尋參數進一步自訂搜尋，例如：使用 Safesearch 篩選出成人內容，或使用 Freshness 根據特定日期傳回結果。

認知服務中有五種可用的 Bing 搜尋 API。

- <a href="https://www.microsoft.com/cognitive-services/en-us/bing-web-search-api" target="_blank">Web 搜尋 API</a> 可透過單一 API 呼叫提供網頁、影像、影片、新聞和相關搜尋結果。

- <a href="https://www.microsoft.com/cognitive-services/en-us/bing-image-search-api" target="_blank">影像搜尋 API</a> 可傳回具有增強中繼資料 (主色、影像類型等等) 的影像結果，並支援多種影像篩選功能以便自訂結果。

- <a href="https://www.microsoft.com/cognitive-services/en-us/bing-video-search-api" target="_blank">影片搜尋 API</a> 可擷取具有豐富中繼資料 (影片大小、品質、價格等等) 的影片結果、影片預覽，並支援多種影像篩選功能以便自訂結果。

- <a href="https://www.microsoft.com/cognitive-services/en-us/bing-news-search-api" target="_blank">新聞搜尋 API</a> 可尋找符合搜尋查詢的全球新聞報導，或目前在網際網路上的熱門趨勢話題。

- <a href="https://www.microsoft.com/cognitive-services/en-us/bing-autosuggest-api" target="_blank">自動建議 API</a> 提供立即查詢完成建議，可讓您用更少的輸入字元，更快完成搜尋查詢。 

深入了解採用 Microsoft 認知服務的[ Web 搜尋][search]。

## <a name="image-and-video-understanding"></a>影像與影片理解

視覺 API 可為您的 Bot 新增進階影像和影片理解技術。 最新驗算法可讓您處理影像或影片，並從中取得相關資訊以採取動作。 例如，您可使用這些資訊辨識物件、人臉、年齡、性別甚至情緒。 

視覺 API 支援各種影像理解功能。 其可識別完整或明確的內容、預估並強調輔色、分類影像內容、進行光學字元辨識，且可以完整英文句子描述影像。 視覺 API 亦支援多種影像和影片處理功能，例如以智慧方式產生影像或影片縮圖，或穩定影片輸出。

認知服務提供四種您可用於處理影像或影片的 API：

- <a href="https://www.microsoft.com/cognitive-services/en-us/computer-vision-api" target="_blank">電腦視覺 API</a> 可擷取與影像相關的豐富資訊 (例如物件或人員)、判斷影像是否包含完整或明確的內容，以及 (使用 OCR) 處理影像中的文字。

- <a href="https://www.microsoft.com/cognitive-services/en-us/emotion-api" target="_blank">表情 API</a> 可分析人類臉部表情，並以人類可能出現的八種情緒分類來辨識情緒。

- <a href="https://www.microsoft.com/cognitive-services/en-us/face-api" target="_blank">臉部 API</a> 可偵測人臉、與類似的臉比較，甚至根據視覺上的相似程度將人員分組。

- <a href="https://www.microsoft.com/cognitive-services/en-us/video-api" target="_blank">影片 API</a> 可分析並處理影片，以穩定影片輸出、偵測動作、追蹤臉部，且可產生影片的動態縮圖摘要。

深入了解採用 Microsoft 認知服務的[影像和影片理解][vision]。

## <a name="additional-resources"></a>其他資源

如需每項產品的完整文件及其對應的 API 參考資料，請參閱<a href="https://docs.microsoft.com/azure/cognitive-services" target="_blank">認知服務文件</a>。

[language]: https://docs.microsoft.com/en-us/azure/cognitive-services/luis/home
[search]: https://docs.microsoft.com/en-us/azure/cognitive-services/bing-web-search/search-the-web
[vision]: https://docs.microsoft.com/en-us/azure/cognitive-services/computer-vision/home
[knowledge]: https://docs.microsoft.com/en-us/azure/cognitive-services/kes/overview
[speech]: https://docs.microsoft.com/en-us/azure/cognitive-services/speech/home
[location]: https://docs.microsoft.com/en-us/azure/cognitive-services/
