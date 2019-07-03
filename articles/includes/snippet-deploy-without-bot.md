---
ms.openlocfilehash: 117f95799df0abbe957000d4979b10f05baf262c
ms.sourcegitcommit: a295a90eac461f8b96770dd902ba44919acf33fc
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/26/2019
ms.locfileid: "67405528"
---
開始部署之前，確定您有最新版的 [Azure cli](https://docs.microsoft.com/cli/azure/install-azure-cli?view=azure-cli-latest) 和 [dotnet cli](https://dotnet.microsoft.com/download)。 如果您沒有 dotnet cli，請從上面提供的連結使用 .Net Core Runtime 選項安裝。 

### <a name="login-to-azure-cli-and-set-your-subscription"></a>登入 Azure CLI 並設定訂用帳戶
您已在本機建立及測試 Bot，而現在想要將其部署至 Azure。 開啟命令提示字元以登入 Azure 入口網站。

```cmd
az login
```
### <a name="set-the-subscription"></a>設定訂用帳戶

設定要使用的預設訂用帳戶。

```cmd
az account set --subscription "<azure-subscription>"
```

如果您不確定哪個訂用帳戶要用於部署 Bot ，可以使用 `az account list` 命令，檢視帳戶的訂用帳戶清單。

瀏覽至 Bot 資料夾。
`cd <local-bot-folder>`

### <a name="create-a-web-app-bot-in-azure"></a>在 Azure 中建立 Web 應用程式 Bot 

如果您還沒有可供發佈本機 Bot 的資源群組，請建立一個：

```cmd
az group create --name <resource-group-name> --location <geographic-location> --verbose
```

| 選項     | 說明 |
|:-----------|:---|
| name     | 資源群組的唯一名稱。 請勿在名稱中包含空格或底線。 |
| location | 用來建立資源群組的地理位置。 例如，`eastus`、`westus` 及 `westus2` 等等。 使用 `az account list-locations` 來取得位置清單。 |

然後，建立您將在其中發佈 Bot 的 Bot 資源。 這會在 Azure 中佈建必要的資源並建立 Bot Web 應用程式，您將以本機 Bot 覆寫該應用程式。 

在繼續之前，請根據您用來登入 Azure 的電子郵件帳戶類型，閱讀您適用的指示。

#### <a name="msa-email-account"></a>MSA 電子郵件帳戶
如果您使用 MSA 電子郵件帳戶，則必須在應用程式註冊入口網站上建立應用程式識別碼和應用程式密碼，以搭配 `az bot create` 命令使用。
1. 移至[**應用程式註冊入口網站**](https://portal.azure.com/#blade/Microsoft_AAD_RegisteredApps/ApplicationsListBlade)。
1. 按一下 [新增應用程式]  以註冊您的應用程式、建立 **Application-id**，以及**產生新密碼**。 如果您已經有應用程式和密碼，但不記得密碼，則必須在應用程式祕密區段中產生新密碼。
1. 儲存應用程式識別碼和您剛產生的新密碼，讓您能用來搭配 `az bot create` 命令。  

```cmd
az bot create --kind webapp --name <bot-name-in-azure> --location <geographic-location> --version v4 --lang <language> --verbose --resource-group <resource-group-name> --appid "<application-id>" --password "<application-password>" --verbose
```

| 選項 | 說明 |
|:---|:---|
| name | 在 Azure 中用於部署 Bot 的唯一名稱。 這可能與您的本機 Bot 同名。 請勿在名稱中包含空格或底線。 |
| location | 用來建立 Bot 服務資源的地理位置。 例如，`eastus`、`westus` 及 `westus2` 等等。 |
| resource-group | 要在其中建立 Bot 的資源群組名稱。 您可以使用 `az configure --defaults group=<name>` 來設定預設群組。 |
| appid | 要用於 Bot 的 Microsoft 帳戶識別碼 (MSA ID)。 |
| password | Bot 的 Microsoft 帳戶 (MSA) 密碼。 |

#### <a name="business-or-school-account"></a>公司或學校帳戶

```cmd
az bot create --kind webapp --name <bot-name-in-azure> --location <geographic-location> --version v4 --lang <language> --verbose --resource-group <resource-group-name>
```
| 選項 | 說明 |
|:---|:---|
| name | 在 Azure 中用於部署 Bot 的唯一名稱。 這可能與您的本機 Bot 同名。 請勿在名稱中包含空格或底線。 |
| location | 用來建立 Bot 服務資源的地理位置。 例如，`eastus`、`westus` 及 `westus2` 等等。 |
| lang | 要用來建立 Bot 的語言：`Csharp` 或 `Node`；預設值是 `Csharp`。 |
| resource-group | 要在其中建立 Bot 的資源群組名稱。 您可以使用 `az configure --defaults group=<name>` 來設定預設群組。 |

#### <a name="update-appsettingsjson-or-env-file"></a>更新 appsettings.json 或 .env 檔案
建立 Bot 之後，您應會看到主控台視窗中顯示的下列資訊： 

```JSON
{
  "appId": "as234-345b-4def-9047-a8a44b4s",
  "appPassword": "34$#w%^$%23@334343",
  "endpoint": "https://mybot.azurewebsites.net/api/messages",
  "id": "mybot",
  "name": "mybot",
  "resourceGroup": "botresourcegroup",
  "serviceName": "mybot",
  "subscriptionId": "234532-8720-5632-a3e2-a1qw234",
  "tenantId": "32f955bf-33f1-43af-3ab-23d009defs47",
  "type": "abs"
}
```

您需要複製 `appId` 和 `appPassword` 值，並將其貼到 appsettings.json 或 .env 檔案中。 例如︰

```JSON
{
  MicrosoftAppId: "as234-345b-4def-9047-a8a44b4s",
  MicrosoftAppPassword: "34$#w%^$%23@334343"
}
```
請注意，如果 appsettings.json 或 .env 檔案中有額外的金鑰適用於您針對 Bot 所佈建的其他服務，請勿刪除這些項目。

儲存檔案。

接下來，視您用來建立 Bot 的程式設計語言而定 (**C#** 或 **JS**)，遵循您所適用的步驟。

**C# Bot：** 

開啟命令提示字元，並巡覽至專案資料夾。 從命令列執行下列命令。

| Task | 命令 |
|:-----|:--------|
| 1.還原專案相依性 | `dotnet restore`|
| 2.建置專案     | `dotnet build` |
| 3.壓縮專案檔案 | 使用任何公用程式來壓縮專案檔案。 移至包含 .csproj 檔案的資料夾，然後選取此層級的所有檔案和資料夾，以建立壓縮的資料夾。 |
| 4.設定組建部署設定 | `az webapp config appsettings set --resource-group <resource-group-name> --name <bot-name> --settings SCM_DO_BUILD_DEPLOYMENT=false`|
| 5.設定指令碼產生器引數 | `az webapp config appsettings set --resource-group <resource-group-name> --name <bot-name> --settings SCM_SCRIPT_GENERATOR_ARGS="--aspNetCore mybot.csproj"`|

**JS Bot：**
1. 從[這裡](https://github.com/projectkudu/kudu/wiki/Using-a-custom-web.config-for-Node-apps)下載 web.config 並將其儲存到專案資料夾中。 
1. 編輯此檔案，並以 "index.js" 取代所有出現的 "server.js" 項目。 
1. 儲存檔案。

開啟命令提示字元，並巡覽至專案資料夾。 從命令列執行下列命令。

| Task | 命令 |
|:-----|:--------|
| 1.安裝節點模組 | `npm install` |
| 2.壓縮專案檔案 | 使用任何公用程式來壓縮專案檔案。 移至包含 .csproj 檔案的資料夾，然後選取此層級的所有檔案和資料夾，以建立壓縮的資料夾。 |
| 3.設定組建部署設定 | `az webapp config appsettings set --resource-group <resource-group-name> --name <bot-name> --settings SCM_DO_BUILD_DEPLOYMENT=false`|
