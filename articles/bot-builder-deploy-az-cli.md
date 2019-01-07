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
ms.date: 12/14/2018
ms.openlocfilehash: 19960940a40fa291534bc1f88290bc6a7da109e0
ms.sourcegitcommit: 8c10aa7372754596a3aa7303a3a893dd4939f7e9
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 12/20/2018
ms.locfileid: "53654333"
---
# <a name="deploy-your-bot-using-azure-cli"></a>使用 Azure CLI 部署 Bot

[!INCLUDE [pre-release-label](./includes/pre-release-label.md)]

建立 Bot 並在本機測試後，您可以將其部署至 Azure，以便從任何地方進行存取。 將 Bot 部署至 Azure 時，需支付您所使用的服務。 [計費與成本管理](https://docs.microsoft.com/en-us/azure/billing/)一文協助您了解 Azure 計費方式、監視使用量和成本，以及管理帳戶和訂用帳戶。

在本文中，我們會示範如何使用 `az` 和 `msbot` 將 C# 和 JavaScript Bot 部署至 Azure。 在依照步驟執行前閱讀本文很有用，您可完全了解部署 Bot 的相關事項。


## <a name="prerequisites"></a>必要條件
- 如果您沒有 Azure 訂用帳戶，請在開始前建立 [免費帳戶](https://azure.microsoft.com/free/) 。
- 安裝最新版的 [Azure CLI 工具](https://docs.microsoft.com/en-us/cli/azure/install-azure-cli?view=azure-cli-latest)。
- 安裝 `az` 工具的最新版 `botservice` 擴充程式。
  - 首先，使用 `az extension remove -n botservice` 命令移除舊版。 接著，使用 `az extension add -n botservice` 命令來安裝最新版本。
- 安裝最新版的 [MSBot](https://github.com/Microsoft/botbuilder-tools/tree/master/packages/MSBot) 工具。
- 安裝 [Bot Framework Emulator](https://aka.ms/Emulator-wiki-getting-started) 的最新發行版本。
- 安裝及設定 [ngrok](https://github.com/Microsoft/BotFramework-Emulator/wiki/Tunneling-%28ngrok%29)。
- [.bot](v4sdk/bot-file-basics.md) 檔案的知識。

## <a name="deploy-javascript-and-c-bots-using-az-cli"></a>使用 az cli 部署 JavaScript 和 C# Bot

您已在本機建立及測試 Bot，而現在想要將其部署至 Azure。 這些步驟假設您已建立必要的 Azure 資源。

開啟命令提示字元以登入 Azure 入口網站。

```cmd
az login
```

瀏覽器視窗會隨即開啟，以便您登入。

### <a name="set-the-subscription"></a>設定訂用帳戶

設定要使用的預設訂用帳戶。

```cmd
az account set --subscription "<azure-subscription>"
```

如果您不確定哪個訂用帳戶要用於部署 Bot ，您可以使用 `az account list` 命令，檢視您帳戶的 `subscriptions` 清單。

瀏覽至 Bot 資料夾。

```cmd
cd <local-bot-folder>
```

### <a name="create-a-web-app-bot"></a>建立 Web 應用程式 Bot

建立您將在其中發佈 Bot 的 Bot 資源。

在繼續之前，請根據您用來登入 Azure 的電子郵件帳戶類型，閱讀您適用的指示。

#### <a name="msa-email-account"></a>MSA 電子郵件帳戶

如果您使用 [MSA](https://en.wikipedia.org/wiki/Microsoft_account) 電子郵件帳戶，則必須在應用程式註冊入口網站上建立應用程式識別碼和應用程式密碼，以搭配 `az bot create` 命令使用。

1. 移至[**應用程式註冊入口網站**](https://apps.dev.microsoft.com/)。
1. 按一下 [新增應用程式] 以註冊您的應用程式、建立 **Application-id**，以及**產生新密碼**。 如果您已經有應用程式和密碼，但不記得密碼，則必須在應用程式祕密區段中產生新密碼。
1. 儲存應用程式識別碼和您剛產生的新密碼，讓您能用來搭配 `az bot create` 命令。  

```cmd
az bot create --kind webapp --name <bot-resource-name> --location <geographic-location> --version v4 --lang <language> --verbose --resource-group <resource-group-name> --appid "<application-id>" --password "<application-password>" --verbose
```

| 選項 | 說明 |
|:---|:---|
| --name | 在 Azure 中用於部署 Bot 的唯一名稱。 這可能與您的本機 Bot 同名。 請勿在名稱中包含空格或底線。 |
| --location | 用來建立 Bot 服務資源的地理位置。 例如，`eastus`、`westus` 及 `westus2` 等等。 |
| --lang | 要用來建立 Bot 的語言：`Csharp` 或 `Node`；預設值是 `Csharp`。 |
| --resource-group | 要在其中建立 Bot 的資源群組名稱。 您可以使用 `az configure --defaults group=<name>` 來設定預設群組。 |
| --appid | 要用於 Bot 的 Microsoft 帳戶識別碼 (MSA ID)。 |
| --password | Bot 的 Microsoft 帳戶 (MSA) 密碼。 |

#### <a name="business-or-school-account"></a>公司或學校帳戶

```cmd
az bot create --kind webapp --name <bot-resource-name> --location <geographic-location> --version v4 --lang <language> --verbose --resource-group <resource-group-name>
```

| 選項 | 說明 |
|:---|:---|
| --name | 在 Azure 中用於部署 Bot 的唯一名稱。 這可能與您的本機 Bot 同名。 請勿在名稱中包含空格或底線。 |
| --location | 用來建立 Bot 服務資源的地理位置。 例如，`eastus`、`westus` 及 `westus2` 等等。 |
| --lang | 要用來建立 Bot 的語言：`Csharp` 或 `Node`；預設值是 `Csharp`。 |
| --resource-group | 要在其中建立 Bot 的資源群組名稱。 您可以使用 `az configure --defaults group=<name>` 來設定預設群組。 |

### <a name="download-the-bot-from-azure"></a>從 Azure 下載 Bot

接下來，下載您剛建立的 Bot。 此命令會在 save-path 之下建立子目錄；不過，指定的路徑必須已經存在。

```cmd
az bot download --name <bot-resource-name> --resource-group <resource-group-name> --save-path "<path>"
```

| 選項 | 說明 |
|:---|:---|
| --name | Azure 中的 Bot 名稱。 |
| --resource-group | Bot 所屬的資源群組名稱。 |
| --save-path | Bot 程式碼要下載至的現有目錄。 |

### <a name="decrypt-the-downloaded-bot-file"></a>將下載的 .bot 檔案解密

.bot 檔案中的敏感性資訊已加密。

取得加密金鑰。

1. 登入 [Azure 入口網站](http://portal.azure.com/)。
1. 為您的 Bot 開啟 Web 應用程式 Bot 資源。
1. 開啟 Bot 的 [應用程式設定]。
1. 在 [應用程式設定] 視窗中，向下捲動至 [應用程式設定]。
1. 找出 **botFileSecret** 並複製其值。

將 .bot 檔案解密。

```cmd
msbot secret --bot <name-of-bot-file> --secret "<bot-file-secret>" --clear
```

| 選項 | 說明 |
|:---|:---|
| --bot | 下載的 .bot 檔案的相對路徑。 |
| --secret | 加密金鑰。 |

### <a name="use-the-downloaded-bot-file-in-your-project"></a>在您的專案中使用下載的 .bot 檔案

將已解密的 .bot 檔案複製到包含本機 Bot 專案的目錄。

更新 Bot 以使用這個新 .bot 檔案。

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

在 **appsettings.json** 中，更新 **botFilePath** 屬性以指向新的 .bot 檔案。

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

在 **.env** 中，更新 **botFilePath** 屬性以指向新的 .bot 檔案。

---

### <a name="update-the-bot-file"></a>更新 .bot 檔案

如果 Bot 使用 LUIS、QnA Maker 或分派服務，您必須將這些參考新增至 .bot 檔案。 否則，您可以略過此步驟。

1. 使用新的 .bot 檔案，在 BotFramework Emulator 中開啟 Bot。 Bot 不需要在本機執行。
1. 在 [BOT 總管] 面板中，展開 [服務] 區段。
1. 若要新增 LUIS 應用程式的參考，請按一下 [服務] 右邊的加號 (+)。
   1. 選取 [新增 Language Understanding (LUIS)]。
   1. 依照提示來登入您的 Azure 帳戶。
   1. 其會呈現您有存取權的 LUIS 應用程式清單。 選取您的 Bot 適用的項目。
1. 若要新增 QnA Maker 知識庫的參考，請按一下 [服務] 右邊的加號 (+)。
   1. 選取 [新增 QnA Maker]。
   1. 依照提示來登入您的 Azure 帳戶。
   1. 其會呈現您有存取權的知識庫清單。 選取您的 Bot 適用的項目。
1. 若要新增分派模型的參考，請按一下 [服務] 右邊的加號 (+)。
   1. 選取 [新增分派]。
   1. 依照提示來登入您的 Azure 帳戶。
   1. 其會呈現您有存取權的分派模型清單。 選取您的 Bot 適用的項目。

### <a name="test-your-bot-locally"></a>在本機測試 Bot

此時，您 Bot 的運作方式應該與使用舊 .bot 檔案時相同。 確保使用新 .bot 檔案能如預期般運作。

### <a name="publish-your-bot-to-azure"></a>將 Bot 發佈至 Azure

<!-- TODO: re-encrypt your .bot file? -->

將本機 Bot 發佈至 Azure。 這個步驟可能需要一些時間。

```cmd
az bot publish --name <bot-resource-name> --proj-file "<project-file-name>" --resource-group <resource-group-name> --code-dir <directory-path> --verbose --version v4
```

<!-- Question: What should --proj-file be for a Node project? -->

| 選項 | 說明 |
|:---|:---|
| --name | Azure 中的 Bot 資源名稱。 |
| --proj-file | 需要發佈的啟動專案檔名 (不含 .csproj)。 例如︰EnterpriseBot。 |
| --resource-group | 資源群組的名稱。 |
| --code-dir | 要從中上傳 Bot 程式碼的目錄。 |

一旦完成並顯示「部署成功！」 訊息，Bot 便已部署在 Azure 中。

<!-- TODO: If we tell them to re-encrypt, this step is not necessary. -->

清除加密金鑰設定。

1. 登入 [Azure 入口網站](http://portal.azure.com/)。
1. 為您的 Bot 開啟 Web 應用程式 Bot 資源。
1. 開啟 Bot 的 [應用程式設定]。
1. 在 [應用程式設定] 視窗中，向下捲動至 [應用程式設定]。
1. 找出 **botFileSecret** 並加以刪除。

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
