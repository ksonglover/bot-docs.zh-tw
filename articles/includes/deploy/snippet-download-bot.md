---
ms.openlocfilehash: 867e65b25878f810e3247eb3cace95f4d31e11db
ms.sourcegitcommit: a47183f5d1c2b2454c4a06c0f292d7c075612cdd
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/19/2019
ms.locfileid: "67252614"
---
使用您目前專案目錄以外的暫存目錄。 

此命令會在 save-path 之下建立子目錄；不過，指定的路徑必須已經存在。

```cmd
az bot download --name <bot-resource-name> --resource-group <resource-group-name> --save-path "<path>"
```

| 選項 | 說明 |
|:---|:---|
| --name | Azure 中的 Bot 名稱。 |
| --resource-group | Bot 所屬的資源群組名稱。 |
| --save-path | Bot 程式碼要下載至的現有目錄。 |