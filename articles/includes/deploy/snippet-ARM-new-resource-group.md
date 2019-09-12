---
ms.openlocfilehash: fd279af81e0c94ac22f54429cb8ae330e5d60661
ms.sourcegitcommit: dd12ddf408c010182b09da88e2aac0de124cef22
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/05/2019
ms.locfileid: "70386088"
---
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
