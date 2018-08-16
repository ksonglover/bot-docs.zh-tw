---
title: 開始使用 Bot Builder 來建置 Bot | Microsoft Docs
description: 開始使用 Bot Builder 來建置功能強大的 Bot。
author: robstand
ms.author: kamrani
manager: kamrani
ms.topic: get-started-article
ms.prod: bot-framework
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: 21186c5d3b0769311e4703ca1dab2f48a0a0081a
ms.sourcegitcommit: 67445b42796d90661afc643c6bb6533e9a662cbc
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/06/2018
ms.locfileid: "39574884"
---
# <a name="develop-bots-with-bot-builder"></a>使用 Bot Builder 來開發 Bot

[!INCLUDE [pre-release-label](includes/pre-release-label-v3.md)]

Bot Builder 提供 SDL、程式庫、範本與工具來協助您建置 Bot 及針對 Bot 進行偵錯。 當您使用 Bot Service 建置 Bot 時，您的 Bot 由 Bot Builder SDK 支援。 您也可以使用 Bot Builder SDK 利用 C# 或 Node.js 從頭開始建立 Bot。 Bot Builder 包括可用於測試 Bot 的 Bot Framework Emulator，以及與用於在不同通道上預覽 Bot 使用者體驗的 Channel Inspector。

若要使用您偏愛的任何程式設計語言來建置 Bot，您可以使用 REST API。

## <a name="bot-builder-sdk-for-net"></a>適用於 .NET 的 Bot Builder SDK
適用於 .NET 的 Bot Builder SDK 利用 C# 來為 .NET 開發人員提供撰寫 Bot 的熟悉方式。 這個功能強大的架構可用於建構 Bot 以處理自由格式互動與功能更強的導向式交談，在這些案例中，使用者可以從可能的值選取。 

[.NET 快速入門](~/dotnet/bot-builder-dotnet-quickstart.md)將引導您用適用於 .NET 的 Bot Builder SDK 來建立 Bot。

適用於 .NET 的 Bot Builder SDK 是以 [NuGet 套件](https://www.nuget.org/packages/Microsoft.Bot.Builder/)的形式提供。

> [!IMPORTANT]
> 適用於 .NET 的 Bot Builder SDK 需要 .NET Framework 4.6 或更新版本。 若您將 SDK 新增到以較低 .NET Framework 版本為目標的專案，您將必須先將您的專案更新為以 .NET Framework 4.6 為目標。

若要在現有的 Visual Studio 專案中安裝 SDK，請完成下列步驟：

1. 在 [方案總管] 中，以滑鼠右鍵按一下專案名稱，然後選取 [管理 NuGet 套件]。
2. 在 [瀏覽] 索引標籤上的搜尋方塊中輸入 "Microsoft.Bot.Builder"。
3. 選取結果清單中的 [Microsoft.Bot.Builder]，按一下 [安裝]，並接受變更。

您也可以從 GitHub 下載適用於 .NET 的 Bot Builder SDK [原始程式碼](https://github.com/Microsoft/BotBuilder/tree/master/CSharp)。

### <a name="visual-studio-project-templates"></a>Visual Studio 專案範本
下載 Visual Studio 專案範本以加快 Bot 開發速度。

* [適用 Visual Studio 的 Bot 範本][bot-template] 可用於使用 C# 來開發 Bot
* [適用於 Visual Studio 的 Cortana 技能範本][cortana-template] 可用於使用 C# 來開發 Cortana 技能

> [!TIP]
> <a href="/visualstudio/ide/how-to-locate-and-organize-project-and-item-templates" target="_blank">深入了解</a>如何安裝 Visual Studio 2017 專案範本。

## <a name="bot-builder-sdk-for-nodejs"></a>適用於 Node.js 的 Bot Builder SDK
適用於 Node.js 的 Bot Builder SDK 為 Node.js 開發人員提供熟悉的 Bot 撰寫方式。 您可以用它建立各種不同的對話式使用者介面，從簡單的提示到自由形式的對話都在其適用範圍內。 適用於 Node.js 的 Bot Builder SDK 會使用 restify 這個受歡迎的 Web 服務建立架構，來建立 Bot 的 Web 伺服器。 SDK 也與 Express 相容。 

[Node.js 快速入門](~/nodejs/bot-builder-nodejs-quickstart.md)將引導您用適用於 Node.js 的 Bot Builder SDK 來建立 Bot。 

適用於 Node.js 的 Bot Builder SDK 是以 npm 套件的形式提供。 若要安裝適用於 Node.js 的 Bot Builder SDK 與其相依組件，請先為您的 Bot 建立資料夾並瀏覽到該資料夾，然後執行下列 **npm** 命令：

```nodejs
npm init
```

接著，執行下列 **npm** 命令以安裝適用於 Node.js 的 Bot Builder SDK 與 <a href="http://restify.com/" target="_blank">Restify</a> 模組：

```nodejs
npm install --save botbuilder
npm install --save restify
```

您也可以從 GitHub 下載適用於 Node.js 的 Bot Builder SDK [原始程式碼](https://github.com/Microsoft/BotBuilder/tree/master/Node)。

## <a name="rest-api"></a>REST API

您可以利用 Bot Framework REST API，以任何程式設計語言來建立 Bot。 [REST 快速入門](rest-api/bot-framework-rest-connector-quickstart.md)將引導您用 REST 來建立 Bot。

Bot Framework 中有三種 REST API：

 - [Bot 連接器 REST API][connectorAPI] 可讓您的 Bot 傳送訊息到 [Bot Framework 入口網站](https://dev.botframework.com/) 中所設定的通道並從該通道接收訊息。 

- [Bot State REST API][stateAPI] 可讓您的 Bot 儲存及擷取與透過 Bot 連接器 REST API 進行之交談相關聯的狀態。

- [直接線路 REST API][directLineAPI] 可讓您將自己的應用程式 (例如用戶端應用程式、Web 聊天控制項或行動裝置應用程式) 直接連結到單一 Bot。

## <a name="bot-framework-emulator"></a>Bot Framework 模擬器
「Bot Framework 模擬器」這個傳統型應用程式可以讓您在本機或從遠端測試您的 Bot 以及針對其進行偵錯。 使用此模擬器時，您能與您的 Bot 聊天，並檢查您的 Bot 所傳送及接收的訊息。 [深入了解 Bot Framework 模擬器](~/bot-service-debug-emulator.md)及[下載適用於您平台的](http://emulator.botframework.com)。

## <a name="bot-framework-channel-inspector"></a>Bot Framework Channel Inspector
使用 Bot Framework [Channel Inspector](bot-service-channel-inspector.md) 快速查看 Bot 功能在不同通道上看起來的樣子。

[bot-template]: http://aka.ms/bf-bc-vstemplate
[cortana-template]: https://aka.ms/bf-cortanaskill-template


[connectorAPI]: https://docs.botframework.com/en-us/restapi/connector/#navtitle
 
[stateAPI]: https://docs.botframework.com/en-us/restapi/state/#navtitle

[directLineAPI]: https://docs.botframework.com/en-us/restapi/directline3/#navtitle
