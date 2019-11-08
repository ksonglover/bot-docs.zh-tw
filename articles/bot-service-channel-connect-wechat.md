---
title: 將 Bot 連線至 WeChat | Microsoft Docs
description: 了解如何設定 Bot 與 WeChat 之間的連線。
keywords: WeChat, Tencent, Bot 通道, WeChat 應用程式, WeChat Bot, 應用程式識別碼, 應用程式密碼, 認證
author: seaen
manager: kamrani
ms.topic: article
ms.author: egorn
ms.service: bot-service
ms.date: 11/01/2019
ms.openlocfilehash: aee02ba1707f08fbcb4479b37edd065fd28efd8f
ms.sourcegitcommit: 4751c7b8ff1d3603d4596e4fa99e0071036c207c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 11/02/2019
ms.locfileid: "73443086"
---
# <a name="connect-a-bot-to-wechat"></a>將 Bot 連線至 WeChat

您可以將 Bot 設定為使用 WeChat 官方帳戶平台與其他人通訊。

## <a name="download-wechat-adapter-for-bot-framework"></a>下載適用於 Bot Framework 的 WeChat 配接器

適用於 Microsoft Bot Framework 的 WeChat 配接器是 GitHub 上的開放原始碼配接器。 [下載適用於 Bot Framework 的 WeChat 配接器](https://github.com/microsoft/BotFramework-WeChat/)。

## <a name="create-a-wechat-account"></a>建立 WeChat 帳戶

若要將 Bot 設定為使用 WeChat 進行通訊，您必須在 [WeChat 官方帳戶平台](https://mp.weixin.qq.com/?lang=en_US)上建立 WeChat 官方帳戶，然後將 Bot 連線至應用程式。 我們目前僅支援服務帳戶。

### <a name="change-your-prefer-language"></a>變更您的慣用語言

您可以在登入之前變更慣用的顯示語言。

 ![change_language](./media/channels/wechat-change-language.png)

### <a name="register-a-service-account"></a>註冊服務帳戶

實際的服務帳戶必須經過 WeChat 驗證，您無法在帳戶通過驗證之前啟用 Webhook。 若要建立您自己的服務帳戶，請遵循[這裡](https://kf.qq.com/product/weixinmp.html#hid=87)的指示。
簡而言之，只要按一下頂端的 [立即註冊]、選取服務帳戶並遵循指示即可。

 ![register_account](./media/channels/wechat-register-account.png)

### <a name="sandbox-account"></a>沙箱帳戶

如果只要測試 WeChat 和 Bot 的整合，您可以使用沙箱帳戶，而不是建立服務帳戶。 深入了解如何[建立沙箱帳戶](https://mp.weixin.qq.com/debug/cgi-bin/sandbox?t=sandbox/login)。

## <a name="enable-wechat-adapter-to-bot"></a>對 Bot 啟用 WeChat 配接器

Bot 專案是一般的 Bot Framework SDK V4 專案。 您必須先確定您可以執行 Bot，才可以啟動專案。 下載[適用於 Bot Framework 的 WeChat 配接器](https://github.com/microsoft/BotFramework-WeChat/)。

### <a name="prerequisites"></a>必要條件

    .NET Core SDK (version 2.2.x)

### <a name="add-reference-to-wechat-adapter-source"></a>將參考新增至 WeChat 配接器來源

請直接參考 WeChat 配接器專案，或是將 ~/BotFramework-WeChat/libraries/csharp_dotnetcore/outputpackages 作為本機 NuGet 來源。

### <a name="inject-wechat-adapter-in-your-bot-startupcs"></a>在 Bot Startup.cs 中插入 WeChat 配接器

```csharp
public void ConfigureServices(IServiceCollection services)
{
    services.AddMvc().SetCompatibilityVersion(CompatibilityVersion.Version_2_2);

    // Create the storage we'll be using for User and Conversation state. (Memory is great for testing purposes.)
    services.AddSingleton<IStorage, MemoryStorage>();

    // Create the User state. (Used in this bot's Dialog implementation.)
    services.AddSingleton<UserState>();

    // Create the Conversation state. (Used by the Dialog system itself.)
    services.AddSingleton<ConversationState>();

    // Load WeChat settings.
    var wechatSettings = new WeChatSettings();
    Configuration.Bind("WeChatSettings", wechatSettings);
    services.AddSingleton<WeChatSettings>(wechatSettings);

    // Configure hosted serivce.
    services.AddSingleton<IBackgroundTaskQueue, BackgroundTaskQueue>();
    services.AddHostedService<QueuedHostedService>();
    services.AddSingleton<WeChatHttpAdapter>();

    // The Dialog that will be run by the bot.
    services.AddSingleton<MainDialog>();

    // Create the bot as a transient. In this case the ASP Controller is expecting an IBot.
    services.AddTransient<IBot, EchoBot>();
}
```

### <a name="update-your-bot-controller"></a>更新 Bot 控制器

```csharp
[Route("api/messages")]
[ApiController]
public class BotController : ControllerBase
{  
    private readonly IBot _bot;
    private readonly WeChatHttpAdapter _weChatHttpAdapter;
    private readonly string Token;
    public BotController(IBot bot, WeChatHttpAdapter weChatAdapter)
    {
        _bot = bot;
        _weChatHttpAdapter = weChatAdapter;
    }

    [HttpPost("/WeChat")]
    [HttpGet("/WeChat")]
    public async Task PostWeChatAsync([FromQuery] SecretInfo secretInfo)
    {
        // Delegate the processing of the HTTP POST to the adapter.
        // The adapter will invoke the bot.
        await _weChatHttpAdapter.ProcessAsync(Request, Response, _bot, secretInfo);
    }
}
```

### <a name="setup-appsettingsjson"></a>設定 appsettings.json

您必須先設定 appsettings.json，才能啟動 Bot，您可以在下方找到所需的項目。

```json
"WeChatSettings": {
    "UploadTemporaryMedia": true,
    "PassiveResponseMode": false,
    "Token": "",
    "EncodingAESKey": "",
    "AppId": "",
    "AppSecret": ""
}
```

#### <a name="service-account"></a>服務帳戶

如果您已經有服務帳戶，並且準備好進行部署，您可以在左側瀏覽列的基本設定中找到 **AppID**、**AppSecret**、**EncodingAESKey** 和 **Token**，如下所示。

別忘了您需要設定 IP 允許清單，否則 WeChat 不會接受您的要求。

 ![serviceaccount_console](./media/channels/wechat-serviceaccount-console.png)

#### <a name="sandbox-account"></a>沙箱帳戶

沙箱帳戶沒有 **EncodingAESKey**，來自 WeChat 的訊息並未加密，將 EncodingAESKey 保留空白即可。 這裡只有三個參數：**appID**、**appsecret** 和 **Token**。

 ![sandbox_account](./media/channels/wechat-sandbox-account.png)

### <a name="start-bot-and-set-endpoint-url"></a>啟動 Bot 並設定端點 URL

現在您可以設定 Bot 後端。 執行此動作之前，您必須先啟動 Bot，然後再儲存設定，WeChat 會傳送驗證 URL 的要求給您。
請以這類模式設定端點： **https://your_end_point/WeChat** ，或是依照您在 BotController.cs 中所做的設定來設定您的個人設定

 ![sandbox_account2](./media/channels/wechat-sandbox-account-2.png)

### <a name="subscribe-your-official-account"></a>訂閱您的官方帳戶

您可以在 WeChat 中找到用來訂閱測試帳戶的 QR 代碼。

 ![訂閱](./media/channels/wechat-subscribe.png)

## <a name="test-through-wechat"></a>透過 WeChat 進行測試

一切完成後，您可以在 WeChat 用戶端中試用此功能。 您可以在 [測試] 資料夾下試用我們的範例 Bot。 此範例 Bot 包含 WeChat 配接器，並且已與回應 Bot 和卡片 Bot 整合。

 ![聊天](./media/channels/wechat-chat.png)
