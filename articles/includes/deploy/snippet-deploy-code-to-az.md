---
ms.openlocfilehash: 0c28cf506f2701acfc705fdcc67ec6a1e7224ad1
ms.sourcegitcommit: dd12ddf408c010182b09da88e2aac0de124cef22
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/05/2019
ms.locfileid: "70386012"
---
此時，我們已準備好將程式碼部署至 Azure Web 應用程式。 從命令列執行下列命令，即可對 Web 應用程式使用 kudu zip 推送部署來執行部署。

```cmd
az webapp deployment source config-zip --resource-group "<resource-group-name>" --name "<name-of-web-app>" --src "code.zip" 
```

| 選項   | 說明 |
|:---------|:------------|
| resource-group | 包含 Bot 的 Azure 資源群組的名稱。 (這會是您為 Bot 建立應用程式註冊時所使用或建立的資源群組。) |
| name | 您稍早使用的 Web 應用程式名稱。 |
| src  | 您所建立的 zip 檔案路徑。 |
