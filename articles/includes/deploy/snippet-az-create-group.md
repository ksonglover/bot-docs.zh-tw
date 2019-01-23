---
ms.openlocfilehash: 53db401b00e53b964027f08c32ca7a38a4915f60
ms.sourcegitcommit: bdb981c0b11ee99d128e30ae0462705b2dae8572
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 01/17/2019
ms.locfileid: "54360845"
---
```cmd
az group create --name <resource-group-name> --location <geographic-location> --verbose
```

| 選項 | 說明 |
|:---|:---|
| --name | 資源群組的唯一名稱。 請勿在名稱中包含空格或底線。 |
| --location | 用來建立資源群組的地理位置。 例如，`eastus`、`westus` 及 `westus2` 等等。 使用 `az account list-locations` 來取得位置清單。 |