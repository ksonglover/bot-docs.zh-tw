---
ms.openlocfilehash: 88732d2d5490962d7a899d936767e7dd148e94c5
ms.sourcegitcommit: bdb981c0b11ee99d128e30ae0462705b2dae8572
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 01/17/2019
ms.locfileid: "54360835"
---
將本機 Bot 發佈至 Azure。 這個步驟可能需要一些時間。

```cmd
az bot publish --name <bot-resource-name> --proj-name "<project-file-name>" --resource-group <resource-group-name> --code-dir <directory-path> --verbose --version v4
```

| 選項 | 說明 |
|:---|:---|
| --name | Azure 中的 Bot 資源名稱。 |
| --proj-name | 對於 C#，使用需要發佈的啟動專案檔名 (不含 .csproj)。 例如： `EnterpriseBot` 。 對於 Node.js，使用 Bot 的主要進入點。 例如： `index.js`。 |
| --resource-group | 資源群組的名稱。 |
| --code-dir | 要從中上傳 Bot 程式碼的目錄。 |

一旦完成並顯示「部署成功！」 訊息，Bot 便已部署在 Azure 中。