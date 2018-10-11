---
title: 使用適用於 .NET 的 Bot Builder SDK 建立 Bot | Microsoft Docs
description: 使用適用於 .NET 的 Bot Builder SDK 建立 Bot，這個 SDK 是功能強大的 Bot 建構架構。
keywords: Bot Builder SDK, 建立 Bot, 快速入門, .NET, 開始使用, C# Bot
author: kamrani
ms.author: kamrani
manager: kamrani
ms.topic: get-started-article
ms.prod: bot-framework
ms.date: 09/23/2018
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 5f3a02783242697fccf267bef2d56ed453880c67
ms.sourcegitcommit: 3bf3dbb1a440b3d83e58499c6a2ac116fe04b2f6
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/23/2018
ms.locfileid: "46707974"
---
# <a name="create-a-bot-with-the-bot-builder-sdk-for-net"></a>使用適用於 .NET 的 Bot Builder SDK 建立 Bot
[!INCLUDE [pre-release-label](../includes/pre-release-label.md)]

本快速入門會逐步引導您藉由使用 C# 範本，然後使用 Bot Framework Emulator 來測試，以建置 Bot。 

## <a name="prerequisites"></a>必要條件
- Visual Studio [2017](https://www.visualstudio.com/downloads)
- Bot Builder SDK 4 版的 [C#](https://botbuilder.myget.org/feed/aitemplates/package/vsix/BotBuilderV4.fbe0fc50-a6f1-4500-82a2-189314b7bea2) 範本
- Bot Framework [模擬器](https://github.com/Microsoft/BotFramework-Emulator/releases) (英文)
- 了解 [ASP.Net Core](https://docs.microsoft.com/aspnet/core/) 和 [C#](https://docs.microsoft.com/en-us/dotnet/csharp/programming-guide/concepts/async/index) 中的非同步程式設計

## <a name="create-a-bot"></a>建立 Bot
安裝您在必要條件區段下載的 BotBuilderVSIX.vsix 範本。 

在 Visual Studio 中建立新 Bot 專案。

![Visual Studio 專案](../media/azure-bot-quickstarts/bot-builder-dotnet-project.png)

> [!TIP] 
> 視需要更新 [NuGet 套件](https://docs.microsoft.com/en-us/nuget/quickstart/install-and-use-a-package-in-visual-studio)。

由於有範本，專案中會包含要在本快速入門建立 Bot 所需的所有程式碼。 您實際上不需要撰寫任何額外的程式碼。

## <a name="start-your-bot-in-visual-studio"></a>在 Visual Studio 中啟動 Bot

當您按一下執行按鈕時，Visual Studio 將會建置應用程式、將其部署到 localhost，並啟動 Web 瀏覽器來顯示應用程式的 `default.htm` 頁面。 此時，Bot 正在本機執行。

## <a name="start-the-emulator-and-connect-your-bot"></a>啟動模擬器並且連線至您的 Bot

接下來，請啟動模擬器，然後在模擬器中連線至您的 Bot：

1. 按一下模擬器 [歡迎使用] 索引標籤中的 [開啟 Bot] 連結。 
2. 選取位於您建立 Visual Studio 解決方案的目錄中的 .bot 檔案。

## <a name="interact-with-your-bot"></a>與您的 Bot 互動

傳送訊息給 Bot，Bot 就會以訊息回應。
![模擬器執行中](../media/emulator-v4/emulator-running.png)

## <a name="next-steps"></a>後續步驟

> [!div class="nextstepaction"]
> [Bot 的運作方式](../v4sdk/bot-builder-basics.md) 
