---
title: 企業 Bot 範本部署 | Microsoft Docs
description: 了解如何針對企業 Bot 部署所有支援的 Azure 資源
author: darrenj
ms.author: darrenj
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 09/18/2018
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 0f4c5e0db9dae86f81414ccd9bbb1e5de4dce624
ms.sourcegitcommit: b8bd66fa955217cc00b6650f5d591b2b73c3254b
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/15/2018
ms.locfileid: "49326395"
---
# <a name="enterprise-bot-template---deploying-your-bot"></a>企業 Bot 範本 - 部署您的 Bot

> [!NOTE]
> 本主題適用於 SDK 的 v4 版本。 

## <a name="prerequisites"></a>必要條件

- 確定已安裝[節點套件管理員](https://nodejs.org/en/)。

- 安裝 Azure Bot 服務命令列 (CLI) 工具。 即使您之前已使用此工具，請務必這麼做，以確保擁有最新版本。

```shell
npm install -g ludown luis-apis qnamaker botdispatch msbot luisgen chatdown
```

- 從[這裡](https://docs.microsoft.com/en-us/cli/azure/install-azure-cli-windows?view=azure-cli-latest)安裝 Azure 命令列工具 (CLI)

- 安裝適用於 Bot 服務的 AZ 擴充功能
```shell
az extension add -n botservice
```

## <a name="configuration"></a>組態

- 擷取您的 LUIS 撰寫金鑰
   - 檢閱[這個](https://docs.microsoft.com/en-us/azure/cognitive-services/luis/luis-reference-regions)文件頁面，以取得您打算部署至區域的正確 LUIS 入口網站。 
   - 登入後，按一下右上角您的名稱。
   - 選擇 [設定]，並記下編寫索引鍵，以供下一個步驟使用。

## <a name="deployment"></a>部署

>如果您有多個 Azure 訂用帳戶，並且想要確保部署選取正確的訂用帳戶，請先執行下列命令再繼續。

 遵循瀏覽器登入程序來登入您的 Azure 帳戶
```shell
az login
az account list
az account set --subscription "YOUR_SUBSCRIPTION_NAME"
```

企業範本 Bot 需要下列相依性以便進行端對端作業。
- Azure Web 應用程式
- Azure 儲存體帳戶 (文字記錄)
- Azure Application Insights (遙測)
- Azure CosmosDb (狀態)
- Azure 認知服務 - Language Understanding
- Azure 認知服務 - QnAMaker (包括 Azure 搜尋服務、Azure Web 應用程式)
- Azure 認知服務 - 內容審查工具 (選用手動步驟)

新的 Bot 專案具有一個部署配方，可讓 `msbot clone services` 命令自動將上述所有服務部署到您的 Azure 訂用帳戶，並確保以所有的服務 (包括能夠讓 Bot 順暢運作的金鑰) 更新您專案中的 .bot 檔案。

> 部署後，請檢閱所建立服務的定價層，並調整為符合您的案例。

所建專案中的 README.md 包含以您建立的 Bot 名稱更新的範例 msbot 複製服務命令列，泛型版本如下所示。 確保您會更新來自上一個步驟的撰寫金鑰，並選擇您想要使用的 Azure 資料中心位置 (例如 westus 或 westeurope)。

> 確保在上一個步驟擷取的 LUIS 撰寫金鑰可用於您在下面指定的區域。

```shell
msbot clone services --name "YOUR_BOT_NAME" --luisAuthoringKey "YOUR_AUTHORING_KEY" --folder "DeploymentScripts\msbotClone" --location "westus"
```

msbot 工具會概述部署計畫，包括位置和 SKU。 確保在繼續前進行檢閱。

![部署確認](./media/enterprise-template/EnterpriseBot-ConfirmDeployment.png)

>部署完成之後，您**必須**記下所提供的 .bot 檔案祕密，因為後續步驟需要此祕密。

- 以新建的 .bot 檔案名稱和 .bot 檔案祕密更新 `appsettings.json` 檔案。
- 執行下列命令，擷取您 Application Insights 執行個體的 InstrumentationKey，並更新 `appsettings.json` 檔案中的 InstrumentationKey。

`msbot list --bot YOURBOTFILE.bot --secret YOUR_BOT_SECRET`

        {
          "botFilePath": ".\\YOURBOTFILE.bot",
          "botFileSecret": "YOUR_BOT_SECRET",
          "ApplicationInsights": {
            "InstrumentationKey": "YOUR_INSTRUMENTATION_KEY"
          }
        }

## <a name="testing"></a>測試

完成後，請在開發環境中執行您的 Bot 專案，並開啟 Bot Framework 模擬器。 在模擬器中，從 [檔案] 功能表中選擇 [開啟 Bot]，並瀏覽至您目錄中的 .bot 檔案。

然後輸入 ```hi``` 來確認一切運作正常。

## <a name="deploy-to-azure"></a>部署至 Azure

您可以在本機端執行對端測試。 當您準備好將 Bot 部署至 Azure 進行額外測試時，您可以使用下列命令來發行原始程式碼，每當您想要推送原始程式碼更新時即可執行此原始程式碼。

```shell
az bot publish -g YOUR_BOT_NAME -n YOUR_BOT_NAME --proj-file YOUR_BOT_NAME.csproj --sdk-version v4
```

## <a name="enabling-more-scenarios"></a>啟用更多案例

您的 Bot 專案會提供您可透過下列步驟啟用的額外功能。

### <a name="authentication"></a>驗證

若要啟用驗證，在 Azure 入口網站中您 Bot 的 [設定] 中設定 [驗證連線名稱] 之後，請遵循下列步驟。 您可以在[文件](https://docs.microsoft.com/en-us/azure/bot-service/bot-builder-tutorial-authentication?view=azure-bot-service-3.0)中找到進一步資訊。

在 MainDialog 建構函式中註冊 `SignInDialog`：
    
`AddDialog(new SignInDialog(_services.AuthConnectionName));`

在您的程式碼中所要的位置新增下列內容，來測試簡單登入流程：
    
`var signInResult = await dc.BeginAsync(SignInDialog.Name);`

### <a name="content-moderation"></a>內容仲裁

內容審查可用來識別傳送至 Bot 的訊息中的個人識別資訊 (PII) 和成人內容。 若要啟用這項功能，請前往 Azure 入口網站並建立新的內容審查工具服務。 收集您的訂用帳戶金鑰和區域，以設定您的 .bot 檔案。 

> 這個步驟將在未來自動執行。

一開始就將下列程式碼新增至 service.AddBot<>() 的底部，以便每次啟用內容審查。 透過您的 Bot 狀態可以存取內容審查的結果 
    
```
    // Content Moderation Middleware (analyzes incoming messages for inappropriate content including PII, profanity, etc.)
    var moderatorService = botConfig.Services.Where(s => s.Name == ContentModeratorMiddleware.ServiceName).FirstOrDefault();
    if (moderatorService != null)
    {
        var moderator = moderatorService as GenericService;
        var moderatorKey = moderator.Configuration["subscriptionKey"];
        var moderatorRegion = moderator.Configuration["region"];
        var moderatorMiddleware = new ContentModeratorMiddleware(moderatorKey, moderatorRegion);
        options.Middleware.Add(moderatorMiddleware);
    }
```
從對話堆疊呼叫此程式碼以存取中介軟體結果
```     
    var cm = dc.Context.TurnState.Get<Microsoft.CognitiveServices.ContentModerator.Models.Screen>(ContentModeratorMiddleware.TextModeratorResultKey);
```

## <a name="customize-your-bot"></a>自訂您的 Bot

確認成功部署現成的 Bot 之後，您可以針對您的案例和需求自訂 Bot。 繼續[自訂 Bot](bot-builder-enterprise-template-customize.md)。
