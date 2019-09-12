---
ms.openlocfilehash: db7c4b447c5a27fe2047cfa41e65fac0d3e3a86c
ms.sourcegitcommit: dd12ddf408c010182b09da88e2aac0de124cef22
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/05/2019
ms.locfileid: "70386056"
---
當您使用 [Visual Studio 範本](https://docs.microsoft.com/azure/bot-service/dotnet/bot-builder-dotnet-sdk-quickstart?view=azure-bot-service-4.0)或 [Yeoman 範本](https://docs.microsoft.com/azure/bot-service/javascript/bot-builder-javascript-quickstart?view=azure-bot-service-4.0)建立 Bot 時，產生的原始程式碼中會有包含 ARM 範本的 `deploymentTemplates` 資料夾。 此處說明的部署程序會使用其中一個 ARM 範本，然後以 Azure CLI 在 Azure 中佈建 Bot 所需的資源。 

> [!NOTE]
> 在 Bot Framework SDK 4.3 版本中，我們已_不再_使用 .bot 檔案。 我們會使用 appsettings.json 或 .env 檔案來管理 Bot 資源。 若要了解如何將設定從 .bot 檔案遷移至 appsettings.json 或 .env 檔案，請參閱[管理 Bot 資源](https://docs.microsoft.com/azure/bot-service/bot-file-basics?view=azure-bot-service-4.0)。
