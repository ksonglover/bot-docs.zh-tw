---
ms.openlocfilehash: 0b991c438c0006d1fb4bafa90982f73f4a18be77
ms.sourcegitcommit: fa6e775dcf95a4253ad854796f5906f33af05a42
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/16/2019
ms.locfileid: "68230666"
---
Bot Builder Framework 可讓您的 Bot 儲存和擷取與使用者、對話或特定對話內容中的特定使用者相關聯的狀態資料。 狀態資料有許多用途，例如判斷上一個對話最後的內容，或是單純地以名字問候回來的使用者。 如果您儲存使用者的喜好設定，下次聊天時就可以使用該資訊來自訂對話。 例如，您可能會在有和使用者感興趣之主題相關的新聞文章，或有可用約會時警示使用者。 

針對測試和原型設計目的，您可以使用 Bot Builder Framework 的記憶體內部資料儲存體。 針對生產環境的 Bot，您可以實作自己的儲存體配接器，或使用其中一個 Azure 擴充功能。 Azure 擴充功能可讓您將 Bot 的狀態資料儲存於表格儲存體、CosmosDB 或 SQL。 本文將示範如何使用記憶體內部儲存體配接器來儲存 Bot 的狀態資料。 

> [!IMPORTANT]
> 建議您不要將 Bot Framework 狀態服務 API 用於生產環境，因為未來版本可能會淘汰它。 建議您更新 Bot 程式碼，以針對測試目的使用記憶體內部儲存體配接器，或針對生產環境 Bot 使用其中一個 **Azure 擴充功能**。
