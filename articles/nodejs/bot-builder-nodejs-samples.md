---
title: 適用於 Node.js 的 Bot 建立器 SDK 範例 Bot | Microsoft Docs
description: 探索大量的範例 Bot 選項，以協助您以適用於 Node.js 的 Bot 建立器 SDK 開始開發 Bot。
author: v-ducvo
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: 43c178c4bbdf0bb04384bb8ada397066e6f7dd12
ms.sourcegitcommit: 67445b42796d90661afc643c6bb6533e9a662cbc
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/06/2018
ms.locfileid: "39574614"
---
# <a name="bot-builder-sdk-for-nodejs-samples"></a>適用於 Node.js 的 Bot 建立器 SDK 範例

[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]

這些範例會示範以工作為主的 Bot，顯示如何利用適用於 Node.js 的 Bot 建立器 SDK 功能。 您可使用這些範例，協助您快速開始建置具備豐富功能的絕佳 Bot。

## <a name="get-the-samples"></a>取得範例
若要取得範例，請使用 Git 複製 [BotBuilder-Samples](https://github.com/Microsoft/BotBuilder-Samples) \(英文\) GitHub 存放庫。

```cmd
git clone https://github.com/Microsoft/BotBuilder-Samples.git
cd BotBuilder-Samples
```

使用適用於 Node.js 的 Bot 建立器 SDK 所建置的範例 Bot，已整理於 **Node** 目錄中。

您也可以在 GitHub 上檢視範例，並直接將它們部署至 Azure。

## <a name="core"></a>核心
下列範例說明建置功能豐富且具有能力之 Bot 的基本技術。

範例 | 說明
------------ | ------------- 
[傳送附件](https://github.com/Microsoft/BotBuilder-Samples/tree/master/Node/core-SendAttachment) \(英文\) | 會將簡單的媒體附件 (影像) 傳送給使用者的範例 Bot。 
[接收附件](https://github.com/Microsoft/BotBuilder-Samples/tree/master/Node/core-ReceiveAttachment) \(英文\) | 會接收使用者所傳送的附件並下載它們的範例 Bot。 
[建立新對話](https://github.com/Microsoft/BotBuilder-Samples/tree/master/Node/core-CreateNewConversation) \(英文\)  | 會使用先前儲存的使用者位址開始新對話的範例 Bot。
[取得對話的成員](https://github.com/Microsoft/BotBuilder-Samples/tree/master/Node/core-GetConversationMembers) \(英文\) | 會擷取對話的成員清單並偵測清單變更的的範例 Bot。 
[直接線路](https://github.com/Microsoft/BotBuilder-Samples/tree/master/Node/core-DirectLine) \(英文\) | 會使用直接線路 API 與彼此通訊的範例 Bot 和自訂用戶端。 
[直接線路 (WebSocket)](https://github.com/Microsoft/BotBuilder-Samples/tree/master/Node/core-DirectLineWebSockets) \(英文\) | 會使用直接線路 API + WebSocket 與彼此通訊的範例 Bot 和自訂用戶端。 
[多個對話](https://github.com/Microsoft/BotBuilder-Samples/tree/master/Node/core-MultiDialogs) \(英文\) | 會顯示不同種類對話的範例 Bot。
[狀態 API](https://github.com/Microsoft/BotBuilder-Samples/tree/master/Node/core-State) \(英文\) | 會追蹤對話內容的無狀態範例 Bot。
[自訂狀態 API](https://github.com/Microsoft/BotBuilder-Samples/tree/master/Node/core-CustomState) \(英文\) | 會使用自訂儲存體提供者來追蹤對話內容的無狀態範例 Bot。
[ChannelData](https://github.com/Microsoft/BotBuilder-Samples/tree/master/Node/core-ChannelData) \(英文\) | 會使用 ChannelData 將原生中繼資料傳送至 Facebook 的範例 Bot。
[AppInsights](https://github.com/Microsoft/BotBuilder-Samples/tree/master/Node/core-AppInsights) | 會將遙測記錄到 Application Insights 執行個體的範例 Bot。

## <a name="search"></a>Search
此範例示範如何在資料驅動的 Bot 中利用 Azure 搜尋服務。

範例 | 說明
------------ | -------------
[Azure 搜尋服務](https://github.com/Microsoft/BotBuilder-Samples/tree/master/Node/demo-Search) | 可協助使用者瀏覽大量內容的兩個範例 Bot。


## <a name="cards"></a>卡片
下列範例示範如何在 Bot Framework 中傳送複合式資訊卡。

範例 | 說明
------------ | -------------
[複合式資訊卡](https://github.com/Microsoft/BotBuilder-Samples/tree/master/Node/cards-RichCards) \(英文\) | 會傳送許多不同類型之複合式資訊卡的範例 Bot。
[浮動切換的卡片](https://github.com/Microsoft/BotBuilder-Samples/tree/master/Node/cards-CarouselCards) \(英文\) | 會使用浮動切換版面配置，在單一訊息內傳送多張複合式資訊卡的範例 Bot。

## <a name="intelligence"></a>智慧
下列範例示範如何使用 Bing 和 Microsoft 認知服務 API，對 Bot 新增人工智慧功能。

範例 | 說明
------------ | -------------
[LUIS](https://github.com/Microsoft/BotBuilder-Samples/tree/master/Node/intelligence-LUIS) | 會使用 LuisDialog 來與 LUIS.ai 應用程式整合的範例 Bot。
[影像標題](https://github.com/Microsoft/BotBuilder-Samples/tree/master/Node/intelligence-ImageCaption) \(英文\) | 會使用 Microsoft 認知服務視覺 API 來取得影像標題的範例 Bot。
[語音轉換文字](https://github.com/Microsoft/BotBuilder-Samples/tree/master/Node/intelligence-SpeechToText) \(英文\)  | 會使用 Bing 語音 API 從音訊取得文字的範例 Bot。
[類似產品](https://github.com/Microsoft/BotBuilder-Samples/tree/master/Node/intelligence-SimilarProducts) \(英文\) | 會使用 Bing 影像搜尋 API 來尋找具有類似外觀之產品的範例 Bot。 
[Zummer](https://github.com/Microsoft/BotBuilder-Samples/tree/master/Node/intelligence-Zummer) \(英文\) | 會使用 Bing 搜尋 API 來尋找維基百科文章的範例 Bot。

## <a name="reference-implementation"></a>參考實作
下列範例的設計目的是展示端對端案例。 如果您想要在 Bot 中實作更複雜的功能，這是一個非常好的程式碼片段來源。


範例 | 說明
------------ | -------------
[Contoso Flowers](https://github.com/Microsoft/BotBuilder-Samples/tree/master/Node/demo-ContosoFlowers) \(英文\) | 會使用 Bot Framework 許多功能的範例 Bot。
