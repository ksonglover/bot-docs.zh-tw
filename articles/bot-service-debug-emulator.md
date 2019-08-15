---
title: 使用 Bot Framework 模擬器對 Bot 進行測試和偵錯 | Microsoft Docs
description: 了解如何使用 Bot Framework 模擬器桌面應用程式對 Bot 進行檢查、測試和偵錯。
keywords: 文字記錄, msbot 工具, 語言服務, 語音辨識
author: DeniseMak
ms.author: v-demak
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 2/26/2019
ms.openlocfilehash: 847ae51791ae66ef190ebefee765f2806ec91c5e
ms.sourcegitcommit: 6a83b2c8ab2902121e8ee9531a7aa2d85b827396
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/12/2019
ms.locfileid: "68484042"
---
# <a name="debug-with-the-emulator"></a>使用模擬器進行偵錯

Bot Framework 模擬器是一項桌面應用程式，可讓 Bot 開發人員在本機或從遠端對其 Bot 進行測試和偵錯。 使用此模擬器時，您能與您的 Bot 聊天，並檢查您的 Bot 所傳送及接收的訊息。 模擬器會顯示在 Web 聊天 UI 中出現的相同訊息，並在您與 Bot 交換訊息時記錄 JSON 要求和回應。 將您的 Bot 部署至雲端之前，請先在本機使用模擬器加以測試。 即使您尚未使用 Azure Bot Service [建好](./bot-service-quickstart.md) Bot 或將其設定為在任何通道上執行，您仍可使用模擬器來測試 Bot。

## <a name="prerequisites"></a>必要條件
- 安裝 [Bot Framework 模擬器](https://aka.ms/Emulator-wiki-getting-started)

## <a name="run-a-bot-locally"></a>在本機執行聊天機器人
在將聊天機器人連線至 Bot Framework Emulator 之前，您必須先在本機執行聊天機器人。 您可以使用 Visual Studio 或 Visual Studio Code 來執行聊天機器人，也可以使用命令列。 若要使用命令列來執行聊天機器人，請執行下列動作：


# <a name="ctabcsharp"></a>[C#](#tab/csharp)

* 移至命令提示字元，並將目錄變更為聊天機器人的專案目錄。
* 執行下列命令來啟動聊天機器人： 
    ```
    dotnet run
    ```
* 複製 *Application started. Press CTRL+C to shut down.* 前一行的連接埠號碼。

    ![C# 連接埠號碼](media/bot-service-debug-emulator/csharp_port_number.png)


# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

* 移至命令提示字元，並將目錄變更為聊天機器人的專案目錄。
* 執行下列命令來啟動聊天機器人：
    ```
    node index.js
    ```
* 複製 restify 正在接聽的連接埠號碼。

    ![JS 連接埠號碼](media/bot-service-debug-emulator/js_port_number.png)
---

至此，聊天機器人應該正在本機上執行。 


## <a name="connect-to-a-bot-running-on-localhost"></a>連線至執行於本機主機的 Bot

![模擬器 UI](media/emulator-v4/emulator-welcome.png)

若要連線至在本機上執行的聊天機器人，請按一下 [開啟聊天機器人]  。 將稍早複製的連接埠號碼新增至下列 URL，並在 [Bot URL] 列中貼上更新後的 URL：

*http://localhost:**埠號碼**/api/messages*

![模擬器 UI](media/bot-service-debug-emulator/open_bot_emulator.png)

如果您的聊天機器人搭配 [Microsoft 帳戶 (MSA) 認證](#use-bot-credentials)執行，請也輸入這些認證。


### <a name="use-bot-credentials"></a>使用聊天機器人認證

當您開啟聊天機器人時，請設定 **Microsoft 應用程式識別碼**和 **Microsoft 應用程式密碼** (如果您的聊天機器人使用這些認證來執行)。 如果您使用 Azure Bot Service 建立了聊天機器人，則可於聊天機器人的 App Service 上 (在 [設定] -> [組態]  區段底下) 取得這些認證。 如果您不知道這些值，則可以從本機執行的聊天機器人組態檔中移除這些值，然後在模擬器中執行聊天機器人。 如果聊天機器人並未使用這些設定來執行，則不需要使用任一設定來執行模擬器。 

## <a name="view-detailed-message-activity-with-the-inspector"></a>使用偵測器檢視詳細訊息活動

傳送訊息給聊天機器人，而聊天機器人應該會有所回應。 您可以按一下對話視窗內的訊息泡泡，並使用視窗右側的**偵測器**功能檢查原始 JSON 活動。 訊息泡泡經選取後會變成黃色，且活動 JSON 物件將會顯示在聊天視窗左側。 此 JSON 資訊包含主要中繼資料，包括通道識別碼、活動類型、交談識別碼、文字訊息，端點 URL 等。 您可以檢查使用者所傳送的活動，以及 Bot 所回應的活動。

![模擬器訊息活動](media/emulator-v4/emulator-view-message-activity-03.png)

> [!TIP]
> 您可以藉由在聊天機器人中新增[檢查中介軟體](bot-service-debug-inspection-middleware.md)，來對連線至通道的聊天機器人進行狀態變更的偵錯。

<!--
## Save and load conversations with bot transcripts

Activities in the emulator can be saved as transcripts. From an open live chat window, select **Save Transcript As** to the transcript file. The **Start Over** button can be used any time to clear a conversation and restart a connection to the bot.  

![Emulator save transcripts](media/emulator-v4/emulator-save-transcript.png)

To load transcripts, simply select **File > Open Transcript File** and select the transcript. A new Transcript window will open and render the message activity to the output window. 

![Emulator load transcripts](media/emulator-v4/emulator-load-transcript.png)
--->
<!---
## Add services 

You can easily add a LUIS app, QnA knowledge base, or dispatch model to your bot directly from the emulator. When the bot is loaded, select the services button on the far left of the emulator window. You will see options under the **Services** menu to add LUIS, QnA Maker, and Dispatch. 

To add a service app, simply click on the **+** button and select the service you want to add. You will be prompted to sign in to the Azure portal to add the service to the bot file, and connect the service to your bot application. 

> [!IMPORTANT]
> Adding services only works if you're using a `.bot` configuration file. Services will need to be added independently. For details on that, see [Manage bot resources](v4sdk/bot-file-basics.md) or the individual how to articles for the service you're trying to add.
>
> If you are not using a `.bot` file, the left pane won't have your services listed (even if your bot uses services) and will display *Services not available*.

![LUIS connect](media/emulator-v4/emulator-connect-luis-btn.png)

When either service is connected, you can go back to a live chat window and verify that your services are connected and working. 

![QnA connected](media/emulator-v4/emulator-view-message-activity.png)

--->

## <a name="inspect-services"></a>檢查服務

透過新的 v4 模擬器，您也可以檢查 LUIS 和 QnA 的 JSON 回應。 使用已連接語言服務的 Bot 時，您可以在右下方選取 [記錄] 視窗中的 [追蹤]  。 這項新工具也提供直接從模擬器更新語言服務的功能。 

![LUIS 偵測器](media/emulator-v4/emulator-luis-inspector.png)

在已連接 LUIS 服務的情況下，您會發現追蹤連結指定了 **Luis 追蹤**。 加以選取後，您會看到 LUIS 服務的原始回應，其中包含意圖、實體及其指定的分數。 您也可以選擇重新指派使用者語句的意圖。 

![QnA 偵測器](media/emulator-v4/emulator-qna-inspector.png)

在已連接 QnA 服務的情況下，記錄將會顯示 **QnA 追蹤**，若加以選取，您將可預覽與該活動相關聯的問答組和信賴分數。 在此處，您可以新增答案的替代問題用語。

<!--## Configure ngrok

If you are using Windows and you are running the Bot Framework Emulator behind a firewall or other network boundary and want to connect to a bot that is hosted remotely, you must install and configure **ngrok** tunneling software. The Bot Framework Emulator integrates tightly with ngrok tunnelling software (developed by [inconshreveable][inconshreveable]), and can launch it automatically when it is needed.

Open the **Emulator Settings**, enter the path to ngrok, select whether or not to bypass ngrok for local addresses, and click **Save**.

![ngrok path](media/emulator-v4/emulator-ngrok-path.png)
-->

<!---## Login to Azure

You can use Emulator to login in to your Azure account. This is particularly helpful for you to add and manage services your bot depends on. 
See [above](#add-services) to learn more about services you can manage using the Emulator.
-->

### <a name="login-to-azure"></a>登入 Azure
您可以使用模擬器來登入您的 Azure 帳戶。 這特別適合用來讓您新增及管理 Bot 所依賴的服務。 遵循下列步驟來登入 Azure：
- 按一下 [檔案] -> [使用 Azure 登入] ![Azure 登入](media/emulator-v4/emulator-azure-login.png)
- 在歡迎畫面上按一下 [使用 Azure 帳戶登入]，您可以選擇性地讓模擬器保留您的登入狀態，以在模擬器應用程式重新啟動之間保持登入。
![Azure 登入](media/emulator-v4/emulator-azure-login-success.png)

## <a name="disabling-data-collection"></a>停用資料收集

如果您決定不再允許模擬器收集使用資料，您可以依照下列步驟來輕鬆地停用資料收集：

1. 按一下左側瀏覽列中的 [設定] 按鈕 (齒輪圖示)，瀏覽至模擬器的 [設定] 頁面。

    ![停用資料收集](media/emulator-v4/emulator-disable-data-1.png)

2. 在 [資料收集]  區段底下，取消選取標示 [允許我們收集使用資料，以協助改善模擬器]  的核取方塊。

    ![停用資料收集](media/emulator-v4/emulator-disable-data-2.png)

3. 按一下 [儲存] 按鈕。

    ![停用資料收集](media/emulator-v4/emulator-disable-data-3.png)
    
如果您改變心意，一律可以藉由重新選取核取方塊來將其啟用。

## <a name="additional-resources"></a>其他資源

Bot Framework 模擬器是開放原始碼。 您可以[參與][EmulatorGithubContribute]開發，並[提交 Bug 和建議][EmulatorGithubBugs]。

如需進行疑難排解，請參閱[針對一般問題疑難排解](bot-service-troubleshoot-bot-configuration.md)以及該區段中的其他疑難排解文章。

## <a name="next-steps"></a>後續步驟

使用檢查中介軟體來對連線至通道的聊天機器人進行偵錯。

> [!div class="nextstepaction"]
> [使用文字記錄檔進行 Bot 偵錯](bot-service-debug-inspection-middleware.md)

<!--
Saving a conversation to a transcript file allows you to quickly draft and replay a certain set of interactions for debugging.

> [!div class="nextstepaction"]
> [Debug your bot using transcript files](~/v4sdk/bot-builder-debug-transcript.md)
-->

<!-- Footnote-style URLs -->

[EmulatorGithubContribute]: https://github.com/Microsoft/BotFramework-Emulator/wiki/How-to-Contribute
[EmulatorGithubBugs]: https://github.com/Microsoft/BotFramework-Emulator/wiki/Submitting-Bugs-%26-Suggestions

[ngrokDownload]: https://ngrok.com/
[inconshreveable]: https://inconshreveable.com/
