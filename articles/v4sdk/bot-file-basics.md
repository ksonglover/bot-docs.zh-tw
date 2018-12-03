---
title: 使用 .bot 檔案管理資源 | Microsoft Docs
description: 說明 Bot 檔案的用途與使用方式。
keywords: bot 檔案, .bot, .bot 檔案, msbot, bot 資源, 管理 bot 資源
author: ivorb
ms.author: v-ivorb
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 11/23/2018
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 276b553a6990ed286acbf073825afa7c4656de32
ms.sourcegitcommit: 6c719b51c9e4e84f5642100a33fe346b21360e8a
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 11/28/2018
ms.locfileid: "52452020"
---
# <a name="manage-resources-with-a-bot-file"></a>使用 .bot 檔案管理資源

Bot 通常會耗用很多不同的服務，例如 [LUIS.ai](https://luis.ai) 或 [QnaMaker.ai](https://qnamaker.ai)。 當您開發 bot 時，沒有統一位置可儲存使用中服務的相關中繼資料。  這導致我們無法建置可整體查看 Bot 的工具。

為了解決此問題，我們建立了 **.bot 檔案** 作為將所有服務參考匯合在一起的位置，以啟用工具功能。  比方說，Bot Framework 模擬器 ([V4](https://aka.ms/Emulator-wiki-getting-started)) 使用 .bot 檔案，建立 Bot 所用連線服務的統一檢視。  

使用 .bot 檔案，您可以註冊一些服務，例如：

* **Localhost** 本機偵錯工具端點
* [**Azure Bot Service**](https://azure.microsoft.com/en-us/services/bot-service/) Azure Bot Service 註冊。
* [**LUIS.AI**](https://www.luis.ai/) LUIS 讓 Bot 能夠與使用自然語言的人員通訊。 
* [**QnA Maker**](https://qnamaker.ai/) 根據常見問題集 URL、結構化文件或期刊內容，在短短幾分鐘內建置、定型及發佈簡單的問答 Bot。
* [**分派**](https://github.com/Microsoft/botbuilder-tools/tree/master/Dispatch) 模型，可供分派於多項服務。
* [**Azure Application Insights**](https://azure.microsoft.com/en-us/services/application-insights/)，可提供見解和 Bot 分析。
* [**Azure Blob 儲存體**](https://azure.microsoft.com/en-us/services/storage/blobs/)可供保存 Bot 狀態。 
* [**Azure Cosmos DB**](https://azure.microsoft.com/en-us/services/cosmos-db/) - 全球發行的多模型資料庫服務，用以保存 Bot 狀態。

除了這些以外，Bot 還可能依賴其他自訂服務。 您可以利用[泛型服務](https://github.com/Microsoft/botbuilder-tools/blob/master/packages/MSBot/docs/add-services.md)功能來連線一般服務組態。

## <a name="when-is-a-bot-file-created"></a>何時建立 .bot 檔案？ 
- 如果您使用 [Azure Bot Service](https://ms.portal.azure.com/#blade/Microsoft_Azure_Marketplace/GalleryResultsListBlade/selectedSubMenuItemId/%7B%22menuItemId%22%3A%22gallery%2FCognitiveServices_MP%2FBotService%22%2C%22resourceGroupId%22%3A%22%22%2C%22resourceGroupLocation%22%3A%22%22%2C%22dontDiscardJourney%22%3Afalse%2C%22launchingContext%22%3A%7B%22source%22%3A%5B%22GalleryFeaturedMenuItemPart%22%5D%2C%22menuItemId%22%3A%22CognitiveServices_MP%22%2C%22subMenuItemId%22%3A%22BotService%22%7D%7D) 建立 Bot，系統就會為您自動建立 .bot 檔案，其中已佈建連線服務清單。 預設會加密 .bot。
- 如果您使用適用於 Visual Studio 的 Bot Builder V4 SDK [範本](https://marketplace.visualstudio.com/items?itemName=BotBuilder.botbuilderv4)或使用 Bot Builder [Yeoman Generator](https://www.npmjs.com/package/generator-botbuilder) 建立 Bot，則會自動建立 .bot 檔案。 此流程中不會佈建任何連線服務，而且 Bot 檔案不會加密。
- 如果您開始使用 [BotBuilder-samples](https://github.com/Microsoft/botbuilder-samples)，Bot Builder V4 SDK 的每個範例都會包含 .bot 檔案，且 .bot 檔案不會加密。 
- 您也可以使用 [MSBot](https://github.com/Microsoft/botbuilder-tools/blob/master/packages/MSBot/README.md) 工具建立 Bot 檔案。

## <a name="what-does-a-bot-file-look-like"></a>bot 檔案的外觀為何？ 
讓我們看一下範例 [.bot](https://github.com/Microsoft/botbuilder-tools/blob/master/packages/MSBot/docs/sample-bot-file.json) 檔案。
若要深入了解 .bot 檔案的加密和解密，請參閱 [Bot 祕密](https://github.com/Microsoft/botbuilder-tools/blob/master/packages/MSBot/docs/bot-file-encryption.md)。

## <a name="why-do-i-need-a-bot-file"></a>為何需要 .bot 檔案？
使用 Bot Builder SDK 建置 Bot 時，**不**需要 .bot 檔案。 您可以繼續使用 appsettings.json、web.config、env、keyvault 或任何適當機制，追蹤您的 Bot 所依賴的服務參考和索引鍵。 不過，若要使用模擬器測試 Bot，您會需要 .bot 檔案。 好消息是模擬器可以建立 .bot 檔案以供測試。 若要這樣做，請啟動模擬器，按一下 [歡迎] 頁面上的 [建立新的 Bot 組態] 連結。 在出現的對話方塊中，輸入 [Bot 名稱] 和 [端點 URL]。 然後連線。

使用 .bot 檔案的優點：
- 不論您使用的語言/平台為何，請提供儲存資源的標準方式。   
- Bot Framework 模擬器和 CLI 工具會依賴且妥善處理以一致的格式 (.bot 檔案) 追蹤連線服務。 
- 若沒有定義完善的結構描述 (.bot 檔案)，難以打造以服務建立和管理為主的精緻工具解決方案。  

## <a name="using-bot-file-in-your-bot-builder-sdk-bot"></a>在 Bot Builder SDK Bot 中使用 .bot 檔案
您可以使用 .bot 檔案來取得 Bot 程式碼中的服務組態資訊。 可供 [C#](https://www.nuget.org/packages/Microsoft.Bot.Configuration) 和 [JS](https://www.npmjs.com/package/botframework-config) 使用的 BotFramework-Configuration 程式庫可協助您載入 Bot 檔案，並支援數種方法來查詢和取得適當的服務組態資訊。

## <a name="additional-resources"></a>其他資源
如需使用 bot 檔案的詳細資訊，請參閱 [MSBot](https://github.com/Microsoft/botbuilder-tools/blob/master/packages/MSBot/README.md) 讀我檔案。
