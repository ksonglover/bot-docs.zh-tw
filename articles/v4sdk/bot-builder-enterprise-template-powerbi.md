---
title: 使用 PowerBI 的對話式分析 | Microsoft Docs
description: 深入了解企業 Bot 範本如何利用 Application Insights 以透過 PowerBI 啟用見解
author: darrenj
ms.author: darrenj
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 09/18/2018
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 88208a2f5b0eb88d3b2964e63a21585484166d73
ms.sourcegitcommit: 2d84d5d290359ac3cfb8c8f977164f799666f1ab
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 01/09/2019
ms.locfileid: "54152171"
---
# <a name="enterprise-bot-template---conversational-analytics-using-powerbi-dashboard-and-application-insights"></a>企業 Bot 範本 - 使用 PowerBI 儀表板和 Application Insights 的對話式分析

> [!NOTE]
> 本主題適用於 SDK 的 v4 版本。 

部署您的 Bot 並開始處理訊息之後，您會看到遙測流入您資源群組內的 Application Insights 執行個體。 

在 Azure 入口網站的 [Application Insights] 刀鋒視窗中以及使用 Log Analytics，都可以檢視此遙測。 此外，PowerBI 可以使用相同的遙測來提供更多關於 Bot 使用量的一般商務見解。

在[交談 AI 遙測](https://aka.ms/botPowerBiTemplate)中提供範例 PowerBI 儀表板。 

這是針對範例用途所提供的，可示範如何開始建立自己的見解。 經過一段時間，我們會強化這些視覺效果。 


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
    - FromName
    - ConversationId
    - ConversationName
    - Locale
    - Text
    - RecipientId
    - RecipientName
```
  
```
-BotMessageSent
    - ActivityId,
    - Channel
    - RecipientId
    - ConversationId
    - ConversationName
    - Locale
    - RecipientId
    - RecipientName
    - ReplyToId
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
    - DialogId
```

```
- QnAMaker
    - ActivityId
    - ConversationId
    - OriginalQuestion
    - FromName
    - ArticleFound
    - Question
    - Answer
    - Score
```