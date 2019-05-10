---
title: 管理 Bot 資源 |Microsoft Docs
description: 說明 Bot 檔案的用途與使用方式。
keywords: bot 檔案, .bot, .bot 檔案, msbot, bot 資源, 管理 bot 資源
author: ivorb
ms.author: v-ivorb
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 05/01/2019
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 20b434c4fe5106ffe953c1a9ba9a282254511c9c
ms.sourcegitcommit: f84b56beecd41debe6baf056e98332f20b646bda
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 05/03/2019
ms.locfileid: "65032316"
---
# <a name="manage-bot-resources"></a>管理 Bot 資源

[!INCLUDE [applies-to-v4](../includes/applies-to.md)]

Bot 通常會取用不同的服務，例如 [LUIS.ai](https://luis.ai) 或 [QnaMaker.ai](https://qnamaker.ai)。 當您開發 Bot 時，您需要能夠追蹤一切。 您可以使用各種方法，例如 appsettings.json、web.config 或 .env。 

> [!IMPORTANT]
> 在 Bot Framework SDK 4.3 發行之前，我們提供 .bot 檔案作為管理資源的機制。 不過，我們建議您今後使用 appsettings.json 或 .env 檔案來管理這些資源。 即使 .bot 檔案已經「淘汰」，使用 .bot 檔案的 Bot 目前仍會繼續運作。 如果您已使用 .bot 檔案來管理資源，請遵循適用於遷移設定的步驟。 

## <a name="migrating-settings-from-bot-file"></a>從 .bot 檔案遷移設定
下列各節討論如何從 .bot 檔遷移設定。 請遵循您適用的案例。

**案例 #1：具有 .bot 檔案的本機 Bot**

在此案例中，您具有使用 .bot 檔案的本機 Bot，但是「Bot 尚未遷移」至 Azure 入口網站。 請遵循下列步驟，將設定從 .bot 檔案遷移至 appsettings.json 或 .env 檔案。

- 如果 .bot 檔案已加密，您必須使用下列命令將其解密：

```cli
msbot secret --bot <name-of-bot-file> --secret "<bot-file-secret>" --clear
```

- 開啟已解密的 .bot 檔案，複製其中的值並將其新增到 appsettings.json 或 .env 檔案。
- 從 appsettings.json 或 .env 檔案將程式碼更新為讀取設定。

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

在 `ConfigureServices` 方法中，使用 ASP.NET Core 提供的組態物件，例如： 

**Startup.cs**
```csharp
var appId = Configuration.GetSection("MicrosoftAppId").Value;
var appPassword = Configuration.GetSection("MicrosoftAppPassword").Value;
options.CredentialProvider = new SimpleCredentialProvider(appId, appPassword);
```
# <a name="javascripttabjs"></a>[JavaScript](#tab/js)

在 JavaScript 中，參考 `process.env` 物件外的 .env 變數，例如：
   
**index.js**

```js
const adapter = new BotFrameworkAdapter({
    appId: process.env.MicrosoftAppId,
    appPassword: process.env.MicrosoftAppPassword
});
```
---

如有需要，佈建資源並將其連線到使用 appsettings.json 或 .env 檔案的 Bot。

**案例 2：Bot 部署至具有 .bot 檔案的 Azure**

在此案例中，您已將 Bot 部署至使用 .bot 檔案的 Azure 入口網站，而您現在想要將設定從 .bot 遷移至 appsettings.json 或 .env 檔案。

- 從 Azure 入口網站下載 Bot 程式碼。 當您下載程式碼時，系統會提示您包含 appsettings.json 或 .env 檔案，此檔案會有您的 MicrosoftAppId 和 MicrosoftAppPassword 以及任何其他設定。 
- 開啟「已下載」的 appsettings.json 或 .env 檔案，並將其中的設定複製到「本機」 appsettings.json 或 .env 檔案。 別忘了從本機 appsettings.json 或 .env 檔案中移除 botSecret 和 botFilePath 項目。
- 從 appsettings.json 或 .env 檔案將程式碼更新為讀取設定。

# <a name="ctabcsharp"></a>[C#](#tab/csharp)
在 `ConfigureServices` 方法中，使用 ASP.NET Core 提供的組態物件，例如： 

**Startup.cs**
```csharp
var appId = Configuration.GetSection("MicrosoftAppId").Value;
var appPassword = Configuration.GetSection("MicrosoftAppPassword").Value;
options.CredentialProvider = new SimpleCredentialProvider(appId, appPassword);
```
# <a name="javascripttabjs"></a>[JavaScript](#tab/js)
在 JavaScript 中，參考 `process.env` 物件外的 .env 變數，例如：
   
**index.js**

```js
const adapter = new BotFrameworkAdapter({
    appId: process.env.MicrosoftAppId,
    appPassword: process.env.MicrosoftAppPassword
});
```
---

您也需要從 **Azure 入口網站**的 [應用程式設定] 區段中移除 `botFilePath` 和 `botFileSecret`。

如有需要，佈建資源並將其連線到使用 appsettings.json 或 .env 檔案的 Bot。

**案例 3：對於使用 appsettings.json 或 .env 檔案的 Bot**

此案例涵蓋以下情況：您使用 SDK 4.3 從頭開始開發 Bot，且沒有要遷移的現有 .bot 檔案。 您想要在 Bot 中使用的所有設定都位於 appsettings.json 或 .env 檔案中，如下所示：

```JSON
{
  "MicrosoftAppId": "<your-AppId>",
  "MicrosoftAppPassword": "<your-AppPwd>"
}
```

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

若要讀取 C# 程式碼中的上述設定，您可使用 ASP.NET Core 提供的組態物件，例如：**Startup.cs**
```csharp
var appId = Configuration.GetSection("MicrosoftAppId").Value;
var appPassword = Configuration.GetSection("MicrosoftAppPassword").Value;
options.CredentialProvider = new SimpleCredentialProvider(appId, appPassword);
```

# <a name="javascripttabjs"></a>[JavaScript](#tab/js)
在 JavaScript 中，參考 `process.env` 物件外的 .env 變數，例如：**index.js**
```js
const adapter = new BotFrameworkAdapter({
    appId: process.env.MicrosoftAppId,
    appPassword: process.env.MicrosoftAppPassword
});
```

---

如有需要，佈建資源並將其連線到使用 appsettings.json 或 .env 檔案的 Bot。

## <a name="additional-resources"></a>其他資源
- 如需部署 Bot 的步驟，請參閱[部署](../bot-builder-deploy-az-cli.md)主題。
- 了解如何使用 [Azure Key Vault](https://docs.microsoft.com/en-us/azure/key-vault/key-vault-overview) 來保護及管理雲端應用程式和服務所使用的密碼編譯金鑰和密碼。
