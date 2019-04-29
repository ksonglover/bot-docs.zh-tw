---
ms.openlocfilehash: 0891b9652154f8ed086cc45ce6018aa0be1a67b8
ms.sourcegitcommit: aea57820b8a137047d59491b45320cf268043861
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 04/22/2019
ms.locfileid: "59905161"
---
若要將本機 JavaScript Bot 發佈回到 Azure，您必須先手動建立單一壓縮檔案，其中包含用來在本機建置及執行 Bot 的所有檔案。 這包括所有下載到 `node_modules` 資料夾中的所有 npm 程式庫。 建立此 zip 檔案時，_請確定您使用的根目錄是 index.js 檔案所在的相同目錄_。

建立包含所有 Bot 原始程式碼的 ZIP 檔案後，請開啟命令提示字元視窗並執行下列 _Az cli_ 命令。 

這個步驟可能需要一些時間。

```cmd
az webapp deployment source config-zip --resource-group <resource-group-name> --name <bot-resource-name> --src <directory-path>
```

| 選項 | 說明 |
|:---|:---|
| --resource-group | Azure 中資源群組的名稱。 |
| --name | Azure 中的 Bot 資源名稱。 |
| --src | 要從中上傳壓縮 Bot 程式碼的完整目錄路徑。 例如 `c:\my-local-repository\this-app-folder\my-zipped-code.zip` |

成功完成後，您的 Bot 就會部署在 Azure 中。
