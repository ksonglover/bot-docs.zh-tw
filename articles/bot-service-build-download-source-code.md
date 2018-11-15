---
title: 下載並重新部署 Bot 服務原始程式碼 | Microsoft Docs
description: 了解如何下載及發佈 Bot 服務。
keywords: download source code, redeploy, deploy, zip file, publish, 下載原始程式碼, 重新部署, 部署, 壓縮文件, 發佈
author: ivorb
ms.author: v-ivorb
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 09/26/2018
ms.openlocfilehash: 19dd474c16224cc811a214acea6e9cb51da95b3f
ms.sourcegitcommit: cb0b70d7cf1081b08eaf1fddb69f7db3b95b1b09
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 11/09/2018
ms.locfileid: "51332762"
---
# <a name="download-and-redeploy-bot-code"></a>下載並重新部署 Bot 程式碼
Azure Bot Service 允許您下載 Bot 的整個來源專案，以便您使用所選的 IDE 在本機作業。 程式碼更新完成後，您可以將變更發佈回到 Azure 入口網站。 我們將示範如何使用 Azure 入口網站和 `az` cli 下載程式碼。 我們也會說明如何使用 Visual Studio 和 `az` cli 工具重新部署已更新的 Bot 程式碼。 您可以選擇最適合您的方法。

## <a name="prerequisites"></a>必要條件
- 安裝 [Azure CLI](https://docs.microsoft.com/en-us/cli/azure/?view=azure-cli-latest)
- 使用 `az extension add -n botservice` 命令安裝 az botservice 擴充功能

### <a name="download-code-using-the-azure-portal"></a>使用 Azure 入口網站下載程式碼
若要從 [Azure 入口網站](https://portal.azure.com)下載程式碼，請執行下列作業：
1. 開啟 Bot 的刀鋒視窗。
1. 在 [Bot 管理] 區段下，按一下 [建置]。
1. 在 [下載原始程式碼] 下，按一下 [下載 zip 檔案]。
1. 等待 Azure 準備您的下載 URI，然後按一下通知中的 [下載 zip 檔案]。
1. 儲存 .zip 檔案並解壓縮至本機目錄。

如果您有 C# Bot，請更新 `appsettings.json` 檔案以納入 .bot 檔案資訊，如下所示：

```
{
  "botFilePath": "yourbasicBot.bot",
  "botFileSecret": "ukxxxxxxxxxxxs="
}
```

如果您有 node.js Bot，請新增含有下列項目的 `.env` 檔案：
```
botFilePath=yourbasicBot.bot
botFileSecret=ukxxxxxxxxxxxxs=
```

`botFilePath` 會參考您的 Bot 名稱，只要以您自己的 Bot 名稱取代 "yourbasicBot.bot" 即可。 若要取得 `botFileSecret` 金鑰，請參閱 [Bot 檔案加密](https://aka.ms/bot-file-encryption)一文，了解如何為您的 Bot 產生金鑰。

接下來，編輯現有的來源檔案，或在專案中新增來源檔案，對您的來源進行變更。 使用模擬器測試您的程式碼。 當您準備要將修改過的程式碼重新部署至 Azure 入口網站時，請遵循下列指示。

### <a name="publish-code-using-visual-studio"></a>使用 Visual Studio 發佈程式碼
1. 在 Visual Studio 中，以滑鼠右鍵按一下專案名稱，然後按一下 [發佈]。[發佈] 視窗隨即開啟。

![Azure 發佈](~/media/azure-bot-build/azure-csharp-publish.png)

2. 選取您專案的設定檔。
3. 在您的專案中複製 _publish.cmd_ 檔案中所列的密碼。
4. 按一下 [發佈] 。
5. 出現提示時，輸入您在步驟 3 複製的密碼。   

設定您的專案之後，您的專案變更會發佈至 Azure。 

接下來，讓我們了解如何使用 `az` cli 來下載和重新部署程式碼。

### <a name="download-code-using-azure-cli"></a>使用 Azure CLI 下載程式碼

首先，使用 az cli 工具登入 Azure 入口網站。

```azcli
az login
```

系統會提示您提供唯一的暫時性驗證碼。 若要登入，請使用網頁瀏覽器瀏覽 Microsoft [裝置登入](https://microsoft.com/devicelogin)，並貼上 CLI 所提供的程式碼，以繼續操作。

若要使用 `az` cli 上傳程式碼，請使用下列命令：
```azcli
az bot download --name "my-bot-name" --resource-group "my-resource-group"`
```
下載程式碼之後，請執行下列作業：
- 針對 C# Bot，更新 appsettings.json 檔案以納入 .bot 檔案資訊，如下所示：

```
{
  "botFilePath": "yourbasicBot.bot",
  "botFileSecret": "ukxxxxxxxxxxxs="
}
```

- 針對 node.js Bot，新增含有下列項目的 .env 檔案：

```
botFilePath=yourbasicBot.bot
botFileSecret=ukxxxxxxxxxxxxs=
```

接下來，編輯現有的來源檔案，或在專案中新增來源檔案，對您的來源進行變更。 使用模擬器測試您的程式碼。 當您準備要將修改過的程式碼重新部署至 Azure 入口網站時，請遵循下列指示。

### <a name="login-to-azure-cli-by-running-the-following-command"></a>執行下列命令以登入 Azure CLI。
如果您已經登入，則可略過此步驟。

```azcli
az login
```
系統會提示您提供唯一的暫時性驗證碼。 若要登入，請使用網頁瀏覽器瀏覽 Microsoft [裝置登入](https://microsoft.com/devicelogin)，並貼上 CLI 所提供的程式碼，以繼續操作。

### <a name="publish-code-using-azure-cli"></a>使用 Azure CLI 發佈程式碼
若要使用 `az` cli 將程式碼發佈回到 Azure，請使用下列命令：
```azcli
az bot publish --name "my-bot-name" --resource-group "my-resource-group" --code-dir <path to directory> 
```

您可以使用 `code-dir` 選項，指出要使用哪個目錄。 如未提供，`az bot publish` 命令會使用本機目錄來發佈。

## <a name="next-steps"></a>後續步驟
您現在已了解如何將變更上傳回到 Azure，即可為您的 Bot 設定持續部署。

> [!div class="nextstepaction"]
> [設定持續部署](bot-service-build-continuous-deployment.md)
