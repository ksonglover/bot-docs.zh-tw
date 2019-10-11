---
ms.openlocfilehash: b3fcdea923daa96a93b7b9d223354a16878b0ac2
ms.sourcegitcommit: 7e901f5f39a0cfb0d37e532321b90a1dcf4baadd
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/08/2019
ms.locfileid: "72039785"
---
使用未設定的 [zip 部署 API](https://github.com/projectkudu/kudu/wiki/Deploying-from-a-zip-file-or-url)來部署您 Bot 的程式碼時，Web 應用程式/Kudu 的行為會如下所示：

_根據預設，Kudu 會假設來自 zip 檔案的部署都已可供執行，而且部署期間不需要額外的建置步驟，例如 npm install 或 dotnet restore/dotnet publish。_

因此，務必在要部署至 Web 應用程式的 zip 檔案中包含建置程式碼和所有必要相依性，否則您的 Bot 不會如預期般運作。

> [!IMPORTANT]
> 壓縮專案檔之前，請確定您_位在_專案資料夾中。 
> - 針對 C# Bot，這是具有 .csproj 檔案的資料夾。 
> - 針對 JavaScript Bot，這是具有 app.js 或 index.js 檔案的資料夾。 
> - 針對 TypeScript Bot，這是包含 _src_ 資料夾的資料夾( bot.ts 和 index.ts 檔案的所在位置)。 
>
>在專案資料夾**內**選取所有檔案並加以壓縮，然後在該資料夾中執行命令。 
>
> 如果您的根資料夾位置不正確，**Bot 將無法在 Azure 入口網站中執行**。
