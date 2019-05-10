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
ms.openlocfilehash: 307a6bf697e274391336a0d216c64da85232616d
ms.sourcegitcommit: f84b56beecd41debe6baf056e98332f20b646bda
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 05/03/2019
ms.locfileid: "65033322"
---
# <a name="debug-with-the-emulator"></a>使用模擬器進行偵錯

Bot Framework 模擬器是一項桌面應用程式，可讓 Bot 開發人員在本機或從遠端對其 Bot 進行測試和偵錯。 使用此模擬器時，您能與您的 Bot 聊天，並檢查您的 Bot 所傳送及接收的訊息。 模擬器會顯示在 Web 聊天 UI 中出現的相同訊息，並在您與 Bot 交換訊息時記錄 JSON 要求和回應。 將您的 Bot 部署至雲端之前，請先在本機使用模擬器加以測試。 即使您尚未使用 Azure Bot Service [建好](./bot-service-quickstart.md) Bot 或將其設定為在任何通道上執行，您仍可使用模擬器來測試 Bot。

## <a name="prerequisites"></a>必要條件
- 安裝[模擬器](https://aka.ms/Emulator-wiki-getting-started)

## <a name="connect-to-a-bot-running-on-localhost"></a>連線至執行於本機主機的 Bot

![模擬器 UI](media/emulator-v4/emulator-welcome.png)

若要連線到本機上執行的 Bot，請按一下 [開啟 Bot]，或選取您預先設定的組態檔 (.bot 檔案)。 您不需要用組態檔來連線到 Bot，但如果您的 Bot 有組態檔，模擬器仍會與其搭配使用。 如果您的 Bot 搭配 Microsoft 帳戶 (MSA) 認證執行，請也輸入這些認證。

![模擬器 UI](media/emulator-v4/emulator-open-bot.png)

## <a name="view-detailed-message-activity-with-the-inspector"></a>使用偵測器檢視詳細訊息活動

傳送訊息給您的 Bot，而 Bot 應該會有所回應。 您可以按一下對話視窗內的訊息泡泡，並使用視窗右側的**偵測器**功能檢查原始 JSON 活動。 訊息泡泡經選取後會變成黃色，且活動 JSON 物件將會顯示在聊天視窗左側。 JSON 資訊包含主要中繼資料，包括 channelID、活動類型、交談識別碼、文字訊息，端點 URL 等。您可以檢查使用者所傳送的活動，以及 Bot 所回應的活動。 

![模擬器訊息活動](media/emulator-v4/emulator-view-message-activity-03.png)

## <a name="save-and-load-conversations-with-bot-transcripts"></a>儲存及載入與 Bot 交談的文字記錄

模擬器中的活動可以儲存為文字記錄。 從開啟的即時聊天視窗中，選取 [另存文字記錄]，以儲存文字記錄檔案。 您可以隨時使用 [重新開始] 按鈕來清除對話，並重新連線至 Bot。  

![模擬器儲存文字記錄](media/emulator-v4/emulator-save-transcript.png)

若要載入文字記錄，只需選取 [檔案] > [開啟文字記錄檔案]，再選取文字記錄即可。 新的 [文字記錄] 視窗隨即開啟，並將訊息活動轉譯為輸出視窗。 

![模擬器載入文字記錄](media/emulator-v4/emulator-load-transcript.png)

## <a name="add-services"></a>新增服務 

您可以直接從模擬器輕鬆地將 LUIS 應用程式、QnA 知識庫或分派模型新增到 Bot。 載入 Bot 後，請選取模擬器視窗最左側的 [服務] 按鈕。 您會在 [服務] 功能表下方看到用來新增 LUIS、QnA Maker 和分派的選項。 

若要新增服務應用程式，只需按一下 [+] 按鈕，然後選取您想要新增的服務。 系統會提示您登入 Azure 入口網站，以將服務新增至 Bot 檔案，並將服務連線到您的 Bot 應用程式。 

> [!IMPORTANT]
> 只有在您使用 `.bot` 組態檔時，才能新增服務。 服務需要個別加入。 如需詳細資訊，請參閱[管理 Bot 資源](v4sdk/bot-file-basics.md)，或針對您要新增的服務，參閱個別的操作說明文章。
>
> 如果您不是使用 `.bot` 檔案，左側窗格將不會列出您的服務 (即使您的 Bot 有使用服務)，而且會顯示*服務無法使用*。

![LUIS 連線](media/emulator-v4/emulator-connect-luis-btn.png)

在任一服務連線時，您都可以返回即時聊天視窗，並確認您的服務已連線並運作中。 

![QnA 已連線](media/emulator-v4/emulator-view-message-activity.png)

## <a name="inspect-services"></a>檢查服務

透過新的 v4 模擬器，您也可以檢查 LUIS 和 QnA 的 JSON 回應。 使用已連接語言服務的 Bot 時，您可以在右下方選取 [記錄] 視窗中的 [追蹤]。 這項新工具也提供直接從模擬器更新語言服務的功能。 

![LUIS 偵測器](media/emulator-v4/emulator-luis-inspector.png)

在已連接 LUIS 服務的情況下，您會發現追蹤連結指定了 **Luis 追蹤**。 加以選取後，您會看到 LUIS 服務的原始回應，其中包含意圖、實體及其指定的分數。 您也可以選擇重新指派使用者語句的意圖。 

![QnA 偵測器](media/emulator-v4/emulator-qna-inspector.png)

在已連接 QnA 服務的情況下，記錄將會顯示 **QnA 追蹤**，若加以選取，您將可預覽與該活動相關聯的問答組和信賴分數。 在此處，您可以新增答案的替代問題用語。

<!--## Configure ngrok

If you are using Windows and you are running the Bot Framework Emulator behind a firewall or other network boundary and want to connect to a bot that is hosted remotely, you must install and configure **ngrok** tunneling software. The Bot Framework Emulator integrates tightly with ngrok tunnelling software (developed by [inconshreveable][inconshreveable]), and can launch it automatically when it is needed.

Open the **Emulator Settings**, enter the path to ngrok, select whether or not to bypass ngrok for local addresses, and click **Save**.

![ngrok path](media/emulator-v4/emulator-ngrok-path.png)
-->

## <a name="login-to-azure"></a>登入 Azure

您可以使用模擬器來登入您的 Azure 帳戶。 這特別適合用來讓您新增及管理 Bot 所依賴的服務。 若要深入了解可使用模擬器來管理的服務，請參閱[上述內容](#add-services)。

### <a name="to-login"></a>登入

![Azure 登入](media/emulator-v4/emulator-azure-login.png)

登入
- 您可以按一下 [檔案]-> [使用 Azure 登入]
- 在歡迎畫面上按一下 [使用 Azure 帳戶登入]，您可以選擇性地讓模擬器保留您的登入狀態，以在模擬器應用程式重新啟動之間保持登入。

![Azure 登入](media/emulator-v4/emulator-azure-login-success.png)

## <a name="disabling-data-collection"></a>停用資料收集

如果您決定不再允許模擬器收集使用資料，您可以依照下列步驟來輕鬆地停用資料收集：

1. 按一下左側瀏覽列中的 [設定] 按鈕 (齒輪圖示)，瀏覽至模擬器的 [設定] 頁面。

    ![停用資料收集](media/emulator-v4/emulator-disable-data-1.png)

2. 在 [資料收集] 區段底下，取消選取標示 [允許我們收集使用資料，以協助改善模擬器] 的核取方塊。

    ![停用資料收集](media/emulator-v4/emulator-disable-data-2.png)

3. 按一下 [儲存] 按鈕。

    ![停用資料收集](media/emulator-v4/emulator-disable-data-3.png)
    
如果您改變心意，一律可以藉由重新選取核取方塊來將其啟用。

## <a name="additional-resources"></a>其他資源

Bot Framework 模擬器是開放原始碼。 您可以[參與][EmulatorGithubContribute]開發，並[提交 Bug 和建議][EmulatorGithubBugs]。

如需進行疑難排解，請參閱[針對一般問題疑難排解](bot-service-troubleshoot-bot-configuration.md)以及該區段中的其他疑難排解文章。

## <a name="next-steps"></a>後續步驟

將對話儲存為文字記錄檔，可讓您快速地對一組互動撰寫草稿及加以重新執行，以用於偵錯。

> [!div class="nextstepaction"]
> [使用文字記錄檔進行 Bot 偵錯](~/v4sdk/bot-builder-debug-transcript.md)

<!-- Footnote-style URLs -->

[EmulatorGithubContribute]: https://github.com/Microsoft/BotFramework-Emulator/wiki/How-to-Contribute
[EmulatorGithubBugs]: https://github.com/Microsoft/BotFramework-Emulator/wiki/Submitting-Bugs-%26-Suggestions

[ngrokDownload]: https://ngrok.com/
[inconshreveable]: https://inconshreveable.com/
