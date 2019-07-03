---
ms.openlocfilehash: f95c8a37b1207a26dab0a714b86412a9dba2dcb4
ms.sourcegitcommit: a47183f5d1c2b2454c4a06c0f292d7c075612cdd
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/19/2019
ms.locfileid: "67252634"
---
```cmd
az bot create --kind webapp --name <bot-resource-name> --location <geographic-location> --version v4 --lang <language> --verbose --resource-group <resource-group-name>
```

| 選項 | 說明 |
|:---|:---|
| --name | 在 Azure 中用於部署 Bot 的唯一名稱。 這可能與您的本機 Bot 同名。 請勿在名稱中包含空格或底線。 |
| --location | 用來建立 Bot 服務資源的地理位置。 例如，`eastus`、`westus` 及 `westus2` 等等。 |
| --lang | 要用來建立 Bot 的語言：`Csharp` 或 `Node`；預設值是 `Csharp`。 |
| --resource-group | 要在其中建立 Bot 的資源群組名稱。 您可以使用 `az configure --defaults group=<name>` 來設定預設群組。 |