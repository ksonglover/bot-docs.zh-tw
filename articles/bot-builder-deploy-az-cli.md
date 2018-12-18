---
title: 使用 Azure CLI 部署 Bot | Microsoft Docs
description: 將您的 Bot 部署至 Azure 雲端。
keywords: 部署 bot, azure 部署, 發佈 bot, az 部署 bot, visual studio 部署 bot, msbot 發佈, msbot 複製
author: ivorb
ms.author: v-ivorb
manager: kamrani
ms.topic: get-started-article
ms.service: bot-service
ms.subservice: abs
ms.date: 12/11/2018
ms.openlocfilehash: a7cb9cb1e3df14f2f46bc5a4c3a633f5212b5dfd
ms.sourcegitcommit: 0b421ff71617f03faf55ea175fb91d1f9e348523
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 12/12/2018
ms.locfileid: "53286614"
---
# <a name="deploy-your-bot-using-azure-cli"></a>使用 Azure CLI 部署 Bot

[!INCLUDE [pre-release-label](./includes/pre-release-label.md)]

建立 Bot 並在本機測試後，您可以將其部署至 Azure，以便從任何地方進行存取。 將 Bot 部署至 Azure 時，需支付您所使用的服務。 [計費與成本管理](https://docs.microsoft.com/en-us/azure/billing/)一文協助您了解 Azure 計費方式、監視使用量和成本，以及管理帳戶和訂用帳戶。

在本文中，我們會示範如何使用 `msbot` 工具將 C# 和 JavaScript Bot 部署至 Azure。 在依照步驟執行前閱讀本文很有用，您可完全了解部署 Bot 的相關事項。


## <a name="prerequisites"></a>必要條件
- 如果您沒有 Azure 訂用帳戶，請在開始前建立 [免費帳戶](https://azure.microsoft.com/free/) 。
- 安裝 [.NET Core SDK](https://dotnet.microsoft.com/download) >=v2.2。 
- 安裝最新版的 [Azure CLI 工具](https://docs.microsoft.com/en-us/cli/azure/install-azure-cli?view=azure-cli-latest)。
- 安裝 `az` 工具的最新版 `botservice` 擴充程式。 
  - 首先，使用 `az extension remove -n botservice` 命令移除舊版。 接著，使用 `az extension add -n botservice` 命令來安裝最新版本。
- 安裝最新版的 [MSBot](https://github.com/Microsoft/botbuilder-tools/tree/master/packages/MSBot) 工具。
  - 如果複製作業包含 LUIS 或分派資源，則需要 [LUIS CLI](https://github.com/Microsoft/botbuilder-tools/tree/master/packages/LUIS#installation)。
  - 如果複製作業包含 QnA Maker 資源，則需要 [LUIS CLI](https://github.com/Microsoft/botbuilder-tools/tree/master/packages/QnAMaker#as-a-cli)。
- 安裝 [Bot Framework 模擬器](https://aka.ms/Emulator-wiki-getting-started)。
- 安裝及設定 [ngrok](https://github.com/Microsoft/BotFramework-Emulator/wiki/Tunneling-%28ngrok%29)。
- [.bot](v4sdk/bot-file-basics.md) 檔案的知識。

## <a name="deploy-javascript-and-c-bots-using-az-cli"></a>使用 az cli 部署 JavaScript 和 C# Bot
您已建立 Bot，而現在想要將其部署至 Azure。 這些步驟假設您已建立必要的 Azure 資源，並使用 `msbot connect` 命令或 Bot Framework 模擬器來更新 .bot 檔案中的服務參考。 如果 .bot 檔案不是最新狀態，則部署程序可能會在沒有任何錯誤或警告的情形下完成，但已部署的 Bot 無法運作。

開啟命令提示字元以登入 Azure 入口網站。

```cmd
az login
```
瀏覽器視窗會隨即開啟，以便您登入。 

### <a name="set-the-subscription"></a>設定訂用帳戶 
使用下列命令來設定訂用帳戶：

```cmd
az account set --subscription "<azure-subscription>"
``` 

如果您不確定哪個訂用帳戶要用於部署 Bot ，您可以使用 `az account list` 命令，檢視您帳戶的 `subscriptions` 清單。

瀏覽至 Bot 資料夾。 
```cmd 
cd <local-bot-folder>
```

### <a name="azure-subscription-account"></a>Azure 訂用帳戶
在繼續之前，請根據您用來登入 Azure 的電子郵件帳戶類型，閱讀您適用的指示。

**MSA 電子郵件帳戶**

如果您使用 [MSA](https://en.wikipedia.org/wiki/Microsoft_account) 電子郵件帳戶，您必須建立 appId 和 appSecret 以搭配 `msbot clone services` 命令使用。 

- 移至[應用程式註冊入口網站](https://apps.dev.microsoft.com/)。 按一下 [新增應用程式] 以註冊您的應用程式、建立 **Application-id**，以及**產生新密碼**。 
- 儲存應用程式識別碼和您剛產生的新密碼，讓您能用來搭配 `msbot clone services` 命令。 
- 若要部署，請使用您 Bot 適用的命令。

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

`msbot clone services --folder deploymentScripts/msbotClone --location "<geographic-location>" --proj-file "<your.csproj>" --name "<bot-name>" --appid "xxxxxxxx" --password "xxxxxxx" --verbose`

[!INCLUDE [deployment note](./includes/deployment-note-cli.md)]

# <a name="javascripttabjs"></a>[JavaScript](#tab/js)

`msbot clone services --folder deploymentScripts/msbotClone --location "<geographic-location>"   --code-dir . --name "<bot-name>" --appid "xxxxxxxx" --password "xxxxxxx" --verbose`


[!INCLUDE [deployment note](./includes/deployment-note-cli.md)]

---

**公司或學校帳戶**

如果您使用貴公司或學校提供給您的電子郵件帳戶來登入 Azure，則不需要建立應用程式識別碼和密碼。 若要部署，請使用您 Bot 適用的命令。

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

`msbot clone services --folder deploymentScripts/msbotClone --location "<geographic-location>" --verbose --proj-file "<your-project-file>" --name "<bot-name>"`

[!INCLUDE [deployment note](./includes/deployment-note-cli.md)]

# <a name="javascripttabjs"></a>[JavaScript](#tab/js)

`msbot clone services --folder deploymentScripts/msbotClone --location "<geographic-location>" --verbose --code-dir . --name "<bot-name>"`


[!INCLUDE [deployment note](./includes/deployment-note-cli.md)]

---

#### <a name="save-the-secret-used-to-decrypt-bot-file"></a>儲存用來解密 .bot 檔案的祕密
請務必注意，部署程序會建立 _新的 .bot 檔案並使用祕密進行加密_。 在部署 Bot 時，您會在命令列中看到下列訊息，要求您儲存 .bot 檔案祕密。 

`The secret used to decrypt myAzBot.bot is:`
`hT6U1SIPQeXlebNgmhHYxcdseXWBZlmIc815PpK0WWA=`

`NOTE: This secret is not recoverable and you should save it in a safe place according to best security practices.
      Copy this secret and use it to open the <file.bot> the first time.`
      
儲存 .bot 檔案祕密，以供稍後使用。 新的已加密 .bot 檔案會在 Azure 入口網站中搭配 botFileSecret 使用。 如果您稍後需要變更 Bot 檔案名稱或祕密，請移至入口網站中的 [App Service 設定] -> [應用程式設定] 區段。 請注意，在 appsettings.json 或 .env 檔案中，會使用所建立的最新 Bot 檔案更新 Bot 檔案名稱。  

### <a name="test-your-bot"></a>測試 Bot
在模擬器中，使用生產端點來測試您的應用程式。 如果您想要在本機進行測試，請確定您的 Bot 正在本機電腦上執行。 

### <a name="to-update-your-bot-code-in-azure"></a>在 Azure 中更新 Bot 程式碼
請勿使用 `msbot clone services` 命令在 Azure 中更新 Bot 程式碼。 您必須使用 `az bot publish` 命令，如下所示：

```cmd
az bot publish --name "<your-azure-bot-name>" --proj-file "<your-proj-file>" --resource-group "<azure-resource-group>" --code-dir "<folder>" --verbose --version v4
```

| 引數        | 說明 |
|----------------  |-------------|
| `name`      | Bot 第一次部署至 Azure 時所使用的名稱。|
| `proj-file` | 若為 C# Bot，這是 .csproj 檔案。 若為 JS/TS Bot，這是您本機 Bot 的啟動專案檔案名稱 (例如 index.js 或 index.ts)。|
| `resource-group` | `msbot clone services` 命令使用的 Azure 資源群組。|
| `code-dir`  | 指向本機 Bot 資料夾。|



## <a name="additional-resources"></a>其他資源

當您部署 Bot 時，這些資源通常會建立於 Azure 入口網站中：

| 資源      | 說明 |
|----------------|-------------|
| Web 應用程式 Bot | 已部署至 Azure App Service 的 Azure Bot Service Bot。|
| [App Service](https://docs.microsoft.com/en-us/azure/app-service/)| 可讓您建置及裝載 Web 應用程式。|
| [App Service 計劃](https://docs.microsoft.com/en-us/azure/app-service/azure-web-sites-web-hosting-plans-in-depth-overview)| 針對要執行的 Web 應用程式定義一組計算資源。|
| [Application Insights](https://docs.microsoft.com/en-us/azure/application-insights/app-insights-overview)| 提供用來收集和分析遙測的工具。|
| [儲存體帳戶](https://docs.microsoft.com/en-us/azure/storage/common/storage-introduction)| 提供高可用性、安全、耐用、可擴充和備援性雲端儲存體。|

若要查看 `az bot` 命令的文件，請參閱[參考](https://docs.microsoft.com/en-us/cli/azure/bot?view=azure-cli-latest)主題。

如果您不熟悉 Azure 資源群組，請參閱這個[術語](https://docs.microsoft.com/en-us/azure/azure-resource-manager/resource-group-overview#terminology)主題。

## <a name="next-steps"></a>後續步驟
> [!div class="nextstepaction"]
> [設定持續部署](bot-service-build-continuous-deployment.md)
