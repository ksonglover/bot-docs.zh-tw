---
title: 開發 Direct Line Speech Bot | Microsoft Docs
description: 開發 DirectLine Speech Bot
keywords: 開發 Direct Line Speech Bot, 語音 Bot
author: ivorb
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 11/01/2019
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 4fc2eb84751f64a1ca1493515ccb231a0a78bbd4
ms.sourcegitcommit: 490810d278d1c8207330b132f28a5eaf2b37bd07
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 11/05/2019
ms.locfileid: "73592254"
---
# <a name="use-direct-line-speech-in-your-bot"></a>在 Bot 中使用 Direct Line Speech

[!INCLUDE [applies-to-v4](includes/applies-to.md)]

Direct Line Speech 使用 Bot Framework 的新 WebSocket 型串流功能，在 Direct Line Speech 通道與您的 Bot 之間交換訊息。 在 Azure 入口網站中啟用 Direct Line Speech 通道後，您必須更新 Bot 以接聽並接受這些 WebSocket 連線。 這些指示說明如何執行這項操作。  

## <a name="step-1-upgrade-to-the-46-sdk"></a>步驟 1：升級至 4.6 SDK 

針對 Direct Line Speech，請確定您是使用 4.6 版或更高版本的 Bot Builder SDK。 

## <a name="step-2-update-your-net-core-bot-codeif-your-bot-uses-addbot-and-usebotframework-instead-of-a-botcontroller"></a>步驟 2：更新 .NET Core Bot 程式碼 (如果 Bot 使用 AddBot 和 UseBotFramework 而非 BotController) 

如果您已使用 4.3.2 版之前的 Bot Builder SDK v4 建立 Bot，Bot 可能不包含 BotController，但在 Startup.cs 檔案中改為使用 AddBot() 和 UseBotFramework() 方法來公開 Bot 接收訊息的 POST 端點。 若要公開新的串流端點，您必須新增 BotController 及移除 AddBot() 和 UseBotFramework() 方法。 這些指示會逐步引導需要進行的變更。 如果您已經有這些變更，請繼續進行下一個步驟。 

藉由新增名為 BotController.cs 的檔案，將新的 MVC 控制器新增至 Bot 專案。 將此檔案中新增控制器程式碼： 

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

在 **Startup.cs** 檔案中，找出 Configure 方法。 移除 `UseBotFramework()` 行並確定您有 `UseWebSockets` 的這幾行： 

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

同時在 Startup.cs 檔案中，找出 ConfigureServices 方法。 移除 `AddBot()` 行，並確定您有用來新增 `IBot` 和 `BotFrameworkHttpAdapter` 的這幾行： 

```cs

public void ConfigureServices(IServiceCollection services) 
{ 
    services.AddMvc().SetCompatibilityVersion(CompatibilityVersion.Version_2_1); 
    services.AddSingleton<ICredentialProvider, ConfigurationCredentialProvider>(); 
    services.AddSingleton<IChannelProvider, ConfigurationChannelProvider>(); 
    
    // Create the Bot Framework Adapter. 
    services.AddSingleton<IBotFrameworkHttpAdapter, BotFrameworkHttpAdapter>(); 

    // Create the bot as a transient. In this case the ASP Controller is expecting an IBot. 
    services.AddTransient<IBot, EchoBot>(); 
} 
```

您 Bot 程式碼的其餘部分維持不變！ 

## <a name="step3-ensure-websockets-are-enabled"></a>步驟 3：確定已啟用 Websocket 

當您在 Azure 入口網站中使用其中一個範本 (例如 EchoBot) 建立新的 Bot 時，您會收到一個 Bot，其中包含可公開 GET 和 POST 端點的 ASP.NET MVC 控制器，而且將會使用 WebSocket。 這些指示會說明當您要進行升級或未從範本建立 Bot 時，如何將這些元素新增至 Bot。 

在您解決方案的 Controllers 資料夾之下開啟 **BotController.cs** 

在類別中找到 `PostAsync` 方法，並將其裝飾從 [HttpPost] 更新為 [HttpPost, HttpGet]： 

```cs

[HttpPost, HttpGet] 
public async Task PostAsync() 
{ 
    await _adapter.ProcessAsync(Request, Response, _bot); 
} 
```

儲存並關閉 BotController.cs 

在您解決方案的根目錄中開啟 **Startup.cs**。 

在 Startup.cs 中，瀏覽至 Configure 方法的底部。 在呼叫  `app.UseMvc()` 之前，請先將呼叫新增至  `app.UseWebSockets()`。 這很重要，因為這些使用呼叫的順序很重要。 方法結尾應該類似這樣： 

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
您 Bot 程式碼的其餘部分維持不變！ 

 

步驟 4：(選擇性) 設定活動的 [說話] 欄位，以自訂要對使用者說的話 

根據預設，說話內容是透過 Direct Line Speech 傳送給使用者的所有訊息。  

您可以藉由對 Bot 所傳送的任何活動設定 [說話] 欄位，選擇性地自訂說出的訊息方式： 

```cs 

public IActivity Speak(string message) 
{ 
    var activity = MessageFactory.Text(message); 
    string body = @"<speak version='1.0' xmlns='https://www.w3.org/2001/10/synthesis' xml:lang='en-US'> 

        <voice name='Microsoft Server Speech Text to Speech Voice (en-US, JessaRUS)'>" + 
        $"{message}" + "</voice></speak>"; 

    activity.Speak = body; 
    return activity; 
} 
```

下列程式碼片段示範如何使用先前的 Speak 函式： 

```cs

protected override async Task OnMessageActivityAsync(ITurnContext<IMessageActivity> turnContext, CancellationToken cancellationToken) 
{ 
    await turnContext.SendActivityAsync(Speak($"Echo: {turnContext.Activity.Text}"), cancellationToken); 
} 
``` 

## <a name="additional-information"></a>其他資訊 

- 如需建立和使用啟用語音的 Bot 的完整範例，請參閱 [教學課程：使用語音 SDK 透過聲音啟用 Bot](https://docs.microsoft.com/azure/cognitive-services/speech-service/tutorial-voice-enable-your-bot-speech-sdk)。 

- 如需有關使用活動的詳細資訊，請參閱  [Bot 的運作方式](https://docs.microsoft.com/azure/bot-service/bot-builder-basics)和 [如何傳送及接收文字訊息 https://docs.microsoft.com/azure/bot-service/bot-builder-howto-send-messages?view=azure-bot-service-4.0]()。 

 
