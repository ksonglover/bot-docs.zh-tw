---
title: 企業 Bot 範本的詳細介紹 | Microsoft Docs
description: 深入了解企業 Bot 範本背後的設計決策
author: darrenj
ms.author: darrenj
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 09/18/2018
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 5f5559ae16bc346d34e5cddfa741853044ec685f
ms.sourcegitcommit: b78fe3d8dd604c4f7233740658a229e85b8535dd
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/24/2018
ms.locfileid: "49997345"
---
# <a name="enterprise-template---detailed-overview"></a>企業範本 - 詳細介紹

> [!NOTE]
> 本主題適用於 SDK 的 v4 版本。 

企業 Bot 範本結合了數個我們在建置對話式體驗時發現的最佳做法，並自動整合對 Azure Bot Service 開發人員非常有幫助的元件。 本節涵蓋了一些重要決策的背景知識，以協助說明為什麼範本會如此運作。

## <a name="introduction-card"></a>簡介卡片

許多對話式體驗的主要問題是使用者不知道如何開始，這會導致 Bot 無法有效地回答許多常見問題。 第一印象非常重要！ 簡介卡片提供了向終端使用者介紹 Bot 功能的機會，並且提供一些建議的初始問題給使用者，以作為使用此功能的起點。 這也是讓 Bot 展現特質的絕佳機會。

我們會以標準格式提供簡單的簡介卡片，但您可以視需要加以調整。

## <a name="basic-language-understanding-luis-intents"></a>基本 Language Understanding (LUIS) 意圖

每個 Bot 都應該能處理基礎等級的對話式語言理解。 例如，問候語就是每個 Bot 應能輕鬆處理的基本項目。 一般而言，開發人員需要建立這些基本意圖，並提供用於入門的初始訓練資料。 企業 Bot 範本提供 LU 範例檔案來協助您入門，同時避免在每次使用專案時都必須建立這些檔案，並確保有現成的基礎功能可用。

LU 檔案會以英文、法文、義大利文、德文和西班牙文提供下列意圖。

> 問候語、說明、取消、重新啟動、向上增加、確認是、確認否、確認更多、下一個、道別

[LU](https://github.com/Microsoft/botbuilder-tools/blob/master/packages/Ludown/docs/lu-file-format.md) 格式類似於 MarkDown，可讓您輕鬆進行修改與原始檔控制。 [LuDown](https://github.com/Microsoft/botbuilder-tools/tree/master/packages/Ludown) 工具則可用來將 .LU 檔案轉換成 LUIS 模型，然後透過入口網站或相關聯的 [LUIS](https://github.com/Microsoft/botbuilder-tools/tree/master/packages/LUIS) CLI (命令列) 工具將其發行至您的 LUIS 訂用帳戶。

## <a name="content-moderator"></a>內容仲裁

內容審查工具可偵測潛在的不雅內容，並協助檢查個人識別資訊 (PII)。 將此功能整合到 Bot 會很有幫助，這可讓 Bot 針對不雅內容或當使用者共用 PII 資訊時，做出適當反應。 比方說，Bot 可能會道歉並移交給人類，或在偵測到 PII 資訊時，不儲存遙測記錄。

系統會提供中介軟體元件，以透過 TurnState 物件上的 ```TextModeratorResult``` 將該畫面轉換成文字並顯示。

## <a name="telemetry"></a>遙測

經過證實，提供您 Bot 的業務開發見解相當有價值。 此見解可協助您了解業務開發的層級、使用者所使用的 Bot 功能 (意圖)，以及人們問哪些問題時 Bot 無法回答 - 將 Bot 的知識缺口標示出來，這可能可以透過 QnAMaker 之類的文件來解決。

整合 Application insights 可提供現成的重要作業/技術見解，但這也可用來擷取特定的 Bot 相關事件 - 傳送和接收的訊息及 LUIS 和 QnAMaker 作業。

Bot 層級的遙測在本質上與技術和作業方面的遙測相連結，因此可讓您檢查 Bot 如何回答指定使用者的問題，反之亦然。

中介軟體元件結合 QnAMaker 和 LuisRecognizer SDK 類別的包裝函式類別，以提供簡潔的方式來收集一組一致的事件。 接著，這些一致的事件可交由 Applicatin Insights 工具和 PowerBI 之類的工具來使用。

使用企業 Bot 範本所建立的每個專案，都會提供 PowerBI 儀表板範例。 如需詳細資訊，請參閱 [PowerBI](bot-builder-enterprise-template-powerbi.md) 一節。

## <a name="dispatcher"></a>發送器

在第一波對話式體驗中產生良好效果的主要設計模式就是利用 Language Understanding (LUIS) 和 QnAMaker。 LUIS 可透過 Bot 為終端使用者執行的工作來訓練，而 QnAMaker 可透過較廣泛的知識來訓練。

所有傳入的語句 (問題) 都會路由至 LUIS 以進行分析。 如果無法識別指定語句的意圖，則會將其標示為「無」意圖。 接著會使用 QnAMaker 來嘗試為終端使用者尋找回答。

儘管此模式運作良好，還是有可能在兩個主要案例中遇到問題。

- 如果 LUIS 模型和 QnAMaker 中的語句在部分時間上稍微重疊，這可能會導致異常行為，其中 LUIS 可能會在問題應已導向 QnaMaker 時，嘗試處理該問題。
- 如果有兩個或兩個以上的 LUIS 模型，Bot 必須叫用每一個模型，並執行某種形式的意圖評估比較，以識別傳送指定語句的目的地。 因為模型之間沒有通用的基準分數比較，無法有效運作會導致不良的使用者體驗。

[發送器](https://docs.microsoft.com/en-us/azure/bot-service/bot-builder-tutorial-dispatch?view=azure-bot-service-4.0&tabs=csaddref%2Ccsbotconfig)為此提供了簡潔的解決方案，方法是從每個設定的 LUIS 模型擷取語句，以及從 QnAMaker 擷取問題，然後建立中央分派 LUIS 模型。

這可讓 Bot 快速找出應處理指定語句的 LUIS 模型或元件，並確保系統會認為 QnAMaker 資料在意圖處理的最上層，而不是跟之前一樣視為「無」意圖。

此分派工具也會啟用評估，並在部署之前，醒目提示 LUIS 模型之間的混淆和重疊及已標示的 QnAMaker 知識庫問題。

發送器會在每個專案 (使用企業 Bot 範本所建立) 的核心上使用。 分派模型會在 `MainDialog` 類別內使用，可識別目標是 LUIS 模型或 QnA。 在 LUIS 的案例中，次要 LUIS 模型會受到叫用，並如往常般傳回意圖和實體。

## <a name="qnamaker"></a>QnAMaker

[QnAMaker](https://www.qnamaker.ai/) 可讓非開發人員以問題和答案配對的格式來策畫通用知識。 此知識可以從常見問題集資料來源和產品手冊中匯入，也可在 QnaMaker 入口網站內以互動方式匯入。

CogSvcModels 的 QnA 資料夾中有以 [LU](https://github.com/Microsoft/botbuilder-tools/blob/master/packages/Ludown/docs/lu-file-format.md) 檔案格式提供的一組 QnA 實體範例。 [LuDown](https://github.com/Microsoft/botbuilder-tools/tree/master/packages/Ludown) 接著會用來當作部署指令碼的一部分，以建立 QnAMaker JSON 檔案，然後 [QnAMaker](https://github.com/Microsoft/botbuilder-tools/tree/master/packages/QnAMaker) CLI (命令列) 工具會使用此檔案來將項目發行至 QnAMaker 知識庫。
