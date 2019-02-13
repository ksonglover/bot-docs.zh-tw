---
title: 部署 Bot botbuilder-samples 存放庫 | Microsoft Docs
description: 將您的 Bot 部署至 Azure 雲端。
keywords: 部署 bot, azure 部署, 發佈 bot, az 部署 bot, visual studio 部署 bot, msbot 發佈, msbot 複製
author: ivorb
ms.author: v-ivorb
manager: kamrani
ms.topic: get-started-article
ms.service: bot-service
ms.subservice: abs
ms.date: 01/15/2019
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 3ca8ac4bfe14ed20f11a0ab26d8102ac21e60e2b
ms.sourcegitcommit: 32615b88e4758004c8c99e9d564658a700c7d61f
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 02/04/2019
ms.locfileid: "55711952"
---
# <a name="deploy-bots-from-botbuilder-samples-repo"></a>部署 Bot botbuilder-samples 存放庫

[!INCLUDE [pre-release-label](./includes/pre-release-label.md)]

在本文中，我們會示範如何將 [botbuilder-samples](https://github.com/Microsoft/BotBuilder-Samples) 存放庫中的 C# 和 JavaScript 範例部署至 Azure。

以下兩項部署作業的指示「不同」：部署範例 Bot，以及[部署您使用 Azure 中已經佈建的所有資源所建立的 Bot ](https://docs.microsoft.com/en-us/azure/bot-service/bot-builder-deploy-az-cli?view=azure-bot-service-4.0&tabs=csharp)。

> [!IMPORTANT]
> 將 [botbuilder-samples](https://github.com/Microsoft/BotBuilder-Samples) 存放庫中的 Bot 部署至 Azure，就會佈建 Azure 服務並且需要支付您所使用的服務。
> [計費與成本管理](https://docs.microsoft.com/en-us/azure/billing/)一文協助您了解 Azure 計費方式、監視使用量和成本，以及管理帳戶和訂用帳戶。

在依照步驟執行前詳細閱讀本文很有用，您可完全了解部署 Bot 的相關事項。

## <a name="prerequisites"></a>必要條件

- 如果您沒有 Azure 訂用帳戶，請在開始前建立 [免費帳戶](https://azure.microsoft.com/free/) 。
- 安裝 [.NET Core SDK](https://dotnet.microsoft.com/download) >= v2.2。 使用 `dotnet --version` 來查看您擁有的版本。
- 安裝最新版的 [Azure CLI 工具](https://docs.microsoft.com/en-us/cli/azure/install-azure-cli?view=azure-cli-latest)。 使用 `az --version` 來查看您擁有的版本。
- 安裝最新版的 [MSBot](https://github.com/Microsoft/botbuilder-tools/tree/master/packages/MSBot) 工具。
  - 如果複製作業包含 LUIS 或分派資源，則需要 [LUIS CLI](https://github.com/Microsoft/botbuilder-tools/tree/master/packages/LUIS#installation)。
  - 如果複製作業包含 QnA Maker 資源，則需要 [LUIS CLI](https://github.com/Microsoft/botbuilder-tools/tree/master/packages/QnAMaker#as-a-cli)。
- 安裝 [Bot Framework 模擬器](https://aka.ms/Emulator-wiki-getting-started)。
- 安裝及設定 [ngrok](https://github.com/Microsoft/BotFramework-Emulator/wiki/Tunneling-%28ngrok%29)。
- [.bot](v4sdk/bot-file-basics.md) 檔案的知識。

使用 msbot 4.3.2 和更新版本，您需要 Azure CLI 2.0.54 版或更新版本。 如果您安裝了 botservice 擴充功能，則使用此命令加以移除。

```cmd
az extension remove --name botservice
```

### <a name="c"></a>C\#

 `msbot clone services` 不會將程式碼檔案上傳至 Azure，只會上傳 DLL 和一些其他檔案。 在執行此命令之前，您需要先編譯範例。

### <a name="service-names"></a>服務名稱

除了部署 Bot，`msbot clone services` 命令還會將所有相關聯的服務佈建到您所選的訂用帳戶。

如果訂用帳戶中已存在這些名稱服務組合，則此命令會產生錯誤，而您必須先刪除部分的部署，再重新開始。 這包括 LUIS 應用程式、QnA Maker 知識庫和分派模型。

## <a name="deploy-javascript-and-c-bots-using-az-cli"></a>使用 az cli 部署 JavaScript 和 C# Bot

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

### <a name="clone-the-sample"></a>複製範例

**Azure 訂用帳戶**：在繼續之前，請根據您用來登入 Azure 的電子郵件帳戶類型，閱讀您適用的指示。

**建立服務**：`msbot clone services` 命令會為 Bot 建立 Azure 服務。

1. 命令會列出所要建立的服務，並提示您在繼續之前先確認作業。 如果您拒絕，此命令則會結束，且不會建立任何服務。
1. 命令會讓您先向 Azure 進行驗證，再繼續進行。

**LUIS 服務**：如果 Bot 使用 LUIS 或分派，您必須在 `msbot clone services` 命令中包含您的 LUIS 撰寫金鑰。

#### <a name="msa-email-account"></a>MSA 電子郵件帳戶

如果您使用 [MSA](https://en.wikipedia.org/wiki/Microsoft_account) 電子郵件帳戶，您必須建立 appId 和 appSecret 以搭配 `msbot clone services` 命令使用。

- 移至[應用程式註冊入口網站](https://apps.dev.microsoft.com/)。 按一下 [新增應用程式] 以註冊您的應用程式、建立 **Application-id**，以及**產生新密碼**。
> 注意 - 如果產生的密碼包含 "|" 字元，Azure 將會拒絕此密碼。 若要解決這個問題，請產生另一個新密碼。
- 儲存應用程式識別碼和您剛產生的新密碼，讓您能用來搭配 `msbot clone services` 命令。
- 若要部署，請使用您 Bot 適用的命令。

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

`msbot clone services --folder deploymentScripts/msbotClone --location "<geographic-location>" --proj-file "<your.csproj>" --name "<bot-name>" --appId "xxxxxxxx" --appSecret "xxxxxxx" --verbose --luisAuthoringKey <luis-authoring-key>`

[!INCLUDE [deployment note](./includes/deployment-note-cli.md)]

# <a name="javascripttabjs"></a>[JavaScript](#tab/js)

`msbot clone services --folder deploymentScripts/msbotClone --location "<geographic-location>"   --code-dir . --name "<bot-name>" --appId "xxxxxxxx" --appSecret "xxxxxxx" --verbose --luisAuthoringKey <luis-authoring-key>`

[!INCLUDE [deployment note](./includes/deployment-note-cli.md)]

---

#### <a name="business-or-school-account"></a>公司或學校帳戶

如果您使用貴公司或學校提供給您的電子郵件帳戶來登入 Azure，則不需要建立應用程式識別碼和密碼。 若要部署，請使用您 Bot 適用的命令。

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

`msbot clone services --folder deploymentScripts/msbotClone --location "<geographic-location>" --verbose --proj-file "<your-project-file>" --name "<bot-name>" --luisAuthoringKey <luis-authoring-key>`

[!INCLUDE [deployment note](./includes/deployment-note-cli.md)]

# <a name="javascripttabjs"></a>[JavaScript](#tab/js)

`msbot clone services --folder deploymentScripts/msbotClone --location "<geographic-location>" --verbose --code-dir . --name "<bot-name>" --luisAuthoringKey <luis-authoring-key>`

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
az bot publish --name "<your-azure-bot-name>" --proj-name "<your-proj-name>" --resource-group "<azure-resource-group>" --code-dir "<folder>" --verbose --version v4
```

| 引數        | 說明 |
|----------------  |-------------|
| `name`      | Bot 第一次部署至 Azure 時所使用的名稱。|
| `proj-name` | 對於 C#，使用需要發佈的啟動專案檔名 (不含 .csproj)。 例如： `EnterpriseBot` 。 對於 Node.js，使用 Bot 的主要進入點。 例如： `index.js`。 |
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
