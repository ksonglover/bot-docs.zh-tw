---
title: 針對 Bot HTTP 500 錯誤進行疑難排解 | Microsoft Docs
description: 如何針對已部署 Bot 中的 HTTP 500 錯誤進行疑難排解。
keywords: 疑難排解, HTTP 500, 問題。
author: jonathanFingold
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 4/30/2019
ms.openlocfilehash: 3dcb22c2310f8c686f02fae27617061681910d01
ms.sourcegitcommit: a6d02ec4738e7fc90b7108934740e9077667f3c5
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/04/2019
ms.locfileid: "70298601"
---
# <a name="troubleshoot-http-500-errors"></a>針對 HTTP 500 錯誤進行疑難排解

針對 500 錯誤進行疑難排解的第一個步驟是啟用 Application Insights。

<!-- TODO: Add links back in once there's a fresh AppInsights sample.
The luis-with-appinsights ([C# sample](https://aka.ms/cs-luis-with-appinsights-sample) / [JS sample](https://aka.ms/js-luis-with-appinsights-sample)) and qna-with-appinsights ([C# sample](https://aka.ms/qna-with-appinsights) / [JS sample](https://aka.ms/js-qna-with-appinsights-sample)) samples demonstrate bots that support Azure Application Insights.
-->
如需有關如何將 Application Insights 新增至現有 Bot 的資訊，請參閱[對話分析遙測](https://aka.ms/botframeworkanalytics)。

## <a name="enable-application-insights-on-aspnet"></a>在 ASP.Net 上啟用 Application Insights

如需基本 Application Insights 支援，請參閱如何[設定 ASP.NET 網站的 Application Insights](https://docs.microsoft.com/azure/application-insights/app-insights-asp-net)。 Bot Framework (從 4.2 版開始) 會提供一層額外的 Application Insights 遙測，但並非診斷 HTTP 500 錯誤的必要項目。

## <a name="enable-application-insights-on-nodejs"></a>在 Node.js 上啟用 Application Insights

如需基本 Application Insights 支援，請參閱如何[使用 Application Insights 監視 Node.js 服務和應用程式](https://docs.microsoft.com/azure/azure-monitor/learn/nodejs-quick-start)。 Bot Framework (從 4.2 版開始) 會提供一層額外的 Application Insights 遙測，但並非診斷 HTTP 500 錯誤的必要項目。

## <a name="query-for-exceptions"></a>查詢例外狀況

分析 HTTP 狀態碼 500 錯誤的最簡單方法是從例外狀況著手。

下列查詢會告訴您最近的例外狀況：

```sql
exceptions
| order by timestamp desc
| project timestamp, operation_Id, appName
```

從第一個查詢，選取一些作業識別碼並尋找更多資訊：

```sql
let my_operation_id = "d298f1385197fd438b520e617d58f4fb";
let union_all = () {
    union
    (traces | where operation_Id == my_operation_id),
    (customEvents | where operation_Id == my_operation_id),
    (requests | where operation_Id == my_operation_id),
    (dependencies | where operation_Id  == my_operation_id),
    (exceptions | where operation_Id == my_operation_id)
};

union_all
    | order by timestamp desc
```

如果您只有 `exceptions`，請分析詳細資料並查看是否對應於您程式碼中的各行。 如果您只看到來自通道連接器的例外狀況 (`Microsoft.Bot.ChannelConnector`)，請參閱[沒有 Application Insights 事件](#no-application-insights-events)以確保 Application Insights 已正確設定且您的程式碼正在記錄事件。

## <a name="no-application-insights-events"></a>沒有 Application Insights 事件

如果您收到 500 錯誤，而且 Application Insights 中沒有來自您 Bot 的進一步事件，請檢查下列各項：

### <a name="ensure-bot-runs-locally"></a>確保 Bot 在本機執行

先使用模擬器確保 Bot 在本機執行。

### <a name="ensure-configuration-files-are-being-copied-net-only"></a>確保正在複製組態檔案 (僅限 .NET)

確保在部署過程中正確封裝 `appsettings.json` 和其他設定檔。

#### <a name="application-assemblies"></a>應用程式組件

確保在部署過程中正確封裝 Application Insights 組件。

- Microsoft.ApplicationInsights
- Microsoft.ApplicationInsights.TraceListener
- Microsoft.AI.Web
- Microsoft.AI.WebServer
- Microsoft.AI.ServeTelemetryChannel
- Microsoft.AI.PerfCounterCollector
- Microsoft.AI.DependencyCollector
- Microsoft.AI.Agent.Intercept

確保在部署過程中正確封裝 `appsettings.json` 和其他設定檔。

#### <a name="appsettingsjson"></a>appsettings.json

在 `appsettings.json` 檔案內，確保已設定檢測金鑰。

## <a name="aspnet-web-apitabdotnetwebapi"></a>[ASP.NET Web API](#tab/dotnetwebapi)

```json
{
    "Logging": {
        "IncludeScopes": false,
        "LogLevel": {
            "Default": "Debug",
            "System": "Information",
            "Microsoft": "Information"
        },
        "Console": {
            "IncludeScopes": "true"
        }
    }
}
```

## <a name="aspnet-coretabdotnetcore"></a>[ASP.NET Core](#tab/dotnetcore)

```json
{
    "ApplicationInsights": {
        "InstrumentationKey": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx"
    }
}
```

---

### <a name="verify-config-file"></a>確認設定檔

確保設定檔包含 Application Insights 金鑰。

```json
{
    "ApplicationInsights": {
        "type": "appInsights",
        "tenantId": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
        "subscriptionId": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
        "resourceGroup": "my resource group",
        "name": "my appinsights name",
        "serviceName": "my service name",
        "instrumentationKey": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
        "applicationId": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
        "apiKeys": {},
        "id": ""
    }
},
```

### <a name="check-logs"></a>查看記錄

Bot ASP.Net 和 Node 會在伺服器層級發出可以檢查的記錄。

#### <a name="set-up-a-browser-to-watch-your-logs"></a>設定要監看您記錄的瀏覽器

1. 在 [Azure 入口網站](http://portal.azure.com/)中開啟 Bot。
1. 開啟 [App Service 設定 / 所有 App Service 設定]  頁面，以查看所有服務設定。
1. 開啟應用程式服務的 [監視 / 診斷記錄]  頁面。
   - 確保已啟用 [應用程式記錄 (檔案系統)]  。 如果變更此設定，請務必按一下 [儲存]  。
1. 切換至 [監視 / 記錄資料流]  頁面。
   - 選取 [Web 伺服器記錄]  ，並確保您看到已連線的訊息。 您應該會看到如下的內容：

     ```bash
     Connecting...
     2018-11-14T17:24:51  Welcome, you are now connected to log-streaming service.
     ```

     讓此視窗保持開啟。

#### <a name="set-up-browser-to-restart-your-bot-service"></a>設定瀏覽器以重新啟動您的 Bot 服務

1. 使用不同的瀏覽器，在 Azure 入口網站中開啟 Bot。
1. 開啟 [App Service 設定 / 所有 App Service 設定]  頁面，以查看所有服務設定。
1. 切換至應用程式服務的 [概觀]  頁面，按一下 [重新啟動]  。
   - 系統會提示您是否確定選取 [是]  。
1. 返回第一個瀏覽器視窗並監看記錄。
1. 確認您正在接收新的記錄。
   - 如果沒有任何活動，請重新部署您的 Bot。
   - 然後切換到 [應用程式記錄]  頁面並尋找是否有任何錯誤。
