---
ms.openlocfilehash: 80cc69ca3d863a4f96f1dd241e6471074cc9976f
ms.sourcegitcommit: dd12ddf408c010182b09da88e2aac0de124cef22
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/05/2019
ms.locfileid: "70385997"
---
使用未設定的 [zip 部署 API](https://github.com/projectkudu/kudu/wiki/Deploying-from-a-zip-file-or-url)來部署您 Bot 的程式碼時，Web 應用程式/Kudu 的行為會如下所示：

_根據預設，Kudu 會假設來自 zip 檔案的部署都已可供執行，而且部署期間不需要額外的建置步驟，例如 npm install 或 dotnet restore/dotnet publish。_

因此，務必在要部署至 Web 應用程式的 zip 檔案中包含建置程式碼和所有必要相依性，否則您的 Bot 不會如預期般運作。

> [!IMPORTANT]
> 壓縮專案檔之前，請確定您_位在_專案資料夾中。 
> - 針對 C# Bot，這是具有 .csproj 檔案的資料夾。 
> - 針對 JS Bot，這是具有 app.js 或 index.js 檔案的資料夾。 
>
>在專案資料夾**內**選取所有檔案並加以壓縮，然後在該資料夾中執行命令。 
>
> 如果您的根資料夾位置不正確，**Bot 將無法在 Azure 入口網站中執行**。