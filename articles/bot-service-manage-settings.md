---
title: 配置 Bot 設定 |Microsoft Docs
description: 了解如何使用 Azure 入口網站，為 Bot 配置各種選項。
keywords: 配置 bot 設定、顯示名稱、圖示、 Application Insights、設定刀鋒視窗
author: v-royhar
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 12/13/2017
ms.openlocfilehash: 7497726039823777afeb8a7d317212d859d66a36
ms.sourcegitcommit: a6d02ec4738e7fc90b7108934740e9077667f3c5
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/04/2019
ms.locfileid: "70297557"
---
# <a name="configure-bot-settings"></a>配置 bot 設定

Bot 設定 (例如顯示名稱、 圖示和 Application Insights) 可在 [**設定**] 刀鋒視窗中檢視及修改。

![Bot 設定刀鋒視窗](~/media/bot-service-portal-configure-settings/bot-settings-blade.png)

以下是 [**設定**] 刀鋒視窗中的欄位清單：

| 欄位 | 說明 |
| :---  | :---        |
| 圖示 | 自訂圖示，以便透過視覺化方式在頻道中識別您的 Bot，並做為 Skype、 Cortana 和其他服務的圖示。 此圖示必須採用 PNG 格式，並且不得大於 30K。 此值隨時可以變更。 |
| 顯示名稱 | 您的 Bot 在頻道與目錄中的名稱。 此值隨時可以變更。 35 個字元限制。 |
| Bot 控制代碼 | Bot 的唯一識別碼。 藉由 Bot 服務建立 Bot 之後就無法變更此值。 |
| 傳訊端點 | 要與您的 Bot 進行通訊的端點。 |
| Microsoft 應用程式識別碼 | Bot 的唯一識別碼。 無法變更此值。 您可以按一下 [**管理**] 連結以產生新的密碼。 |
| Application Insights 檢測金鑰 | Bot 遙測的唯一索引鍵。 如果您想要接收此 Bot 的 Bot 遙測資料，請將您的 Azure Application Insights 金鑰複製到此欄位中。 此為選用值。 在 Azure 入口網站中建立的 Bot 會自行產生此金鑰。 如需此欄位的更多詳細資料，請參閱 [Application Insights 金鑰](~/bot-service-resources-app-insights-keys.md)。 |
| Application Insights API | Bot 分析的唯一金鑰。 如果您想要在儀表板中檢視 Bot 相關分析，請將您的 Azure Application Insights API 金鑰複製到此欄位中。 此為選用值。 如需此欄位的更多詳細資料，請參閱 [Application Insights 金鑰](~/bot-service-resources-app-insights-keys.md)。 |
| Application Insights 應用程式識別碼 | Bot 分析的唯一金鑰。 如果您想要在儀表板中檢視 bot 相關分析，請將您的 Azure Insights 應用程式識別碼金鑰複製到此欄位中。 此為選用值。 在 Azure 入口網站中建立的 Bot 會自行產生此金鑰。 如需此欄位的更多詳細資料，請參閱 [Application Insights 金鑰](~/bot-service-resources-app-insights-keys.md)。 |

> [!NOTE]
> 在變更自己的 Bot 的設定後，請按一下刀鋒視窗頂端的 [**儲存**] 按鈕以儲存新的 Bot 設定。

## <a name="next-steps"></a>後續步驟
現在，您已了解如何配置自己的 Bot 服務的設定值，請深入了解如何配置語音預備。
> [!div class="nextstepaction"]
> [語音預備](bot-service-manage-speech-priming.md)