---
title: 使用適用於 .NET 的 Bot Builder SDK 建立 Bot | Microsoft Docs
description: 使用適用於 .NET 的 Bot Builder SDK 建立 Bot，Bot Builder SDK 是功能強大的 Bot 建構架構。
keywords: Bot Builder SDK, 建立 Bot, 快速入門, .NET, 開始使用
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: get-started-article
ms.prod: bot-framework
ms.date: 04/27/2018
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 98c6103f0cd38bf6d3f477735dafcf01952304d5
ms.sourcegitcommit: f576981342fb3361216675815714e24281e20ddf
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/18/2018
ms.locfileid: "39300339"
---
# <a name="create-a-bot-with-the-bot-builder-sdk-v4-for-net"></a>使用適用於 .NET 的 Bot Builder SDK 4 版建立 Bot
[!INCLUDE [pre-release-label](../includes/pre-release-label.md)]

本快速入門會逐步引導您使用 Bot 應用程式範本和適用於 .NET 的 Bot Builder SDK 建置 Bot，然後使用 Bot Framework 模擬器進行測試。 本文以 [Microsoft Bot Builder SDK 4 版](https://github.com/Microsoft/botbuilder-dotnet)作為基礎。

## <a name="prerequisites"></a>必要條件
- Visual Studio [2017](https://www.visualstudio.com/downloads)
- Bot Builder SDK 4 版的 [C#](https://marketplace.visualstudio.com/items?itemName=BotBuilder.botbuilderv4) 範本
- [Bot Framework 模擬器](https://github.com/Microsoft/BotFramework-Emulator/releases)
- 了解 [ASP.Net Core](https://docs.microsoft.com/aspnet/core/) 和 [C#](https://docs.microsoft.com/en-us/dotnet/csharp/programming-guide/concepts/async/index) 中的非同步程式設計

## <a name="create-a-bot"></a>建立 Bot
安裝您在必要條件區段下載的 BotBuilderVSIX.vsix 範本。 

在 Visual Studio 中建立新 Bot 專案。

![Visual Studio 專案](../media/azure-bot-quickstarts/bot-builder-dotnet-project.png)

> [!TIP] 
> 視需要更新 [NuGet 套件](https://docs.microsoft.com/en-us/nuget/quickstart/install-and-use-a-package-in-visual-studio)。

## <a name="explore-code"></a>探索程式碼
開啟 Startup.cs 檔案來檢閱 `ConfigureServices(IServiceCollection services)` 方法中的程式碼。 `CatchExceptionMiddleware` 中介軟體已新增至訊息管道。 其可處理由其他中介軟體或 OnTurn 方法擲回任的何例外狀況。 

```cs
options.Middleware.Add(new CatchExceptionMiddleware<Exception>(async (context, exception) =>
{
    await context.TraceActivity("EchoBot Exception", exception);
    await context.SendActivity("Sorry, it looks like something went wrong!");
}));
```

在記憶體內部儲存體中使用的對話狀態中介軟體。 它會在儲存體中讀取和寫入 EchoState。  EchoState 類別中的回合計數會追蹤傳送至 Bot 的訊息數目。 您可以使用類似技巧，讓狀態維持到下個回合。

```cs
 IStorage dataStore = new MemoryStorage();
 options.Middleware.Add(new ConversationState<EchoState>(dataStore));
```

`Configure(IApplicationBuilder app, IHostingEnvironment env)`呼叫路由`.UseBotFramework`傳入 Bot 配接器活動的方法。 

EchoBot.cs 檔案的 `OnTurn(ITurnContext context)` 方法可用於檢查傳入的活動類型，並傳送回覆給使用者。 

```cs
public async Task OnTurn(ITurnContext context)
{
    // This bot is only handling Messages
    if (context.Activity.Type == ActivityTypes.Message)
    {
        // Get the conversation state from the turn context
        var state = context.GetConversationState<EchoState>();

        // Bump the turn count. 
        state.TurnCount++;

        // Echo back to the user whatever they typed.
        await context.SendActivity($"Turn {state.TurnCount}: You sent '{context.Activity.Text}'");
    }
}
```
## <a name="start-your-bot"></a>啟動 Bot

- 您的 `default.html` 頁面會顯示在瀏覽器中。
- 請記下該頁面的 localhost 連接埠號碼。 稍後與 Bot 互動時，您會需要使用此資訊。

### <a name="start-the-emulator-and-connect-your-bot"></a>啟動模擬器並且連線至您的 Bot

此時，Bot 正在本機執行。
接下來，請啟動模擬器，然後在模擬器中連線至您的 Bot：

1. 按一下模擬器 [歡迎使用] 索引標籤中的 [建立新的 Bot 組態] 連結。 

2. 輸入 **Bot 名稱**，然後輸入 Bot 程式碼的目錄路徑。 Bot 組態檔會儲存至這個路徑。

3. 將 `http://localhost:port-number/api/messages` 輸入至 [端點 URL] 欄位，其中連接埠號碼必須符合應用程式執行所在瀏覽器中顯示的連接埠號碼。

4. 按一下 [連線] 來連線至 Bot。 您不需要指定 **Microsoft 應用程式識別碼**和 **Microsoft 應用程式密碼**。 目前可以將這些欄位保留空白。 稍後註冊 Bot 時，您會取得這項資訊。

## <a name="interact-with-your-bot"></a>與您的 Bot 互動

傳送 "Hi" 給 Bot，然後 Bot 就會對該訊息回應 "Turn 1: You sent Hi"。

## <a name="next-steps"></a>後續步驟

接下來，請[將 Bot 部署至 Azure](../bot-builder-howto-deploy-azure.md) 或跳至說明 Bot 及其運作方式的概念。

> [!div class="nextstepaction"]
> [基本 Bot 概念](../v4sdk/bot-builder-basics.md)
