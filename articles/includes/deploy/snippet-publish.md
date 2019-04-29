---
ms.openlocfilehash: bb7520e8e99ad6326d7d00d8190dae306bf11afa
ms.sourcegitcommit: aea57820b8a137047d59491b45320cf268043861
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 04/22/2019
ms.locfileid: "59905160"
---
將本機 Bot 發佈至 Azure。 這個步驟可能需要一些時間。

```cmd
az bot publish --name <bot-resource-name> --proj-file-path "<project-file-name>" --resource-group <resource-group-name> --code-dir <directory-path> --verbose --version v4
```

| 選項 | 說明 |
|:---|:---|
| --name | Azure 中的 Bot 資源名稱。 |
| --proj-file-path | 對於 C#，使用需要發佈的啟動專案檔名 (不含 .csproj)。 例如： `EnterpriseBot` 。 對於 Node.js，使用 Bot 的主要進入點。 例如： `index.js`。 |
| --resource-group | 資源群組的名稱。 |
| --code-dir | 要從中上傳 Bot 程式碼的目錄。 |

一旦完成並顯示「部署成功！」 訊息，Bot 便已部署在 Azure 中。
