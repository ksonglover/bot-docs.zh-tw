---
title: 使用 .NET Core 驗證活動 | Microsoft Docs
description: 了解如何使用 .NET Core 驗證 Bot 活動。
author: v-ducvo
ms.author: v-ducvo
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 12/13/17
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: 6df7923caa708ac2b10af37d860dfac317e113a0
ms.sourcegitcommit: f576981342fb3361216675815714e24281e20ddf
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/18/2018
ms.locfileid: "39299202"
---
# <a name="authenticating-activities-using-net-core"></a>使用 .NET Core 驗證活動

如果您選擇使用 [.NET Core](/dotnet/core/index) 來開發您的 Bot，可以使用 [Bot Framework Connector](bot-builder-dotnet-connector.md) 從您的 Bot 傳送及接收[活動](https://docs.botframework.com/en-us/csharp/builder/sdkreference/dc/d2f/class_microsoft_1_1_bot_1_1_connector_1_1_activity.html)訊息。 若要使用 Connector 服務，您必須為目標架構版本設定適當的驗證模型。

Bot Framework Connector.AspNetCore 支援下列版本的 ASP.NET：
* 針對 .NET Core v1.1/AspNetCore1.x 請使用 **Microsoft.Bot.Connector.AspNetCore 1.x**
* 針對 .NET Core v2.0/AspNetCore2.x 請使用 **Microsoft.Bot.Connector.AspNetCore 2.x**

本文將說明如何為 Bot 的目標特定架構設定驗證模型。

## <a name="prerequisites"></a>必要條件

* [Visual Studio 2017](https://www.visualstudio.com/downloads/)
* [.NET Core](https://www.microsoft.com/net/download/windows). 安裝目標 .NET Core 版本 (例如：.NET Core 1.1 版或 .NET Core 2.0 版)。
* [註冊 Bot](~/bot-service-quickstart-registration.md)。 註冊您的 Bot，以取得驗證程序所需的應用程式識別碼和密碼。

## <a name="create-a-net-core-project"></a>建立 .NET Core 專案

若要建立 .NET Core 專案，請執行下列作業：

1. 開啟 Visual Studio 2017，然後按一下 [檔案] > [新增] > [專案...]。
2. 展開 [Visual C#] 節點，然後按一下 [.NET Core]。
3. 選擇 [ASP.NET Core Web 應用程式] 專案，鍵入及填入專案資訊 (例如：[名稱]、[位置] 和 [解決方案] 名稱欄位)。
4. 按一下 [確定]。
5. 請確定專案的目標設為您想要的 .NET Core 和 ASP.NET Core 版本。 例如，以下的螢幕擷取畫面顯示專案目標設為 **.NET Core** 和 **ASP.NET Core 2.0**：

![建立 ASP.NET Core 2.0 版專案](~/media/dotnet-core-authentication/create-asp-net-core-2x-project.png)
 
  > [!NOTE]
  > 如果您的目標設為 **ASP.NET Core 2.0**，請確定在我們的電腦上所安裝的版本至少是 2.x 和更新版本。

6. 選擇 [Web API] 專案類型。
7. 按一下 [確定]  以建立專案。

## <a name="download-the-nuget-package"></a>下載 NuGet 套件

若要使用 **Bot Framework Connector**，請安裝適合您 .NET Core 版本的 NuGet 套件。 若要安裝 NuGet 套件，請執行下列動作：

1. 從 [方案總管] 中，以滑鼠右鍵按一下專案名稱和 [管理 NuGet 套件...]
2. 按一下 [瀏覽] 並搜尋 **Connector.ASPNetCore**。 
3. 選擇您想要設為目標的版本。 例如，以下的螢幕擷取畫面顯示已選取 **2.0.0.3** 版。 若要安裝 1.1.3.2 版，請按一下 [版本] 下拉式清單，然後改為選擇 [1.1.3.2] 版。
![適用於 ASP Net Core 2.0.0.3 版的 NuGet 套件](~/media/dotnet-core-authentication/nuget-package-net-core-version.png)
4. 按一下 [Install] 。

## <a name="update-the-appsettingsjson"></a>更新 appsettings.json

Bot Framework Connector 需要您的應用程式識別碼和密碼，才能驗證 Bot。 您可以在您 Web 應用程式專案的 **appsettings.json** 中設定這些值。

> [!NOTE]
> 若要尋找您 Bot 的**應用程式識別碼**和**應用程式密碼**，請參閱 [MicrosoftAppID 和 MicrosoftAppPassword](~/bot-service-manage-overview.md#microsoftappid-and-microsoftapppassword)。

### <a name="appsettingsjson-for-net-core-v11"></a>適用於 .NET Core 1.1 版的 appsettings.json：

```json
{
  "Logging": {
    "IncludeScopes": false,
    "LogLevel": {
      "Default": "Warning"
    }
  },
  "MicrosoftAppId": "<MicrosoftAppId>",
  "MicrosoftAppPassword": "<MicrosoftAppPassword>"
}
```

### <a name="appsettingsjson-for-net-core-v20"></a>適用於 .NET Core 2.0 版的 appsettings.json：

```json
{
  "Logging": {
    "IncludeScopes": false,
    "Debug": {
      "LogLevel": {
        "Default": "Warning"
      }
    },
    "Console": {
      "LogLevel": {
        "Default": "Warning"
      }
    }
  },
  "MicrosoftAppId": "<MicrosoftAppId>",
  "MicrosoftAppPassword": "<MicrosoftAppPassword>"
}
```

## <a name="update-the-startupcs-class"></a>更新 startup.cs 類別

更新 **startup.cs** 類別。 請根據您的專案版本，更新適當的程式碼片段，讓驗證得以運作。

### <a name="startupcs-for-net-core-v11"></a>適用於 .NET Core 1.1 版的 startup.cs

```cs
public class Startup
    {
        public Startup(IHostingEnvironment env)
        {
            var builder = new ConfigurationBuilder()
                .SetBasePath(env.ContentRootPath)
                .AddJsonFile("appsettings.json", optional: true, reloadOnChange: true)
                .AddJsonFile($"appsettings.{env.EnvironmentName}.json", optional: true)
                .AddEnvironmentVariables();
            Configuration = builder.Build();
        }

        public IConfigurationRoot Configuration { get; }

        // This method gets called by the runtime. Use this method to add services to the container.
        public void ConfigureServices(IServiceCollection services)
        {
            services.AddSingleton(_ => Configuration);

            services.AddSingleton(typeof(ICredentialProvider), new StaticCredentialProvider(
                Configuration.GetSection(MicrosoftAppCredentials.MicrosoftAppIdKey)?.Value,
                Configuration.GetSection(MicrosoftAppCredentials.MicrosoftAppPasswordKey)?.Value));

            // Add framework services.
            services.AddMvc(options =>
            {
                options.Filters.Add(typeof(TrustServiceUrlAttribute));
            });
        }

        // This method gets called by the runtime. Use this method to configure the HTTP request pipeline.
        public void Configure(IApplicationBuilder app, IServiceProvider serviceProvider, IHostingEnvironment env, ILoggerFactory loggerFactory)
        {
            loggerFactory.AddConsole(Configuration.GetSection("Logging"));
            loggerFactory.AddDebug();

            app.UseStaticFiles();

            ICredentialProvider credentialProvider = serviceProvider.GetService<ICredentialProvider>();

            app.UseBotAuthentication(credentialProvider);

            app.UseMvc();
        }
    }
```

### <a name="startupcs-for-net-core-v20"></a>適用於 .NET Core 2.0 版的 startup.cs：

```cs
public class Startup
    {
        public Startup(IHostingEnvironment env)
        {
            var builder = new ConfigurationBuilder()
                .SetBasePath(env.ContentRootPath)
                .AddJsonFile("appsettings.json", optional: true, reloadOnChange: true)
                .AddJsonFile($"appsettings.{env.EnvironmentName}.json", optional: true)
                .AddEnvironmentVariables();
            Configuration = builder.Build();
        }

        public IConfiguration Configuration { get; }
        // This method gets called by the runtime. Use this method to add services to the container.
        public void ConfigureServices(IServiceCollection services)
        {
            services.AddSingleton(_ => Configuration);

            var credentialProvider = new StaticCredentialProvider(Configuration.GetSection(MicrosoftAppCredentials.MicrosoftAppIdKey)?.Value,
                Configuration.GetSection(MicrosoftAppCredentials.MicrosoftAppPasswordKey)?.Value);

            services.AddAuthentication(
                    options =>
                    {
                        options.DefaultAuthenticateScheme = JwtBearerDefaults.AuthenticationScheme;
                        options.DefaultChallengeScheme = JwtBearerDefaults.AuthenticationScheme;
                    }
                )
                .AddBotAuthentication(credentialProvider);

            services.AddSingleton(typeof(ICredentialProvider), credentialProvider);

            services.AddMvc(options =>
            {
                options.Filters.Add(typeof(TrustServiceUrlAttribute));
            });
        }


        // This method gets called by the runtime. Use this method to configure the HTTP request pipeline.
        public void Configure(IApplicationBuilder app, IHostingEnvironment env)
        {
            app.UseStaticFiles();
            app.UseAuthentication();
            app.UseMvc();
        }
    }
```

## <a name="create-the-messagescontrollercs-class"></a>建立 MessagesController.cs 類別

從 [方案總管] 中新增名為 **MessagesController.cs** 的新空白類別。 在 **MessagesController.cs** 類別中，使用以下的程式碼更新 **Post** 方法。 這樣可讓您的 Bot 為使用者傳送及接收訊息。

```cs
private IConfiguration configuration;

public MessagesController(IConfiguration configuration)
{
    this.configuration = configuration;
}


[Authorize(Roles = "Bot")]
[HttpPost]
public async Task<OkResult> Post([FromBody] Activity activity)
{
    if (activity.Type == ActivityTypes.Message)
    {
        //MicrosoftAppCredentials.TrustServiceUrl(activity.ServiceUrl);
        var appCredentials = new MicrosoftAppCredentials(configuration);
        var connector = new ConnectorClient(new Uri(activity.ServiceUrl), appCredentials);

        // return our reply to the user
        var reply = activity.CreateReply("HelloWorld");
        await connector.Conversations.ReplyToActivityAsync(reply);
    }
    else
    {
        //HandleSystemMessage(activity);
    }
    return Ok();
}
```
