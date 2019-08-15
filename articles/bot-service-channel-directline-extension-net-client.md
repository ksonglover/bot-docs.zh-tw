---
title: 建立連線至 Direct Line App Service 擴充功能的 .NET 用戶端
titleSuffix: Bot Service
description: 連線至 Direct Line App Service 擴充功能的 .NET 用戶端 (C#)
services: bot-service
manager: kamrani
ms.service: bot-service
ms.topic: conceptual
ms.author: kamrani
ms.date: 07/25/2019
ms.openlocfilehash: 62fda46569c6134f798b4d253a0676a037fdfa0f
ms.sourcegitcommit: 6a83b2c8ab2902121e8ee9531a7aa2d85b827396
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/09/2019
ms.locfileid: "68866461"
---
# <a name="create-net-client-to-connect-to-direct-line-app-service-extension"></a>建立連線至 Direct Line App Service 擴充功能的 .NET 用戶端

本文說明如何以 C# 建立連線至 Direct Line App Service 擴充功能的 .NET 用戶端。

## <a name="gather-your-direct-line-extension-keys"></a>收集您的 Direct Line 擴充功能金鑰

1. 在瀏覽器中，導覽至 [Azure 入口網站](https://portal.azure.com/)
1. 在 Azure 入口網站中，找出您的 **Azure Bot Service** 資源
1. 按一下 [通道]  以設定 Bot 的通道
1. 如果尚未啟用，請按一下 [Direct Line]  通道加以啟用。 
1. 如果已啟用，請在 [連線到通道] 資料表中，按一下[Direct Line] 資料列上的 [編輯]  連結。
1. 捲動至 [網站] 區段。 一般應會有預設網站，除非您已將其刪除或重新命名。
1. 按一下 [顯示連結]  以顯示其中一個金鑰，然後複製其值。

    ![App Service 擴充功能金鑰](./media/channels/direct-line-extension-extension-keys-net-client.png)

> [!NOTE]
> 此值是您用來連線至 Direct Line App Service 擴充功能的 Direct Line 用戶端密碼。 您想要的話，也可以建立其他網站，並同樣使用這些秘密值。

## <a name="add-the-preview-nuget-package-source"></a>新增預覽 Nuget 套件來源

建立 C# Direct Line 用戶端所需的預覽 NuGet 套件，可在 NuGet 摘要中找到。

1. 在 Visual Studio 中，導覽至 [工具] -> [選項]  功能表項目。
1. 選取 [NuGet 套件管理員] -> [套件來源]  項目。
1. 按一下 + 按鈕，以使用下列值新增套件來源：
    - 名稱：DL ASE 預覽
    - 來源： https://botbuilder.myget.org/F/experimental/api/v3/index.json
1. 按一下 [更新]  按鈕以儲存這些值。
1. 按一下 [確定]  以結束套件來源設定。

## <a name="create-a-c-direct-line-client"></a>建立 C# Direct Line 用戶端

Direct Line App Service 擴充功能的互動方式與傳統 Direct Line 有所不同，因為大部分的通訊都會透過 *WebSocket* 進行。 更新的 Direct Line 用戶端包含用來開啟和關閉 *WebSocket*、透過 WebSocket 傳送命令，以及從 Bot 接收活動的協助程式類別。 本節說明如何建立簡單的 C# 用戶端與 Bot 進行互動。

1. 在 Visual Studio 中，建立新的 .NET Core 2.2 主控台應用程式專案。
1. 將 **DirectLine 用戶端 NuGet** 新增至您的專案
    - 按一下解決方案樹狀結構中的 [相依性]
    - 選取 [管理 NuGet 套件...] 
    - 將套件來源變更為您先前定義的來源 (DL ASE 預覽)
    - 尋找套件 *Microsoft.Bot.Connector.Directline* v3.0.3-Preview1 版或更新版本。
    - 按一下 [安裝套件]  。
1. 建立用戶端，並使用秘密產生權杖。 此步驟與建置任何其他 C# Direct Line 用戶端的步驟相同，差別在於，您需要在 Bot 中使用的端點會附加 **.bot/** 路徑，如下所示。 別忘了尾端的 **/** 。

    ```csharp
    string endpoint = "https://<YOUR_BOT_HOST>.azurewebsites.net/.bot/";
    string secret = "<YOUR_BOT_SECRET>";

    var tokenClient = new DirectLineClient(
        new Uri(endpoint),
        new DirectLineClientCredentials(secret));
    var conversation = await tokenClient.Tokens.GenerateTokenForNewConversationAsync();
    ```

1. 一旦您在產生權杖後取得對話參考，即可使用此對話識別碼在 `DirectLineClient` 上開啟具有新 `StreamingConversations` 屬性的 WebSocket。 若要這樣做，您必須建立在 Bot 想要將 `ActivitySets` 傳送至用戶端時所將叫用的回呼：

    ```csharp
    public static void ReceiveActivities(ActivitySet activitySet)
    {
        if (activitySet != null)
        {
            foreach (var a in activitySet.Activities)
            {
                if (a.Type == ActivityTypes.Message && a.From.Id.Contains("bot"))
                {
                    Console.WriteLine($"<Bot>: {a.Text}");
                }
            }
        }
    }
    ```

1. 現在，您已可使用對話的權杖 `conversationId` 開啟具有 `StreamingConversations` 屬性的 WebSocket，和您的 `ReceiveActivities` 回呼：

    ```csharp
    var client = new DirectLineClient(
        new Uri(endpoint),
        new DirectLineClientCredentials(conversation.Token));

    await client.StreamingConversations.ConnectAsync(
        conversation.ConversationId,
        ReceiveActivities);
    ```

1. 用戶端現在可以用來啟動對話，並將 `Activities` 傳送至 Bot：

    ```csharp

    var startConversation = await client.StreamingConversations.StartConversationAsync();
    var from = new ChannelAccount() { Id = "123", Name = "Fred" };
    var message = Console.ReadLine();

    while (message != "end")
    {
        try
        {
            var response = await client.StreamingConversations.PostActivityAsync(
                startConversation.ConversationId,
                new Activity()
                {
                    Type = "message",
                    Text = message,
                    From = from
                });
        }
        catch (OperationException ex)
        {
            Console.WriteLine(
                $"OperationException when calling PostActivityAsync: ({ex.StatusCode})");
        }
        message = Console.ReadLine();
    }
    ```
