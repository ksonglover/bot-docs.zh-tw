---
title: Enterprise Bot Template 的詳細介紹 | Microsoft Docs
description: 深入了解 Enterprise Bot Template 背後的設計決策
author: darrenj
ms.author: darrenj
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 09/18/2018
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 582ec21d36e15fcbaef2d26616a9e55a04d8e5f2
ms.sourcegitcommit: b2245df2f0a18c5a66a836ab24a573fd70c7d272
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 03/07/2019
ms.locfileid: "57568215"
---
# <a name="enterprise-bot-template---overview"></a>Enterprise Bot Template - 概觀

> [!NOTE]
> 本主題適用於 SDK 的 v4 版本。 

Enterprise Bot Template 為打造絕佳的對話式體驗、減少工作量和提高品質標準所需的最佳做法和服務，提供了穩固的基礎。 此範本利用 [Bot Builder SDK v4](https://github.com/Microsoft/botbuilder) 和 [Bot Builder 工具](https://github.com/Microsoft/botbuilder-tools)，提供下列功能：

功能      | 說明 |
------------ | -------------
簡介 | 在開始對話時，採用[調適型卡片]()的簡介訊息
輸入指標  | 在對話期間自動化視覺輸入指標，並針對長時間執行的作業重複此動作
基本 LUIS 模型  | 支援常見的意圖，例如**取消**、**說明**、**呈報**等等
基本對話方塊 | 擷取基本使用者資訊，以及取消和說明意圖等中斷邏輯的對話方塊流程
基本回應  | 基本意圖和對話方塊的文字及語音回應
常見問題集 | 與 [QnA Maker](https://www.qnamaker.ai) 整合以回答知識庫的一般問題 
閒聊 | 專業人員閒聊模型，以提供常用查詢的標準解答 ([了解更多](https://docs.microsoft.com/en-us/azure/cognitive-services/qnamaker/how-to/chit-chat-knowledge-base))
發送器 | 整合式[分派](https://docs.microsoft.com/en-us/azure/bot-service/bot-builder-tutorial-dispatch?view=azure-bot-service-4.0&tabs=csaddref%2Ccsbotconfig)模型，用來識別指定的語句應該由 LUIS 或 QnA Maker 處理。
語言支援 | 提供英文、法文、義大利文、德文、西班牙文和中文
文字記錄 | Azure 儲存體中儲存的所有對話的文字記錄
遙測  | [Application Insights](https://azure.microsoft.com/en-gb/services/application-insights/) 整合，以收集所有對話的遙測資料
分析 | Power BI 儀表板範例，可讓您深入了解對話式體驗。
自動化部署 | 使用 [Bot Builder 工具](https://github.com/Microsoft/botbuilder-tools)輕鬆部署所有上述服務

# <a name="background"></a>背景

## <a name="introduction-card"></a>簡介卡片
簡介卡片可以透過提供 Bot 功能的概觀，並提供範例語句簡化使用者入門方法，從而改善對話。 其也會建立使用者的 Bot 個人特質。

## <a name="base-luis-model-with-common-intents"></a>基本 LUIS 模型與一般意圖
範本中包含的基本 LUIS 模型，涵蓋了大部分 Bot 需要處理的最常見意圖選取範圍。 您可以在 Bot 中使用以下立即可用的意圖：

意圖       | 範例語句 |
-------------|-------------|
取消       |取消、沒關係|
呈報     |我可以洽詢人員嗎？|
FinishTask   |完成、全部完成|
GoBack       |返回|
說明         |可以請您提供協助嗎？|
Repeat       |可以再說一次嗎？|
SelectAny    |任一|
SelectItem   |第一個|
SelectNone   |以上皆非|
ShowNext     |顯示較多|
ShowPrevious |顯示上一個|
StartOver    |*restart*|
Stop         |*stop*|

## <a name="qna-maker"></a>QnA Maker

[QnA Maker](https://www.qnamaker.ai/) 為非開發人員提供產生問題和答案配對集合的功能，以便在 Bot 中使用。 此知識可以從常見問題集資料來源和產品手冊中匯入，或在 QnA Maker 入口網站內手動建立。

範本中提供了兩個 QnA Maker 模型範例，一個用於常見問題集，另一個用於[專業人員閒聊](https://docs.microsoft.com/en-us/azure/cognitive-services/qnamaker/how-to/chit-chat-knowledge-base)。 

## <a name="dispatch"></a>分派

分派服務是透過從每個服務中擷取語句並建立中央分派 LUIS 模型，來管理多個 LUIS 模型和 QnA Maker 知識庫之間的路由。

這可讓 Bot 快速找出應處理指定語句的元件，並確保系統會認為 QnA Maker 資料在意圖處理的最上層，而不是在階層的最下層。

這個分派工具也可以評估您的模型，反白顯示服務中重疊的語句和差異。

## <a name="telemetry"></a>遙測

對 Bot 對話提供深入了解，協助您了解使用者參與層級、使用者正在使用的功能以及 Bot 無法處理的問題。

[Application Insights](https://docs.microsoft.com/en-us/azure/azure-monitor/app/app-insights-overview) 擷取 Bot 服務的操作遙測，以及特定事件的對話。 可以使用 [Power BI](https://powerbi.microsoft.com/en-us/what-is-power-bi/) 等工具將這些事件彙總為可以操作的資訊。 Power BI 儀表板範例可用於示範這項功能的 Enterprise Bot Template。

# <a name="next-steps"></a>後續步驟
請參閱[使用者入門](bot-builder-enterprise-template-getting-started.md)以了解如何建立與部署 Enterprise Bot。 

# <a name="resources"></a>資源
Enterprise Bot Template 的完整原始程式碼位於 [GitHub](https://github.com/Microsoft/AI/tree/master/templates/Enterprise-Template)。
