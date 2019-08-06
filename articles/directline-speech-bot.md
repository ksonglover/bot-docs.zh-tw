---
title: 開發 Direct Line Speech Bot | Microsoft Docs
description: 開發 DirectLine Speech Bot
keywords: 開發 DirectLine Speech Bot, 語音 Bot
author: ivorb
ms.author: v-ivorb
manager: kamrani
ms.topic: conceptuals
ms.service: bot-service
ms.subservice: abs
ms.date: 07/15/2019
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 4f33f799af3c7a6aecec172f5512791b52fdbffe
ms.sourcegitcommit: f3fda6791f48ab178721b72d4f4a77c373573e38
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/31/2019
ms.locfileid: "68671488"
---
# <a name="use-direct-line-speech-in-your-bot"></a>在 Bot 中使用 Direct Line Speech 

[!INCLUDE [applies-to-v4](includes/applies-to.md)]

Direct Line Speech 使用 Bot Framework 的新 WebSocket 型串流功能，在 Direct Line Speech 通道與您的 Bot 之間交換訊息。 在 Azure 入口網站中啟用 Direct Line Speech 通道後，您必須更新 Bot 以接聽並接受這些 WebSocket 連線。 這些指示說明如何執行這項操作。

## <a name="add-the-nuget-package"></a>新增 NuGet 套件
在 Direct Line Speech 預覽版中，有您需要新增至 Bot 的其他 NuGet 套件。 這些套件裝載於 myget.org NuGet 摘要上：
1.  在 Visual Studio 中開啟 Bot 的專案。

2.  移至 Bot 專案屬性之下的 [管理 Nuget 套件]。

3.  新增 `Microsoft.Bot.Builder.StreamingExtensions` 套件。 您必須核取 [包含發行前版本] 方塊，以查看預覽套件。

4.  接受任何提示，可完成將套件新增至您的專案。

## <a name="set-the-speak-field-on-activities-you-want-spoken-to-the-user"></a>在想要與使用者對話的活動上，設定「說話」欄位
您必須在想要與使用者對話的任何活動上 (由 Bot 傳送)，設定「說話」欄位。 

```cs
public IActivity Speak(string message)
{
    var activity = MessageFactory.Text(message);
    string body = @"<speak version='1.0' xmlns='https://www.w3.org/2001/10/synthesis' xml:lang='en-US'>
        <voice name='Microsoft Server Speech Text to Speech Voice (en-US, JessaNeural)'>" +
        $"{message}" + "</voice></speak>";
    activity.Speak = body;
    return activity;
}
```

## <a name="option-1-update-your-net-core-bot-code-if-your-bot-has-a-botcontrollercs"></a>選項 1：更新 .NET Core Bot 程式碼 _(如果 Bot 有 BotController.cs)_
當您在 Azure 入口網站中使用其中一個範本 (例如 EchoBot) 建立新的 Bot 時，您會收到一個 Bot，其中包含可公開單一 POST 端點的 ASP.NET MVC 控制器。 下列指示說明如何將該控制器擴展為同時公開一個端點來接受 WebSocket 串流端點 (GET 端點)。
1.  在您解決方案的 Controllers 資料夾之下開啟 BotController.cs

2.  在類別中找到 PostAsync 方法，並將其裝飾從 [HttpPost] 更新為 [HttpPost, HttpGet]：
```cs
[HttpPost, HttpGet]
public async Task PostAsync()
{ 
    await _adapter.ProcessAsync(Request, Response, _bot);
}
```

3.  儲存並關閉 BotController.cs

4.  在您解決方案的根目錄中開啟 Startup.cs。

5.  新增命名空間：

```cs
using Microsoft.Bot.Builder.StreamingExtensions;
```

6.  在 ConfigureServices 方法中，在適當的 services.AddSingleton 呼叫中以 WebSocketEnabledHttpAdapter 取代使用 AdapterWithErrorHandler：

```cs
public void ConfigureServices(IServiceCollection services)
{
    ...    

    // Create the Bot Framework Adapter.
    services.AddSingleton<IBotFrameworkHttpAdapter, WebSocketEnabledHttpAdapter>();

    services.AddTransient<IBot, EchoBot>();

    ...
}
```

7. 仍然在 Startup.cs 中，巡覽至 Configure 方法的底部。 在 app.UseMvc(); 呼叫 (這很重要，因為 Use 呼叫的順序關係重大) 之前，新增 app.UseWebSockets();。 方法結尾應該類似以下內容：

```cs
public void Configure(IApplicationBuilder app, IHostingEnvironment env)
{
    ...

    app.UseDefaultFiles();
    app.UseStaticFiles();
    app.UseWebSockets();
    app.UseMvc();

    ...
}
```

8.  您 Bot 程式碼的其餘部分維持不變！

## <a name="option-2-update-your-net-core-bot-code-if-your-bot-uses-addbot-and-usebotframework-instead-of-a-botcontroller"></a>選項 2：更新 .NET Core Bot 程式碼 _(如果 Bot 使用 AddBot 和 UseBotFramework 而非 BotController)_

如果您已使用 4.3.2 版之前的 Bot Builder SDK v4 建立 Bot，Bot 可能不包含 BotController，但在 Startup.cs 檔案中改為使用 AddBot() 和 UseBotFramework() 方法來公開 Bot 接收訊息的 POST 端點。 若要公開新的串流端點，您必須新增 BotController 及移除 AddBot() 和 UseBotFramework() 方法。 這些指示會逐步引導需要進行的變更。

1.  藉由新增名為 BotController.cs 的檔案，將新的 MVC 控制器新增至 Bot 專案。 將此檔案中新增控制器程式碼：

```cs
[Route("api/messages")]
[ApiController]
public class BotController : ControllerBase
{
    private readonly IBotFrameworkHttpAdapter _adapter;
    private readonly IBot _bot;

    public BotController(IBotFrameworkHttpAdapter adapter, IBot bot)
    {
        _adapter = adapter;
        _bot = bot;
    }

    [HttpPost, HttpGet]
    public async Task ProcessMessageAsync()
    {
        await _adapter.ProcessAsync(Request, Response, _bot);
    }
}
```
2.  在 Startup.cs 檔案中，找出 Configure 方法。 移除 UseBotFramework() 行並確定您有這幾行：

```cs
public void Configure(IApplicationBuilder app, IHostingEnvironment env)
{
    ...

    app.UseDefaultFiles();
    app.UseStaticFiles();
    app.UseWebSockets();
    app.UseMvc();

    ...
}
```

3.  同時在 Startup.cs 檔案中，找出 ConfigureServices 方法。 移除 AddBot() 行並確定您有這幾行：

```cs
public void ConfigureServices(IServiceCollection services)
{
    services.AddMvc().SetCompatibilityVersion(CompatibilityVersion.Version_2_1);

    services.AddSingleton<ICredentialProvider, ConfigurationCredentialProvider>();

    services.AddSingleton<IChannelProvider, ConfigurationChannelProvider>();

    // Create the Bot Framework Adapter.
    services.AddSingleton<IBotFrameworkHttpAdapter, WebSocketEnabledHttpAdapter>();

    // Create the bot as a transient. In this case the ASP Controller is expecting an IBot.
    services.AddTransient<IBot, EchoBot>();
}
```
4.  您 Bot 程式碼的其餘部分維持不變！

## <a name="next-steps"></a>後續步驟
> [!div class="nextstepaction"]
> [將 Bot 連線至 Direct Line Speech (預覽)](./bot-service-channel-connect-directlinespeech.md)
