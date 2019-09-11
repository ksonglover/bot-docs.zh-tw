---
title: Bot 分析 | Microsoft Docs
description: 了解如何使用資料收集和分析，利用 Bot Framework 中的分析來改善 Bot。
keywords: bot 分析, application insights, 流量, 延遲, 整合, AppInsights
author: JonathanFingold
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 12/04/2018
ms.openlocfilehash: 324050c625f5d9666811f63191d783643816104c
ms.sourcegitcommit: a6d02ec4738e7fc90b7108934740e9077667f3c5
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/04/2019
ms.locfileid: "70298695"
---
# <a name="bot-analytics"></a>Bot 分析

Analytics 是 [Application Insights](/azure/application-insights/app-insights-analytics) 的擴充功能。 Application Insights 提供**服務層級**和檢測資料，例如流量、延遲和整合。 Analytics 提供關於使用者、訊息和通道資料的**對話層級**報告。

## <a name="view-analytics-for-a-bot"></a>檢視針對 Bot 的分析

若要存取 Analytics，請在 Azure 入口網站中開啟 Bot，然後按一下 [Analytics]  。

資料太多嗎？ 您可以對連結到 Bot 的 Application Insights [啟用並設定取樣](/azure/application-insights/app-insights-sampling)。 這可減少遙測流量和儲存體，同時能夠維持統計上正確的分析。

### <a name="specify-channel"></a>指定通道

選擇哪個通道會出現在下圖中。 請注意，如果 Bot 未在通道上啟用，就不會有來自該通道的任何資料。

![選取通道](~/media/analytics-channels.png)

* 勾選核取方塊以在圖表中包含通道。
* 清除核取方塊以從圖表中移除通道。

### <a name="specify-time-period"></a>指定時間週期

分析僅適用於過去 90 天。 當 Application Insights 啟用時，資料收集隨即開始。

![選取時間週期](~/media/analytics-timepick.png)

按一下下拉式功能表，然後按一下應該要顯示圖表的時間量。
請注意，變更整體時間範圍會導致圖表上的時間增量 (X 軸) 據以變更。

### <a name="grand-totals"></a>總計

在指定時間範圍內的作用中使用者，以及傳送及接收的訊息總數。
破折號 `--` 表示沒有活動。

### <a name="retention"></a>保留

「保留」會追蹤多少使用者傳送一則訊息，然後又返回再傳送另一則。
圖表是累積的 10 天時間範圍；變更時間範圍不會影響結果。

![保留圖表](~/media/analytics-retention.png)

請注意，可能的最近日期是在兩天前；使用者在前天傳送訊息，然後在昨天「返回」  。

### <a name="user"></a>使用者

使用者圖表會追蹤有多少使用者在指定時間範圍期間，使用各個通道存取 Bot。

![使用者圖表](~/media/analytics-users.png)

* 百分比圖表會顯示使用者使用各個通道的百分比。
* 折線圖會指出有多少使用者曾經在特定時間存取 Bot。
* 折線圖的圖例會指出哪個色彩表示哪個通道，並且包含指定時間週期期間的使用者總數。

### <a name="activities"></a>活動

「活動」圖表會追蹤在指定的時間範圍內，使用了哪個通道傳送及接收了多少活動。

![活動圖表](~/media/analytics-activities.png)

* 百分比圖表會顯示透過各個通道通訊的活動百分比。
* 折線圖會指出在指定的時間範圍內，傳送及接收了多少活動。
* 折線圖的圖例會指出哪個線條色彩表示哪個通道，以及指定的時間週期內，在該通道傳送及接收的活動總數。

## <a name="enable-analytics"></a>啟用分析

在啟用並設定 Application Insights 之前，都無法使用 Analytics。 Application Insights 一旦啟用，就會開始收集資料。 例如，如果 Application Insights 在一週前已針對六個月的 Bot 啟用，它就會收集一週的資料。

> [!NOTE]
> Analytics 同時需要 Azure 訂用帳戶與 Application Insights [資源](/azure/application-insights/app-insights-create-new-resource)。
若要存取 Application Insights，請在 [Azure 入口網站](https://portal.azure.com/)中開啟 Bot，然後按一下 [設定]  。

建立 Bot 資源時，您可以新增 Application Insights。

您也可以稍後建立 Application Insights 資源並將其連線到 Bot。

1. 建立 Application Insights [資源](/azure/application-insights/app-insights-create-new-resource)。
2. 在儀表板中開啟 Bot。 按一下 [設定]  ，然後向下捲動至 [Analytics]  區段。
3. 輸入資訊以便將 Bot 連線至 Application Insights。 所有欄位皆為必填項目。

![連線 Insights](~/media/analytics-enable.png)

<!--Snip: As of 12/04/2018, parts of this appear to be out of date. However, ~/bot-service-resources-app-insights-keys.md appears to be up to date.

### AppInsights Instrumentation Key

To find this value, open the Application Insights resource for your bot and navigate to **Configure** > **Properties**.

### AppInsights API key

Provide an Azure App Insights API key. Learn how to [generate a new API key](https://dev.applicationinsights.io/documentation/Authorization/API-key-and-App-ID). Only **Read** permission is required.

### AppInsights Application ID

To find this value, open Application Insights and navigate to **Configure** > **API Access**.

/Snip-->

如需如何找出這些值的詳細資訊，請參閱 [Application Insights 金鑰](~/bot-service-resources-app-insights-keys.md)。

## <a name="additional-resources"></a>其他資源
* [Application Insights 金鑰](~/bot-service-resources-app-insights-keys.md)