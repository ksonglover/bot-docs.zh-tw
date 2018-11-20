---
title: 企業 Bot 範本的概觀 | Microsoft Docs
description: 深入了解企業 Bot 範本的功能
author: darrenj
ms.author: darrenj
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 09/18/2018
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: f5aacc693fb2e8987d6b59db67d0272423a4cf44
ms.sourcegitcommit: 8b7bdbcbb01054f6aeb80d4a65b29177b30e1c20
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 11/14/2018
ms.locfileid: "51645551"
---
# <a name="enterprise-bot-template"></a>企業 Bot 範本 

> [!NOTE]
> 本主題適用於 SDK 的 v4 版本。 

建立高品質對話式體驗時，需要一組基本功能。 為了協助您成功打造絕佳的對話式體驗，我們建立了企業 Bot 範本。 此範本結合了我們透過打造對話式體驗所找出的所有最佳做法與支援元件。 

此範本大幅簡化新 Bot 專案的建立。 此範本利用 [Bot Builder SDK v4](https://github.com/Microsoft/botbuilder) 和 [Bot Builder Tools](https://github.com/Microsoft/botbuilder-tools)，提供下列立即可用的功能。

功能 | 說明 |
------------ | -------------
簡介訊息 | 在開始對話時，採用調適型卡片的簡介訊息。 其會說明 Bot 功能，並會提供按鈕來引導初始的問題。 開發人員可以接著視需要加以自訂。
自動化輸入指標  | 在對話期間傳送視覺輸入指標，並針對長時間執行的作業重複此動作。
.bot 檔案驅動的組態 | Bot 的所有組態資訊 (例如 LUIS、發送器端點、Application Insights) 會包裝在 .bot 檔案內，而且用於促使 Bot 啟動。
基本對話式意圖  | 以英文、法文、義大利文、德文、西班牙文提供的基本意圖 (問候語、道別、協助、取消等等)。 這些意圖會以 .LU (語言理解) 檔案提供，以便輕鬆修改。
基本對話式回應  | 對基本對話式意圖的回應會提取到個別的檢視類別中。 這些回應未來會移至新的語言生成 (LG) 檔案。
不適內容或 PII (個人識別資訊) 偵測  |透過在中介軟體元件中使用[內容審查工具](https://azure.microsoft.com/en-us/services/cognitive-services/content-moderator/)，偵測傳入交談中的不當或 PII 資料。
文字記錄  | Azure 儲存體中儲存的所有對話的文字記錄
發送器 | 整合式[分派](https://docs.microsoft.com/en-us/azure/bot-service/bot-builder-tutorial-dispatch?view=azure-bot-service-4.0&tabs=csaddref%2Ccsbotconfig)模型，用來識別指定的語句應該由 LUIS 與 Code 處理或傳遞至 QnAMaker。
QnAMAker 整合  | 與 [QnAMaker](https://www.qnamaker.ai) 整合，可以運用現有的資料來源 (例如 PDF 手冊) 來回答知識庫中的一般問題。
對話式見解  | 與 [Application Insights](https://azure.microsoft.com/en-gb/services/application-insights/) 整合，可收集所有對話的遙測，而範例 PowerBI 儀表板可讓您開始深入了解您的對話式體驗。

此外，會自動部署 Bot 所需的所有 Azure 資源：Bot 註冊、Azure App Service、LUIS、QnAMaker、內容審查工具、CosmosDB、Azure 儲存體和 Application Insights。 此外，會建立、定型及發行基底 LUIS、QnAMaker 和分派模型，以便立即測試基本意圖和路由。

建立範本並執行部署步驟之後，您可以按 F5 來進行端對端測試。 這可提供開始對話式體驗的穩固基礎，減少每個專案必須承擔的許多天投入時間，並提高交談式品質標準。

若要開始使用，請繼續[建立專案](bot-builder-enterprise-template-create-project.md)。 若要深入了解已推動上述功能的最佳做法和知識，請參閱[範本詳細資料](bot-builder-enterprise-template-overview-detail.md)主題。 

在此 [GitHub 位置](https://github.com/Microsoft/AI/tree/master/templates/Enterprise-Template)可取得企業 Bot 範本的完整原始程式碼
