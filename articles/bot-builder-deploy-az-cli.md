---
title: 部署您的 Bot | Microsoft Docs
description: 將您的 Bot 部署至 Azure 雲端
keywords: 部署 bot, azure 部署 bot, 發佈 bot
author: ivorb
ms.author: kamrani
manager: kamrani
ms.topic: conceptual
ms.service: bot-service
ms.date: 08/06/2019
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: d44b1339b367c7243cfb6d2311a553becb6245ff
ms.sourcegitcommit: c200cc2db62dbb46c2a089fb76017cc55bdf26b0
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/27/2019
ms.locfileid: "70037500"
---
# <a name="deploy-your-bot"></a>部署您的 Bot

[!INCLUDE [applies-to](./includes/applies-to.md)]

在本文中，我們將說明如何將基本 Bot 部署至 Azure。 我們將說明如何準備您的 Bot 以進行部署、將 Bot 部署至 Azure，以及在網路聊天中測試您的 Bot。 在依照步驟執行前閱讀本文很有用，您可完全了解部署 Bot 的相關事項。

<!-- create your Azure Application, 2) prepare your source code for deployment, and 3) deploy your code to your Azure Application.  -->

## <a name="prerequisites"></a>必要條件
- [Microsoft Azure](https://azure.microsoft.com/free/)訂用帳戶。
- 您已在本機電腦上開發的 C#、JavaScript 或 TypeScript Bot。
- 最新版的 [Azure CLI](https://docs.microsoft.com/cli/azure/?view=azure-cli-latest)。
- 熟悉 [Azure CLI 和 ARM 範本](https://docs.microsoft.com/azure/azure-resource-manager/resource-group-overview)。

## <a name="prepare-for-deployment"></a>準備部署
當您使用 [Visual Studio 範本](https://docs.microsoft.com/azure/bot-service/dotnet/bot-builder-dotnet-sdk-quickstart?view=azure-bot-service-4.0)或 [Yeoman 範本](https://docs.microsoft.com/azure/bot-service/javascript/bot-builder-javascript-quickstart?view=azure-bot-service-4.0)建立 Bot 時，所產生的原始程式碼會包含 `deploymentTemplates` 資料夾與 ARM 範本。 此處敘述的部署程序會使用 ARM 範本，然後以 Azure CLI 在 Azure 中佈建所需的 Bot 資源。 

> [!NOTE]
> 在 Bot Framework SDK 4.3 版本中，我們已_不再_使用 .bot 檔案，而是改用 appsettings.json 或 .env 檔案來管理資源。 若要了解如何將設定從 .bot 檔案遷移至 appsettings.json 或 .env 檔案，請參閱[管理 Bot 資源](v4sdk/bot-file-basics.md)。

### <a name="1-login-to-azure"></a>1.登入 Azure

您已在本機建立及測試 Bot，而現在想要將其部署至 Azure。 開啟命令提示字元以登入 Azure 入口網站。

```cmd
az login
```
瀏覽器視窗會隨即開啟，以便您登入。

> [!NOTE]
> 如果您將 Bot 部署至非 Azure 雲端 (例如 US Gov)，您必須在 `az login` 之前執行 `az cloud set --name <name-of-cloud>`，其中，&lt;name-of-cloud> 是已註冊的雲端名稱，例如 `AzureUSGovernment`。 如果您要返回公用雲端，可以執行 `az cloud set --name AzureCloud`。 

### <a name="2-set-the-subscription"></a>2.設定訂用帳戶

設定要使用的預設訂用帳戶。

```cmd
az account set --subscription "<azure-subscription>"
```

如果您不確定哪個訂用帳戶要用於部署 Bot ，可以使用 `az account list` 命令，檢視帳戶的訂用帳戶清單。 瀏覽至 Bot 資料夾。

### <a name="3-create-an-app-registration"></a>3.建立應用程式註冊

註冊應用程式意謂著您可以使用 Azure AD 來驗證使用者，以及要求存取使用者資源。 在 Azure 中，您的 Bot 需要已註冊的應用程式來允許 Bot 存取 Bot Framework 服務，以便傳送和接收已驗證的訊息。 若要透過 Azure CLI 建立註冊應用程式，請執行下列命令：

```cmd
az ad app create --display-name "displayName" --password "AtLeastSixteenCharacters_0" --available-to-other-tenants
```

| 選項   | 說明 |
|:---------|:------------|
| display-name | 應用程式的顯示名稱。 |
| password | 應用程式密碼，也稱為「用戶端密碼」。 密碼必須至少有 16 個字元長，至少包含 1 個大寫或小寫字母，以及至少包含 1 個特殊字元。|
| available-to-other-tenants| 應用程式可以從任何 Azure AD 租用戶使用。 這必須設為 `true`，才能讓您的 Bot 使用 Azure Bot 服務通道。|

上述命令會輸出具有 `appId` 索引鍵的 JSON，請儲存此索引鍵的值以用於 ARM 部署，該值會用在 `appId` 參數上。 提供的密碼將會用於 `appSecret` 參數。

> [!NOTE] 
> 如果您想要使用現有的應用程式註冊，您可以使用命令：
> ``` cmd
> az bot create --kind webapp --resource-group "<name-of-resource-group>" --name "<name-of-web-app>" --appid "<existing-app-id>" --password "<existing-app-password>" --lang <Javascript|Csharp>
> ```

### <a name="4-deploy-via-arm-template"></a>4.透過 ARM 範本進行部署
您可以將 Bot 部署在新的資源群組或現有的資源群組中。 選擇最適合您的選項。 
<!--
## [Deploy via ARM template (with **new**  Resource Group)](#tab/nerg)
-->
#### <a name="deploy-via-arm-template-with-new-resource-group"></a>**透過 ARM 範本部署 (使用**新**資源群組)**
<!-- ##### Create Azure resources -->
您會在 Azure 中建立新的資源群組，然後使用 ARM 範本在其中建立指定資源。 在此案例中，我們提供 App Service 方案、Web 應用程式和 Bot 通道註冊。

```cmd
az deployment create --name "<name-of-deployment>" --template-file "template-with-new-rg.json" --location "location-name" --parameters appId="<msa-app-guid>" appSecret="<msa-app-password>" botId="<id-or-name-of-bot>" botSku=F0 newAppServicePlanName="<name-of-app-service-plan>" newWebAppName="<name-of-web-app>" groupName="<new-group-name>" groupLocation="<location>" newAppServicePlanLocation="<location>"
```

| 選項   | 說明 |
|:---------|:------------|
| name | 部署的自訂名稱。 |
| template-file | ARM 範本的路徑。 您可以使用專案資料夾 `deploymentTemplates` 中提供的 `template-with-new-rg.json` 檔案。 |
| location |位置。 值的來源：`az account list-locations`。 您可以使用 `az configure --defaults location=<location>` 來設定預設位置。 |
| parameters | 提供部署參數值。 執行 `az ad app create` 命令時所得的 `appId` 值。 `appSecret` 是您在上一個步驟中提供的密碼。 `botId` 參數應該是全域唯一，而且會用來作為不可變的 Bot 識別碼。 此參數也會用來設定 Bot 的顯示名稱，而這是可變動的。 `botSku` 是定價層，可以是 F0 (免費) 或 S1 (標準)。 `newAppServicePlanName` 是 App Service 方案的名稱。 `newWebAppName` 是您要建立的 Web 應用程式名稱。 `groupName` 是您要建立的 Azure 資源群組名稱。 `groupLocation` 是 Azure 資源群組的位置。 `newAppServicePlanLocation` 是 App Service 方案的位置。 |

#### <a name="deploy-via-arm-template-with-existing--resource-group"></a>**透過 ARM 範本部署 (使用**現有的**資源群組)**
<!--
## [Deploy via ARM template (with **existing**  Resource Group)](#tab/erg)
##### Create Azure resources
-->
使用現有的資源群組時，您可以使用現有的 App Service 方案或建立新的方案。 這兩個選項的步驟如下所列。 

**選項 1：現有的 App Service 方案** 

在此案例中，我們使用現有的 App Service 方案，但建立新的 Web 應用程式和 Bot 通道註冊。 

> [!NOTE]
> botId 參數應該是全域唯一，而且會用來作為不可變的 Bot 識別碼。 此參數也會用來設定 Bot 的 displayName，而這是可變動的。

```cmd
az group deployment create --name "<name-of-deployment>" --resource-group "<name-of-resource-group>" --template-file "template-with-preexisting-rg.json" --parameters appId="<msa-app-guid>" appSecret="<msa-app-password>" botId="<id-or-name-of-bot>" newWebAppName="<name-of-web-app>" existingAppServicePlan="<name-of-app-service-plan>" appServicePlanLocation="<location>"
```

**選項 2：新的 App Service 方案**

在此案例中，我們會建立 App Service 方案、Web 應用程式和 Bot 通道註冊。 

```cmd
az group deployment create --name "<name-of-deployment>" --resource-group "<name-of-resource-group>" --template-file "template-with-preexisting-rg.json" --parameters appId="<msa-app-guid>" appSecret="<msa-app-password>" botId="<id-or-name-of-bot>" newWebAppName="<name-of-web-app>" newAppServicePlanName="<name-of-app-service-plan>" appServicePlanLocation="<location>"
```

| 選項   | 說明 |
|:---------|:------------|
| name | 部署的自訂名稱。 |
| resource-group | Azure 資源群組的名稱 |
| template-file | ARM 範本的路徑。 您可以使用專案資料夾 `deploymentTemplates` 中提供的 `template-with-preexisting-rg.json` 檔案。 |
| location |位置。 值的來源：`az account list-locations`。 您可以使用 `az configure --defaults location=<location>` 來設定預設位置。 |
| parameters | 提供部署參數值。 執行 `az ad app create` 命令時所得的 `appId` 值。 `appSecret` 是您在上一個步驟中提供的密碼。 `botId` 參數應該是全域唯一，而且會用來作為不可變的 Bot 識別碼。 此參數也會用來設定 Bot 的顯示名稱，而這是可變動的。 `newWebAppName` 是您要建立的 Web 應用程式名稱。 `newAppServicePlanName` 是 App Service 方案的名稱。 `newAppServicePlanLocation` 是 App Service 方案的位置。 |

---

### <a name="5-prepare-your-code-for-deployment"></a>5.準備您的程式碼以進行部署
#### <a name="51-retrieve-or-create-necessary-iiskudu-files"></a>5.1 擷取或建立必要的 IIS/Kudu 檔案

<!-- **C# bots** -->
##### <a name="ctabcsharp"></a>[C#](#tab/csharp)

```cmd
az bot prepare-deploy --lang Csharp --code-dir "." --proj-file-path "MyBot.csproj"
```

您必須提供與 --code-dir 對應的 .csproj 檔案路徑。 這可以透過 --proj-file-path 引數來執行。 此命令會將 --code-dir 和 --proj-file-path 解析為 "./MyBot.csproj"

<!-- **JavaScript bots** -->
##### <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

```cmd
az bot prepare-deploy --code-dir "." --lang Javascript
```

此命令會擷取 web.config，這是讓 Node.js 應用程式可以在 Azure App Service 上與 IIS 搭配運作的必要項目。 請確定 web.config 已儲存到您 Bot 的根目錄。

<!-- **TypeScript bots** -->
##### <a name="typescripttabtypescript"></a>[TypeScript](#tab/typescript)

```cmd
az bot prepare-deploy --code-dir "." --lang Typescript
```

此命令的運作方式類似上述的 JavaScript，但這適用於 Typescript Bot。

---

> [!NOTE]
> 執行上述命令之後，您應該能夠在 Bot 專案資料夾中看到 `.deployment` 檔案。

#### <a name="52-zip-up-the-code-directory-manually"></a>5.2 手動壓縮程式碼目錄

使用未設定的 [zip 部署 API](https://github.com/projectkudu/kudu/wiki/Deploying-from-a-zip-file-or-url)來部署您 Bot 的程式碼時，Web 應用程式/Kudu 的行為會如下所示：

_根據預設，Kudu 會假設來自 zip 檔案的部署都已可供執行，而且部署期間不需要額外的建置步驟，例如 npm install 或 dotnet restore/dotnet publish。_

因此，務必在要部署至 Web 應用程式的 zip 檔案中包含建置程式碼和所有必要相依性，否則您的 Bot 不會如預期般運作。

> [!IMPORTANT]
> 壓縮專案檔之前，請確定您_位在_正確的資料夾。 
> - 針對 C# Bot，這是具有 .csproj 檔案的資料夾。 
> - 針對 JS Bot，這是具有 app.js 或 index.js 檔案的資料夾。 
>
>在專案資料夾**內**選取所有檔案並加以壓縮，然後在該資料夾中執行命令。 
>
> 如果您的根資料夾位置不正確，**Bot 將無法在 Azure 入口網站中執行**。

## <a name="deploy-code-to-azure"></a>將程式碼部署至 Azure
此時，我們已準備好將程式碼部署至 Azure Web 應用程式。 從命令列執行下列命令，即可對 Web 應用程式使用 kudu zip 推送部署來執行部署。

```cmd
az webapp deployment source config-zip --resource-group "<new-group-name>" --name "<name-of-web-app>" --src "code.zip" 
```

| 選項   | 說明 |
|:---------|:------------|
| resource-group | 您先前在 Azure 中建立的資源群組名稱。 |
| name | 您稍早使用的 Web 應用程式名稱。 |
| src  | 您所建立的 zip 檔案路徑。 |

## <a name="test-in-web-chat"></a>在網路聊天中測試

1. 在瀏覽器中導覽至 [Azure 入口網站](https://ms.portal.azure.com)。
2. 在左側面板中，按一下 [資源群組]  。
3. 在右側面板中，搜尋您的群組。
4. 按一下群組名稱。
5. 按一下 Bot 通道註冊的連結。
6. 在 [Bot 通道註冊]  刀鋒視窗中，按一下 [在網路聊天中測試]  。
或者，在右側面板中按一下 [測試] 方塊。

如需通道註冊的詳細資訊，請參閱[向 Bot 服務註冊聊天機器人](https://docs.microsoft.com/azure/bot-service/bot-service-quickstart-registration?view=azure-bot-service-3.0)。

> [!NOTE]
> 刀鋒視窗是用來顯示所選服務功能或導覽元素的介面。

## <a name="additional-information"></a>其他資訊
將 Bot 部署至 Azure 時，需支付您所使用的服務。 [計費與成本管理](https://docs.microsoft.com/azure/billing/)一文協助您了解 Azure 計費方式、監視使用量和成本，以及管理帳戶和訂用帳戶。

## <a name="next-steps"></a>後續步驟
> [!div class="nextstepaction"]
> [設定持續部署](bot-service-build-continuous-deployment.md)
