---
title: 部署您的 Bot | Microsoft Docs
description: 將您的 Bot 部署至 Azure 雲端。
keywords: 部署 bot, azure 部署 bot, 發佈 bot
author: ivorb
ms.author: v-ivorb
manager: kamrani
ms.topic: get-started-article
ms.service: bot-service
ms.subservice: abs
ms.date: 02/13/2019
ms.openlocfilehash: 53889703d58983a87a7a2d16622f1298d56c87db
ms.sourcegitcommit: 05ddade244874b7d6e2fc91745131b99cc58b0d6
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 02/21/2019
ms.locfileid: "56591026"
---
# <a name="deploy-your-bot"></a>部署您的 Bot

[!INCLUDE [pre-release-label](./includes/pre-release-label.md)]

建立 Bot 並在本機測試後，您可以將其部署至 Azure，以便從任何地方進行存取。 將 Bot 部署至 Azure 時，需支付您所使用的服務。 [計費與成本管理](https://docs.microsoft.com/en-us/azure/billing/)一文協助您了解 Azure 計費方式、監視使用量和成本，以及管理帳戶和訂用帳戶。

在本文中，我們會示範如何將 C# 和 JavaScript Bot 部署至 Azure。 在依照步驟執行前閱讀本文很有用，您可完全了解部署 Bot 的相關事項。

## <a name="prerequisites"></a>必要條件

- 安裝最新版的 [msbot](https://github.com/Microsoft/botbuilder-tools/tree/master/packages/MSBot) 工具。
- 您已在本機電腦上開發的 [CSharp](./dotnet/bot-builder-dotnet-sdk-quickstart.md) 或 [JavaScript](./javascript/bot-builder-javascript-quickstart.md) Bot。

## <a name="1-prepare-for-deployment"></a>1.準備部署
若要進行部署程序，Azure 中必須要有作為目標的 Web 應用程式 Bot，以便能在其中部署您的本機 Bot。 本機 Bot 會使用作為目標的 Web 應用程式 Bot 和使用此 Bot 在 Azure 中佈建的資源來進行部署。 這些是必要資源，因為本機 Bot 並不會佈建所有必要的 Azure 資源。 在建立作為目標的 Web 應用程式 Bot 時，系統會為您佈建下列資源：
-   Web 應用程式 Bot – 您會使用此 Bot 以便將本機 Bot 部署到其中。
-   App Service 方案 - 會提供 App Service 應用程式執行所需的資源。
-   App Service - 用於裝載 Web 應用程式的服務
-   儲存體帳戶 - 包含您所有的 Azure 儲存體資料物件：Blob、檔案、佇列、資料表和磁碟。

在建立作為目標的 Web 應用程式 Bot 期間，系統也會為您的 Bot 產生應用程式識別碼和密碼。 在 Azure 中，應用程式識別碼和密碼可支援[服務驗證和授權](https://docs.microsoft.com/azure/app-service/overview-authentication-authorization)。 您會擷取此資訊的某些資料，以便在本機 Bot 程式碼中使用。 

> [!IMPORTANT]
> 服務的 Bot 範本語言必須符合 Bot 的撰寫語言。

如果您已在 Azure 中建立想要使用的 Bot，則是否要建立新 Web 應用程式 Bot 的選擇權在您。

1. 登入 [Azure 入口網站](https://portal.azure.com)。
1. 按一下 Azure 入口網站左上角的 [建立新資源] 連結，然後選取 [AI + 機器學習服務] > [Web 應用程式 Bot]。
1. 隨即開啟含有 Web 應用程式 Bot 相關資訊的新刀鋒視窗。 
1. 在 [Bot 服務] 刀鋒視窗中，提供有關 Bot 的必要資訊。
1. 按一下 [建立] 以建立服務，並將 Bot 部署到雲端。 此程序可能需要幾分鐘的時間。

### <a name="download-the-source-code"></a>下載原始程式碼
在建立作為目標的 Web 應用程式 Bot 之後，您必須從 Azure 入口網站將 Bot 程式碼下載到本機電腦。 下載程式碼的原因是為了取得 [.bot 檔案](./v4sdk/bot-file-basics.md)中的服務參考。 這些服務參考適用於 Web 應用程式 Bot、App Service 方案、App Service 和儲存體帳戶。 

1. 在 [Bot 管理] 區段中，按一下 [組建]。
1. 在右窗格中按一下**下載 Bot 原始程式碼**連結。
1. 遵循提示以下載程式碼，然後將資料夾解壓縮。

### <a name="decrypt-the-bot-file"></a>將 .bot 檔案解密

您從 Azure 入口網站下載的原始程式碼包含已加密的 .bot 檔案。 您必須將其解密，以將其值複製到您的本機 .bot 檔案。 這是必要步驟，可讓您複製實際的服務參考而非加密版本。  

1. 在 Azure 入口網站中，為您的 Bot 開啟 Web 應用程式 Bot 資源。
1. 開啟 Bot 的 [應用程式設定]。
1. 在 [應用程式設定] 視窗中，向下捲動至 [應用程式設定]。
1. 找出 **botFileSecret** 並複製其值。

使用 `msbot cli` 將檔案解密。

```cmd
msbot secret --bot <name-of-bot-file> --secret "<bot-file-secret>" --clear
```

### <a name="update-your-local-bot-file"></a>更新本機 .bot 檔案

開啟您已解密的 .bot 檔案。 複製 `services` 區段底下所列出的**全部**項目，然後將其新增至您的本機 .bot 檔案。 解決任何重複的服務項目或重複的服務識別碼。 Bot 所仰賴的其他服務參考則全都保留下來。 例如︰

```json
"services": [
    {
        "type": "abs",
        "tenantId": "<tenant-id>",
        "subscriptionId": "<subscription-id>",
        "resourceGroup": "<resource-group-name>",
        "serviceName": "<bot-service-name>",
        "name": "<friendly-service-name>",
        "id": "1",
        "appId": "<app-id>"
    },
    {
        "type": "blob",
        "connectionString": "<connection-string>",
        "tenantId": "<tenant-id>",
        "subscriptionId": "<subscription-id>",
        "resourceGroup": "<resource-group-name>",
        "serviceName": "<blob-service-name>",
        "id": "2"
    },
    {
        "type": "endpoint",
        "appId": "",
        "appPassword": "",
        "endpoint": "<local-endpoint-url>",
        "name": "development",
        "id": "3"
    },
    {
        "type": "endpoint",
        "appId": "<app-id>",
        "appPassword": "<app-password>",
        "endpoint": "<hosted-endpoint-url>",
        "name": "production",
        "id": "4"
    },
    {
        "type": "appInsights",
        "instrumentationKey": "<instrumentation-key>",
        "applicationId": "<appinsights-app-id>",
        "apiKeys": {},
        "tenantId": "<tenant-id>",
        "subscriptionId": "<subscription-id>",
        "resourceGroup": "<resource-group>",
        "serviceName": "<appinsights-service-name>",
        "id": "5"
    }
],
```

儲存檔案。

### <a name="setup-a-repository"></a>設定存放庫

若要支援持續部署，請使用慣用的 git 原始檔控制提供者建立 git 存放庫。 將程式碼認可至存放庫。 

確定您的存放庫根目錄具有正確檔案，如[準備存放庫](https://docs.microsoft.com/azure/app-service/deploy-continuous-deployment#prepare-your-repository)底下所述。

### <a name="update-app-settings-in-azure"></a>在 Azure 中更新應用程式設定
本機 Bot 不會使用加密的 .bot 檔案，但 Azure 入口網站則設定為使用加密的 .bot 檔案。 
1. 在 Azure 入口網站中，為您的 Bot 開啟 [Web 應用程式 Bot] 資源。
1. 開啟 Bot 的 [應用程式設定]。
1. 在 [應用程式設定] 視窗中，向下捲動至 [應用程式設定]。
1. 找出 **botFileSecret** 並加以刪除。
1. 更新 Bot 檔案名稱，以符合您簽入存放庫的檔案。
1. 儲存變更。

## <a name="2-deploy-using-azure-deployment-center"></a>2.使用 Azure 部署中心進行部署

現在，您需要將 Bot 程式碼上傳至 Azure。 請遵循[持續部署至 Azure App Service](https://docs.microsoft.com/azure/app-service/deploy-continuous-deployment)主題中的指示來進行。

請注意，建議使用 `App Service Kudu build server` 建置。

在設定好持續部署後，系統便會將您已認可至存放庫的變更發行出去。 不過，如果您將服務新增至 Bot，則需要將這些服務的項目新增至 .bot 檔案中。

## <a name="3-test-your-deployment"></a>3.測試您的部署

在成功部署後等候幾秒，選擇性地重新啟動您的 Web 應用程式，以清除任何快取。 回到您的 Web 應用程式 Bot 刀鋒視窗，使用 Azure 入口網站中提供的網路聊天進行測試。

## <a name="additional-resources"></a>其他資源

- [如何調查連續部署的常見問題](https://github.com/projectkudu/kudu/wiki/Investigating-continuous-deployment)

<!--

## Prerequisites

[!INCLUDE [prerequisite snippet](~/includes/deploy/snippet-prerequisite.md)]


## Deploy JavaScript and C# bots using az cli

You've already created and tested a bot locally, and now you want to deploy it to Azure. These steps assume that you have created the required Azure resources.

[!INCLUDE [az login snippet](~/includes/deploy/snippet-az-login.md)]

### Create a Web App Bot

If you don't already have a resource group to which to publish your bot, create one:

[!INCLUDE [az create group snippet](~/includes/deploy/snippet-az-create-group.md)]

[!INCLUDE [az create web app snippet](~/includes/deploy/snippet-create-web-app.md)]

Before proceeding, read the instructions that apply to you based on the type of email account you use to log in to Azure.

#### MSA email account

If you are using an [MSA](https://en.wikipedia.org/wiki/Microsoft_account) email account, you will need to create the app ID and app password on the Application Registration Portal to use with `az bot create` command.

[!INCLUDE [create bot msa snippet](~/includes/deploy/snippet-create-bot-msa.md)]

#### Business or school account

[!INCLUDE [create bot snippet](~/includes/deploy/snippet-create-bot.md)]

### Download the bot from Azure

Next, download the bot you just created. 
[!INCLUDE [download bot snippet](~/includes/deploy/snippet-download-bot.md)]

### Decrypt the downloaded .bot file and use in your project

The sensitive information in the .bot file is encrypted.

[!INCLUDE [decrypt bot snippet](~/includes/deploy/snippet-decrypt-bot.md)]

### Update the .bot file

If your bot uses LUIS, QnA Maker, or Dispatch services, you will need to add references to them to your .bot file. Otherwise, you can skip this step.

1. Open your bot in the BotFramework Emulator, using the new .bot file. The bot does not need to be running locally.
1. In the **BOT EXPLORER** panel, expand the **SERVICES** section.
1. To add references to LUIS apps, click the plus-sign (+) to the right of **SERVICES**.
   1. Select **Add Language Understanding (LUIS)**.
   1. If it prompts you to log into your Azure account, do so.
   1. It presents a list of LUIS applications you have access to. Select the ones for your bot.
1. To add references to a QnA Maker knowledge base, click the plus-sign (+) to the right of **SERVICES**.
   1. Select **Add QnA Maker**.
   1. If it prompts you to log into your Azure account, do so.
   1. It presents a list of knowledge bases you have access to. Select the ones for your bot.
1. To add references to Dispatch models, click the plus-sign (+) to the right of **SERVICES**.
   1. Select **Add Dispatch**.
   1. If it prompts you to log into your Azure account, do so.
   1. It presents a list of Dispatch models you have access to. Select the ones for your bot.

### Test your bot locally

At this point, your bot should work the same way it did with the old .bot file. Make sure that it works as expected with the new .bot file.

### Publish your bot to Azure

[!INCLUDE [publish snippet](~/includes/deploy/snippet-publish.md)]


[!INCLUDE [clear encryption snippet](~/includes/deploy/snippet-clear-encryption.md)]

## Additional resources

[!INCLUDE [additional resources snippet](~/includes/deploy/snippet-additional-resources.md)]

## Next steps
> [!div class="nextstepaction"]
> [Set up continous deployment](bot-service-build-continuous-deployment.md)

-->
