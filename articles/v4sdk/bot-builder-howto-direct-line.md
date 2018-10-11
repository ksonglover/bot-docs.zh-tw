---
title: 建立直接線路 Bot 與用戶端 | Microsoft Docs
description: 了解如何使用 V4 版的適用於 .NET 的 Bot 建立器 SDK 建立直接線路 Bot 與用戶端。
keywords: direct line bot, direct line client, custom channel, console-based, publish, 直接線路 Bot, 直接線路用戶端, 自訂通道, 主控台型, 發佈
author: v-royhar
ms.author: v-royhar
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 4/16/18
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 81b6f1f9373c18bd3aedb393cfc4966587bf24cb
ms.sourcegitcommit: 6c2426c43cd2212bdea1ecbbf8ed245145b3c30d
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/08/2018
ms.locfileid: "48852203"
---
# <a name="create-a-direct-line-bot-and-client"></a>建立直接線路 Bot 與用戶端

[!INCLUDE [pre-release-label](../includes/pre-release-label.md)]

Microsoft Bot Framework 直接線路 Bot 是能與您自有設計自訂用戶端搭配運作的 Bot。 直接線路 Bot 明顯地與一般 Bot 非常相似。 它們不需要使用提供的通道。

直接線路用戶端可以撰寫為您想要的樣子。 您可以撰寫 Android 用戶端、iOS 用戶端或甚至主控台型用戶端應用程式。

在此主題中，您將了解如何建立及部署直接用戶端 Bot，以及如何建立及執行主控台型直接線路用戶端應用程式。

## <a name="create-your-bot"></a>建立您的 Bot

每個語言都依照不同的路徑來建立 Bot。 Javascript 會在 Azure 中建立 Bot 並修改程式碼，而 C# 會在本機建立 Bot 並將它發佈到 Azure，但這兩種都是有效的方法，而且可以搭配任一語言使用。 如需有關將您的 Bot 發佈到 Azure 的詳細資料，請參閱[在 Azure 中部署您的 Bot](../bot-builder-howto-deploy-azure.md)。

# <a name="ctabcscreatebot"></a>[C#](#tab/cscreatebot)

### <a name="create-the-solution-in-visual-studio"></a>在 Visual Studio 中建立方案

在 Visual Studio 2015 或更新版本中建立直接線路 Bot 方案：

1. 在 **Visual C#** > **.NET Core** 中建立新的 [ASP.NET Core Web 應用程式]。

1. 針對名稱，請輸入 **DirectLineBotSample**，然後按一下 [確定]。

1. 確定已選取 [.NET Core] 與 [ASP.NET Core 2.0]，選擇 [空白] 專案範本，然後按一下 [確定]。

#### <a name="add-dependencies"></a>新增相依性

1. 在 [方案總管] 中，以滑鼠右鍵按一下 [相依性]，然後選擇 [管理 NuGet 套件]。

1. 按一下 [瀏覽]，然後確定已選取 [包含發行前版本] 核取方塊。

1. 搜尋並安裝下列 NuGet 封裝：
    - Microsoft.Bot.Builder
    - Microsoft.Bot.Builder.Core.Extensions
    - Microsoft.Bot.Builder.Integration.AspNet.Core
    - Microsoft.Rest.ClientRuntime
    - Newtonsoft.Json

### <a name="create-the-appsettingsjson-file"></a>建立 appsettings.json 檔案

appsettings.json 檔案將會包含 Microsoft 應用程式識別碼、應用程式密碼，以及資料連線字串。 因為此 Bot 將不會儲存狀態資訊，資料連線字串會維持為空字串。 此外，若您只使用 Bot Framework Emulator，可以將它們都維持為空字串。

建立 **appsettings.json** 檔案：

1. 以滑鼠右鍵按一下 [DirectLineBotSample] 專案，選擇 [加入] > [新項目]。

1. 在 [ASP.NET Core] 中，按一下 [ASP.NET 組態檔]。

1. 輸入 **appsettings.json** 做為名稱，然後按一下 [加入]。

使用下列程式碼取代 appsettings.json 檔案的內容：

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

    "MicrosoftAppId": "",
    "MicrosoftAppPassword": "",
    "DataConnectionString": ""
}
```

### <a name="edit-startupcs"></a>編輯 Startup.cs

使用下列程式碼取代 Startup.cs 檔案的內容：

**注意** ：**DirectBot** 將會在下一個步驟中定義，因此您可以忽略其紅色底線。

```csharp
using System;
using System.Collections.Generic;
using System.Linq;
using System.Threading.Tasks;
using Microsoft.AspNetCore.Builder;
using Microsoft.AspNetCore.Hosting;
using Microsoft.AspNetCore.Http;
using Microsoft.Bot.Builder.BotFramework;
using Microsoft.Bot.Builder.Integration.AspNet.Core;
using Microsoft.Extensions.Configuration;
using Microsoft.Extensions.DependencyInjection;

namespace DirectLineBotSample
{
    public class Startup
    {
        public IConfiguration Configuration { get; }

        public Startup(IHostingEnvironment env)
        {
            var builder = new ConfigurationBuilder()
                .SetBasePath(env.ContentRootPath)
                .AddJsonFile("appsettings.json", optional: true, reloadOnChange: true)
                .AddJsonFile($"appsettings.{env.EnvironmentName}.json", optional: true)
                .AddEnvironmentVariables();
            Configuration = builder.Build();
        }

        // This method gets called by the runtime. Use this method to add services to the container.
        public void ConfigureServices(IServiceCollection services)
        {
            services.AddSingleton(_ => Configuration);
            services.AddBot<DirectBot>(options =>
            {
                options.CredentialProvider = new ConfigurationCredentialProvider(Configuration);
            });
        }

        // This method gets called by the runtime. Use this method to configure the HTTP request pipeline.
        public void Configure(IApplicationBuilder app, IHostingEnvironment env)
        {
            if (env.IsDevelopment())
            {
                app.UseDeveloperExceptionPage();
            }

            app.UseDefaultFiles();
            app.UseStaticFiles();
            app.UseBotFramework();
        }
    }
}
```

### <a name="create-the-directbot-class"></a>建立 DirectBot 類別

DirectBot 類別包含此 Bot 的大部分邏輯。

1. 以滑鼠右鍵按一下 [DirectLineBotSample] > [加入] > [新項目]。

1. 在 [ASP.NET Core] 中，按一下 [類別]。

1. 輸入 **DirectBot.cs** 做為名稱，然後按一下 [加入]。

使用下列程式碼取代 DirectBot.cs 檔案的內容：

```csharp
using System.Threading.Tasks;
using Microsoft.Bot;
using Microsoft.Bot.Builder;
using Microsoft.Bot.Builder.Core.Extensions;
using Microsoft.Bot.Schema;

namespace DirectLineBotSample
{
    public class DirectBot : IBot
    {
        public Task OnTurn(ITurnContext context)
        {
            // Respond to the various activity types.
            switch (context.Activity.Type)
            {
                case ActivityTypes.Message:
                    // Respond to the incoming text message.
                    RespondToMessage(context);
                    break;

                case ActivityTypes.ConversationUpdate:
                    break;

                case ActivityTypes.ContactRelationUpdate:
                    break;

                case ActivityTypes.Typing:
                    break;

                case ActivityTypes.DeleteUserData:
                    break;
            }

            return Task.CompletedTask;
        }

        /// <summary>
        /// Responds to the incoming message by either sending a hero card, an image, 
        /// or echoing the user's message.
        /// </summary>
        /// <param name="context">The context of this conversation.</param>
        private void RespondToMessage(ITurnContext context)
        {
            switch (context.Activity.Text.Trim().ToLower())
            {
                case "hi":
                case "hello":
                case "help":
                    // Send the user an instruction message.
                    context.SendActivity("Welcome to the Bot to showcase the DirectLine API. " +
                    "Send \"Show me a hero card\" or \"Send me a BotFramework image\" to see how the " +
                    "DirectLine client supports custom channel data. Any other message will be echoed.");
                    break;

                case "show me a hero card":
                    // Create the hero card.
                    HeroCard heroCard = new HeroCard()
                    {
                        Title = "Sample Hero Card",
                        Text = "Displayed in the DirectLine client"
                    };

                    // Attach the hero card to a new activity.
                    context.SendActivity(MessageFactory.Attachment(heroCard.ToAttachment()));
                    break;

                case "send me a botframework image":
                    // Create the image attachment.
                    Attachment imageAttachment = new Attachment()
                    {
                        ContentType = "image/png",
                        ContentUrl = "https://docs.microsoft.com/en-us/bot-framework/media/how-it-works/architecture-resize.png",
                    };

                    // Attach the image attachment to a new activity.
                    context.SendActivity(MessageFactory.Attachment(imageAttachment));
                    break;

                default:
                    // No command was encountered. Echo the user's message.
                    context.SendActivity($"You said \"{context.Activity.Text}\"");
                    break;
            }
        }
    }
}
```

**注意：** 不需要變更現有的 Program.cs 檔案。

若要確認項目是否都正確，請按 **F6** 以建置專案。 不應該有警告或錯誤。

### <a name="publish-the-bot-from-visual-studio"></a>從 Visual Studio 發佈 Bot

1. 在 Visual Studio 的 [方案總管] 中，以滑鼠右鍵按一下 [DirectLineBotSample] 專案，然後按一下 [設定為啟始專案]。

    ![Visual Studio 發佈頁面](media/bot-builder-howto-direct-line/visual-studio-publish-page.png)

1. 再次以滑鼠右鍵按一下 [DirectLineBotSample]，然後按一下 [發佈]。 Visual Studio [發佈] 頁面隨即出現。

1. 若您已經有此 Bot 的發行設定檔，請選取它並按一下 [發行] 按鈕，然後移至下一節。

1. 若要建立新的發行設定檔，請選取 [Microsoft Azure App Service] 圖示。

1. 選取 [建立新的]。

1. 按一下 [發佈]  按鈕。 [建立 App Service] 對話方塊隨即出現。

    ![[建立 App Service] 對話方塊](media/bot-builder-howto-direct-line/create-app-service-dialog.png)

    - 針對 [應用程式名稱]，請將它命名為稍後您可以找到的名稱。 例如，您可以將您的電子郵件名稱加到「應用程式名稱」開頭。

    - 確認您使用的是正確的訂用帳戶。

    - 針對 [資源群組]，請確認您使用的是正確的資源群組。 您可以在 Bot 的 [概觀] 刀鋒視窗上找到資源群組。 **注意：** 不正確的資源群組難以修正。

    - 確認您使用的是正確的 App Service 方案。
    
1. 按一下 [ **建立** ] 按鈕。 Visual Studio 將會開始部署您的 Bot。

發佈 Bot 之後，將會開啟瀏覽器，其中會顯示您 Bot 的 URL 端點。

### <a name="create-bot-channels-registration-bot-on-microsoft-azure"></a>在 Microsoft Azure 上建立 Bot 通道註冊

直接線路 Bot 可以在任何平台上裝載。 在此範例中，Bot 將裝載在 Microsoft Azure 上。 

在 Microsoft Azure 上建立 Bot：

1. 在 Microsoft Azure 入口網站中，按一下 [建立資源]，然後搜尋「Bot 通道註冊」。

1. 按一下頁面底部的 [新增] 。 [Bot 通道註冊] 刀鋒視窗隨即出現。

    ![Bot 通道註冊刀鋒視窗，其中顯示 Bot 名稱、訂用帳戶、資源群組、位置、定價層、傳訊端點與其他欄位。](media/bot-builder-howto-direct-line/bot-service-registration-blade.png)

1. 在 [Bot 通道註冊] 刀鋒視窗上，輸入 **Bot 名稱**、**訂用帳戶**、**資源群組**、**位置**與**定價層**。

1. 將 [傳訊端點] 留白。 此值將會稍後再填入。

1. 按一下 [Microsoft 應用程式識別碼與密碼] [自動建立應用程式識別碼與密碼]。

1. 選取 [釘選到儀表板] 核取方塊。

1. 按一下 [ **建立** ] 按鈕。

部署需要幾分鐘時間，但部署完成時，它應該會出現在您的儀表板中。

### <a name="update-the-appsettingsjson-file"></a>更新 appsettings.json 檔案

1. 按一下您儀表板上的 Bot，或按一下 [所有資源] 並搜尋您的 Bot 通道註冊名稱。

1. 按一下 [概觀]。

1. 將資源群組的名稱複製到 **DirectLineClientSample** 專案之 **Program.cs** 的 **botId** 字串中。

1. 按一下 [設定]，然後按一下 [Microsoft 應用程式識別碼] 欄位附近的 [管理]。

1. 複製應用程式識別碼並將它貼到 **appsettings.json** 檔案的 **"MicrosoftAppId"** 欄位。

1. 按一下 [產生新密碼] 按鈕。

1. 複製新密碼，並將它貼到 **appsettings.json** 檔案的 **"MicrosoftAppPassword"** 欄位中。

### <a name="set-the-messaging-endpoint"></a>設定傳訊端點

加入端點到您的 Bot：

1. 從發佈您的 Bot 之後彈出的瀏覽器複製 URL 網址。

    ![Bot 通道註冊的設定刀鋒視窗，其中的傳訊端點已反白顯示](media/bot-builder-howto-direct-line/bot-channels-registration-settings.png)

1. 叫出您 Bot 的 [設定] 刀鋒視窗。

1. 將位址貼到 [傳訊端點] 中。

1. 編輯位址，使其開頭為 "https://" 且結尾為 "/api/messages"。 例如，若從瀏覽器複製的位址是 "http://v-royhar-dlbot-directlinebotsample20180329044602.azurewebsites.net"，請將它編輯為 "https://v-royhar-dlbot-directlinebotsample20180329044602.azurewebsites.net/api/messages"。

1. 按一下 [設定] 刀鋒視窗上的 [儲存] 按鈕。

**注意：** 若發佈失敗，您可能必須先停止 Azure 上的 Bot，才能發佈對您的 Bot 所做的變更。

1. 尋找您的 App Service 名稱 (它介於 "https://" 到 ".azurewebsites.net" 之間)。

1. 在 Azure 入口網站「所有資源」中搜尋該資源。

1. 按一下 App Service 資源。 App Serivce 刀鋒視窗隨即開啟。

1. 按一下 App Service 刀鋒視窗上的 [停止] 按鈕。

1. 在 Visual Studio 中，再次發佈您的 Bot。

1. 按一下 App Service 刀鋒視窗上的 [開始] 按鈕。

### <a name="test-your-bot-in-webchat"></a>在 webchat 中測試您的 Bot

若要確認您的 Bot 是否正常運作，請在 Webchat 中檢查您的 Bot：

1. 在您 Bot 的 Bot 通道註冊刀鋒視窗中，按一下 [設定]，然後按一下 [在 Web Chat 中測試]。

1. 輸入 "Hi"。 Bot 應該會以歡迎訊息來回應。

1. 輸入 "show me a hero card"。 Bot 應該會顯示主圖卡。

1. 輸入 "send me a botframework image"。 Bot 應該會顯示來自 Bot Framework 文件的影像。

1. 輸入任何其他東西，Bot 應該會回應， "You said" 與您的訊息 (放在引號中)。

### <a name="troubleshooting"></a>疑難排解

若您的 Bot 無法運作，請確認或重新輸入您的 Microsoft 應用程式識別碼與密碼。 即使您先前曾複製過它，也請檢查 Bot 通道註冊之設定刀鋒視窗上的 Microsoft 應用程式識別碼是否與 appsettings.json 檔案中 **"MicrosoftAppId"** 欄位的值相符。

檢查您的密碼或建立並使用新密碼： 

1. 按一下您 Bot 通道註冊刀鋒視窗上 [Microsoft 應用程式識別碼] 欄位旁的 [管理]。

1. 登入應用程式註冊入口網站。

1. 檢查您密碼的前三個字元是否符合 **appsettings.json** 檔案中的 **"MicrosoftAppPassword"** 欄位。 

1. 若值不相符，請產生新密碼並將該值儲存在 **appsettings.json** 檔案的 **"MicrosoftAppPassword"** 欄位中。

# <a name="javascripttabjscreatebot"></a>[JavaScript](#tab/jscreatebot)

### <a name="create-web-app-bot-on-microsoft-azure"></a>在 Microsoft Azure 上建立 Web 應用程式 Bot

在 Microsoft Azure 上建立 Bot： 

1. 在 Microsoft Azure 入口網站中，按一下 [建立資源]，然後搜尋「Web 應用程式 Bot」。

1. 按一下頁面底部的 [新增] 。 [Web 應用程式 Bot] 刀鋒視窗隨即出現。

![Web 應用程式 Bot 註冊](media/bot-builder-howto-direct-line/web-app-bot-registration.png)

1. 在 [Web 應用程式 Bot] 刀鋒視窗上，輸入**Bot 名稱**、**訂用帳戶**、**資源群組**、**位置**、**定價層**、**應用程式名稱**。

1. 選取您要使用的 Bot 範本。 **SDK 版本：SDK v4** 與 **SDK 語言：Node.js** 

1. 選取基本 v4 預覽版範本。 

1. 選擇您的 [App Service 方案/位置]、[Azure 儲存體] 與 [Application Insights 位置]。

1. 選取 [釘選到儀表板] 核取方塊。

1. 按一下 [建立] 按鈕。

1. 等候系統部署您的 Bot。 因為您已選取 [釘選到儀表板] 核取方塊，您的 Bot 將會出現在儀表板上。

### <a name="edit-your-direct-line-bot"></a>編輯您的直接線路 Bot

現在讓我們新增一些功能到 echo Bot。

1. 在 [Web 應用程式 Bot] 刀鋒視窗上，按一下 [建置]。 

1. 按一下 [下載 zip 檔]。 

1. 將 .zip 檔案解壓縮至本機目錄。

1. 瀏覽到解壓縮的資料夾並在您慣用的 IDE 中開啟原始程式檔。

1. 複製下面的程式碼並貼到 app.js 檔案中。

```javascript
// Copyright (c) Microsoft Corporation. All rights reserved.
// Licensed under the MIT License.

const { BotFrameworkAdapter, MessageFactory, CardFactory } = require('botbuilder');
const restify = require('restify');

// Create server
let server = restify.createServer();
server.listen(process.env.port || process.env.PORT || 3978, function () {
    console.log(`${server.name} listening to ${server.url}`);
});

// Create adapter
const adapter = new BotFrameworkAdapter({
    appId: process.env.MicrosoftAppId,
    appPassword: process.env.MicrosoftAppPassword
});

// Responds to the incoming message by either sending a hero card, an image, 
// or echoing the user's message.

// Listen for incoming requests 
server.post('/api/messages', (req, res) => {
    // Route received request to adapter for processing
    adapter.processActivity(req, res, (context) => {
        if (context.activity.type === 'message') {
            const text = (context.activity.text || '').trim().toLowerCase()

            switch(text){
                case 'hi':
                case "hello":
                case "help":
                    // Send the user an instruction message.
                    return context.sendActivity("Welcome to the Bot to showcase the DirectLine API. " +
                    "Send \"Show me a hero card\" or \"Send me a BotFramework image\" to see how the " +
                    "DirectLine client supports custom channel data. Any other message will be echoed.");
                    break;

                case 'show me a hero card':
                    // Create the hero card.
                    const message = MessageFactory.attachment(
                        CardFactory.heroCard(   
                        'Sample Hero Card', //cards title
                        'Displayed in the DirectLine client' //cards text
                        )
                    );
                    return context.sendActivity(message);
                    break;
                    
                case 'send me a botframework image':
                    // Create the image attachment.
                    const imageOrVideoMessage = MessageFactory.contentUrl('https://docs.microsoft.com/en-us/azure/bot-service/media/how-it-works/architecture-resize.png', 'image/png')
                    return context.sendActivity(imageOrVideoMessage);
                    break;
                
                default:
                    // No command was encountered. Echo the user's message.
                    return context.sendActivity(`You said ${context.activity.text}`);
                    break;
                    
            }
        }
    });
});

```

當您就緒時，您可以將原始程式檔發佈回 Azure。 依照這些步驟了解如何將您的 Bot 發佈回 [Azure](../bot-service-build-download-source-code.md)

---


## <a name="create-the-console-client-app"></a>建立主控台用戶端應用程式

# <a name="ctabcsclientapp"></a>[C#](#tab/csclientapp)

主控台用戶端應用程式會以兩個執行緒運作。 主要執行緒會接受使用者輸入並將訊息傳送到Bot。 次要執行緒會每秒輪詢一次 Bot，以從 Bot 擷取任何訊息，然後顯示收到的訊息。

建立主控台專案：

1. 在 [方案總管] 中，以滑鼠右鍵按一下 [方案 'DirectLineBotSample']。

1. 選擇 [加入] > [新增專案]。

1. 在 **Visual C#** > [Windows 傳統桌面]* 中，選擇 [主控台應用程式 (.NET Framework)]。
 
    **注意：** 請勿選擇 [主控台應用程式 (.NET Core)]。

1. 針對名稱，請輸入 **DirectLineClientSample**。

1. 按一下 [確定]。

### <a name="add-the-nuget-packages-to-the-console-app"></a>將 NuGet 套件新增至主控台應用程式

1. 以滑鼠右鍵按一下 [參考] 。

1. 按一下 [管理 NuGet 封裝] 。

1. 按一下 [瀏覽] 。 確定已選取 [包含發行前版本] 核取方塊。

1. 搜尋並安裝下列 NuGet 封裝：
    - Microsoft.Bot.Connector.DirectLine (v3.0.2)
    - Newtonsoft.Json

### <a name="edit-the-programcs-file"></a>編輯 Program.cs 檔案

使用下列程式碼取代 DirectLineClientSample **Program.cs** 檔案的內容：

```csharp
using System;
using System.Diagnostics;
using System.Linq;
using System.Threading.Tasks;
using Microsoft.Bot.Connector.DirectLine;
using Newtonsoft.Json;

namespace DirectLineClientSample
{
    class Program
    {
        // ************
        // Replace the following values with your Direct Line secret and the name of your bot resource ID.
        //*************
        private static string directLineSecret = "*** Replace with Direct Line secret ***";
        private static string botId = "*** Replace with the resource name of your bot ***";

        // This gives a name to the bot user.
        private static string fromUser = "DirectLineClientSampleUser";

        static void Main(string[] args)
        {
            StartBotConversation().Wait();
        }


        /// <summary>
        /// Drives the user's conversation with the bot.
        /// </summary>
        /// <returns></returns>
        private static async Task StartBotConversation()
        {
            // Create a new Direct Line client.
            DirectLineClient client = new DirectLineClient(directLineSecret);

            // Start the conversation.
            var conversation = await client.Conversations.StartConversationAsync();

            // Start the bot message reader in a separate thread.
            new System.Threading.Thread(async () => await ReadBotMessagesAsync(client, conversation.ConversationId)).Start();

            // Prompt the user to start talking to the bot.
            Console.Write("Type your message (or \"exit\" to end): ");

            // Loop until the user chooses to exit this loop.
            while (true)
            {
                // Accept the input from the user.
                string input = Console.ReadLine().Trim();

                // Check to see if the user wants to exit.
                if (input.ToLower() == "exit")
                {
                    // Exit the app if the user requests it.
                    break;
                }
                else
                {
                    if (input.Length > 0)
                    {
                        // Create a message activity with the text the user entered.
                        Activity userMessage = new Activity
                        {
                            From = new ChannelAccount(fromUser),
                            Text = input,
                            Type = ActivityTypes.Message
                        };

                        // Send the message activity to the bot.
                        await client.Conversations.PostActivityAsync(conversation.ConversationId, userMessage);
                    }
                }
            }
        }


        /// <summary>
        /// Polls the bot continuously and retrieves messages sent by the bot to the client.
        /// </summary>
        /// <param name="client">The Direct Line client.</param>
        /// <param name="conversationId">The conversation ID.</param>
        /// <returns></returns>
        private static async Task ReadBotMessagesAsync(DirectLineClient client, string conversationId)
        {
            string watermark = null;

            // Poll the bot for replies once per second.
            while (true)
            {
                // Retrieve the activity set from the bot.
                var activitySet = await client.Conversations.GetActivitiesAsync(conversationId, watermark);
                watermark = activitySet?.Watermark;

                // Extract the activies sent from our bot.
                var activities = from x in activitySet.Activities
                                 where x.From.Id == botId
                                 select x;

                // Analyze each activity in the activity set.
                foreach (Activity activity in activities)
                {
                    // Display the text of the activity.
                    Console.WriteLine(activity.Text);

                    // Are there any attachments?
                    if (activity.Attachments != null)
                    {
                        // Extract each attachment from the activity.
                        foreach (Attachment attachment in activity.Attachments)
                        {
                            switch (attachment.ContentType)
                            {
                                // Display a hero card.
                                case "application/vnd.microsoft.card.hero":
                                    RenderHeroCard(attachment);
                                    break;

                                // Display the image in a browser.
                                case "image/png":
                                    Console.WriteLine($"Opening the requested image '{attachment.ContentUrl}'");
                                    Process.Start(attachment.ContentUrl);
                                    break;
                            }
                        }
                    }

                    // Redisplay the user prompt.
                    Console.Write("\nType your message (\"exit\" to end): ");
                }

                // Wait for one second before polling the bot again.
                await Task.Delay(TimeSpan.FromSeconds(1)).ConfigureAwait(false);
            }
        }


        /// <summary>
        /// Displays the hero card on the console.
        /// </summary>
        /// <param name="attachment">The attachment that contains the hero card.</param>
        private static void RenderHeroCard(Attachment attachment)
        {
            const int Width = 70;
            // Function to center a string between asterisks.
            Func<string, string> contentLine = (content) => string.Format($"{{0, -{Width}}}", string.Format("{0," + ((Width + content.Length) / 2).ToString() + "}", content));

            // Extract the hero card data.
            var heroCard = JsonConvert.DeserializeObject<HeroCard>(attachment.Content.ToString());

            // Display the hero card.
            if (heroCard != null)
            {
                Console.WriteLine("/{0}", new string('*', Width + 1));
                Console.WriteLine("*{0}*", contentLine(heroCard.Title));
                Console.WriteLine("*{0}*", new string(' ', Width));
                Console.WriteLine("*{0}*", contentLine(heroCard.Text));
                Console.WriteLine("{0}/", new string('*', Width + 1));
            }
        }
    }
}
```

若要確認項目是否都正確，請按 **F6** 以建置專案。 不應該有警告或錯誤。

# <a name="javascripttabjsclientapp"></a>[JavaScript](#tab/jsclientapp)

### <a name="create-a-direct-line-client"></a>建立直接線路用戶端 

現在您已部署您的 Wb 應用程式 Bot，我們可以建立直接線路用戶端了。

```
    md DirectLineClient
    cd DirectLineClient
    npm init
```

接著，從 npm 安裝這些套件

```
    npm install --save open
    npm install --save request
    npm install --save request-promise
    npm install --save swagger-client
```

建立 **app.js** 檔案。 複製下面的程式碼並貼到此檔案中。

我們將會在下一個步驟取得 directLineSecret。
```javascript
var Swagger = require('swagger-client');
var open = require('open');
var rp = require('request-promise');

// config items
var pollInterval = 1000;
// Change the Direct Line Secret to your own 
var directLineSecret = 'your secret here';
var directLineClientName = 'DirectLineClient';
var directLineSpecUrl = 'https://docs.botframework.com/en-us/restapi/directline3/swagger.json';

var directLineClient = rp(directLineSpecUrl)
    .then(function (spec) {
        // client
        return new Swagger({
            spec: JSON.parse(spec.trim()),
            usePromise: true
        });
    })
    .then(function (client) {
        // add authorization header to client
        client.clientAuthorizations.add('AuthorizationBotConnector', new Swagger.ApiKeyAuthorization('Authorization', 'Bearer ' + directLineSecret, 'header'));
        return client;
    })
    .catch(function (err) {
        console.error('Error initializing DirectLine client', err);
    });

// once the client is ready, create a new conversation
directLineClient.then(function (client) {
    client.Conversations.Conversations_StartConversation()                          // create conversation
        .then(function (response) {
            return response.obj.conversationId;
        })                            // obtain id
        .then(function (conversationId) {
            sendMessagesFromConsole(client, conversationId);                        // start watching console input for sending new messages to bot
            pollMessages(client, conversationId);                                   // start polling messages from bot
        })
        .catch(function (err) {
            console.error('Error starting conversation', err);
        });
});

// Read from console (stdin) and send input to conversation using DirectLine client
function sendMessagesFromConsole(client, conversationId) {
    var stdin = process.openStdin();
    process.stdout.write('Command> ');
    stdin.addListener('data', function (e) {
        var input = e.toString().trim();
        if (input) {
            // exit
            if (input.toLowerCase() === 'exit') {
                return process.exit();
            }

            // send message
            client.Conversations.Conversations_PostActivity(
                {
                    conversationId: conversationId,
                    activity: {
                        textFormat: 'plain',
                        text: input,
                        type: 'message',
                        from: {
                            id: directLineClientName,
                            name: directLineClientName
                        }
                    }
                }).catch(function (err) {
                    console.error('Error sending message:', err);
                });

            process.stdout.write('Command> ');
        }
    });
}

// Poll Messages from conversation using DirectLine client
function pollMessages(client, conversationId) {
    console.log('Starting polling message for conversationId: ' + conversationId);
    var watermark = null;
    setInterval(function () {
        client.Conversations.Conversations_GetActivities({ conversationId: conversationId, watermark: watermark })
            .then(function (response) {
                watermark = response.obj.watermark;                                 // use watermark so subsequent requests skip old messages
                return response.obj.activities;
            })
            .then(printMessages);
    }, pollInterval);
}

// Helpers methods
function printMessages(activities) {
    if (activities && activities.length) {
        // ignore own messages
        activities = activities.filter(function (m) { return m.from.id !== directLineClientName });

        if (activities.length) {
            process.stdout.clearLine();
            process.stdout.cursorTo(0);

            // print other messages
            activities.forEach(printMessage);

            process.stdout.write('Command> ');
        }
    }
}

function printMessage(activity) {
    if (activity.text) {
        console.log(activity.text);
    }

    if (activity.attachments) {
        activity.attachments.forEach(function (attachment) {
            switch (attachment.contentType) {
                case "application/vnd.microsoft.card.hero":
                    renderHeroCard(attachment);
                    break;

                case "image/png":
                    console.log('Opening the requested image ' + attachment.contentUrl);
                    open(attachment.contentUrl);
                    break;
            }
        });
    }
}

function renderHeroCard(attachment) {
    var width = 70;
    var contentLine = function (content) {
        return ' '.repeat((width - content.length) / 2) +
            content +
            ' '.repeat((width - content.length) / 2);
    }

    console.log('/' + '*'.repeat(width + 1));
    console.log('*' + contentLine(attachment.content.title) + '*');
    console.log('*' + ' '.repeat(width) + '*');
    console.log('*' + contentLine(attachment.content.text) + '*');
    console.log('*'.repeat(width + 1) + '/');
}
```

---

## <a name="configure-the-direct-line-channel"></a>設定直接線路通道

新增直接線路通道可讓此 Bot 成為 Direct Line Bot。

設定直接線路通道：

![具有已反白顯示之 [直接線路通道] 按鈕的 [連線到通道] 刀鋒視窗。](media/bot-builder-howto-direct-line/direct-line-channel-button.png)

1. 在 [Bot 通道註冊] 刀鋒視窗上，按一下 [通道]。 將會出現 [連線到通道] 刀鋒視窗。

    ![具有兩個祕密金鑰的 [設定直接線路] 刀鋒視窗隨即顯示。](media/bot-builder-howto-direct-line/configure-direct-line.png)

1. 在 [連線到通道] 刀鋒視窗上，按一下 [設定直接線路通道] 按鈕。 

1. 若未選取 [3.0]，請選取該核取方塊。

1. 按一下至少一個祕密金鑰的 [顯示]。

1. 複製其中一個祕密金鑰並將它貼到您的用戶端應用程式的 **directLineSecret** 字串中。

1. 按一下 [完成]。

## <a name="run-the-client-app"></a>執行用戶端應用程式

# <a name="ctabcsrunclient"></a>[C#](#tab/csrunclient)

您的 Bot 現在準備好與直接線路主控台用戶端應用程式通訊了。 若要執行主控台應用程式，請執行下列動作：

1. 在 Visual Studio 中，以滑鼠右鍵按一下 [DirectLineClientSample] 專案並選取 [設定為起始專案]。

1. 檢查 **DirectLineClientSample** 專案中的 **Program.cs** 檔案。

    - 確認一個直接線路祕密程式碼位於 **directLineSecret** 字串中。

1. 若 **directLineSecret** 不正確，請移至 Azure 入口網站，按一下您的直接線路 Bot，按一下 [通道]，按一下 [設定直接線路通道] (或者 [編輯])，顯示金鑰。然後將該金鑰複製到 **directLineSecret** 字串中。

    - 確認資源群組位於 **botId** 字串中。

1. 若沒有，請從 [概觀] 刀鋒視窗複製 [資源群組] 並貼到 **botId** 字串中。

1. 按 F5 或 [開始偵錯]。

主控台用戶端應用程式將會啟動。 測試應用程式： 

1. 輸入 "Hi"。 Bot 應該會顯示 'You said "Hi"'。

1. 輸入 "show me a hero card"。 Bot 應該會顯示主圖卡。

1. 輸入 "send me a botframework image"。 Bot 將會啟動瀏覽器並顯示來自 Bot Framework 文件的影像。

1. 輸入任何其他東西，Bot 應該會回應， "You said" 與您的訊息 (放在引號中)。

# <a name="javascripttabjsrunclient"></a>[JavaScript](#tab/jsrunclient)

若要執行範例，您將必須執行您的 DirectLineClient 應用程式。

1. 開啟 CMD 主控台並 CD 到 DirectLineClient 目錄

1. 執行 `node app.js`

若要測試自訂訊息，請在用戶端的主控台輸入 show me a hero card 或 send me a botframework image，您應該會看到下列結果。

![結果](media/bot-builder-howto-direct-line/outcome.png)

---


### <a name="next-steps"></a>後續步驟
