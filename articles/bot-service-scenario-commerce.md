---
title: 商務 Bot 案例 | Microsoft Docs
description: 使用 Bot Framework 來探索商務 Bot 案例。
author: BrianRandell
ms.author: v-brra
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: 15745bc25013df2fd18b0a2045ae2314d6c361e2
ms.sourcegitcommit: 67445b42796d90661afc643c6bb6533e9a662cbc
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/06/2018
ms.locfileid: "39574854"
---
# <a name="commerce-bot-scenario"></a>商務 Bot 案例

[!INCLUDE [pre-release-label](includes/pre-release-label-v3.md)]

[商務 Bot](bot-service-scenario-commerce.md) 案例說明 Bot 如何取代傳統電子郵件和電話互動等常見的旅館接待服務。 Bot 會利用認知服務，以更順暢地處理客戶的文字和語音要求，並搭配蒐集自與後端服務整合的內容。

![應用程式 Bot 圖表](~/media/scenarios/bot-service-scenario-commerce-bot.png)

以下是作為旅館接待的商務 Bot 邏輯流程：

1. 客戶可使用旅館的行動應用程式。
2. 使用者透過 Azure AD B2C 進行驗證。
3. 使用者透過自訂應用程式 Bot 來要求資訊。 
4. 認知服務可協助處理自然語言要求。
5. 回應會由能利用自然對話修正問題的客戶進行檢閱。
6. 若使用者滿意該結果，應用程式 Bot 就會更新客戶的預約。
7. Application Insights 會收集執行階段遙測，以協助開發 Bot 效能和使用量。

## <a name="sample-bot"></a>範例 Bot
範例商務 Bot 的設計訴求是打造虛構的旅館接待服務。 在透過鏈結的成員服務行動應用程式向旅館驗證 Azure AD B2C 後，客戶可存取以 C# 撰寫的 Bot。 鏈結會將預約儲存在 SQL Database 中。 客戶可以使用自然的問句，如「住宿期間租一間泳池小屋的費用多少」。 Bot 因此會擁有來賓住在哪間旅館和住宿期間的相關內容。 此外，Language Understanding (LUIS) 服務可讓 Bot 從簡單的詞語如「泳池小屋」等輕鬆取得內容。 Bot 會提供答案，然後為來賓預訂小屋，並提供天數和小屋類型的選擇。 在 Bot 擁有所有必要資料後，即會依要求進行預訂。 來賓也可以使用語音進行相同的要求。

您可以從[常見的 Bot Framework 案例範例](https://aka.ms/bot/scenarios)下載或複製此範例 Bot 的來源程式碼。

## <a name="components-youll-use"></a>您將使用的元件
商務 Bot 會使用下列元件：
-   用以驗證的 Azure AD
-   認知服務：LUIS
-   Application Insights

### <a name="azure-active-directory-azure-ad"></a>Azure Active Directory (Azure AD)
Azure Active Directory (Azure AD) 是 Microsoft 的多租用戶雲端型目錄和身分識別管理服務。 身為 Bot 開發人員，Azure AD 可讓您專注於建置 Bot，快速而簡單地整合數百萬個全球各地組織所使用的世界級身分識別管理解決方案。 Azure AD 支援 B2C 連接器，可讓您使用外部識別碼 (例如 Google、Facebook 或 Microsoft 帳戶) 來識別個人。 Azure AD 讓您不必管理使用者的認證，改而專注於 Bot 的解決方案，了解您可以將 Bot 使用者與應用程式所公開的正確資料相互關聯。

### <a name="cognitive-services-luis"></a>認知服務：LUIS
作為認知服務系列技術之一員，Language Understanding (LUIS) 會為您的應用程式帶來機器學習的強大功能。 目前，LUIS 支援數種語言，可讓 Bot 了解人想要什麼。 與 LUIS 整合時，您可表達意圖並定義 Bot 能了解的實體。 然後使用範例語句進行訓練，教導 Bot 了解這些意圖和實體。 您可以使用片語清單和 RegEx 功能調校整合，針對特定的對話需求讓 Bot 盡可能順暢。

### <a name="application-insights"></a>Application Insights
Application Insights 可協助您透過應用程式效能管理和立即分析，取得可操作的見解。 根據預設，您可以取得豐富的效能監視、強大的警示與便於取用的儀表板，以協助確保您的 Bot 可供使用，並如預期般執行。 您可以快速查看是否遇到問題，然後執行根本原因分析以找出問題，並加以修正。

## <a name="next-steps"></a>後續步驟
接下來，深入了解 Cortana 技能 Bot 案例。

> [!div class="nextstepaction"]
> [Cortana 技能 Bot 案例](bot-service-scenario-cortana-skill.md)
