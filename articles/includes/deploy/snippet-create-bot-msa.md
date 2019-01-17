---
ms.openlocfilehash: fed6222fb9cf2d7793776c5575dfbcda49b54224
ms.sourcegitcommit: bdb981c0b11ee99d128e30ae0462705b2dae8572
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 01/17/2019
ms.locfileid: "54360848"
---
1. 移至[**應用程式註冊入口網站**](https://apps.dev.microsoft.com/)。
1. 按一下 [新增應用程式] 以註冊您的應用程式、建立 **Application-id**，以及**產生新密碼**。 如果您已經有應用程式和密碼，但不記得密碼，則必須在應用程式祕密區段中產生新密碼。
1. 儲存應用程式識別碼和您剛產生的新密碼，讓您能用來搭配 `az bot create` 命令。  

```cmd
az bot create --kind webapp --name <bot-resource-name> --location <geographic-location> --version v4 --lang <language> --verbose --resource-group <resource-group-name> --appid "<application-id>" --password "<application-password>" --verbose
```

| 選項 | 說明 |
|:---|:---|
| --name | 在 Azure 中用於部署 Bot 的唯一名稱。 這可能與您的本機 Bot 同名。 請勿在名稱中包含空格或底線。 |
| --location | 用來建立 Bot 服務資源的地理位置。 例如，`eastus`、`westus` 及 `westus2` 等等。 |
| --lang | 要用來建立 Bot 的語言：`Csharp` 或 `Node`；預設值是 `Csharp`。 |
| --resource-group | 要在其中建立 Bot 的資源群組名稱。 您可以使用 `az configure --defaults group=<name>` 來設定預設群組。 |
| --appid | 要用於 Bot 的 Microsoft 帳戶識別碼 (MSA ID)。 |
| --password | Bot 的 Microsoft 帳戶 (MSA) 密碼。 |
