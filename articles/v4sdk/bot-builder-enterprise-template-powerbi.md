---
title: 使用 PowerBI 的對話式分析 | Microsoft Docs
description: 深入了解企業 Bot 範本如何利用 Application Insights 以透過 PowerBI 啟用見解
author: darrenj
ms.author: darrenj
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 09/18/2018
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 18dcaeaf2967a90ca3ecb8ff32c1ef6399d5f498
ms.sourcegitcommit: 3bf3dbb1a440b3d83e58499c6a2ac116fe04b2f6
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/23/2018
ms.locfileid: "46708526"
---
# <a name="enterprise-bot-template---conversational-analytics-using-powerbi-dashboard-and-application-insights"></a>企業 Bot 範本 - 使用 PowerBI 儀表板和 Application Insights 的對話式分析

> [!NOTE]
> 本主題適用於 SDK 的 v4 版本。 

部署您的 Bot 並開始處理訊息之後，您會看到遙測流入您資源群組內的 Application Insights 執行個體。 

在 Azure 入口網站的 [Application Insights] 刀鋒視窗中以及使用 Log Analytics，都可以檢視此遙測。 此外，PowerBI 可以使用相同的遙測來提供更多關於 Bot 使用量的一般商務見解。

在已建立專案的 PowerBI 資料夾內會提供一個範例 PowerBI 儀表板。 這是針對範例用途所提供的，可示範如何開始建立自己的見解。 經過一段時間，我們會強化這些視覺效果。 

## <a name="getting-started"></a>開始使用

- 從[這裡](https://powerbi.microsoft.com/en-us/desktop/)下載 PowerBI Desktop
 
- 擷取 Bot 所用 Application Insights 資源的 ```Application Id```。 瀏覽至 Azure Application Insights 刀鋒視窗的 [設定] 區段下的 [API 存取] 頁面，即可取得此識別碼。

按兩下所提供的 PowerBI 範本檔案，其位於您的解決方案的 PowerBI 資料夾內。 系統會提示您輸入在上一個步驟中擷取的 ```Application Id```。 如果系統提示您使用 Azure 訂用帳戶認證來完成驗證，您可能需要按一下 [組織帳戶] 設定進行登入。

所產生的儀表板現在會連結到您的 Application Insights 執行個體，如果已傳送及接收訊息，您應會在儀表板內看到初始見解。

>請注意，情感視覺效果不會顯示資料，因為部署指令碼目前並未在發佈 LUIS 模型時啟用情感功能。 如果您[重新發行](https://docs.microsoft.com/en-us/azure/cognitive-services/luis/luis-how-to-publish-app) LUIS 模型並啟用情感功能，就能運作。

## <a name="middleware-processing"></a>中介軟體處理

系統會提供以 QnAMaker 和 LuisRecognizer 類別為主的遙測包裝函式，確保不論何種案例都有一致的遙測輸出，並且讓標準儀表板能夠作用於每個專案。

```TelemetryLuisRecognizer``` 和 ```TelemetryQnAMaker``` 兩者都提供建構函式的屬性，讓開發人員能夠停用使用者名稱和原始訊息的記錄功能。 不過，這會減少可用的見解量。

## <a name="telemetry-captured"></a>擷取的遙測

利用企業範本中預設啟用的 ```TelemetryLuisRecognizer``` 和 ```TelemetryQnAMaker```，可擷取 4 個不同的遙測事件。 

您的專案所用的每個 LUIS 意圖前面都會加上 LuisIntent， 以便儀表板輕鬆識別意圖。

```
-BotMessageReceived
    - ActivityId
    - Channel
    - FromId
    - Conversationid
    - ConversationName
    - Locale
    - UserName
    - Text
```
  
```
-BotMessageSent
    - ActivityId,
    - Channel
    - RecipientId
    - Conversationid
    - ConversationName
    - Locale
    - ReceipientName
    - Text
```

```
- LuisIntent.*
    - ActivityId
    - Intent
    - IntentScore
    - SentimentLabel
    - SentimentScore
    - ConversationId
    - Question
```

```
- QnAMaker
    - ActivityId
    - ConversationId
    - OriginalQuestion
    - UserName
    - QnAItemFound
    - Question
    - Answer
    - Score
```