---
title: Bot 分析 | Microsoft Docs
description: 了解如何使用資料收集和分析，利用 Bot Framework 中的分析來改善 Bot。
keywords: bot 分析, application insights, 流量, 延遲, 整合, AppInsights
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 12/13/2017
ms.openlocfilehash: 503e9b2231b198346f5a7cd767a1e6a866e9e5b3
ms.sourcegitcommit: f576981342fb3361216675815714e24281e20ddf
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/18/2018
ms.locfileid: "39299206"
---
# <a name="bot-analytics"></a>Bot 分析
Analytics 是 [Application Insights](/azure/application-insights/app-insights-analytics) 的擴充功能。 Application Insights 提供**服務層級**和檢測資料，例如流量、延遲和整合。 Analytics 提供關於使用者、訊息和通道資料的**對話層級**報告。

## <a name="view-analytics-for-a-bot"></a>檢視針對 Bot 的分析
若要存取 Analytics，請在開發人員入口網站開啟 Bot，然後按一下 [Analytics]。

資料太多嗎？ [啟用並設定取樣](/azure/application-insights/app-insights-sampling)以減少遙測流量和儲存體，同時能夠維持統計上正確的分析。 

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
在指定時間範圍期間的作用中使用者，以及傳送及接收的訊息總數。
破折號 `--` 表示沒有活動。

### <a name="retention"></a>保留
「保留」會追蹤多少使用者傳送一則訊息，然後又返回再傳送另一則。
圖表是累積的 10 天時間範圍；變更時間範圍不會影響結果。

![保留圖表](~/media/analytics-retention.png)

請注意，可能的最近日期是在兩天前；使用者在前天傳送訊息，然後在昨天「返回」。

### <a name="user"></a>使用者
使用者圖表會追蹤有多少使用者在指定時間範圍期間，使用各個通道存取 Bot。

![使用者圖表](~/media/analytics-users.png)

* 百分比圖表會顯示使用者使用各個通道的百分比。
* 折線圖會指出有多少使用者曾經在特定時間存取 Bot。
* 折線圖的圖例會指出哪個色彩表示哪個通道，並且包含指定時間週期期間的使用者總數。

### <a name="messages"></a>訊息
訊息圖表會追蹤指定時間範圍期間，使用了哪個通道傳送及接收了多少訊息。

![訊息圖表](~/media/analytics-messages.png)

* 百分比圖表會顯示透過各個通道通訊的訊息百分比。
* 折線圖會指出一段指定時間範圍內，已傳送及接收了多少訊息。
* 折線圖的圖例會指出哪個線條色彩表示哪個通道，以及指定時間週期期間在該通道傳送及接收的訊息總數。 

## <a name="enable-analytics"></a>啟用分析
在啟用並設定 Application Insights 之前，都無法使用 Analytics。 Application Insights 一旦啟用，就會開始收集資料。 例如，如果 Application Insights 在一週前已針對六個月的 Bot 啟用，它就會收集一週的資料。
> [!NOTE]
> Analytics 同時需要 Azure 訂用帳戶與 Application Insights [資源](/azure/application-insights/app-insights-create-new-resource)。
若要存取 Application Insights，請在 [Bot Framework 入口網站](https://dev.botframework.com/)中開啟 Bot，然後按一下 [設定]。

1. 建立 Application Insights [資源](/azure/application-insights/app-insights-create-new-resource)。
2. 在儀表板中開啟 Bot。 按一下 [設定]，然後向下捲動至 [Analytics] 區段。
3. 輸入資訊以便將 Bot 連線至 Application Insights。 所有欄位皆為必填項目。

![連線 Insights](~/media/analytics-enable.png)

### <a name="appinsights-instrumentation-key"></a>AppInsights 檢測金鑰
若要尋找此值，請開啟 Application Insights 然後瀏覽至 [設定] > [屬性]。

### <a name="appinsights-api-key"></a>AppInsights API 金鑰
提供 Azure App Insights API 金鑰。 了解如何[產生新的 API 金鑰](https://dev.applicationinsights.io/documentation/Authorization/API-key-and-App-ID)。 只需要**讀取**權限。

### <a name="appinsights-application-id"></a>AppInsights 應用程式識別碼
若要尋找此值，請開啟 Application Insights 然後瀏覽至 [設定] > [API 存取]。

如需如何找出這些值的詳細資訊，請參閱 [Application Insights 金鑰](~/bot-service-resources-app-insights-keys.md)。

## <a name="additional-resources"></a>其他資源
* [Application Insights 金鑰](~/bot-service-resources-app-insights-keys.md)