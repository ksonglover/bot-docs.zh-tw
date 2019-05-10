---
ms.openlocfilehash: eac6abae509d92ea4714bc01221f180ea575950f
ms.sourcegitcommit: 4ff7a8772124a567f43e2c3e13aded368c4002e3
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 05/03/2019
ms.locfileid: "65035742"
---
取得加密金鑰。

1. 登入 [Azure 入口網站](http://portal.azure.com/)。
1. 為您的 Bot 開啟 Web 應用程式 Bot 資源。
1. 開啟 Bot 的 [應用程式設定]。
1. 在 [應用程式設定] 視窗中，向下捲動至 [應用程式設定]。
1. 找出 **botFileSecret** 並複製其值。

將 .bot 檔案解密。

```cmd
msbot secret --bot <name-of-bot-file> --secret "<bot-file-secret>" --clear
```

| 選項 | 說明 |
|:---|:---|
| --bot | 下載的 .bot 檔案的相對路徑。 |
| --secret | 加密金鑰。 |

將已解密的 `.bot` 檔案複製到包含本機 Bot 專案的目錄，將您的 Bot 更新為使用這個新 `.bot` 檔案，然後移除舊的 `.bot` 檔案。

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

在 **appsettings.json** 中，更新 **botFilePath** 屬性，以指向您已新增至本機目錄的新 `.bot` 檔案。

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

在 **.env** 中，更新 **botFilePath** 屬性，以指向您已新增至本機目錄的新 `.bot` 檔案。

---

在您的 Bot 更新後，刪除已下載 Bot 的暫存目錄。