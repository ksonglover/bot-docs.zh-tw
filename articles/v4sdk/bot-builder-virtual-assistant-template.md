---
title: 虛擬助理範本的概觀 | Microsoft Docs
description: 深入了解虛擬助理範本
author: darrenj
ms.author: darrenj
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 05/23/2019
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 82385510894213a386c3f38836c85aad44306a23
ms.sourcegitcommit: eacf1522d648338eebefe2cc5686c1f7866ec6a2
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/30/2019
ms.locfileid: "70167059"
---
# <a name="virtual-assistant---template-outline"></a>虛擬助理 - 範本大綱

> [!NOTE]
> 本主題適用於 SDK 的 v4 版本。 

虛擬助理範本結合了數個我們在建置交談式體驗時發現的最佳做法，並自動整合對 Bot Framework 開發人員非常有幫助的元件。 本節涵蓋了一些重要決策的背景知識，以協助說明為什麼範本會如此運作。

功能      | 說明 |
------------ | -------------
簡介 | 在開始對話時，採用[調適型卡片]()的簡介訊息
輸入指標  | 在對話期間自動化視覺輸入指標，並針對長時間執行的作業重複此動作
基本 LUIS 模型  | 支援常見的意圖，例如**取消**、**說明**、**呈報**等等
基本對話方塊 | 擷取基本使用者資訊，以及取消和說明意圖等中斷邏輯的對話方塊流程
基本回應  | 基本意圖和對話方塊的文字及語音回應
常見問題集 | 與 [QnA Maker](https://www.qnamaker.ai) 整合以回答知識庫的一般問題 
閒聊 | 專業人員閒聊模型，以提供常用查詢的標準解答 ([了解更多](https://docs.microsoft.com/azure/cognitive-services/qnamaker/how-to/chit-chat-knowledge-base))
發送器 | 整合式[分派](https://docs.microsoft.com/azure/bot-service/bot-builder-tutorial-dispatch?view=azure-bot-service-4.0&tabs=csaddref%2Ccsbotconfig)模型，用來識別指定的語句應該由 LUIS 或 QnA Maker 處理。
語言支援 | 提供英文、法文、義大利文、德文、西班牙文和中文
文字記錄 | Azure 儲存體中儲存的所有對話的文字記錄
遙測  | [Application Insights](https://azure.microsoft.com/services/application-insights/) 整合，以收集所有對話的遙測資料
分析 | Power BI 儀表板範例，可讓您深入了解對話式體驗。
自動化部署 | 使用 Azure ARM 範本輕鬆部署所有上述服務。

## <a name="introduction-card"></a>簡介卡片

許多對話式體驗的主要問題是使用者不知道如何開始，這會導致 Bot 無法有效地回答許多常見問題。 第一印象非常重要！ 簡介卡片提供了向終端使用者介紹 Bot 功能的機會，並且提供一些建議的初始問題給使用者，以作為使用此功能的起點。 這也是讓 Bot 展現特質的絕佳機會。

系統會提供簡單的簡介卡片，作為您可視需要適應的標準，而當使用者完成上線對話 (由簡介卡片上的 [開始使用] 按鈕所觸發) 時，後續互動時會顯示傳回的使用者卡片。

![簡介卡片範例](./media/enterprise-template/vabotintrocard.png)

## <a name="basic-language-understanding-luis-intents"></a>基本 Language Understanding (LUIS) 意圖

每個 Bot 都應該能處理基礎等級的對話式語言理解。 例如，問候語就是每個 Bot 應能輕鬆處理的基本項目。 一般而言，開發人員需要建立這些基本意圖，並提供用於入門的初始訓練資料。 虛擬助理範本提供範例 .lu 檔案來協助您入門，同時避免在每次使用專案時都必須建立這些檔案，並確保有現成的基礎功能可用。

.lu 檔案會以英文、中文、法文、義大利文、德文、西班牙文提供下列意圖。

Intent       | 範例語句 |
-------------|-------------|
取消       |取消  、沒關係 |
呈報     |我可以洽詢人員嗎？ |
FinishTask   |完成  、全部完成 |
GoBack       |返回 |
說明         |可以請您提供協助嗎？ |
Repeat       |可以再說一次嗎？ |
SelectAny    |任一 |
SelectItem   |第一個 |
SelectNone   |以上皆非 |
ShowNext     |顯示較多 |
ShowPrevious |顯示上一個 |
StartOver    |*restart*|
Stop         |*stop*|

[.lu](https://github.com/Microsoft/botbuilder-tools/blob/master/packages/Ludown/docs/lu-file-format.md) 格式類似於 Markdown，可讓您輕鬆進行修改與原始檔控制。 [LuDown](https://github.com/Microsoft/botbuilder-tools/tree/master/packages/Ludown) 工具則可用來將 .lu 檔案轉換成 LUIS 模型，然後透過入口網站或相關聯的 [LUIS](https://github.com/Microsoft/botbuilder-tools/tree/master/packages/LUIS) CLI (命令列) 工具將其發行至您的 LUIS 訂用帳戶。

## <a name="telemetry"></a>遙測

經過證實，提供您 Bot 的業務開發見解相當有價值。 此見解可協助您了解業務開發的層級、使用者所使用的 Bot 功能 (意圖)，以及人們問哪些問題時 Bot 無法回答 - 將 Bot 的知識缺口標示出來，這可能可以透過 QnA Maker 之類的文件來解決。

整合 Application Insights 可提供現成的重要作業/技術見解，但這也可用來擷取特定的 Bot 相關事件 - 傳送和接收的訊息及 LUIS 和 QnA Maker 作業。

Bot 層級的遙測在本質上與技術和作業方面的遙測相連結，因此可讓您檢查 Bot 如何回答指定使用者的問題，反之亦然。

中介軟體元件結合 QnA Maker 和 LuisRecognizer SDK 類別的包裝函式類別，以提供簡潔的方式來收集一組一致的事件。 接著，這些一致的事件可交由 Application Insights 工具和 PowerBI 之類的工具來使用。

範例 Power BI 儀表板是 Bot Framework 解決方案 github 存放庫的一部分，可立即與每個虛擬助理範本搭配運作方式。 如需詳細資訊，請參閱[分析](https://aka.ms/bfs-analytics)一節。

![分析範例](./media/enterprise-template/powerbi-conversationanalytics-luisintents.png)

## <a name="dispatcher"></a>發送器

在第一波交談方式體驗中，產生良好效果的主要設計模式就是利用 Language Understanding (LUIS) 和 QnA Maker。 LUIS 可透過 Bot 為終端使用者執行的工作來訓練，而 QnA Maker 可透過較廣泛的知識來訓練。

所有傳入的語句 (問題) 都會路由至 LUIS 以進行分析。 如果無法識別指定語句的意圖，則會將其標示為*無*意圖。 接著會使用 QnA Maker 來嘗試為終端使用者尋找回答。

儘管此模式運作良好，還是有可能在兩個主要案例中遇到問題。

- 如果 LUIS 模型和 QnA Maker 中的語句在部分時間上稍微重疊，這可能會導致異常行為，其中 LUIS 可能會在問題應已導向 Qna Maker 時，嘗試處理該問題。
- 如果有兩個或兩個以上的 LUIS 模型，Bot 必須叫用每一個模型，並執行某種形式的意圖評估比較，以識別傳送指定語句的目的地。 因為模型之間沒有通用的基準分數比較，無法有效運作會導致不良的使用者體驗。

[發送器](https://docs.microsoft.com/azure/bot-service/bot-builder-tutorial-dispatch?view=azure-bot-service-4.0&tabs=csaddref%2Ccsbotconfig)為此提供了簡潔的解決方案，方法是從每個設定的 LUIS 模型擷取語句，以及從 QnA Maker 擷取問題，然後建立中央分派 LUIS 模型。

這可讓 Bot 快速找出應處理指定語句的 LUIS 模型或元件，並確保系統會認為 QnA Maker 資料在意圖處理的最上層，而不是跟之前一樣視為*無*意圖。

此分派工具也會啟用評估，並在部署之前，醒目提示 LUIS 模型之間的混淆、問題和重疊及 QnA Maker 知識庫問題。

發送器會使用於每個專案 (使用範本所建立) 的核心。 分派模型會在 `MainDialog` 類別內使用，可識別目標是 LUIS 模型或 QnA。 在 LUIS 的案例中，次要 LUIS 模型會受到叫用，並傳回意圖和實體。 發送器也用於中斷偵測。

![分派範例](./media/enterprise-template/dispatchexample.png)

## <a name="qna-maker"></a>QnA Maker

[QnA Maker](https://www.qnamaker.ai/) 可讓非開發人員以問題和答案配對的格式來策展通用知識。 此知識可以從常見問題集資料來源和產品手冊中匯入，也可在 QnA Maker 入口網站內以互動方式匯入。

在 CognitiveModels 的 QnA 資料夾內會以 [.lu](https://github.com/Microsoft/botbuilder-tools/blob/master/packages/Ludown/docs/lu-file-format.md) 檔案格式提供兩個範例 QnA Maker 模型，一個用於常見問題集，一個用於閒聊。 [LUDown](https://github.com/Microsoft/botbuilder-tools/tree/master/packages/Ludown) 接著會用來當作部署指令碼的一部分，以建立 QnA Maker JSON 檔案，然後 [QnA Maker](https://github.com/Microsoft/botbuilder-tools/tree/master/packages/QnAMaker) CLI (命令列) 工具會使用此檔案來將項目發行至 QnA Maker 知識庫。

![QnA ChitChat 範例](./media/enterprise-template/qnachitchatexample.png)

## <a name="content-moderator"></a>內容仲裁

內容審查工具為選用元件，可偵測潛在的不雅內容，並協助檢查個人識別資訊 (PII)。 比方說，Bot 可能會在發生粗話時道歉並移交給人類，或在偵測到 PII 資訊時，不儲存遙測記錄。

系統會提供中介軟體元件，以透過 TurnState 物件上的 ```TextModeratorResult``` 將該畫面轉換成文字並顯示。

## <a name="next-steps"></a>後續步驟
請參閱[教學課程](https://aka.ms/bfs-tutorials)以了解如何建立和部署虛擬助理。 

## <a name="additional-resources"></a>其他資源
虛擬助理範本的完整原始程式碼位於 [GitHub](https://aka.ms/bf-solutions)。

