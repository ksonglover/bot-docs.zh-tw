---
ms.openlocfilehash: 8073e5f635d20de457e1bf5880c1b5c4c564fab4
ms.sourcegitcommit: dd12ddf408c010182b09da88e2aac0de124cef22
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/05/2019
ms.locfileid: "70386037"
---
您必須先備妥專案檔，才能部署 Bot。 
<!-- **C# bots** -->
##### <a name="ctabcsharp"></a>[C#](#tab/csharp)

```cmd
az bot prepare-deploy --lang Csharp --code-dir "." --proj-file-path "MyBot.csproj"
```

您必須提供與 --code-dir 對應的 .csproj 檔案路徑。 這可以透過 --proj-file-path 引數來執行。 此命令會將 --code-dir 和 --proj-file-path 解析為 "./MyBot.csproj"

<!-- **JavaScript bots** -->
##### <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

```cmd
az bot prepare-deploy --code-dir "." --lang Javascript
```

此命令會擷取 web.config，這是讓 Node.js 應用程式可以在 Azure App Service 上與 IIS 搭配運作的必要項目。 請確定 web.config 已儲存到您 Bot 的根目錄。

<!-- **TypeScript bots** -->
##### <a name="typescripttabtypescript"></a>[TypeScript](#tab/typescript)

```cmd
az bot prepare-deploy --code-dir "." --lang Typescript
```

此命令的運作方式類似上述的 JavaScript，但這適用於 Typescript Bot。

---

> [!NOTE]
> `az bot prepare-depoloy` 命令應該會在您的 Bot 專案資料夾中產生 `.deployment` 檔案。
