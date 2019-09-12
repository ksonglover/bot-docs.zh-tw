---
ms.openlocfilehash: c664749bc6ad63d1b0f60e001b2603898e58a9ea
ms.sourcegitcommit: dd12ddf408c010182b09da88e2aac0de124cef22
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/05/2019
ms.locfileid: "70386065"
---
在本機建立並測試 Bot 之後，您即可將其部署至 Azure。 開啟命令提示字元以登入 Azure 入口網站。

```cmd
az login
```
瀏覽器視窗會隨即開啟，以便您登入。

> [!NOTE]
> 如果您將 Bot 部署至非 Azure 雲端 (例如 US Gov)，您必須在 `az login` 之前執行 `az cloud set --name <name-of-cloud>`，其中，&lt;name-of-cloud> 是已註冊的雲端名稱，例如 `AzureUSGovernment`。 如果您要返回公用雲端，可以執行 `az cloud set --name AzureCloud`。 
