---
title: 具有 Direct Line App Service 擴充功能的 .NET Bot
titleSuffix: Bot Service
description: 讓 .NET Bot 使用 Direct Line App Service 擴充功能
services: bot-service
manager: kamrani
ms.service: bot-service
ms.topic: conceptual
ms.author: kamrani
ms.date: 07/25/2019
ms.openlocfilehash: 3ed589bff5c3740dddcfb62226714006313ae330
ms.sourcegitcommit: 312a4593177840433dfee405335100ce59aac347
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 11/12/2019
ms.locfileid: "73933682"
---
# <a name="configure-net-bot-for-extension"></a>設定 .NET Bot 擴充功能

[!INCLUDE[applies-to-v4](includes/applies-to.md)]

本文說明如何更新 Bot 以使用**具名管道**，以及如何在裝載 Bot 的 **Azure App Service** 資源中啟用 Direct Line App Service 擴充功能。  

## <a name="prerequisites"></a>必要條件

若要執行後續說明的步驟，您在 Azure 中必須具有 **Azure App Service** 資源和相關的 **App Service**。

## <a name="enable-direct-line-app-service-extension"></a>啟用 Direct Line App Service 擴充功能

本文說明如何使用 Bot 通道組態中的金鑰，在裝載 Bot 的 **Azure App Service** 資源中啟用 Direct Line App Service 擴充功能。

## <a name="update-net-bot-to-use-direct-line-app-service-extension"></a>更新 .NET Bot 以使用 Direct Line App Service 擴充功能

1. 在 Visual Studio 中，開啟您的 Bot 專案。
1. 將**串流擴充功能 NuGet** 套件新增至您的專案：
    1. 在您的專案中，以滑鼠右鍵按一下 [相依性]  ，然後選取 [管理 NuGet 套件]  。
    1. 在 [瀏覽]  索引標籤下，按一下 [包含發行前版本]  ，以顯示預覽套件。
    1. 選取套件 **Microsoft.Bot.Builder.StreamingExtensions**。
    1. 按一下 [安裝]  按鈕以安裝套件；閱讀並同意授權合約。
1. 允許您的應用程式使用 **Bot Framework NamedPipe**：
    - 開啟 `Startup.cs` 檔案。
    - 在 ``Configure`` 方法中，將程式碼新增至 ``UseBotFrameworkNamedPipe``

    ```csharp

    using Microsoft.Bot.Builder.StreamingExtensions;

    public void Configure(IApplicationBuilder app, IHostingEnvironment env)
    {
        if (env.IsDevelopment())
        {
            app.UseDeveloperExceptionPage();
        }
        else
        {
            app.UseHsts();
        }

        app.UseDefaultFiles();
        app.UseStaticFiles();

        // Allow bot to use named pipes.
        app.UseBotFrameworkNamedPipe();

        app.UseMvc();
    }
    ```

1. 儲存 `Startup.cs` 檔案。
1. 開啟 `appsettings.json` 檔案並輸入下列值：
    1. `"MicrosoftAppId": "<secret Id>"`
    1. `"MicrosoftAppPassword": "<secret password>"`

    這些值是與服務註冊群組相關聯的 **appid** 和 **appSecret**。

1. 將 Bot **發佈**至您的 Azure App Service。
1. 在瀏覽器中，導覽至 https://<your_app_service>.azurewebsites.net/.bot。 如果一切都正確無誤，頁面將會傳回下列 JSON 內容：`{"k":true,"ib":true,"ob":true,"initialized":true}`。 這是您在**一切運作正常**時所取得的資訊，其中

    - **k** 決定 Direct Line App Service 擴充功能 (ASE) 是否可以從其組態中讀取擴充功能金鑰。 
    - **initialized** 可決定 Direct Line ASE 是否可以使用擴充功能金鑰從 Azure Bot Service 下載 Bot 中繼資料
    - **ib** 決定 Direct Line ASE 是否可以與 Bot 建立輸入連線。
    - **ob** 可決定 Direct Line ASE 是否可以與 Bot 建立輸出連線。 


### <a name="gather-your-direct-line-extension-keys"></a>收集您的 Direct Line 擴充功能金鑰

1. 在瀏覽器中，導覽至 [Azure 入口網站](https://portal.azure.com/)
1. 在 Azure 入口網站中，找出您的 **Azure Bot Service** 資源
1. 按一下 [通道]  以設定 Bot 的通道
1. 如果尚未啟用，請按一下 [Direct Line]  通道加以啟用。 
1. 如果已啟用，請在 [連線到通道] 資料表中，按一下[Direct Line] 資料列上的 [編輯]  連結。
1. 向下捲動至 [App Service 擴充功能金鑰] 區段。 
1. 按一下 [顯示連結]  以顯示其中一個金鑰，然後複製其值。

![App Service 擴充功能金鑰](./media/channels/direct-line-extension-extension-keys.png)

### <a name="enable-the-direct-line-app-service-extension"></a>啟用 Direct Line App Service 擴充功能

1. 在瀏覽器中，導覽至 [Azure 入口網站](https://portal.azure.com/)
1. 在 Azure 入口網站中，找出您的 Bot 裝載所在 Web 應用程式的 [Azure App Service]  資源頁面
1. 按一下 [組態]  。 在 [應用程式設定]  區段下方，新增下列新設定：

    |名稱|值|
    |---|---|
    |DirectLineExtensionKey|<App_Service_Extension_Key_From_Section_1>|
    |DIRECTLINE_EXTENSION_VERSION|最新|

1. 在 [組態]  區段中按一下 [一般設定]  區段，然後開啟 [Web 通訊端] 
1. 按一下 [儲存]  以儲存設定。 這會使 Azure App Service 重新啟動。
