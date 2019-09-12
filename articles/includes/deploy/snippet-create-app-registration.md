---
ms.openlocfilehash: 28f49af1610f3c154e76d1cfcdb1333f08374f6c
ms.sourcegitcommit: dd12ddf408c010182b09da88e2aac0de124cef22
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/05/2019
ms.locfileid: "70386028"
---
註冊應用程式意謂著您可以使用 Azure AD 來驗證使用者，以及要求存取使用者資源。 在 Azure 中，您的 Bot 需要已註冊的應用程式來允許 Bot 存取 Bot Framework 服務，以便傳送和接收已驗證的訊息。 若要透過 Azure CLI 建立註冊應用程式，請執行下列命令：

```cmd
az ad app create --display-name "displayName" --password "AtLeastSixteenCharacters_0" --available-to-other-tenants
```

| 選項   | 說明 |
|:---------|:------------|
| display-name | 應用程式的顯示名稱。 |
| password | 應用程式密碼，也稱為「用戶端密碼」。 密碼必須至少有 16 個字元長，至少包含 1 個大寫或小寫字母，以及至少包含 1 個特殊字元。|
| available-to-other-tenants| 指出是否可從任何 Azure AD 租用戶使用應用程式。 設定為 `true` 可讓您的 Bot 使用 Azure Bot 服務通道。|

上述命令會輸出具有 `appId` 索引鍵的 JSON，請儲存此索引鍵的值以用於 ARM 部署，該值會用在 `appId` 參數上。 提供的密碼將會用於 `appSecret` 參數。

> [!NOTE] 
> 如果您想要使用現有的應用程式註冊，您可以使用命令：
> ``` cmd
> az bot create --kind webapp --resource-group "<name-of-resource-group>" --name "<name-of-web-app>" --appid "<existing-app-id>" --password "<existing-app-password>" --lang <Javascript|Csharp>
> ```
