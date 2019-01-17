---
ms.openlocfilehash: f8aad539a2d1e415833609f66cd5b398c88206f1
ms.sourcegitcommit: bdb981c0b11ee99d128e30ae0462705b2dae8572
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 01/17/2019
ms.locfileid: "54360831"
---
開啟命令提示字元以登入 Azure 入口網站。

```cmd
az login
```

瀏覽器視窗會隨即開啟，以便您登入。

### <a name="set-the-subscription"></a>設定訂用帳戶

設定要使用的預設訂用帳戶。

```cmd
az account set --subscription "<azure-subscription>"
```

如果您不確定哪個訂用帳戶要用於部署 Bot ，您可以使用 `az account list` 命令，檢視您帳戶的 `subscriptions` 清單。

瀏覽至 Bot 資料夾。

```cmd
cd <local-bot-folder>
```