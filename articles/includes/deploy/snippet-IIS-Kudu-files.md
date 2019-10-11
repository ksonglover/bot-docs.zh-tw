---
ms.openlocfilehash: bbe74a9a82d3bd04593384d825d373bfab35e3db
ms.sourcegitcommit: 7e901f5f39a0cfb0d37e532321b90a1dcf4baadd
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/08/2019
ms.locfileid: "72039773"
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
>  針對 C# Bots 和 JavaScript Bot，`az bot prepare-depoloy` 命令應該會在您的 Bot 專案資料夾中產生 `.deployment` 檔案。
> 針對 TypeScript Bot，此命令應該會產生兩個 `web.config` 檔案。 其中一個位於您的專案資料夾中，而另一個位於專案資料夾中的 **src** 資料夾內。 
