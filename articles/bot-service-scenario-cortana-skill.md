---
title: Cortana 技能 Bot 案例 | Microsoft Docs
description: 使用 Bot Framework 探索 Cortana 技能 Bot 案例。
author: BrianRandell
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: d6ba5b414ff7e600fac1e5d4ebce27363340f42f
ms.sourcegitcommit: eacf1522d648338eebefe2cc5686c1f7866ec6a2
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/30/2019
ms.locfileid: "70167048"
---
# <a name="cortana-skills-bot-scenario"></a>Cortana 技能 Bot 案例

[!INCLUDE [pre-release-label](includes/pre-release-label-v3.md)]

Cortana 技能 Bot 擴充了 Cortana，可以輕鬆地從您行事曆的內容，使用語音預訂行動裝置自動維護預約。

Cortana 是您的個人助理。 使用您的語音和自訂的 Cortana 技能 Bot 的自然介面，您可以讓 Cortana 與組織 (如汽車商店) 交談，以協助您預約。 該服務可以提供一份服務、可用時間和持續時間的清單。 Cortana 可以查看您的行事曆，以查看您是否在衝突的時間有某些內容，如果沒有，則建立約會並將它新增至您的行事曆中。

![Cortana 技能 Bot 圖表](~/media/scenarios/bot-service-scenario-cortana-skill.png)

以下是 Cortana 技能 Bot 針對汽車商店的邏輯流程：

1. 使用者從他們的電腦或行動裝置存取 Cortana。
2. 使用文字或語音命令，使用者要求進行汽車保養預約。
3. 由於 Bot 與 Cortana 整合，因此它可以存取使用者的行事曆，並將邏輯套用至要求。
4. 使用該資訊，Bot 可以查詢自動服務以取得有效的約會。
5. 使用者可以利用內容相關的選項，來預約約會。
6. Application Insights 會收集執行階段遙測，以協助開發 Bot 效能和使用量。

## <a name="sample-bot"></a>範例 Bot
使用 Cortana 技能 Bot 完全取決於個人背景。 使用 Cortana，您可以使用語音要求「Bob 的行動裝置維護」根據您的位置來處理您的車輛。 使用透過 Cortana 公開的個人資訊，您的 Bot 可以根據使用者與 Bot 交談時的位置來確認位置。

您可以從[常見的 Bot Framework 案例範例](https://aka.ms/abs-scenarios)下載或複製此範例 Bot 的原始程式碼。

## <a name="components-youll-use"></a>您將使用的元件
Cortana Bot 會使用下列元件：
-   Cortana
-   Application Insights

### <a name="cortana"></a>Cortana
現在，您可以透過建立 Cortana 技能，將支援新增至您的 Bot。 您可以使用 Cortana 技能集為 Cortana 建置新功能 (稱為技能)。 技能是一種允許 Cortana 做更多事情的建構。 您可以建置與 Bot 整合的技能，從而讓 Cortana 完成工作。 作為叫用程序的一部分，Cortana 可以 (在使用者同意之後) 在執行階段將使用者的相關資訊傳遞給技能，以便技能可以相應地自訂其體驗。 Cortana 的內容相關的知識可讓您的 Bot 變得實用，甚至可能更聰明。 一旦叫用，特定類型的技能可以操作 Cortana 的介面，以在技能和使用者之間進行交談。 發佈之後，使用者就可以在 Cortana for Windows 10 年度更新+ (電腦和行動裝置)、iOS 和 Android 上查看和使用您的技能。

### <a name="application-insights"></a>Application Insights
Application Insights 可協助您透過應用程式效能管理和立即分析，取得可操作的見解。 根據預設，您可以取得豐富的效能監視、強大的警示與便於取用的儀表板，以協助確保您的 Bot 可供使用，並如預期般執行。 您可以快速查看是否遇到問題，然後執行根本原因分析以找出問題，並加以修正。

## <a name="next-steps"></a>後續步驟
接下來，了解企業生產力 Bot 案例。

> [!div class="nextstepaction"]
> [企業生產力 Bot 案例](bot-service-scenario-enterprise-productivity.md)
