---
title: 將遙測資料新增至 Bot | Microsoft Docs
description: 了解如何整合 Bot 與新的遙測功能。
keywords: 遙測, appinsights, 監視 bot
author: ivorb
ms.author: v-ivorb
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 02/06/2019
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 75e12ab44915783c33c3b2ee10775cc6f00487bb
ms.sourcegitcommit: aea57820b8a137047d59491b45320cf268043861
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 04/22/2019
ms.locfileid: "59905031"
---
# <a name="add-telemetry-to-your-bot"></a>將遙測新增至 Bot

[!INCLUDE[applies-to](../includes/applies-to.md)]

在 Bot Framework sdk 4.2 版中，已在產品中新增遙測記錄功能。  這可讓 Bot 應用程式將事件資料傳送至 Application Insights 之類的服務。

本文件說明如何整合 Bot 與新的遙測功能。  

## <a name="using-bot-configuration-option-1-of-2"></a>使用 Bot 組態 (選項 1/2)
設定 Bot 有兩種方法。  第一種方法假設您要與 Application Insights 整合。

Bot 組態檔包含有關 Bot 在執行時所用外部服務的中繼資料。  例如，CosmosDB、Application Insights 和 Language Understanding (LUIS) 服務連線和中繼資料都會儲存在這裡。   

如果您想要「貯存」Application Insights，不需要額外的 Application Insights 專屬設定 (例如遙測初始設定式)，只要在初始化期間傳入 Bot 組態物件即可。   這是最簡單的初始化方法並且會設定 Application Insights，以開始追蹤要求、其他服務的外部呼叫，以及跨服務相互關聯事件。

```csharp
// ASP.Net Core - Startup.cs

public void ConfigureServices(IServiceCollection services)
{
     ...
     // Add Application Insights - pass in the bot configuration
     services.AddBotApplicationInsights(botConfig);
     ...
}

public void Configure(IApplicationBuilder app, IHostingEnvironment env)
{
     app.UseBotApplicationInsights()
                 ...
                .UseDefaultFiles()
                .UseStaticFiles()
                .UseBotFramework();
                ...
}
```

```JavaScript
const appInsightsClient = new ApplicationInsightsTelemetryClient(botConfig);
```

**Bot 組態中沒有 Application Insights** 如果 Bot 組態未包含 Application Insights 會怎麼樣？  沒問題，它會預設為 null 用戶端，此方法不會呼叫任何作業。

**多個 Application Insights** 您的 Bot 組態中有多個 Application Insight 區段嗎？  您可以指定您想要在 Bot 組態內使用的 Application Insights 服務執行個體。

```csharp
// ASP.Net Core - Startup.cs

public void ConfigureServices(IServiceCollection services)
{
     // Add Application Insights
     services.AddBotApplicationInsights(botConfig, "myAppInsights");
     ...
}
```

```JavaScript
const appInsightsClient = new ApplicationInsightsTelemetryClient(botConfig);
```

## <a name="overriding-the-telemetry-client-option-2-of-2"></a>覆寫遙測用戶端 (選項 2/2)

如果您想要自訂 Application Insights 用戶端，或者想要登入完全不同的服務，則必須以不同的方式設定系統。

**修改 Application Insights 組態**

```csharp

public void ConfigureServices(IServiceCollection services)
{
     ...
     // Create Application Insight Telemetry Client
     // with custom configuration.
     var telemetryClient = TelemetryClient(myCustomConfiguration)
     
     // Add Application Insights
     services.AddBotApplicationInsights(new BotTelemetryClient(telemetryClient), "InstrumentationKey");
```

```JavaScript
const appInsightsClient = new ApplicationInsightsTelemetryClient(botConfig);
```

**使用自訂遙測** 如果您想要將 Bot Framework 所產生的遙測事件記錄到完全不同的系統，請建立自基底介面衍生的新類別並加以設定。  

```csharp
public void ConfigureServices(IServiceCollection services)
{
     ...
     // Create my IBotTelemetryClient-based logger
     var myTelemetryClient = MyTelemetryLogger();
     
     // Add Application Insights
     services.AddBotApplicationInsights(myTelemetryClient);
     ...
}
```

```JavaScript
const appInsightsClient = new ApplicationInsightsTelemetryClient(botConfig);
```

## <a name="add-custom-logging-to-your-bot"></a>將自訂記錄新增至您的 Bot

一旦 Bot 設定了新的遙測記錄支援，您即可開始將遙測新增至您的 Bot。  `BotTelemetryClient` (採用 C#，`IBotTelemetryClient`) 有數種方法可記錄不同類型的事件。  選擇適當的事件類型，可讓您利用 Application Insights 現有的報告 (如果您使用 Application Insights 的話)。  在一般情況下，通常會使用 `TraceEvent`。  使用 `TraceEvent` 記錄的資料會落在 Kusto 的 `CustomEvent` 資料表中。

如果在 Bot 內使用對話，則每個對話式物件 (包括提示) 都會包含新的 `TelemetryClient` 屬性。  這是可讓您執行記錄的 `BotTelemetryClient`。  這不是只為了方便而已，我們會在本文稍看到如果已設定這個屬性，`WaterfallDialogs` 將會產生事件。

### <a name="identifiers-and-custom-events"></a>識別碼和自訂事件

將事件記錄到 Application Insights 時，所產生的事件包含您不必填入的預設屬性。  比方說，每個自訂事件 (以 `TraceEvent` API 產生) 都會包含 `user_id` 和 `session_id` 屬性。  此外，也會新增 `activitiId`、`activityType` 和 `channelId`。

>注意：系統不會提供這些值給自訂遙測用戶端。

屬性 |類型 | 詳細資料
--- | --- | ---
`user_id`| `string` | [ChannelID](https://github.com/Microsoft/botframework-obi/blob/master/botframework-activity/botframework-activity.md#channel-id) + [From.Id](https://github.com/Microsoft/botframework-obi/blob/master/botframework-activity/botframework-activity.md#from)
`session_id`| `string`|  [ConversationID](https://github.com/Microsoft/botframework-obi/blob/master/botframework-activity/botframework-activity.md#conversation)
`customDimensions.activityId`| `string` | [Bot 活動識別碼](https://github.com/Microsoft/botframework-obi/blob/master/botframework-activity/botframework-activity.md#id)
`customDimensions.activityType` | `string` | [Bot 活動類型](https://github.com/Microsoft/botframework-obi/blob/master/botframework-activity/botframework-activity.md#channel-id)
`customDimensions.channelId` | `string` |  [Bot 活動通道識別碼](https://github.com/Microsoft/botframework-obi/blob/master/botframework-activity/botframework-activity.md#channel-id)

## <a name="waterfalldialog-events"></a>WaterfallDialog 事件

除了產生您自己的事件，SDK 內的 `WaterfallDialog` 物件現在也會產生事件。 下一節說明從 Bot Framework 內產生的事件。 在 `WaterfallDialog` 上設定 `TelemetryClient` 屬性，就會儲存這些事件。

以下範例示範如何修改利用 `WaterfallDialog` 來記錄遙測事件的範例 (BasicBot)。  BasicBot 會利用當 `WaterfallDialog` 置於 `ComponentDialog` (`GreetingDialog`) 內時使用的通用樣式。

```csharp
// IBotTelemetryClient is direct injected into our Bot
public BasicBot(BotServices services, UserState userState, ConversationState conversationState, IBotTelemetryClient telemetryClient)
...

// The IBotTelemetryClient passed to the GreetingDialog
...
Dialogs = new DialogSet(_dialogStateAccessor);
Dialogs.Add(new GreetingDialog(_greetingStateAccessor, telemetryClient));
...

// The IBotTelemetryClient to the WaterfallDialog
...
AddDialog(new WaterfallDialog(ProfileDialog, waterfallSteps) { TelemetryClient = telemetryClient });
...

```

一旦 `WaterfallDialog` 有已設定的 `IBotTelemetryClient`，它就會開始記錄事件。

### <a name="customevent-waterfallstart"></a>CustomEvent："WaterfallStart" 

當 WaterfallDialog 開始時，就會記錄 `WaterfallStart` 事件。

- `user_id`  ([從遙測初始設定式](https://aka.ms/telemetry-initializer))
- `session_id` ([從遙測初始設定式](https://aka.ms/telemetry-initializer))
- `customDimensions.activityId`  ([從遙測初始設定式](https://aka.ms/telemetry-initializer))
- `customDimensions.activityType`  ([從遙測初始設定式](https://aka.ms/telemetry-initializer))
- `customDimensions.channelId` ([從遙測初始設定式](https://aka.ms/telemetry-initializer))
- `customDimensions.DialogId` (這是傳入您瀑布的 dialogId (字串)。  您可以將此視為「瀑布類型」)
- `customDimensions.InstanceID` (每個對話執行個體特有)

### <a name="customevent-waterfallstep"></a>CustomEvent："WaterfallStep" 

記錄瀑布式對話中的個別步驟。

- `user_id`  ([從遙測初始設定式](https://aka.ms/telemetry-initializer))
- `session_id` ([從遙測初始設定式](https://aka.ms/telemetry-initializer))
- `customDimensions.activityId`  ([從遙測初始設定式](https://aka.ms/telemetry-initializer))
- `customDimensions.activityType`  ([從遙測初始設定式](https://aka.ms/telemetry-initializer))
- `customDimensions.channelId` ([從遙測初始設定式](https://aka.ms/telemetry-initializer))
- `customDimensions.DialogId` (這是傳入您瀑布的 dialogId (字串)。  您可以將此視為「瀑布類型」)
- `customDimensions.StepName` (若為 lambda 則是方法名稱或 `StepXofY`)
- `customDimensions.InstanceID` (每個對話執行個體特有)

### <a name="customevent-waterfalldialogcomplete"></a>CustomEvent："WaterfallDialogComplete"

記錄於瀑布式對話完成時。

- `user_id`  ([從遙測初始設定式](https://aka.ms/telemetry-initializer))
- `session_id` ([從遙測初始設定式](https://aka.ms/telemetry-initializer))
- `customDimensions.activityId`  ([從遙測初始設定式](https://aka.ms/telemetry-initializer))
- `customDimensions.activityType`  ([從遙測初始設定式](https://aka.ms/telemetry-initializer))
- `customDimensions.channelId` ([從遙測初始設定式](https://aka.ms/telemetry-initializer))
- `customDimensions.DialogId` (這是傳入您瀑布的 dialogId (字串)。  您可以將此視為「瀑布類型」)
- `customDimensions.InstanceID` (每個對話執行個體特有)

### <a name="customevent-waterfalldialogcancel"></a>CustomEvent："WaterfallDialogCancel" 

記錄於瀑布式對話取消時。

- `user_id`  ([從遙測初始設定式](https://aka.ms/telemetry-initializer))
- `session_id` ([從遙測初始設定式](https://aka.ms/telemetry-initializer))
- `customDimensions.activityId`  ([從遙測初始設定式](https://aka.ms/telemetry-initializer))
- `customDimensions.activityType`  ([從遙測初始設定式](https://aka.ms/telemetry-initializer))
- `customDimensions.channelId` ([從遙測初始設定式](https://aka.ms/telemetry-initializer))
- `customDimensions.DialogId` (這是傳入您瀑布的 dialogId (字串)。  您可以將此視為「瀑布類型」)
- `customDimensions.StepName` (若為 lambda 則是方法名稱或 `StepXofY`)
- `customDimensions.InstanceID` (每個對話執行個體特有)


## <a name="events-generated-by-the-bot-framework-service"></a>Bot Framework Service 所產生的事件

除了 `WaterfallDialog` (可從 Bot 程式碼產生事件)，Bot Framework Channel 服務也會記錄事件。  這可協助您診斷通道或整體 Bot 失敗的問題。

### <a name="customevent-activity"></a>CustomEvent："Activity"
**記錄來源：** 通道服務 在收到訊息時由通道服務記錄。

### <a name="exception-bot-errors"></a>例外狀況：「Bot 錯誤」
**記錄來源：** 通道服務 在 Bot 呼叫傳回非 2XX Http 回應時由通道記錄。

## <a name="additional-events"></a>其他事件

[企業範本](https://github.com/Microsoft/AI/tree/master/templates/Enterprise-Template)是可任意複製的開源程式碼。  這包含數個可重複使用和修改以符合您報告需求的元件。

### <a name="customevent-botmessagereceived"></a>CustomEvent：BotMessageReceived 
**記錄來源：** TelemetryLoggerMiddleware (**企業範例**)

記錄於 Bot 收到新訊息時。

- UserID  ([從遙測初始設定式](https://aka.ms/telemetry-initializer))
- ConversationID ([從遙測初始設定式](https://aka.ms/telemetry-initializer))
- ActivityID  ([從遙測初始設定式](https://aka.ms/telemetry-initializer))
- Channel  ([從遙測初始設定式](https://aka.ms/telemetry-initializer))
- ActivityType  ([從遙測初始設定式](https://aka.ms/telemetry-initializer))
- 文字 (PII 選用)
- FromId
- FromName
- RecipientId
- RecipientName
- ConversationId
- ConversationName
- 地區設定

### <a name="customevent-botmessagesend"></a>CustomEvent：BotMessageSend 
**記錄來源：** TelemetryLoggerMiddleware (**企業範例**)

記錄於 Bot 傳送訊息時。

- UserID  ([從遙測初始設定式](https://aka.ms/telemetry-initializer))
- ConversationID ([從遙測初始設定式](https://aka.ms/telemetry-initializer))
- ActivityID  ([從遙測初始設定式](https://aka.ms/telemetry-initializer))
- Channel  ([從遙測初始設定式](https://aka.ms/telemetry-initializer))
- ActivityType  ([從遙測初始設定式](https://aka.ms/telemetry-initializer))
- ReplyToID
- Channel  (來源通道 - 例如 Skype、Cortana、Teams)
- RecipientId
- ConversationName
- 地區設定
- 文字 (PII 選用)
- RecipientName (PII 選用)

### <a name="customevent-botmessageupdate"></a>CustomEvent：BotMessageUpdate
**記錄來源：** TelemetryLoggerMiddleware 記錄於 Bot 更新訊息時 (罕見案例)

### <a name="customevent-botmessagedelete"></a>CustomEvent：BotMessageDelete
**記錄來源：** TelemetryLoggerMiddleware 記錄於 Bot 刪除訊息時 (罕見案例)

### <a name="customevent-luisintentinentname"></a>CustomEvent：LuisIntent.INENTName 
**記錄來源：** TelemetryLuisRecognizer (**企業範例**)

記錄 LUIS 服務的結果。

- UserID  ([從遙測初始設定式](https://aka.ms/telemetry-initializer))
- ConversationID ([從遙測初始設定式](https://aka.ms/telemetry-initializer))
- ActivityID  ([從遙測初始設定式](https://aka.ms/telemetry-initializer))
- Channel  ([從遙測初始設定式](https://aka.ms/telemetry-initializer))
- ActivityType  ([從遙測初始設定式](https://aka.ms/telemetry-initializer))
- 意圖
- IntentScore
- 問題
- ConversationId
- SentimentLabel
- SentimentScore
- *LUIS 實體*
- **新的** DialogId

### <a name="customevent-qnamessage"></a>CustomEvent：QnAMessage
**記錄來源：** TelemetryQnaMaker (**企業範例**)

記錄 QnA Maker 服務的結果。

- UserID  ([從遙測初始設定式](https://aka.ms/telemetry-initializer))
- ConversationID ([從遙測初始設定式](https://aka.ms/telemetry-initializer))
- ActivityID  ([從遙測初始設定式](https://aka.ms/telemetry-initializer))
- Channel  ([從遙測初始設定式](https://aka.ms/telemetry-initializer))
- ActivityType  ([從遙測初始設定式](https://aka.ms/telemetry-initializer))
- 使用者名稱
- ConversationId
- OriginalQuestion
- 問題
- Answer
- Score (*選用*：如果我們找到知識)

## <a name="querying-the-data"></a>查詢資料
使用 Application Insights 時，所有資料都會相互關聯 (即使是跨服務)。  我們可以查詢成功的要求，以及查看該要求的所有相關事件。  
下列查詢會告訴您最近的要求：
```sql
requests 
| where timestamp > ago(3d) 
| where resultCode == 200
| order by timestamp desc
| project timestamp, operation_Id, appName
| limit 10
```

從第一個查詢中，選取一些 `operation_Id`，然後尋找更多資訊：

```sql
let my_operation_id = "<OPERATION_ID>";
let union_all = () {
    union
    (traces | where operation_Id == my_operation_id),
    (customEvents | where operation_Id == my_operation_id),
    (requests | where operation_Id == my_operation_id),
    (dependencies | where operation_Id  == my_operation_id),
    (exceptions | where operation_Id == my_operation_id)
};
union_all
    | order by timestamp asc
    | project itemType, name, performanceBucket
```

這可提供單一要求的時間順序明細，以及每次呼叫的持續時間貯體。
![範例呼叫](media/performance_query.png)

> 注意："Activity" `customEvent` 事件時間戳記失序，因為這些事件是以非同步方式記錄。

## <a name="create-a-dashboard"></a>建立儀表板

最簡單的測試方法就是使用 [Azure 入口網站的範本部署頁面](https://portal.azure.com/#create/Microsoft.Template)建立儀表板。  
- 按一下 [在編輯器中建置您自己的範本]
- 複製並貼上為了協助您建立儀表板而提供的其中一個 .json 檔案：
  - [系統健康情況儀表板](https://aka.ms/system-health-appinsights)
  - [交談健康情況儀表板](https://aka.ms/conversation-health-appinsights)
- 按一下 [儲存]
- 填入 `Basics`： 
   - 訂用帳戶：<your test subscription>
   - 資源群組：<a test resource group>
   - 位置：<such as West US>
- 填入 `Settings`：
   - 見解元件名稱：<像是 `core672so2hw`>
   - 見解元件資源群組：<像是 `core67`>
   - 儀表板名稱：<像是 `'ConversationHealth'` 或 `SystemHealth`>
- 按一下 `I agree to the terms and conditions stated above`
- 按一下 `Purchase`
- 驗證
   - 按一下 [`Resource Groups`](https://ms.portal.azure.com/#blade/HubsExtension/Resources/resourceType/Microsoft.Resources%2Fsubscriptions%2FresourceGroups)
   - 從上面選取您的資源群組 (像是 `core67`)。
   - 如果您沒有看到新的資源，則查看 [部署] 並了解是否有任何失敗。
   - 以下是您通常看到的失敗：
     
```json
{"code":"DeploymentFailed","message":"At least one resource deployment operation failed. Please list deployment operations for details. Please see https://aka.ms/arm-debug for usage details.","details":[{"code":"BadRequest","message":"{\r\n \"error\": {\r\n \"code\": \"InvalidTemplate\",\r\n \"message\": \"Unable to process template language expressions for resource '/subscriptions/45d8a30e-3363-4e0e-849a-4bb0bbf71a7b/resourceGroups/core67/providers/Microsoft.Portal/dashboards/Bot Analytics Dashboard' at line '34' and column '9'. 'The template parameter 'virtualMachineName' is not found. Please see https://aka.ms/arm-template/#parameters for usage details.'\"\r\n }\r\n}"}]}
```

若要檢視資料，請移至 Azure 入口網站。 按一下左邊的 [儀表板]，然後從下拉式清單中選取您建立的儀表板。

## <a name="additional-resources"></a>其他資源
您可以參考這些實作遙測的範例：
- C#
  - [LUIS 搭配 AppInsights](https://aka.ms/luis-with-appinsights-cs)
  - [QnA 搭配 AppInsights](https://aka.ms/qna-with-appinsights-cs)
- JS
  - [LUIS 搭配 AppInsights](https://aka.ms/luis-with-appinsights-js)
  - [QnA 搭配 AppInsights](https://aka.ms/qna-with-appinsights-js)

