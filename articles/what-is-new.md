---
title: 新功能 | Microsoft Docs
description: 了解 Bot Framework 中的新功能。
keywords: bot framework, azure bot service
author: ivorb
ms.author: v-ivorb
manager: kamrani
ms.topic: get-started-article
ms.service: bot-service
ms.subservice: abs
ms.date: 07/17/2019
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 1bbea2f12af976a9d967e7c62baf416b8938f8aa
ms.sourcegitcommit: b053c0ca7f2e9e60679f7e82e583c57ae83fcb50
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/19/2019
ms.locfileid: "68336745"
---
# <a name="whats-new-in-bot-framework-july-2019"></a>Bot Framework 中的新功能 (2019 年 7 月)

[!INCLUDE[applies-to](includes/applies-to.md)]

Bot Framework SDK v4 是[開放原始碼 SDK][1a]，可讓開發人員使用其慣用的程式設計語言塑造與建置複雜的對話。

本文摘要說明 Bot Framework 和 Azure Bot Service 中的重要新功能和增強功能。

|   | C#  | JS  | Python |   
|---|:---:|:---:|:------:|
|SDK |[4.5][1] | [4.5][2] | [4.4.0 b2 (預覽)][3] | 
|Docs | [docs][5] |[docs][5] |  | |
|範例 |[.NET Core][6], [WebAPI][10] |[Node.js][7] , [TypeScript][8]、[es6][9]  | [Python][111] | | 

[1a]:https://github.com/microsoft/botframework-sdk/#readme
[1]:https://github.com/Microsoft/botbuilder-dotnet/#packages
[2]:https://github.com/Microsoft/botbuilder-js#packages
[3]:https://github.com/Microsoft/botbuilder-python#packages
[5]:https://docs.microsoft.com/azure/bot-service/?view=azure-bot-service-4.0
[6]:https://github.com/Microsoft/BotBuilder-Samples/tree/master/samples/csharp_dotnetcore
[7]:https://github.com/Microsoft/BotBuilder-Samples/tree/master/samples/javascript_nodejs
[8]:https://github.com/Microsoft/BotBuilder-Samples/tree/master/samples/javascript_typescript
[9]:https://github.com/Microsoft/BotBuilder-Samples/tree/master/samples/javascript_es6
[10]:https://github.com/Microsoft/BotBuilder-Samples/tree/master/samples/csharp_webapi
[111]:https://github.com/Microsoft/botbuilder-python/tree/master/samples


## <a name="bot-framework-channels"></a>Bot Framework 通道
- [Direct Line Speech (公開預覽)](https://aka.ms/streaming-extensions) | [docs](https://docs.microsoft.com/azure/bot-service/directline-speech-bot?view=azure-bot-service-4.0)：Bot Framework 和 Microsoft 的語音服務都會提供通道，以便使用 WebSockets 在用戶端與 Bot 應用程式之間雙向串流語音和文字。  

- [Direct Line App Service 擴充功能 (公開預覽)](https://portal.azure.com) | [docs](https://aka.ms/directline-ase)：可讓用戶端使用 Direct Line API 直接連線到 Bot 的 Direct Line 版本。 這可提供許多優點，包括提升的效能和更多隔離。 Direct Line App Service 擴充功能適用於所有的 Azure App Service，包括 Azure App Service 環境內裝載的服務。 Azure App Service 環境可提供隔離，且很適合在 VNet 中運作。 VNet 可讓您在 Azure 中建立自己的私人空間，而對您的雲端網路而言非常重要，因為它可提供隔離、分割和其他主要優點。 

## <a name="bot-framework-sdk"></a>Bot Framework SDK
- [自適性對話方塊 (SDK v4.6 預覽)](https://github.com/Microsoft/BotBuilder-Samples/tree/master/experimental/adaptive-dialog#readme) | [docs](https://github.com/Microsoft/BotBuilder-Samples/tree/master/experimental/adaptive-dialog/docs) | [C# 範例](https://github.com/Microsoft/BotBuilder-Samples/tree/master/experimental/adaptive-dialog/csharp_dotnetcore)：自適性對話方塊現在可讓開發人員根據內容和事件來動態更新交談流程。 在處理交談過程中的交談環境切換和中斷情形時，這特別有用。 
  
- [Bot Framework Python SDK (預覽 2)](https://github.com/microsoft/botbuilder-python) | [範例](https://github.com/Microsoft/botbuilder-python/tree/master/samples)：Python SDK 現在支援 OAuth、提示、CosmosDB，且包含 SDK 4.5 中的所有主要功能。 此外，這些範例也可協助您了解 SDK 中的新功能。

## <a name="bot-framework-testing"></a>Bot Framework 測試
- [單元測試](http://aka.ms/bot-test-package) | [docs](https://aka.ms/testing-framework) | [C# 範例](https://aka.ms/corebot-test) | [JS 範例](https://aka.ms/js-core-test-sample)：為因應客戶和開發人員對於更佳測試工具的詢問，7 月版的 SDK 引進了新的單元測試功能。 Microsoft.Bot.Builder.testing 套件可簡化 Bot 中單元測試對話的程序。 

- [通道測試](https://github.com/Microsoft/BotFramework-Emulator/releases)：Microsoft Build 2019 引進了 Bot Inspector，這是 Bot Framework Emulator 中的一項新功能，可讓您在 Microsoft Teams、Slack、Cortana 等通道上對 Bot 進行偵錯和測試。 當您在特定通道上使用 Bot 時，訊息便會鏡像傳送到 Bot Framework Emulator，您可以在其中檢查 Bot 所收到的訊息資料。 此外，也會呈現通道和 Bot 之間任何給定回合的 Bot 記憶體狀態快照集。

## <a name="web-chat"></a>網路聊天
- 根據企業客戶的詢問，我們新增了一個[網路聊天範例](https://github.com/microsoft/BotFramework-WebChat/tree/master/samples/19.a.single-sign-on-for-enterprise-apps#single-sign-on-demo-for-enterprise-apps-using-oauth)，示範如何授權使用者透過 Bot 存取企業應用程式上的資源。 有兩種類型的資源可用來示範 OAuth 與 Microsoft Graph 和 GitHub API 的互通性。

## <a name="solutions"></a>解決方案
- [虛擬助理解決方案加速器](https://github.com/Microsoft/botframework-solutions#readme)：提供一組範本、解決方案加速器和技能，協助您打造精細的交談式體驗。 適用於虛擬助理的新 Android 應用程式用戶端，該用戶端與 Direct-Line Speech 和虛擬助理整合，可示範裝置用戶端如何與虛擬助理互動及呈現調適型卡片。 更新也包含 Direct-Line Speech 和 Microsoft Teams 的支援。
  
- [Dynamics 365 Virtual Agent for Customer Service (公開預覽)](https://dynamics.microsoft.com/en-us/ai/virtual-agent-for-customer-service/)：使用公開預覽版，您可以透過智慧型、可調式虛擬代理程式提供出色的客戶服務。 客戶服務專家可以透過 AI 驅動的深入解析，輕鬆地建立及增強 Bot。
  
- [Chat for Dynamics 365](https://www.powerobjects.com/powerpacks/powerchat/)：Chat for Dynamics 365 提供幾項功能，可確保支援專員和終端使用者可以有效地互動並保持高生產力。 在 Microsoft Dynamics 365 中進行即時聊天並追蹤來自網站訪客的交談。

## <a name="additional-information"></a>其他資訊
- 您可以在[這裡](what-is-new-archive.md)查看先前的公告。
