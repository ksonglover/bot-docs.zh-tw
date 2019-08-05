---
title: Bot Framework Service 遙測所產生的事件 | Microsoft Docs
description: 了解新的遙測功能會觸發哪些事件。
keywords: 遙測, appinsights, 監視 bot
author: ivorb
ms.author: v-ivorb
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 05/23/2019
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 929432d51a7bdfbcbd328ae7ea4e0df8873e7392
ms.sourcegitcommit: 23a1808e18176f1704f2f6f2763ace872b1388ae
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/25/2019
ms.locfileid: "68484355"
---
# <a name="events-generated-by-the-bot-framework-service-telemetry"></a>Bot Framework Service 遙測所產生的事件

## <a name="channel-service-events"></a>通道服務事件

除了 `WaterfallDialog` (已於[遙測主題](bot-builder-telemetry.md)討論過，而且會從聊天機器人的程式碼產生事件)，Bot Framework Channel 服務也會記錄事件。  這可協助您診斷通道或整體 Bot 失敗的問題。

### <a name="customevent-activity"></a>CustomEvent："Activity"
**記錄來源：** 通道服務 在收到訊息時由通道服務記錄。

### <a name="exception-bot-errors"></a>例外狀況：「Bot 錯誤」
**記錄來源：** 通道服務 在 Bot 呼叫傳回非 2XX Http 回應時由通道記錄。

## <a name="customevent-waterfallstart"></a>CustomEvent："WaterfallStart" 

當 WaterfallDialog 開始時，就會記錄 `WaterfallStart` 事件。

- `user_id`  ([從遙測初始設定式](https://aka.ms/telemetry-initializer))
- `session_id` ([從遙測初始設定式](https://aka.ms/telemetry-initializer))
- `customDimensions.activityId`  ([從遙測初始設定式](https://aka.ms/telemetry-initializer))
- `customDimensions.activityType`  ([從遙測初始設定式](https://aka.ms/telemetry-initializer))
- `customDimensions.channelId` ([從遙測初始設定式](https://aka.ms/telemetry-initializer))
- `customDimensions.DialogId` (這是傳入您瀑布的 dialogId (字串)。  您可以將此視為「瀑布類型」)
- `customDimensions.InstanceID` (每個對話執行個體特有)

## <a name="customevent-waterfallstep"></a>CustomEvent："WaterfallStep" 

記錄瀑布式對話中的個別步驟。

- `user_id`  ([從遙測初始設定式](https://aka.ms/telemetry-initializer))
- `session_id` ([從遙測初始設定式](https://aka.ms/telemetry-initializer))
- `customDimensions.activityId`  ([從遙測初始設定式](https://aka.ms/telemetry-initializer))
- `customDimensions.activityType`  ([從遙測初始設定式](https://aka.ms/telemetry-initializer))
- `customDimensions.channelId` ([從遙測初始設定式](https://aka.ms/telemetry-initializer))
- `customDimensions.DialogId` (這是傳入您瀑布的 dialogId (字串)。  您可以將此視為「瀑布類型」)
- `customDimensions.StepName` (若為 lambda 則是方法名稱或 `StepXofY`)
- `customDimensions.InstanceID` (每個對話執行個體特有)

## <a name="customevent-waterfalldialogcomplete"></a>CustomEvent："WaterfallDialogComplete"

記錄於瀑布式對話完成時。

- `user_id`  ([從遙測初始設定式](https://aka.ms/telemetry-initializer))
- `session_id` ([從遙測初始設定式](https://aka.ms/telemetry-initializer))
- `customDimensions.activityId`  ([從遙測初始設定式](https://aka.ms/telemetry-initializer))
- `customDimensions.activityType`  ([從遙測初始設定式](https://aka.ms/telemetry-initializer))
- `customDimensions.channelId` ([從遙測初始設定式](https://aka.ms/telemetry-initializer))
- `customDimensions.DialogId` (這是傳入您瀑布的 dialogId (字串)。  您可以將此視為「瀑布類型」)
- `customDimensions.InstanceID` (每個對話執行個體特有)

## <a name="customevent-waterfalldialogcancel"></a>CustomEvent："WaterfallDialogCancel" 

記錄於瀑布式對話取消時。

- `user_id`  ([從遙測初始設定式](https://aka.ms/telemetry-initializer))
- `session_id` ([從遙測初始設定式](https://aka.ms/telemetry-initializer))
- `customDimensions.activityId`  ([從遙測初始設定式](https://aka.ms/telemetry-initializer))
- `customDimensions.activityType`  ([從遙測初始設定式](https://aka.ms/telemetry-initializer))
- `customDimensions.channelId` ([從遙測初始設定式](https://aka.ms/telemetry-initializer))
- `customDimensions.DialogId` (這是傳入您瀑布的 dialogId (字串)。  您可以將此視為「瀑布類型」)
- `customDimensions.StepName` (若為 lambda 則是方法名稱或 `StepXofY`)
- `customDimensions.InstanceID` (每個對話執行個體特有)

## <a name="customevent-botmessagereceived"></a>CustomEvent：BotMessageReceived 
記錄於 Bot 從使用者收到新訊息時。

若未加以覆寫，則會使用 `Microsoft.Bot.Builder.IBotTelemetry.TrackEvent()` 方法從 `Microsoft.Bot.Builder.TelemetryLoggerMiddleware` 記錄此事件。

- 工作階段識別碼  
  - 使用 Application Insights 時，會從 `TelemetryBotIdInitializer` 將此屬性記錄為 Application Insights 中所使用的**工作階段**識別碼 (Temeletry.Context.Session.Id  )。  
  - 對應至 Bot Framework 通訊協定所定義的[對話識別碼](https://github.com/Microsoft/BotBuilder/blob/master/specs/botframework-activity/botframework-activity.md#conversation)。
  - 所記錄的屬性名稱是 `session_id`。

- 使用者識別碼
  - 使用 Application Insights 時，會從 `TelemetryBotIdInitializer` 將此屬性記錄為 Application Insights 中所使用的**使用者**識別碼 (Telemetry.Context.User.Id  )。  
  - 此屬性的值結合了 Bot Framework 通訊協定所定義的[通道識別碼](https://github.com/Microsoft/BotBuilder/blob/master/specs/botframework-activity/botframework-activity.md#channel-id)和[使用者識別碼](https://github.com/Microsoft/BotBuilder/blob/master/specs/botframework-activity/botframework-activity.md#from) (串連在一起) 屬性。
  - 所記錄的屬性名稱是 `user_id`。

- ActivityID 
  - 使用 Application Insights 時，會從 `TelemetryBotIdInitializer` 將此屬性記錄為事件的屬性。
  - 對應至 Bot Framework 通訊協定所定義的[活動識別碼](https://github.com/Microsoft/BotBuilder/blob/master/specs/botframework-activity/botframework-activity.md#Id)。
  - 屬性名稱是 `activityId`。

- 通道識別碼
  - 使用 Application Insights 時，會從 `TelemetryBotIdInitializer` 將此屬性記錄為事件的屬性。  
  - 對應至 Bot Framework 通訊協定的[通道識別碼](https://github.com/Microsoft/BotBuilder/blob/master/specs/botframework-activity/botframework-activity.md#id)。
  - 所記錄的屬性名稱是 `channelId`。

- ActivityType 
  - 使用 Application Insights 時，會從 `TelemetryBotIdInitializer` 將此屬性記錄為事件的屬性。  
  - 對應至 Bot Framework 通訊協定的[活動類型](https://github.com/Microsoft/BotBuilder/blob/master/specs/botframework-activity/botframework-activity.md#type)。
  - 所記錄的屬性名稱是 `activityType`。

- 文字
  - 會在 `logPersonalInformation` 屬性設定為 `true` 時**選擇性地**記錄。
  - 對應至 Bot Framework 通訊協定的[活動文字](https://github.com/Microsoft/BotBuilder/blob/master/specs/botframework-activity/botframework-activity.md#text)欄位。
  - 所記錄的屬性名稱是 `text`。

- Speak

  - 會在 `logPersonalInformation` 屬性設定為 `true` 時**選擇性地**記錄。
  - 對應至 Bot Framework 通訊協定的[活動語音](https://github.com/Microsoft/BotBuilder/blob/master/specs/botframework-activity/botframework-activity.md#speak)欄位。
  - 所記錄的屬性名稱是 `speak`。

  - 

- FromId
  - 對應至 Bot Framework 通訊協定的[來源識別碼](https://github.com/Microsoft/BotBuilder/blob/master/specs/botframework-activity/botframework-activity.md#from)欄位。
  - 所記錄的屬性名稱是 `fromId`。

- FromName
  - 會在 `logPersonalInformation` 屬性設定為 `true` 時**選擇性地**記錄。
  - 對應至 Bot Framework 通訊協定的[來源名稱](https://github.com/Microsoft/BotBuilder/blob/master/specs/botframework-activity/botframework-activity.md#from)欄位。
  - 所記錄的屬性名稱是 `fromName`。

- RecipientId
  - 對應至 Bot Framework 通訊協定的[來源名稱](https://github.com/Microsoft/BotBuilder/blob/master/specs/botframework-activity/botframework-activity.md#from)欄位。
  - 所記錄的屬性名稱是 `fromName`。

- RecipientName
  - 對應至 Bot Framework 通訊協定的[來源名稱](https://github.com/Microsoft/BotBuilder/blob/master/specs/botframework-activity/botframework-activity.md#from)欄位。
  - 所記錄的屬性名稱是 `fromName`。

- ConversationId
  - 對應至 Bot Framework 通訊協定的[來源名稱](https://github.com/Microsoft/BotBuilder/blob/master/specs/botframework-activity/botframework-activity.md#from)欄位。
  - 所記錄的屬性名稱是 `fromName`。

- ConversationName
  - 對應至 Bot Framework 通訊協定的[來源名稱](https://github.com/Microsoft/BotBuilder/blob/master/specs/botframework-activity/botframework-activity.md#from)欄位。
  - 所記錄的屬性名稱是 `fromName`。

- Locale
  - 對應至 Bot Framework 通訊協定的[來源名稱](https://github.com/Microsoft/BotBuilder/blob/master/specs/botframework-activity/botframework-activity.md#from)欄位。
  - 所記錄的屬性名稱是 `fromName`。

## <a name="customevent-botmessagesend"></a>CustomEvent：BotMessageSend 
**記錄來源：** TelemetryLoggerMiddleware 

記錄於 Bot 傳送訊息時。

- UserID  ([從遙測初始設定式](https://aka.ms/telemetry-initializer))
- SessionID ([從遙測初始設定式](https://aka.ms/telemetry-initializer))
- ActivityID  ([從遙測初始設定式](https://aka.ms/telemetry-initializer))
- Channel  ([從遙測初始設定式](https://aka.ms/telemetry-initializer))
- ActivityType  ([從遙測初始設定式](https://aka.ms/telemetry-initializer))
- ReplyToID
- RecipientId
- ConversationName
- Locale
- RecipientName (PII 選用)
- Text (PII 選用)
- Speak (PII 選用)


## <a name="customevent-botmessageupdate"></a>CustomEvent：BotMessageUpdate
**記錄來源：** TelemetryLoggerMiddleware 記錄於 Bot 更新訊息時 (罕見案例)
- UserID  ([從遙測初始設定式](https://aka.ms/telemetry-initializer))
- SessionID ([從遙測初始設定式](https://aka.ms/telemetry-initializer))
- ActivityID ([從遙測初始設定式](https://aka.ms/telemetry-initializer))
- Channel  ([從遙測初始設定式](https://aka.ms/telemetry-initializer))
- ActivityType  ([從遙測初始設定式](https://aka.ms/telemetry-initializer))
- RecipientId
- ConversationId
- ConversationName
- Locale
- Text (PII 選用)


## <a name="customevent-botmessagedelete"></a>CustomEvent：BotMessageDelete
**記錄來源：** TelemetryLoggerMiddleware 記錄於 Bot 刪除訊息時 (罕見案例)
- UserID  ([從遙測初始設定式](https://aka.ms/telemetry-initializer))
- SessionID ([從遙測初始設定式](https://aka.ms/telemetry-initializer))
- ActivityID  ([從遙測初始設定式](https://aka.ms/telemetry-initializer))
- Channel  ([從遙測初始設定式](https://aka.ms/telemetry-initializer))
- ActivityType  ([從遙測初始設定式](https://aka.ms/telemetry-initializer))
- RecipientId
- ConversationId
- ConversationName

## <a name="customevent-luisevent"></a>CustomEvent：LuisEvent
**記錄來源：** LuisRecognizer

記錄 LUIS 服務的結果。

- UserID  ([從遙測初始設定式](https://aka.ms/telemetry-initializer))
- SessionID ([從遙測初始設定式](https://aka.ms/telemetry-initializer))
- ActivityID ([從遙測初始設定式](https://aka.ms/telemetry-initializer))
- Channel ([從遙測初始設定式](https://aka.ms/telemetry-initializer))
- ActivityType ([從遙測初始設定式](https://aka.ms/telemetry-initializer))
- ApplicationId
- Intent
- IntentScore
- Intent2 
- IntentScore2 
- FromId
- SentimentLabel
- SentimentScore
- Entities (json 形式)
- Question (PII 選用)

## <a name="customevent-qnamessage"></a>CustomEvent：QnAMessage
**記錄來源：** QnAMaker

記錄 QnA Maker 服務的結果。

- UserID ([從遙測初始設定式](https://aka.ms/telemetry-initializer))
- SessionID ([從遙測初始設定式](https://aka.ms/telemetry-initializer))
- ActivityID ([從遙測初始設定式](https://aka.ms/telemetry-initializer))
- Channel ([從遙測初始設定式](https://aka.ms/telemetry-initializer))
- ActivityType  ([從遙測初始設定式](https://aka.ms/telemetry-initializer))
- Username (PII 選用)
- Question (PII 選用)
- MatchedQuestion
- QuestionId
- Answer
- Score
- ArticleFound

