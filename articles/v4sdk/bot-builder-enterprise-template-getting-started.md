---
title: 企業 Bot 範本部署 | Microsoft Docs
description: 了解如何針對企業 Bot 部署所有支援的 Azure 資源
author: darrenj
ms.author: darrenj
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 09/18/2018
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: e888b2473269cf576fd9edda0d99a30aa6212f7b
ms.sourcegitcommit: b2245df2f0a18c5a66a836ab24a573fd70c7d272
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 03/07/2019
ms.locfileid: "57571853"
---
# <a name="enterprise-bot-template---getting-started"></a>Enterprise Bot Template - 使用者入門

> [!NOTE]
> 本主題適用於 SDK 的 v4 版本。 

## <a name="prerequisites"></a>必要條件

安裝下列項目：
- [Enterprise Bot Template VSIX](https://marketplace.visualstudio.com/items?itemName=BotBuilder.botbuilderv4enterprise)
- [.NET Core SDK](https://www.microsoft.com/net/download) (最新版本)
- [Node 套件管理員](https://nodejs.org/en/)
- [Bot Framework Emulator](https://docs.microsoft.com/en-us/azure/bot-service/bot-service-debug-emulator?view=azure-bot-service-4.0) (最新版本)
- [Azure CLI](https://docs.microsoft.com/en-us/cli/azure/install-azure-cli?view=azure-cli-latest)
- [Azure Bot Service CLI 工具](https://github.com/Microsoft/botbuilder-tools) (最新版本)
    ```shell
    npm install -g ludown luis-apis qnamaker botdispatch msbot chatdown
    ```
- [LuisGen](https://github.com/Microsoft/botbuilder-tools/blob/master/packages/LUISGen/src/npm/readme.md)
    ```shell
    dotnet tool install -g luisgen
    ```

## <a name="create-your-bot-project"></a>建立 Bot 專案
1. 在 Visual Studio 中，按一下 [檔案] > [新增專案]。
1. 在 [Bot] 下，選取 [Enterprise Bot Template]。

![提供新增專案範本](media/enterprise-template/new_project.jpg)

1. 為您的專案命名，然後按一下 [建立]。
1. 以滑鼠右鍵按一下專案，然後按一下 [建置] 以還原 NuGet 套件。

## <a name="deploy-your-bot"></a>部署您的 Bot

企業範本 Bot 需要下列 Azure 服務以便進行端對端作業：
- Azure Web 應用程式
- Azure 儲存體帳戶 (文字記錄)
- Azure Application Insights (遙測)
- Azure CosmosDb (對話狀態儲存體)
- Azure 認知服務 - Language Understanding
- Azure 認知服務 - QnA Maker (包括 Azure 搜尋服務、Azure Web 應用程式)

下列步驟將協助您使用提供的部署指令碼部署這些服務：

1. 擷取您的 LUIS 撰寫金鑰。
   - 檢閱[這個](https://docs.microsoft.com/en-us/azure/cognitive-services/luis/luis-reference-regions)文件頁面，以取得部署區域的正確 LUIS 入口網站。 請注意， www.luis.ai 涉及美國地區，而從這個入口網站擷取的撰寫金鑰不適用於歐洲部署。
   - 登入後，按一下右上角您的名稱。
   - 選擇 [設定]，並記下撰寫金鑰，以供稍後使用。
    
    ![LUIS 撰寫金鑰螢幕擷取畫面](./media/enterprise-template/luis_authoring_key.jpg)

1. 開啟命令提示字元。
1. 使用 Azure CLI 登入您的 Azure 帳戶。 您可以在 Azure 入口網站[訂用帳戶](https://ms.portal.azure.com/#blade/Microsoft_Azure_Billing/SubscriptionsBlade)頁面中，找到您有權存取的訂用帳戶清單。
    ```shell
    az login
    az account set --subscription "YOUR_SUBSCRIPTION_NAME"
    ```

1. 執行 msbot 複製服務命令，以在您的專案中部署服務和設定 .bot 檔案。 **附註：完成部署之後，您必須記下顯示在命令提示字元視窗的 bot 檔案密碼，以供稍後使用。**

    ```shell
    msbot clone services --name "YOUR-BOT-NAME" --luisAuthoringKey "YOUR_AUTHORING_KEY" --folder "DeploymentScripts\LOCALE_FOLDER" --location "REGION"
    ```

    > **注意**：
    >- **YOUR-BOT-NAME** 參數必須是**全域唯一**，並且只能包含小寫字母、數字和連字號 ("-")。
    >- 請確定您在此步驟中提供的部署區域，與您的 LUIS 撰寫金鑰入口網站相符 (例如，luis.ai 的 westus 或 eu.luis.ai 的 westeurope)。
    >- 某些使用者在佈建 MSA AppId 和密碼時，可能會遇到一個已知問題。 如果您收到此錯誤`ERROR: Unable to provision MSA id automatically. Please pass them in as parameters and try again`，請前往 [https://apps.dev.microsoft.com](https://apps.dev.microsoft.com) 並以手動方式建立新的應用程式、記下 AppId 和密碼/祕密。 再次執行上述 msbot 複製服務命令，提供兩個新引數 `--appId` 和 `--appSecret` 以及剛才擷取的值。 您可能需要將密碼中任何特殊字元逸出，因為這些特殊字元可能會被殼層解譯為命令：
    >   - 針對 *Windows 命令提示字元*，使用雙引號括住 appSecret。 例如，`msbot clone services --name xxxx --luisAuthoringKey xxxx --location xxxx --folder bot.recipe --appSecret "YOUR_APP_SECRET"`
    >   - 針對 * Windows PowerShell，嘗試在 --% 引數之後傳入 appSecret。 例如，`msbot clone services --name xxxx --luisAuthoringKey xxxx --location xxxx --folder bot.recipe --% --appSecret "YOUR_APP_SECRET"`
    >   - 針對 *MacOS 或 Linux*，使用單引號括住 appSecret。 例如，`msbot clone services --name xxxx --luisAuthoringKey xxxx --location xxxx --folder bot.recipe --appSecret 'YOUR_APP_SECRET'`

1. 完成部署之後，更新 **appsettings.json** 與您的 Bot 檔案祕密。 
    
    ```
    "botFilePath": "./YOUR_BOT_FILE.bot",
    "botFileSecret": "YOUR_BOT_SECRET",
    ```
1. 執行下列命令，擷取您 Application Insights 執行個體的 InstrumentationKey。
    ```
    msbot list --bot YOUR_BOT_FILE.bot --secret "YOUR_BOT_SECRET"
    ```

1. 更新**appsettings.json** 與您的檢測金鑰：

    ```
    "ApplicationInsights": {
        "InstrumentationKey": "YOUR_INSTRUMENTATION_KEY"
    }
    ```

## <a name="test-your-bot"></a>測試 Bot

完成後，請在開發環境中執行您的 Bot 專案，並開啟 **Bot Framework Emulator**。 在模擬器中，按一下 [檔案] > [開啟 Bot]，並瀏覽至您目錄中的 .bot 檔案。

![模擬器開發端點螢幕擷取畫面](./media/enterprise-template/development_endpoint.jpg)

當對話開始時，應該會收到簡介訊息。

輸入 ```hi``` 來確認一切運作正常。

## <a name="publish-your-bot"></a>發佈您的 Bot

您可以在本機端執行對端測試。 當您準備好將 Bot 部署至 Azure 進行額外測試時，您可以使用下列命令來發行原始程式碼：

```shell
az bot publish -g YOUR-BOT-NAME -n YOUR-BOT-NAME --proj-name YOUR-BOT-NAME.csproj --version v4
```

## <a name="view-your-bot-analytics"></a>檢視您的 Bot 分析
Enterprise Bot Template 隨附預先設定的 Power BI 儀表板，該儀表板連接至 Application Insights 服務以提供對話分析。 當您在本地測試 Bot 之後，就可以開啟此儀表板檢視資料。 

1. 在[此處](https://github.com/Microsoft/AI/blob/master/solutions/analytics/ConversationalAnalyticsSample_02132019.pbit)下載 Power BI 儀表板 (.pbit 檔案)。
1. 在 [Power BI Desktop](https://powerbi.microsoft.com/en-us/desktop/) 中開啟儀表板。
1. 輸入您的 Application Insights 應用程式識別碼 (位於 .bot 檔案中)。

    ![可以在 Bot 檔案中的哪裡找到 AppInsights 應用程式識別碼](./media/enterprise-template/appInsights_appId.jpg)

1. 在[此處](https://github.com/Microsoft/AI/tree/master/solutions/analytics)深入了解 Power BI 儀表板功能。

# <a name="next-steps"></a>後續步驟

成功部署現成的 Bot 之後，您可以針對案例和需求自訂 Bot。 繼續[自訂 Bot](bot-builder-enterprise-template-customize.md)。
