---
title: 使用 Bot Framework 模擬器對 Bot 進行測試和偵錯 | Microsoft Docs
description: 了解如何使用 Bot Framework 模擬器桌面應用程式對 Bot 進行檢查、測試和偵錯。
keywords: 文字記錄, msbot 工具, 語言服務, 語音辨識
author: DeniseMak
ms.author: v-demak
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 04/30/2018
ms.openlocfilehash: 2a92281e9bbee09d8dfb00fbbc67fe6ac6b6c69b
ms.sourcegitcommit: 1abc32353c20acd103e0383121db21b705e5eec3
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/21/2018
ms.locfileid: "42756659"
---
# <a name="debug-with-the-bot-framework-emulator"></a>使用 Bot Framework 模擬器進行偵錯

Bot Framework 模擬器是一項桌面應用程式，可讓 Bot 開發人員在本機或從遠端對其 Bot 進行測試和偵錯。 使用此模擬器時，您能與您的 Bot 聊天，並檢查您的 Bot 所傳送及接收的訊息。 模擬器會顯示在 Web 聊天 UI 中出現的相同訊息，並在您與 Bot 交換訊息時記錄 JSON 要求和回應。 

> [!TIP] 
> 將您的 Bot 部署至雲端之前，請先在本機使用模擬器加以測試。 即使您尚未向 Bot Framework [註冊](~/bot-service-quickstart-registration.md) Bot 或將其設定為在任何通道上執行，您仍可使用模擬器來測試 Bot。

![模擬器 UI](media/emulator-v4/emulator-welcome.png)

## <a name="download-the-bot-framework-emulator"></a>下載 Bot Framework 模擬器

您可以透過 [GitHub 版本頁面](https://github.com/Microsoft/BotFramework-Emulator/releases)取得適用於 Mac、Windows 和 Linux 的下載套件。

## <a name="connect-to-a-bot-running-on-local-host"></a>連線至執行於本機主機的 Bot

![模擬器端點](media/emulator-v4/emulator-endpoint.png)

若要連線至執行於本機的 Bot，請選取左上方的 [Bot 總管] 索引標籤。 按一下 [端點] 索引標籤下方的 **+** 圖示。在此處，您可以將端點指定給 Bot 在本機執行所在的相同連接埠，以連線至該連接埠。 按一下 [提交]，您即會重新導向至可以與您的 Bot 互動的即時聊天視窗。

## <a name="view-detailed-message-activity-with-the-inspector"></a>使用偵測器檢視詳細訊息活動

![模擬器訊息活動](media/emulator-v4/emulator-view-message-activity-02.png)

您可以按一下交談視窗內的任何訊息泡泡，並使用視窗右側的**偵測器**功能檢查原始 JSON 活動。 訊息泡泡經選取後會變成黃色，且活動 JSON 物件將會顯示在聊天視窗左側。 此 JSON 資訊包含主要中繼資料，包括 channelID、活動類型、交談識別碼、文字訊息，端點 URL 等。您可以檢視使用者所傳送的檢查活動，以及 Bot 所回應的活動。 

您也可以使用[通道偵測器](bot-service-channel-inspector.md)來預覽特定通道支援的功能。


## <a name="save-and-load-conversations-with-bot-transcripts"></a>儲存及載入與 Bot 交談的文字記錄

模擬器中的訊息活動可以儲存為文字記錄。 您可以從開啟的即時聊天視窗中選取 [將文字記錄另存新檔]，以命名輸出文字記錄檔案並選取其位置。 

>[!TIP]
> 您可以隨時使用 [重新開始] 按鈕來清除對話，並重新連線至 Bot。  

![模擬器儲存文字記錄](media/emulator-v4/emulator-live-chat.png)

若要載入文字記錄，只需選取 [檔案] --> [開啟文字記錄檔案]，再選取文字記錄即可。 新的 [文字記錄] 視窗隨即開啟，並將訊息活動轉譯為輸出視窗。 

![模擬器載入文字記錄](media/emulator-v4/emulator-load-transcript.png)

## <a name="author-transcripts-with-chatdown"></a>使用 Chatdown 撰寫文字記錄

[Chatdown](https://github.com/Microsoft/botbuilder-tools/tree/master/Chatdown) 工具是一個文字記錄產生器，會使用 [Markdown](https://daringfireball.net/projects/markdown/syntax) 檔案來產生活動文字記錄。 您可以自行撰寫完全採用 Markdown 格式的文字記錄，並將其儲存為 **.chat** 檔案以產生文字記錄。 這可以讓您在 Bot 開發期間快速建立模擬交談情境。  

### <a name="prerequisites"></a>必要條件

- [Bot Framework 模擬器](https://github.com/Microsoft/BotFramework-Emulator/releases) v.4 或更新版本 
- [Node.js](https://nodejs.org/en/)
 
Chatdown 可作為需要 Node.js 的 npm 模組。 若要安裝 Chatdown，請將其全域安裝到您的電腦。 

```
npm install -g chatdown
```
### <a name="create-and-load-transcript-transcript-files"></a>建立並載入文字記錄檔案 ###

以下範例說明如何撰寫 **.chat** 檔案。 這些檔案是包含 2 個部分的 Markdown 檔案：
- 定義交談參與者 (使用者、Bot) 的標頭
- 參與者之間的相互交談

```
user=John Doe
bot=Bot

bot: Hello!
user: hey
bot: [Typing][Delay=3000]
What can I do for you?
user: Actually nevermind, goodbye.
bot: bye!
```
[按一下這裡](https://github.com/Microsoft/botbuilder-tools/tree/master/Chatdown/Examples)可以檢視更多.chat 檔案的範例。 

若要產生文字記錄檔案，請使用在 CLI 中使用 **chatdown** 命令，輸入 .chat 檔案的名稱，其後再加上 '>' 與輸出文字記錄檔案的名稱。 

```
chatdown sample.chat > sample.transcript
```
## <a name="manage-bot-resources-with-the-msbot-tool"></a>使用 MSBot 工具管理 Bot 資源

新的 [MSBot](https://github.com/Microsoft/botbuilder-tools/tree/master/MSBot) 工具可讓您建立 **.bot** 檔案，其中儲存與 Bot 取用不同服務有關的中繼資料，全都放在同一個位置。 此檔案也可讓您的 Bot 從 CLI 連線至這些服務。此工具可用來作為 npm 模組；若要安裝，請執行：

```
npm install -g msbot 
```
![MSBot CLI 視窗](media/emulator-v4/msbot-cli-window.png)


若要建立 Bot 檔案，請從您的 CLI 輸入 **msbot init**，其後再加上 Bot 的名稱和目標 URL 端點，例如：

```shell
msbot init --name az-cli-bot --endpoint http://localhost:3984/api/messages
```
![Botfile](media/emulator-v4/botfile-generated.png)

>**注意：** 本指南中使用的 Bot 是簡單回應 Bot，由 Azure CLI Bot 擴充功能所產生。 [按一下這裡](https://github.com/Microsoft/botbuilder-tools/tree/master/AzureCli)可以深入了解如何使用 Azure CLI 建置 Bot。 

現在，您可以使用 .bot 檔案輕鬆地將 Bot 載入至模擬器。 若要對 Bot 註冊不同的端點和語言元件，也必須使用 .bot 檔案。 

![Bot-File-Dropdown](media/emulator-v4/bot-file-dropdown.png)

## <a name="add-language-services"></a>新增語言服務 

您可以直接從模擬器輕鬆地將 LUIS 應用程式或 QnA 知識庫註冊到 .bot 檔案。 在 .bot 檔案載入後，請選取模擬器視窗最左側的 [服務] 按鈕。 您會在 [服務] 功能表下方看到可用來新增 LUIS、QnA Maker、分派、端點和 Azure Bot Service 的選項。 

若要新增 LUIS 應用程式，請按一下 LUIS 功能表上的 **+** 按鈕，輸入您的 LUIS 應用程式認證，然後按一下 [提交] 即可。 這會將 LUIS 應用程式註冊至 .bot 檔案，並將服務連線至您的 Bot 應用程式。 

![LUIS 連線](media/emulator-v4/emulator-connect-luis-btn.png)

同樣地，若要新增 QnA 知識庫，請按一下 QnA 功能表上的 **+** 按鈕，輸入您 QnA Maker 知識庫認證，然後按一下 [提交] 即可。 您的知識庫此時將會註冊至 .bot 檔案，並且可供使用。 

![QnA 連線](media/emulator-v4/emulator-connect-qna-btn.png)

在任一服務連線時，您都可以返回即時聊天視窗，並確認您的服務已連線並運作中。 

![QnA 已連線](media/emulator-v4/emulator-view-message-activity.png)

## <a name="inspect-language-services"></a>檢查語言服務

透過新的 v4 模擬器，您也可以檢查 LUIS 和 QnA 的 JSON 回應。 使用已連接語言服務的 Bot 時，您可以在右下方選取 [記錄] 視窗中的 [追蹤]。 這項新工具也提供直接從模擬器更新語言服務的功能。 

![LUIS 偵測器](media/emulator-v4/emulator-luis-inspector.png)

在已連接 LUIS 服務的情況下，您會發現追蹤連結指定了 **Luis 追蹤**。 加以選取後，您會看到 LUIS 服務的原始回應，其中包含意圖、實體及其指定的分數。 您也可以選擇重新指派使用者語句的意圖。 

![QnA 偵測器](media/emulator-v4/emulator-qna-inspector.png)

在已連接 QnA 服務的情況下，記錄將會顯示 **QnA 追蹤**，若加以選取，您將可預覽與該活動相關聯的問答組和信賴分數。 在此處，您可以新增答案的替代問題用語。

[!TIP]
> 這些功能僅適用於 v4 SDK Bot 


## <a name="speech-recognition"></a>語音辨識
Bot Framework 模擬器支援透過[認知服務語音 API](/azure/cognitive-services/Speech/home) 的語音辨識。 這可讓您在開發期間，透過模擬器中的語音來訓練具備語音功能的 Bot 或 Cortana 技能。 Bot Framework 模擬器每天可為每一 Bot 提供最多三小時的免費語音辨識服務。 

## <a id="ngrok"></a>安裝及設定 ngrok

如果您使用 Windows，並且在防火牆或其他網路界限以外執行 Bot Framework 模擬器，而想要連線至遠端裝載的 Bot，您必須安裝並設定 **ngrok** 通道軟體。 Bot Framework 模擬器可與 [ngrok][ngrokDownload] 通道軟體 (由 [inconshreveable][inconshreveable] 所開發) 緊密整合，並且可在需要時自動加以啟動。

若要在 Windows 上安裝 **ngrok**，並設定模擬器加以使用，請完成下列步驟： 

1. 將 [ngrok][ngrokDownload] 可執行檔下載到您的本機電腦。

2. 開啟模擬器的 [應用程式設定] 對話方塊、輸入 ngrok 的路徑、選取是否略過 ngrok 而使用本機位址，然後按一下 [儲存]。

![ngrok 路徑](media/emulator-v4/emulator-ngrok-path.png)

## <a name="additional-resources"></a>其他資源

Bot Framework 模擬器是開放原始碼。 您可以[參與][EmulatorGithubContribute]開發，並[提交 Bug 和建議][EmulatorGithubBugs]。


[EmulatorGithub]: https://github.com/Microsoft/BotFramework-Emulator
[EmulatorGithubContribute]: https://github.com/Microsoft/BotFramework-Emulator/wiki/How-to-Contribute
[EmulatorGithubBugs]: https://github.com/Microsoft/BotFramework-Emulator/wiki/Submitting-Bugs-%26-Suggestions

[ngrokDownload]: https://ngrok.com/
[inconshreveable]: https://inconshreveable.com/
[BotFrameworkDevPortal]: https://dev.botframework.com/


[EmulatorConnectPicture]: ~/media/emulator/emulator-connect_localhost_credentials.png
[EmulatorNgrokPath]: ~/media/emulator/emulator-configure_ngrok_path.png
[EmulatorNgrokMonitor]: ~/media/emulator/emulator-testbot-ngrok-monitoring.png
[EmulatorUI]: ~/media/emulator/emulator-ui-new.png

[TroubleshootingGuide]: ~/bot-service-troubleshoot-general-problems.md
[TroubleshootingAuth]: ~/bot-service-troubleshoot-authentication-problems.md
[NodeGetStarted]: ~/nodejs/bot-builder-nodejs-quickstart.md
[CSGetStarted]: ~/dotnet/bot-builder-dotnet-quickstart.md
