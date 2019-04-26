---
title: 使用 .bot 檔案管理資源 | Microsoft Docs
description: 說明 Bot 檔案的用途與使用方式。
keywords: bot 檔案, .bot, .bot 檔案, bot 資源, 管理 bot 資源
author: ivorb
ms.author: v-ivorb
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 04/18/2019
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 65ad712a4d3cfeebb5c85375e023e301f0e101ca
ms.sourcegitcommit: aea57820b8a137047d59491b45320cf268043861
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 04/22/2019
ms.locfileid: "59904521"
---
# <a name="manage-bot-resources"></a>管理 Bot 資源

[!INCLUDE [applies-to-v4](../includes/applies-to.md)]

Bot 通常會取用不同的服務，例如 [LUIS.ai](https://luis.ai) 或 [QnaMaker.ai](https://qnamaker.ai)。 當您開發 Bot 時，您需要能夠追蹤一切。 您可以使用各種方法，例如 appsettings.json、web.config 或 .env。 

> [!IMPORTANT]
> 在 Bot Framework SDK 4.3 發行之前，我們提供 .bot 檔案作為管理資源的機制。 不過，我們建議您今後使用 appsettings.json 或 .env 檔案來管理這些資源。 即使 .bot 檔案已經「淘汰」，使用 .bot 檔案的 Bot 目前仍會繼續運作。 如果您已使用 .bot 檔案來管理資源，請遵循適用於遷移設定的步驟。 

## <a name="migrating-settings-from-bot-file"></a>從 .bot 檔案遷移設定
下列各節討論如何從 .bot 檔遷移設定。 請遵循您適用的案例。

**案例 1：具有 .bot 檔案的本機 Bot**

在此案例中，您具有使用 .bot 檔案的本機 Bot，但是「Bot 尚未遷移」至 Azure 入口網站。 請遵循下列步驟，將設定從 .bot 檔案遷移至 appsettings.json 或 .env 檔案。

- 如果 .bot 檔案已加密，您必須使用下列命令將其解密：

```cli
msbot secret --bot <name-of-bot-file> --secret "<bot-file-secret>" --clear` command.
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


## <a name="faq"></a>常見問題集
**問：** 我想要在 Azure 入口網站中建立新的 V4 Bot。 .bot 檔案移除對 Azure 入口網站體驗有何改變？

**答：** 當您在 Azure 入口網站中建立 Bot 時，將不會建立 .bot 檔案。 您可以使用 **Azure 入口網站**的 [應用程式設定] 區段來尋找識別碼/金鑰。 當您下載程式碼時，這些設定會儲存在 appsettings.json 或 .env 檔案中。 在您的 Bot 中，您可以先將程式碼更新為讀取這些設定，再對個別服務進行呼叫。 在您更新 Bot 程式碼之後，您可以使用 az bot publish 來部署 Bot。

**問：** V3 Bot 如何？

**答：** V3 Bot 案例類似於不含 Bot 檔案的 V4 Bot 案例。 部署程序將會繼續運作。 

## <a name="additional-resources"></a>其他資源
- 如需部署 Bot 的步驟，請參閱[部署](../bot-builder-deploy-az-cli.md)主題。
- 若要保護金鑰和祕密，建議您使用 Azure Key Vault。 Azure Key Vault 是可安全儲存及存取祕密 (例如 Bot 的端點和撰寫金鑰) 的工具。 其會提供[金鑰管理](https://docs.microsoft.com/en-us/azure/key-vault/key-vault-whatis)解決方案，讓您輕鬆地建立及控制加密金鑰，進而嚴格控制您的祕密。


<!--

# Manage resources with a .bot file

Bots usually consume lots of different services, such as [LUIS.ai](https://luis.ai) or [QnaMaker.ai](https://qnamaker.ai). When you are developing a bot, there is no uniform place to store the metadata about the services that are in use.  This prevents us from building tooling that looks at a bot as a whole.

To address this problem, we have created a **.bot file** to act as the place to bring all service references together in one place to 
enable tooling.  For example, the Bot Framework Emulator ([V4](https://aka.ms/Emulator-wiki-getting-started)) uses a  .bot file to create a unified view over the connected services your bot consumes.  

With a .bot file, you can register services like:

* **Localhost** local debugger endpoints
* [**Azure Bot Service**](https://azure.microsoft.com/en-us/services/bot-service/) Azure Bot Service registrations.
* [**LUIS.AI**](https://www.luis.ai/) LUIS gives your bot the ability to communicate with people using natural language.. 
* [**QnA Maker**](https://qnamaker.ai/) Build, train and publish a simple question and answer bot based on FAQ URLs, structured documents or editorial content in minutes.
* [**Dispatch**](https://github.com/Microsoft/botbuilder-tools/tree/master/packages/Dispatch) models for dispatching across multiple services.
* [**Azure Application Insights**](https://azure.microsoft.com/en-us/services/application-insights/) for insights and bot analytics.
* [**Azure Blob Storage**](https://azure.microsoft.com/en-us/services/storage/blobs/) for bot state persistence. 
* [**Azure Cosmos DB**](https://azure.microsoft.com/en-us/services/cosmos-db/) - globally distributed, multi-model database service to persist bot state.

Apart from these, your bot might rely on other custom services. You can leverage the [generic service](https://github.com/Microsoft/botbuilder-tools/blob/master/packages/MSBot/docs/add-services.md) capability to connect a generic service configuration.

## When is a .bot file created? 
- If you create a bot using [Azure Bot Service](https://ms.portal.azure.com/#blade/Microsoft_Azure_Marketplace/GalleryResultsListBlade/selectedSubMenuItemId/%7B%22menuItemId%22%3A%22gallery%2FCognitiveServices_MP%2FBotService%22%2C%22resourceGroupId%22%3A%22%22%2C%22resourceGroupLocation%22%3A%22%22%2C%22dontDiscardJourney%22%3Afalse%2C%22launchingContext%22%3A%7B%22source%22%3A%5B%22GalleryFeaturedMenuItemPart%22%5D%2C%22menuItemId%22%3A%22CognitiveServices_MP%22%2C%22subMenuItemId%22%3A%22BotService%22%7D%7D), a .bot file is automatically created for you with list of connected services provisioned. The .bot is encrypted by default.
- If you create a bot using Bot Framework V4 SDK [Template](https://marketplace.visualstudio.com/items?itemName=BotBuilder.botbuilderv4) for Visual Studio or using Bot Builder [Yeoman Generator](https://www.npmjs.com/package/generator-botbuilder), a .bot file is automatically created. No connected services are provisioned in this flow and the bot file is not encrypted.
- If you are starting with [BotBuilder-samples](https://github.com/Microsoft/botbuilder-samples), every sample for Bot Framework V4 SDK includes a .bot file and the .bot file is not encrypted. 
- You can also create a bot file using the [MSBot](https://github.com/Microsoft/botbuilder-tools/blob/master/packages/MSBot/README.md) tool.

## What does a bot file look like? 
Take a look at a sample [.bot](https://github.com/Microsoft/botbuilder-tools/blob/master/packages/MSBot/docs/sample-bot-file.json) file.
To learn about encrypting and decrypting the .bot file, see [Bot Secrets](https://github.com/Microsoft/botbuilder-tools/blob/master/packages/MSBot/docs/bot-file-encryption.md).

## Why do I need a .bot file?

A .bot file is **not** a requirement to build bots with Bot Framework SDK. You can continue to use appsettings.json, web.config, env, 
keyvault or any mechanism you see fit to keep track of service references and keys that your bot depends on. However, to test
the bot using the Emulator, you'll need a .bot file. The good news is that Emulator can create a .bot file for testing. To do that, 
start the Emulator, click on the **create a new bot configuration** link on the Welcome page. In the dialog box that appears, type a **Bot name** and an **Endpoint URL**. Then connect.

The advantages of using .bot file are:
- Provides a standard way of storing resources regardless of the language/platform you use.   
- Bot Framework Emulator and CLI tools rely on and work great with tracking connected services in a consistent format (in a .bot file) 
- Elegant tooling solutions around services creation and management is harder without a well defined schema (.bot file).  


## Using .bot file in your Bot Framework SDK bot

You can use the .bot file to get service configuration information in your bot's code. The BotFramework-Configuration library available 
for [C#](https://www.nuget.org/packages/Microsoft.Bot.Configuration) and [JS](https://www.npmjs.com/package/botframework-config) helps you load a bot file and supports several methods to query and get the appropriate service configuration information.

## Additional resources
Refer to [MSBot](https://github.com/Microsoft/botbuilder-tools/blob/master/packages/MSBot/README.md) readme file for more information on using a bot file.

-->

