---
title: 企業 Bot 自訂 | Microsoft Docs
description: 了解如何自訂企業範本所建立的 Bot
author: darrenj
ms.author: darrenj
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 09/18/2018
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 6fc7f73d406c1bbbc2b2671c9336df6bda10ade6
ms.sourcegitcommit: 87b5b0ca9b0d5e028ece9f7cc4948c5507062c2b
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "47029756"
---
# <a name="enterprise-bot-template---customize-your-bot"></a>企業 Bot 範本 - 自訂您的 Bot

> [!NOTE]
> 本主題適用於 SDK 的 v4 版本。 

## <a name="net"></a>.NET
在您依照[這裡](bot-builder-enterprise-template-deployment.md)所列的指示，部署及測試企業 Bot 範本能否端對端運作之後，即可根據您的案例和需求輕鬆地自訂 Bot。 此範本的目標是要提供穩固的基礎來打造對話式體驗。

## <a name="project-structure"></a>專案結構

Bot 的資料夾結構如下所示，且代表我們建議的最佳做法，可供建構 Bot 專案及處理傳入訊息。

    | - YourBot.bot         // The .bot file containing all of your Bot configuration including dependencies
    | - README.md           // README file containing links to documentation
    | - Program.cs          // Default Program.cs file
    | - Startup.cs          // Core Bot Initialisation including Bot Configuration LUIS, Dispatcher, etc. 
    | - <BOTNAME>State.cs   // The Root State class for your Bot
    | - appsettings.json    // References above .bot file for Configuration information. App Insights key
    | - CognitiveModels     
        | - LUIS            // .LU file containing base conversational intents (Greeting, Help, Cancel)
        | - QnA             // .LU file containing example QnA items
    | - DeploymentScripts   // msbot clone recipe for deployment
    | - Dialogs             // All Bot dialogs sit under this folder
        | - Main            // Root Dialog for all messages
            | - MainDialog.cs       // Dialog Logic
            | - MainResponses.cs    // Dialog responses
            | - Resources           // Adaptive Card JSON, Resource File
        | - Onboarding
            | - OnboardingDialog.cs       // Onboarding dialog Logic
            | - OnboardingResponses.cs    // Onboarding dialog responses
            | - OnboardingState.cs        // Localised dialog state
            | - Resources                 Resource File
        | - Cancel
        | - Escalate
        | - Signin
    | - Middleware          // Telemetry, Content Moderator
    | - ServiceClients      // SDK libraries, example GraphClient provided for Auth example
   
## <a name="update-introduction-message"></a>更新簡介訊息

簡介訊息會使用[調適型卡片](https://www.adaptivecards.io)。 若要為 Bot 自訂此簡介，您可以在 Dialogs/Main/Resources 資料夾中尋找名為 ```Intro.json``` 的 JSON 檔案。 使用[調適型卡片視覺化檢視](http://adaptivecards.io/visualizer)來修改調適型卡片，以符合您的 Bot 需求。

## <a name="update-bot-responses"></a>更新 Bot 回應

專案內的每個對話都有一組儲存在支援資源 (.resx) 檔案內的回應。 在每個對話底下的 [資源] 資料夾中可以找到這些檔案。

您可以在 Visual Studio 資源編輯器中變更回應 (如下所示)，以調整 Bot 的回應方式。

![自訂 Bot 回應](media/enterprise-template/EnterpriseBot-CustomisingResponses.png)

這個方法使用標準資源檔案當地語系化方法，支援多語系回應。 在[這裡](https://docs.microsoft.com/en-us/aspnet/core/fundamentals/localization?view=aspnetcore-2.1)可以找到進一步資訊。

## <a name="updating-your-cognitive-models"></a>更新您的認知模型

根據預設，企業範本隨附兩個認知模型：範例常見問題集 QnAMaker 知識庫，以及適用於一般意圖 (問候語、協助、取消等等) 的 LUIS 模型。 您可以自訂這些模型，以符合您的需求。 您也可以新增 LUIS 模型和 QnAMaker 知識庫，以擴充您的 Bot 功能。

### <a name="updating-an-existing-luis-model"></a>更新現有的 LUIS 模型
若要更新企業範本現有的 LUIS 模型，請執行下列步驟：
1. 在 [LUIS 入口網站](http://luis.ai)中或使用 [LuDown](https://github.com/Microsoft/botbuilder-tools/tree/master/packages/Ludown) 和 [Luis](https://github.com/Microsoft/botbuilder-tools/tree/master/packages/LUIS) CLI 工具，對 LUIS 模型進行變更。 
2. 執行下列命令來更新分派模型，以反映您的變更 (確保正確的訊息路由)：
```shell
    dispatch refresh --bot "YOURBOT.bot" --secret YOURSECRET
```
3. 從每個已更新模型的專案根目錄執行下列命令，以更新其相關聯的 LuisGen 類別： 
```shell
    luis export version --appId [LUIS_APP_ID] --versionId [LUIS_APP_VERSION] --authoringKey [YOUR_LUIS_AUTHORING_KEY] | luisgen - -cs [CS_FILE_NAME] -o "\Dialogs\Shared\Resources"
```

### <a name="updating-an-existing-qnamaker-knowledge-base"></a>更新現有的 QnAMaker 知識庫
若要更新現有的 QnAMaker 知識庫，請執行下列步驟：
1. 透過 [LuDown](https://github.com/Microsoft/botbuilder-tools/tree/master/packages/Ludown) 和 [QnAMaker](https://github.com/Microsoft/botbuilder-tools/tree/master/packages/QnAMaker) CLI 工具或 [QnAMaker 入口網站](https://qnamaker.ai)，對您的 QnAMaker 知識庫進行變更。
2. 執行下列命令來更新分派模型，以反映您的變更 (確保正確的訊息路由)：
```shell
    dispatch refresh --bot "YOURBOT.bot" --secret YOURSECRET
```

### <a name="adding-a-new-luis-model"></a>新增 LUIS 模型

在您要將 LUIS 模型新增至您的專案的案例中，您必須更新 Bot 組態和發送器，以確保其知道其他模型。 
1. 透過 LuDown/LUIS CLI 工具或透過 LUIS 入口網站建立 LUIS 模型
2. 執行下列命令，將您的 LUIS 應用程式連線到 .bot 檔案：
```shell
    msbot connect luis --appId [LUIS_APP_ID] --authoringKey [LUIS_AUTHORING_KEY] --subscriptionKey [LUIS_SUBSCRIPTION_KEY] 
```
3. 透過下列命令將這個新的 LUIS 模型新增至您的發送器
```shell
    dispatch add -t luis -id YOUR_LUIS_APPID -bot "YOURBOT.bot" -secret YOURSECRET
```
4. 重新整理分派模型，以透過下列命令反映 LUIS 模型變更
```shell
    dispatch refresh -bot "YOURBOT.bot" -secret YOURSECRET
```

## <a name="adding-a-new-dialog"></a>新增對話方塊 

若要將對話方塊新增至 Bot，您必須先在對話方塊底下建立新的資料夾，並確定此類別衍生自 `EnterpriseDialog`。 接著，您必須連接對話方塊基礎結構。 上線對話方塊會顯示一個簡單範例供您參考，而底下會顯示摘錄以及步驟概觀。

- 將瀑布圖對話方塊新增至您的建構函式
- 定義瀑布圖的步驟
- 建立您的瀑布圖步驟
- 呼叫 AddDialog 傳遞您的瀑布圖
- 呼叫 AddDialog 傳遞您在瀑布圖中使用的任何提示
- 將 InitialDialogId 設定為您希望元件執行的第一個對話方塊

```
InitialDialogId = nameof(OnboardingDialog);

var onboarding = new WaterfallStep[]
{
    AskForName,
    AskForEmail,
    AskForLocation,
    FinishOnboardingDialog,
};

AddDialog(new WaterfallDialog(InitialDialogId, onboarding));
AddDialog(new TextPrompt(NamePrompt));
AddDialog(new TextPrompt(EmailPrompt));
AddDialog(new TextPrompt(LocationPrompt));
```

然後，您必須建立範本管理員來處理回應。 建立新的類別並從 TemplateManager 衍生，OnboardingResponses.cs 檔案中會提供範例，而摘錄如下所示。

```
public const string _namePrompt = "namePrompt";
public const string _haveName = "haveName";
public const string _emailPrompt = "emailPrompt";
      
private static LanguageTemplateDictionary _responseTemplates = new LanguageTemplateDictionary
{
    ["default"] = new TemplateIdMap
    {
        {
            _namePrompt,
            (context, data) => OnboardingStrings.NAME_PROMPT
        },
        {
            _haveName,
            (context, data) => string.Format(OnboardingStrings.HAVE_NAME, data.name)
        },
        {
            _emailPrompt,
            (context, data) => OnboardingStrings.EMAIL_PROMPT
        },
```

若要轉譯回應，您可以透過 `ReplyWith` 或 `RenderTemplate` 提示使用範本管理員執行個體來存取這些回應。 範例如下所示。

```
Prompt = await _responder.RenderTemplate(sc.Context, "en", OnboardingResponses._namePrompt),
await _responder.ReplyWith(sc.Context, OnboardingResponses._haveName, new { name });
```

對話方塊基礎結構的最後一步，是建立範圍僅限於您對話方塊的狀態類別。 建立新的類別，並確保其衍生自 `DialogState`

對話方塊完成後，您必須使用 `AddDialog`，將對話方塊新增至 `MainDialog` 元件。 若要使用新的對話方塊，請從 `RouteAsync` 方法呼叫 `dc.BeginDialogAsync()`，如有需要，請使用適當的 LUIS 意圖觸發。

## <a name="conversational-insights-using-powerbi-dashboard-and-application-insights"></a>使用 PowerBI 儀表板和 Application Insights 的對話式見解
- 若要從取得對話式見解著手，請繼續進行[使用 PowerBI 儀表板設定對話式分析](bot-builder-enterprise-template-powerbi.md)。

