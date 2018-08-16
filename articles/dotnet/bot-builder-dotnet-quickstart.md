---
title: 使用適用於 .NET 的 Bot Builder SDK 建立 Bot | Microsoft Docs
description: 使用適用於 .NET 的 Bot Builder SDK 建立 Bot，這個 SDK 是功能強大的 Bot 建構架構。
keywords: Bot Builder SDK, 建立 Bot, 快速入門, 開始使用
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: get-started-article
ms.prod: bot-framework
ms.date: 04/27/2018
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: fcb21fa38750c09f110a3c71f763a941c4437979
ms.sourcegitcommit: f576981342fb3361216675815714e24281e20ddf
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/18/2018
ms.locfileid: "39300302"
---
# <a name="create-a-bot-with-the-bot-builder-sdk-for-net"></a>使用適用於 .NET 的 Bot Builder SDK 建立 Bot
> [!div class="op_single_selector"]
> - [.NET](../dotnet/bot-builder-dotnet-quickstart.md)
> - [Node.js](../nodejs/bot-builder-nodejs-quickstart.md)
> - [Bot 服務](../bot-service-quickstart.md)
> - [REST](../rest-api/bot-framework-rest-connector-quickstart.md)

<a href="https://github.com/Microsoft/BotBuilder" target="_blank">適用於 .NET 的 Bot Builder SDK</a> 是易於使用的架構，可使用 Visual Studio 和 Windows 來開發 Bot。 這個 SDK 可以利用 C#，為 .NET 開發人員提供熟悉的方法來建立功能強大的 Bot。


本教學課程會逐步引導您使用 Bot 應用程式範本和適用於 .NET 的 Bot Builder SDK 來建置 Bot，然後使用 Bot Framework 模擬器進行測試。

## <a name="prerequisites"></a>必要條件
1. Visual Studio [2017](https://www.visualstudio.com/)。

2. 在 Visual Studio 中，將所有的[延伸模組](https://docs.microsoft.com/en-us/visualstudio/extensibility/how-to-update-a-visual-studio-extension)更新至其最新版本。

3. 適用於 [C#](https://marketplace.visualstudio.com/items?itemName=BotBuilder.BotBuilderV3) 的 Bot 範本。

> [!TIP]
> Visual Studio 2017 專案範本目錄通常位於 `%USERPROFILE%\Documents\Visual Studio 2017\Templates\ProjectTemplates\Visual C#\`，而項目範本目錄位於 `%USERPROFILE%\Documents\Visual Studio 2017\Templates\ItemTemplates\Visual C#\`

## <a name="create-your-bot"></a>建立您的 Bot

開啟 Visual Studio，並建立新的 C# 專案。 針對您的新專案選擇 [Simple Echo Bot Application] 範本。

![Visual Studio 建立專案](../media/connector-getstarted-create-project.png)

> [!NOTE]
> Visual Studio 可能表示您需要下載並安裝 [IIS Express](https://www.microsoft.com/en-us/download/details.aspx?id=48264)。 

由於 Bot 應用程式範本，您的專案會包含在本教學課程中建立 Bot 所需的所有程式碼。 您實際上不需要撰寫任何額外的程式碼。 不過，在我們繼續測試您的 Bot 之前，請快速查看 Bot 應用程式範本所提供的一些程式碼。

> [!TIP] 
> 視需要更新 [NuGet 套件](https://docs.microsoft.com/en-us/nuget/quickstart/install-and-use-a-package-in-visual-studio)。

## <a name="explore-the-code"></a>探索程式碼

首先，**Controllers\MessagesController.cs** 內的 `Post` 方法會接收來自使用者的訊息，並叫用根對話。

```csharp
public async Task<HttpResponseMessage> Post([FromBody]Activity activity)
{
    if (activity.GetActivityType() == ActivityTypes.Message)
    {
        await Conversation.SendAsync(activity, () => new Dialogs.RootDialog());
    }
    else
    {
        HandleSystemMessage(activity);
    }
    var response = Request.CreateResponse(HttpStatusCode.OK);
    return response;
}

```

根對話會處理訊息並產生回應。 **Dialogs\RootDialog.cs** 內的 `MessageReceivedAsync` 方法會傳送回覆，在使用者的訊息前面加上 'You sent' 的文字，並在後面加上 'which was *##* characters' 的文字來回應，其中 *##* 代表使用者訊息中的字元數目。

```csharp
public class RootDialog : IDialog<object>
{
    public Task StartAsync(IDialogContext context)
    {
        context.Wait(MessageReceivedAsync);

        return Task.CompletedTask;
    }

    private async Task MessageReceivedAsync(IDialogContext context, IAwaitable<object> result)
    {
        var activity = await result as Activity;

        // Calculate something for us to return
        int length = (activity.Text ?? string.Empty).Length;

        // Return our reply to the user
        await context.PostAsync($"You sent {activity.Text} which was {length} characters");

        context.Wait(MessageReceivedAsync);
    }
}
```

## <a name="test-your-bot"></a>測試 Bot

[!INCLUDE [Get started test your bot](../includes/snippet-getstarted-test-bot.md)]

### <a name="start-your-bot"></a>啟動 Bot

安裝模擬器之後，使用瀏覽器作為應用程式主機，在 Visual Studio 中啟動您的 Bot。
這個 Visual Studio 螢幕擷取畫面顯示 Bot 將在按一下執行按鈕時，於 Microsoft Edge 中啟動。

![Visual Studio 執行專案](../media/connector-getstarted-start-bot-locally.png)

當您按一下執行按鈕時，Visual Studio 將會建置應用程式、將它部署到 localhost，並啟動 Web 瀏覽器來顯示應用程式的 **default.htm** 頁面。
例如，以下是 Microsoft Edge 中所顯示的應用程式 **default.htm** 頁面：

![Visual Studio 執行 localhost 的 Bot](../media/connector-getstarted-bot-running-localhost.png)

> [!NOTE]
> 您可以在專案內修改 **default.htm** 檔案，以指定 Bot 應用程式的名稱和描述。

### <a name="start-the-emulator-and-connect-your-bot"></a>啟動模擬器並連線到您的 Bot

此時，您的 Bot 正在本機執行。
接下來，啟動模擬器，然後在模擬器中連線到您的 Bot：

1. 建立新的 Bot 設定。 將 `http://localhost:port-number/api/messages` 輸入網址列，其中 *port-number* 必須符合應用程式執行所在之瀏覽器中所顯示的連接埠號碼。

2. 按一下 [儲存並連線]。 您不需要指定 **Microsoft 應用程式識別碼**和 **Microsoft 應用程式密碼**。 目前可以將這些欄位保留空白。 稍後[註冊 Bot](~/bot-service-quickstart-registration.md) 時，您將會取得這項資訊。

### <a name="test-your-bot"></a>測試 Bot

現在，您的 Bot 會在本機執行，並且已連線到模擬器，請在模擬器中輸入一些訊息來測試您的 Bot。
您應該會看到 Bot 回應您所傳送之每個訊息的方式是，在訊息前面加上 'You sent' 的文字並在後面加上 'which was *##* characters' 的文字來回應，其中 *##* 是您所傳送之訊息中的字元總數。


> [!TIP]
> 在模擬器中，按一下對話中的任一個語音泡泡。 有關訊息的詳細資料將會以 JSON 格式出現在 [詳細資料] 窗格中。

您已經使用 Bot 應用程式範本和適用於 .NET 的 Bot Builder SDK 成功建立 Bot！

## <a name="next-steps"></a>後續步驟

在本快速入門中，您可以使用 Bot 應用程式範本和適用於 .NET 的 Bot Builder SDK 建立簡單的 Bot，並使用 Bot Framework 模擬器來驗證 Bot 的功能。

接下來，將了解適用於 .NET 的 Bot Builder SDK 重要概念。

> [!div class="nextstepaction"]
> [適用於 .NET 的 Bot Builder SDK 重要概念](bot-builder-dotnet-concepts.md)
