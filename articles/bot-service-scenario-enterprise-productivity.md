---
title: 企業生產力 Bot 案例 | Microsoft Docs
description: 探索使用 Bot Framework 的企業生產力 Bot 案例。
author: BrianRandell
ms.author: v-brra
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: 1fe68144662be3de349d05ea861a230641ae1efb
ms.sourcegitcommit: b78fe3d8dd604c4f7233740658a229e85b8535dd
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/24/2018
ms.locfileid: "49996706"
---
# <a name="enterprise-productivity-bot-scenario"></a>企業生產力 Bot 案例

[!INCLUDE [pre-release-label](includes/pre-release-label-v3.md)]

企業 Bot 說明如何將 Bot 與您的 Office 365 行事曆和其他服務整合，來提升您的生產力。

企業生產力 Bot 的重點在於能快速地存取客戶資訊，而不需要開啟一大堆的視窗。 使用簡單的聊天命令，銷售人員便能查閱客戶資料，並透過圖形 API 和 Office 365 檢查下一個約會。 他們可以從中存取儲存在 Dynamics CRM 的客戶特定資訊，例如取得案例資訊，或是建立新的案例。

![企業 Bot 圖表](~/media/scenarios/bot-service-scenario-enterprise-bot.png)

以下是企業生產力 Bot 的邏輯流程：

1. 員工存取企業生產力 Bot。
2. Azure Active Directory 驗證該員工的身分。
3. 企業生產力 Bot 可透過 Azure Graph 查詢該員工的 Office 365 行事曆。
4. Bot 可利用從行事曆收集而來的資料，存取 Dynamics CRM 中的案例資訊。
5. 此資訊會傳回給員工，他可以在不離開 Bot 的情況下篩出資料。
6. Application Insights 會收集執行階段遙測，以協助開發 Bot 效能和使用量。

您可以從[常見的 Bot Framework 案例範例](https://aka.ms/bot/scenarios) \(英文\) 下載或複製此範例 Bot 的原始程式碼。

## <a name="sample-bot"></a>範例 Bot
因為 Bot 可透過各種通道來存取，只需要經過驗證，您就可以在辦公室時透過公司入口網站來存取，或在外出時透過 Skype 來存取。 透過與 Azure AD 整合，您的企業生產力 Bot 就會知道您是否已經過 Azure AD 驗證，進而得知您是否能夠存取它。 接著，您就可以要求 Bot，以檢查與下一個特定客戶的約會時間。 Bot 是藉由透過圖形 API 查詢 Office 365 來取得此資訊。 然後，如果接下來一星期有約會，Bot 會查詢 CRM，尋找該客戶任何近期的案例資訊。 Bot 會回應找不到任何案例，或是數個已開立和已結案的案例。 您可以在該處要求 Bot 依類型列出案例，並向下鑽研個別案例。

## <a name="components-youll-use"></a>您將會使用的元件
企業生產力 Bot 使用下列元件：
-   用於驗證的 Azure AD
-   用於 Office 365 的圖形 API
-   Dynamics CRM
-   Application Insights

### <a name="azure-active-directory-azure-ad"></a>Azure Active Directory (Azure AD)
Azure Active Directory (Azure AD) 是 Microsoft 的多租用戶雲端型目錄和身分識別管理服務。 身為 Bot 開發人員，Azure AD 可讓您專注於建置 Bot，快速而簡單地整合數百萬個全球各地組織所使用的世界級身分識別管理解決方案。 透過定義 Azure AD 應用程式，您可以控制哪些人可存取您的 Bot 以及其所公開的資料，而不需要自己實作複雜的驗證和授權系統。

### <a name="graph-api-to-office-365"></a>用於 Office 365 的圖形 API
Microsoft Graph 透過位於 https://graph.microsoft.com 的單一端點，公開多個 Office 365 的 API 和其他 Microsoft 雲端服務。 Microsoft Graph 可讓您和 Bot 更輕鬆地執行查詢。 API 公開多個 Microsoft 雲端服務的資料，包括屬於 Office 365 一部分的 Exchange Online、Azure Active Directory、SharePoint 等等。 您可以使用 API 在實體和關聯性之間瀏覽。 您可以使用 SDK 或 REST 端點，以及其他原生支援 Android、iOS、Ruby、UWP、Xamarin 等的應用程式，從 Bot 使用 API。

### <a name="dynamics-crm"></a>Dynamics CRM
Dynamics CRM 是客戶業務開發平台。 使用 Bot 和 CRM API，您可以建置豐富的互動式 Bot，並存取儲存在 CRM 中的大量資料。 Dynamics CRM 的強大功能，可供您的 Bot 用來建立案例、檢查狀態、知識管理搜尋等等。

### <a name="application-insights"></a>Application Insights
Application Insights 可協助您透過應用程式效能管理和立即分析，取得可操作的見解。 根據預設，您可以取得豐富的效能監視、強大的警示與便於取用的儀表板，以協助確保您的 Bot 可供使用，並如預期般執行。 您可以快速查看是否遇到問題，然後執行根本原因分析以找出問題，並加以修正。

## <a name="next-steps"></a>後續步驟
接下來，深入了解 Bot 案例。

> [!div class="nextstepaction"]
> [資訊 Bot 案例](bot-service-scenario-informational.md)
