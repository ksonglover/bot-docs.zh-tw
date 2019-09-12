---
ms.openlocfilehash: 029275eae6f7b0b4448613ede898575eb491a56f
ms.sourcegitcommit: dd12ddf408c010182b09da88e2aac0de124cef22
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/05/2019
ms.locfileid: "70385989"
---
使用現有的資源群組時，您可以使用現有的 App Service 方案或建立新的方案。 這兩個選項的步驟如下所列。 

**選項 1：現有的 App Service 方案** 

在此案例中，我們將使用現有的 App Service 方案，但建立新的 Web 應用程式和 Bot 通道註冊。 

> [!NOTE]
> 此命令會設定 Bot 的識別碼和顯示名稱。 `botId` 參數應該是全域唯一，而且會用來作為不可變的 Bot 識別碼。 Bot 的顯示名稱是可變動的。

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
| resource-group | Azure 資源群組的名稱。 |
| template-file | ARM 範本的路徑。 您可以使用專案資料夾 `deploymentTemplates` 中提供的 `template-with-preexisting-rg.json` 檔案。 |
| location |位置。 值的來源：`az account list-locations`。 您可以使用 `az configure --defaults location=<location>` 來設定預設位置。 |
| parameters | 提供部署參數值。 執行 `az ad app create` 命令時所得的 `appId` 值。 `appSecret` 是您在上一個步驟中提供的密碼。 `botId` 參數應該是全域唯一，而且會用來作為不可變的 Bot 識別碼。 此參數也會用來設定 Bot 的顯示名稱，而這是可變動的。 `newWebAppName` 是您要建立的 Web 應用程式名稱。 `newAppServicePlanName` 是 App Service 方案的名稱。 `newAppServicePlanLocation` 是 App Service 方案的位置。 |
