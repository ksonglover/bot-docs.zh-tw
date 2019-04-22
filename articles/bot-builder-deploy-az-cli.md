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
ms.date: 04/12/2019
ms.openlocfilehash: 4532fe55705524573de55017e633289255a20ab9
ms.sourcegitcommit: 721bb09f10524b0cb3961d7131966f57501734b8
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 04/18/2019
ms.locfileid: "59508215"
---
# <a name="deploy-your-bot"></a>部署您的 Bot

[!INCLUDE [pre-release-label](./includes/pre-release-label.md)]

建立 Bot 並在本機測試後，您可以將其部署至 Azure，以便從任何地方進行存取。 將 Bot 部署至 Azure 時，需支付您所使用的服務。 [計費與成本管理](https://docs.microsoft.com/en-us/azure/billing/)一文協助您了解 Azure 計費方式、監視使用量和成本，以及管理帳戶和訂用帳戶。

在本文中，我們會示範如何將 C# 和 JavaScript Bot 部署至 Azure。 在依照步驟執行前閱讀本文很有用，您可完全了解部署 Bot 的相關事項。

## <a name="prerequisites"></a>必要條件
- 如果您沒有 Azure 訂用帳戶，請在開始前建立 [免費帳戶](http://portal.azure.com)。
- 您已在本機電腦上開發的 [**CSharp**](./dotnet/bot-builder-dotnet-sdk-quickstart.md) 或 [**JavaScript**](./javascript/bot-builder-javascript-quickstart.md) Bot。

## <a name="1-prepare-for-deployment"></a>1.準備部署
若要進行部署程序，Azure 中必須要有作為目標的 Web 應用程式 Bot，以便能在其中部署您的本機 Bot。 本機 Bot 會使用作為目標的 Web 應用程式 Bot 和使用此 Bot 在 Azure 中佈建的資源來進行部署。 這些是必要資源，因為本機 Bot 並不會佈建所有必要的 Azure 資源。 在建立作為目標的 Web 應用程式 Bot 時，系統會為您佈建下列資源：
-   Web 應用程式 Bot – 您會使用此 Bot 以便將本機 Bot 部署到其中。
-   App Service 方案 - 會提供 App Service 應用程式執行所需的資源。
-   App Service - 用於裝載 Web 應用程式的服務
-   儲存體帳戶 - 包含您所有的 Azure 儲存體資料物件：Blob、檔案、佇列、資料表和磁碟。

在建立作為目標的 Web 應用程式 Bot 期間，系統也會為您的 Bot 產生應用程式識別碼和密碼。 在 Azure 中，應用程式識別碼和密碼可支援[服務驗證和授權](https://docs.microsoft.com/azure/app-service/overview-authentication-authorization)。 您會擷取此資訊的某些資料，以便在本機 Bot 程式碼中使用。 

> [!IMPORTANT]
> Azure 入口網站中所用 Bot 範本的程式設計語言必須符合 Bot 的撰寫語言。

如果您已在 Azure 中建立想要使用的 Bot，則是否要建立新 Web 應用程式 Bot 的選擇權在您。

1. 登入 [Azure 入口網站](https://portal.azure.com)。
1. 按一下 Azure 入口網站左上角的 [建立新資源] 連結，然後選取 [AI + 機器學習服務] > [Web 應用程式 Bot]。
1. 隨即開啟含有 Web 應用程式 Bot 相關資訊的新刀鋒視窗。 
1. 在 [Bot 服務] 刀鋒視窗中，提供有關 Bot 的必要資訊。
1. 按一下 [建立] 以建立服務，並將 Bot 部署到雲端。 此程序可能需要幾分鐘的時間。

### <a name="download-the-source-code"></a>下載原始程式碼
在建立作為目標的 Web 應用程式 Bot 之後，您必須從 Azure 入口網站將 Bot 程式碼下載到本機電腦。 下載程式碼的原因是要取得 appsettings.json 或 .env 檔案中的服務參考 (例如 MicrosoftAppID、MicrosoftAppPassword、LUIS 或 QnA)。 

1. 在 [Bot 管理] 區段中，按一下 [組建]。
1. 在右窗格中按一下**下載 Bot 原始程式碼**連結。
1. 遵循提示以下載程式碼，然後將資料夾解壓縮。
    1. [!INCLUDE [download keys snippet](~/includes/snippet-abs-key-download.md)]

### <a name="update-your-local-appsettingsjson-or-env-file"></a>更新本機 appsettings.json 或 .env 檔案

開啟您下載的 appsettings.json 或 .env 檔案。 複製其中所列的**所有**項目並將其新增到「本機」appsettings.json 或 .env 檔案。 解決任何重複的服務項目或重複的服務識別碼。 Bot 所仰賴的其他服務參考則全都保留下來。

儲存檔案。

### <a name="update-local-bot-code"></a>更新本機 Bot 程式碼
將本機 Startup.cs 或 index.js 檔案更新為使用 appsettings.json 或 .env 檔案，而非使用 .bot 檔案。 .bot 檔案已淘汰，而我們正著手更新 VSIX 範本、Yeoman 產生器、範例，以及所有使用 appsettings.json 或 .env 檔案 (而非 .bot 檔案) 的其餘文件。 您同時必須對 Bot 程式碼進行變更。 

從 appsettings.json 或 .env 檔案將程式碼更新為讀取設定。 

# <a name="ctabcsharp"></a>[C#](#tab/csharp)
在 `ConfigureServices` 方法中，使用 ASP.NET Core 提供的組態物件，例如： 

**Startup.cs**
```csharp
var appId = Configuration.GetSection("MicrosoftAppId").Value;
var appPassword = Configuration.GetSection("MicrosoftAppPassword").Value;
options.CredentialProvider = new SimpleCredentialProvider(appId, appPassword);
```

# <a name="jstabjs"></a>[JS](#tab/js)

在 JavaScript 中，參考 `process.env` 物件外的 .env 變數，例如：
   
**index.js**

```js
const adapter = new BotFrameworkAdapter({
    appId: process.env.MicrosoftAppId,
    appPassword: process.env.MicrosoftAppPassword
});
```
---

- 儲存檔案並測試您的 Bot。

### <a name="setup-a-repository"></a>設定存放庫

若要支援持續部署，請使用慣用的 git 原始檔控制提供者建立 git 存放庫。 將程式碼認可至存放庫。

確定您的存放庫根目錄具有正確檔案，如[準備存放庫](https://docs.microsoft.com/azure/app-service/deploy-continuous-deployment#prepare-your-repository)底下所述。

### <a name="update-app-settings-in-azure"></a>在 Azure 中更新應用程式設定
本機 Bot 不會使用加密的 .bot 檔案，但「如果」Azure 入口網站設定為使用加密的 .bot 檔案，則會使用加密的 .bot 檔案。 您可以透過移除儲存在 Azure Bot 設定中的 **botFileSecret** 來解決這個問題。
1. 在 Azure 入口網站中，為您的 Bot 開啟 [Web 應用程式 Bot] 資源。
1. 開啟 Bot 的 [應用程式設定]。
1. 在 [應用程式設定] 視窗中，向下捲動至 [應用程式設定]。
1. 查看您的 Bot 是否具有 **botFileSecret** 和 **botFilePath** 項目。 如果您這麼做，請刪除。
1. 儲存變更。

## <a name="2-deploy-using-azure-deployment-center"></a>2.使用 Azure 部署中心進行部署

現在，您需要將 Bot 程式碼上傳至 Azure。 請遵循[持續部署至 Azure App Service](https://docs.microsoft.com/azure/app-service/deploy-continuous-deployment)主題中的指示來進行。

請注意，建議使用 `App Service Kudu build server` 建置。

在設定好持續部署後，系統便會將您已認可至存放庫的變更發行出去。 不過，如果您將服務新增至 Bot，則需要將這些服務的項目新增至 .bot 檔案中。

## <a name="3-test-your-deployment"></a>3.測試您的部署

在成功部署後等候幾秒，選擇性地重新啟動您的 Web 應用程式，以清除任何快取。 回到您的 Web 應用程式 Bot 刀鋒視窗，使用 Azure 入口網站中提供的網路聊天進行測試。

## <a name="additional-resources"></a>其他資源
- [如何調查連續部署的常見問題](https://github.com/projectkudu/kudu/wiki/Investigating-continuous-deployment)

