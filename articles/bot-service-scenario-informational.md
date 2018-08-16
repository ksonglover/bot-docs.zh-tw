---
title: 資訊 Bot 案例 | Microsoft Docs
description: 使用 Bot Framework 探索資訊 Bot 案例。
author: BrianRandell
ms.author: v-brra
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: 43eb8e25e2a17e1d6b1d30e767dd15569fcad78b
ms.sourcegitcommit: 67445b42796d90661afc643c6bb6533e9a662cbc
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/06/2018
ms.locfileid: "39574864"
---
# <a name="information-bot-scenario"></a>資訊 Bot 案例

[!INCLUDE [pre-release-label](includes/pre-release-label-v3.md)]

此資訊 Bot 可透過認知服務 QnA Maker 回答在知識集或常見問題集中定義的問題，也會透過 Azure 搜尋服務回答更開放式的問題。

通常資訊會藏在 SQL Server 等結構化的資料存放區，不過透過搜尋即可輕鬆找到。 想像一下如此方便的情境：使用簡單的交談式命令查閱客戶的訂單狀態。 使用者使用認知服務 QnA Maker 後，會看到一組有效的搜尋選項，例如查閱客戶、檢閱客戶的最新訂單等。使用者可以利用 QnA 格式定義，輕鬆詢問 Azure 搜尋服務所支援的問題，並在 SQL Database 中查閱儲存的資料。

![資訊 Bot 圖表](~/media/scenarios/bot-service-scenario-informational-bot.png)

以下是資訊 Bot 的邏輯流程：

1. 員工啟動資訊 Bot。
2. Azure Active Directory 驗證該員工的身分。
3. 員工可詢問機器人支援何種查詢。
4. 認知服務傳回使用 QnA Maker 建置的常見問題集 Bot。
5. 員工定義有效的查詢。
6. Bot 將查詢提交至 Azure 搜尋服務，搜尋服務再傳回應用程式資料的相關資訊。
7. Application Insights 會收集執行階段遙測，以協助開發 Bot 效能和使用量。

## <a name="sample-bot"></a>範例 Bot
範例 Bot 以 C# 撰寫，在 Microsoft Azure 中執行，並搭配 Azure 搜尋服務從 SQL Database 執行個體索引的資料使用。 Bot 會公開一份問題清單，以詢問如何使用認知服務將問題 (答案) 分段：QnA Maker。 Bot 的使用者可以輸入查詢，以透過 Azure 搜尋服務，在索引資料庫中的廣泛或特定區域中查閱資料。 此範例提供簡單的資料庫與客戶和訂單資訊。 Application Insights 可追蹤 Bot 的使用方式，並協助您監視例外狀況的 Bot。 Bot 發行為 Azure AD 應用程式，讓您可以限制存取資訊的人選。

您可以從[常見的 Bot Framework 案例範例](https://aka.ms/bot/scenarios)下載或複製此範例 Bot 的原始程式碼。

## <a name="components-youll-use"></a>您將會使用的元件
資訊 Bot 會使用下列元件：
-   用於驗證的 Azure AD
-   認知服務：QnA Maker
-   Azure 搜尋服務
-   Application Insights

### <a name="azure-active-directory-azure-ad"></a>Azure Active Directory (Azure AD)
Azure Active Directory (Azure AD) 是 Microsoft 的多租用戶雲端型目錄和身分識別管理服務。 身為 Bot 開發人員，Azure AD 可讓您專注於建置 Bot，快速而簡單地整合數百萬個全球各地組織所使用的世界級身分識別管理解決方案。 您可以藉由定義 Azure AD 應用程式，來控制哪些人可存取您的 Bot，以及其所公開的資料，而不需要自己實作複雜的驗證和授權系統。

### <a name="cognitive-services-qna-maker"></a>認知服務：QnA Maker
認知服務 QnA Maker 可協助您提供常見問題集的資料來源，讓使用者可以從 Bot 進行查詢。 遇到儲存在不同系統中的大量資訊時，此服務可以有效協助使用者篩選資訊來源和資訊集。 單一 SQL 資料庫擁有大量資訊，以至於執行自由格式搜尋時會提供過多資訊。 初次使用 QnA Maker 時，您可以為 Bot 使用者定義藍圖，讓使用者了解如何問對問題，以透過 Azure 搜尋服務找到答案。

### <a name="azure-search"></a>Azure 搜尋服務
Azure 搜尋服務是應用程式專用的雲端搜尋服務，可讓您快速啟用搜尋索引並執行。 在 Microsoft Azure 上執行，您可以視需求輕鬆增加或減少使用量。 您可以透過對搜尋排名的極佳控制，並找出隱藏在資料庫中的資料，讓搜尋結果促成商業目標。

### <a name="application-insights"></a>Application Insights
Application Insights 可協助您透過應用程式效能管理和立即分析，取得可操作的見解。 根據預設，您可以取得豐富的效能監視、強大的警示與便於取用的儀表板，以協助確保您的 Bot 可供使用，並如預期般執行。 您可以快速查看是否遇到問題，然後執行根本原因分析以找出問題，並加以修正。

## <a name="next-steps"></a>後續步驟
接下來，請深入了解物聯網 Bot 案例。

> [!div class="nextstepaction"]
> [物聯網 Bot 案例](bot-service-scenario-internet-things.md)
