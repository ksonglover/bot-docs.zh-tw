---
title: 適用於 .NET 的 Bot 建立器 SDK Bot 範例 | Microsoft Docs
description: 探索 Bot 範例，協助您以適用於 .NET 的 Bot 建立器 SDK 開始 Bot 開發。
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 05/03/2018
ms.openlocfilehash: 7e19ec2c3e523003c0831544e42eeb88765d8317
ms.sourcegitcommit: dcbc8ad992a3e242a11ebcdf0ee99714d919a877
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/30/2018
ms.locfileid: "39352857"
---
::: moniker range="azure-bot-service-3.0"

# <a name="bot-builder-sdk-for-net-samples"></a>適用於 .NET 的 Bot 建立器 SDK 範例

這些範例會示範以工作為主的 Bot，示範如何利用適用於 .NET 的 Bot 建立器 SDK 功能。 您可使用這些範例，協助您快速開始建置具備豐富功能的絕佳 Bot。

## <a name="get-the-samples"></a>取得範例
若要取得範例，請複製使用 Git 的 [BotBuilder-Samples](https://github.com/Microsoft/BotBuilder-Samples) GitHub 存放庫。

```cmd
git clone https://github.com/Microsoft/BotBuilder-Samples.git
cd BotBuilder-Samples
```

適用於 .NET 的 Bot 建立器 SDK 建置的 Bot 範例，會在 **CSharp** 目錄中進行組織。

您也可以檢視 GitHub 上的範例，並直接將其部署至 Azure。

## <a name="core"></a>核心
下列範例會說明用於建置豐富且有能力 Bot 的基本技術。

範例 | 說明
------------ | ------------- 
[傳送附件](https://github.com/Microsoft/BotBuilder-Samples/tree/master/CSharp/core-SendAttachment) | Bot 範例，會將簡單的媒體附件 (影像) 傳送給使用者。 
[接收附件](https://github.com/Microsoft/BotBuilder-Samples/tree/master/CSharp/core-ReceiveAttachment) | Bot 範例，會接收使用者所傳送的附件，並將其下載。 
[建立新對話](https://github.com/Microsoft/BotBuilder-Samples/tree/master/CSharp/core-CreateNewConversation)  | Bot 範例，會使用先前儲存的使用者地址開始新的對話。
[取得對話的成員](https://github.com/Microsoft/BotBuilder-Samples/tree/master/CSharp/core-GetConversationMembers) | Bot 範例，會擷取對話的成員清單，並於清單變更時加以偵測到。 
[Direct Line](https://github.com/Microsoft/BotBuilder-Samples/tree/master/CSharp/core-DirectLine) | Bot 範例和自訂用戶端，會使用 Direct Line API 彼此通訊。 
[Direct Line (WebSocket)](https://github.com/Microsoft/BotBuilder-Samples/tree/master/CSharp/core-DirectLineWebSockets) | Bot 範例和自訂用戶端，會使用 Direct Line API + WebSocket 彼此通訊。 
[多個對話](https://github.com/Microsoft/BotBuilder-Samples/tree/master/CSharp/core-MultiDialogs) | Bot 範例，會顯示不同種類的對話。
[狀態 API](https://github.com/Microsoft/BotBuilder-Samples/tree/master/CSharp/core-State) | 無狀態 Bot 範例，會追蹤對話的內容。
[自訂狀態 API](https://github.com/Microsoft/BotBuilder-Samples/tree/master/CSharp/core-CustomState) | 無狀態 Bot 範例，會使用自訂儲存體提供者追蹤對話的內容。
[ChannelData](https://github.com/Microsoft/BotBuilder-Samples/tree/master/CSharp/core-ChannelData) | Bot 範例，會使用 ChannelData 將原生中繼資料傳送至 Facebook。
[AppInsights](https://github.com/Microsoft/BotBuilder-Samples/tree/master/CSharp/core-AppInsights) | Bot 範例，會將遙測記錄到 Application Insights 執行個體。

## <a name="search"></a>Search
此範例會說明如何在資料驅動 Bot 中利用 Azure 搜尋服務。

範例 | 說明
------------ | -------------
[Azure 搜尋服務](https://github.com/Microsoft/BotBuilder-Samples/tree/master/CSharp/demo-Search) | 兩個 Bot 範例，會協助使用者瀏覽大量內容。


## <a name="cards"></a>卡片
下列範例說明如何在 Bot Framework 中傳送複合式資訊卡 (Rich Card)。

範例 | 說明
------------ | -------------
[複合式資訊卡 (Rich Card)](https://github.com/Microsoft/BotBuilder-Samples/tree/master/CSharp/cards-RichCards) | Bot 範例，會傳送許多不同類型的複合式資訊卡 (Rich Card)。
[卡片的浮動切換](https://github.com/Microsoft/BotBuilder-Samples/tree/master/CSharp/cards-CarouselCards) | Bot 範例會使用浮動切換版面配置，在單一訊息內傳送多個複合式資訊卡 (Rich Card)。

## <a name="intelligence"></a>智慧
下列範例會說明如何使用 Bing 和 Microsoft 認知服務 API，對 Bot 新增人工智慧功能。

範例 | 說明
------------ | -------------
[LUIS](https://github.com/Microsoft/BotBuilder-Samples/tree/master/CSharp/intelligence-LUIS) | Bot 範例，會使用 LuisDialog 來與 LUIS.ai 應用程式整合。
[影像標題](https://github.com/Microsoft/BotBuilder-Samples/tree/master/CSharp/intelligence-ImageCaption) | Bot 範例，會使用 Microsoft 認知服務視覺 API 取得影像標題。
[語音轉換文字](https://github.com/Microsoft/BotBuilder-Samples/tree/master/CSharp/intelligence-SpeechToText)  | Bot 範例，會使用 Bing 語音 API 從音訊取得文字。
[類似產品](https://github.com/Microsoft/BotBuilder-Samples/tree/master/CSharp/intelligence-SimilarProducts) | Bot 範例，會使用 Bing 影像搜尋 API 尋找看起來類似的產品。 
[Zummer](https://github.com/Microsoft/BotBuilder-Samples/tree/master/CSharp/intelligence-Zummer) | Bot 範例，會使用 Bing 搜尋 API 尋找維基百科文章。

## <a name="reference-implementation"></a>參考實作
下列範例的設計目的是要展示端對端案例。 如果您想要在 Bot 中實作更複雜的功能，這會是絕佳的程式碼片段來源。


範例 | 說明
------------ | -------------
[Contoso Flowers](https://github.com/Microsoft/BotBuilder-Samples/tree/master/CSharp/demo-ContosoFlowers) | Bot 範例，會使用 Bot Framework 的許多功能。

::: moniker-end

::: moniker range="azure-bot-service-4.0"
# <a name="bot-builder-sdk-v4-net-samples"></a>Bot 建立器 SDK v4 .NET 範例
[!INCLUDE [pre-release-label](../includes/pre-release-label.md)]

這些範例會示範以工作為主的 Bot，示範如何利用適用於 .NET 的 Bot 建立器 SDK 功能。 您可使用這些範例，協助您快速開始建置具備豐富功能的絕佳 Bot。 

注意：SDK v4 目前還在開發階段，因此應僅用於測試。 

若要取得範例，請複製使用 Git 的 [botbuilder-dotnet](https://github.com/Microsoft/botbuilder-dotnet) GitHub 存放庫。
```cmd
git clone https://github.com/Microsoft/botbuilder-dotnet.git
cd botbuilder-dotnet\samples-final
```
以適用於 .NET 的 Bot 建立器 SDK 建置的 Bot 範例，會在 **samples-final** 目錄中進行組織。


::: moniker-end

